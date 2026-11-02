# Customer Portal & Ticketing Audit Report
**Audit Date:** 2026-11-02  
**Platform:** AISecurity / PerfectITSecurity  
**Automated Verification:** `CustomerIsolationTest` (10/10 PASS), `AccountIsolationMatrixTest` (5/5 PASS), `TicketingTest` (6/6 PASS), `InternationalHelpdeskTest` (13/13 PASS), `OrderPaymentE2ETest` (10/10 PASS), `WalletTest` (19/19 PASS)

---

## 1. End-to-End Customer Journey Verification

| Step | Journey Stage | Route(s) & Controller(s) | Status | Verification Evidence |
|---|---|---|---|---|
| **1** | **Customer Registration** | `GET/POST /register` (`Auth\RegisterController`) | **VERIFIED WORKING** | Throttled `5,1`. Assigns `CUS-` member number. In production (`APP_ENV=production`), non-customer role requests default to `customer` with `requested_role` marked `pending` for admin review (`RoleRegistry::registrationOutcome()`). |
| **2** | **Email & Phone OTP Verification** | `/portal/verify-email`, `/portal/verify-phone` | **VERIFIED WORKING** | 6-digit OTPs are SHA-256 hashed before DB storage, expire in 10 minutes, enforce a 60-second resend cooldown, and lock out after 5 failed attempts (`CustomerVerificationTest` 5/5 PASS). |
| **3** | **Login & Session Security** | `GET/POST /login` (`Auth\LoginController`) | **VERIFIED WORKING** | Throttled `5,1`. Blocks inactive/suspended accounts (`is_active=false`). Regenerates session ID on login; supports optional RFC 6238 TOTP MFA (`mfa` middleware). |
| **4** | **Password Recovery** | `/password/reset` (`ForgotPasswordController`, `ResetPasswordController`) | **VERIFIED WORKING** | Throttled `5,1`. Invalidates other active sessions on password reset. |
| **5** | **Customer Profile & Settings** | `GET/PUT /portal/profile`, `/portal/profile/password` | **VERIFIED WORKING** | Whitelisted fields only (`role` and `is_active` cannot be mass-assigned); supports avatar, password update, notification preferences, and 3D glove preferences. |
| **6** | **Support Request & Quote Submission** | `/portal/service-request`, `/portal/orders`, `/get-quote` | **VERIFIED WORKING** | Idempotent 15-minute bucket prevents duplicate double-click submissions; creates `ServiceRequest`, `Lead`, and `ServiceOrder`. |
| **7** | **Ticket Creation** | `GET/POST /portal/tickets` (`Customer\TicketController@store`) | **VERIFIED WORKING** | Auto-generates unique `TK-XXXXXXXX` number, applies SLA policy (`SlaService::applySla()`), stores attachments on private local disk, and notifies support staff. |
| **8** | **Ticket Assignment & SLA Tracking** | `POST /admin/tickets/{ticket}/assign` (`Admin\TicketController@assign`) | **VERIFIED WORKING** | Validates assignee is active staff (`InternationalHelpdeskTest::assignee_must_be_active_staff`); pauses SLA clock when status is `waiting_on_customer` (`2026_10_15_000001_add_sla_pause_to_tickets`). |
| **9** | **Ticket Updates, Replies & Reopen** | `POST /portal/tickets/{id}/reply`, `/reopen` | **VERIFIED WORKING** | Customer can reply to own open tickets and reopen resolved tickets within the policy window (`2026_10_15_000004_widen_ticket_status_check` fixed the `reopened` status constraint). Internal staff notes (`is_internal_note=true`) are filtered out of customer views. |
| **10** | **File Uploads & Downloads** | `/portal/tickets/{ticket}/attachments/{attachment}`, `/portal/documents` | **VERIFIED WORKING** | Stored on private `local` disk (`storage/app/ticket-attachments` and `customer-documents`), never under `public/storage`. Extensions restricted (`pdf,doc,docx,xls,xlsx,png,jpg,jpeg,txt,zip`, max 10 MB). Ownership verified before streaming download. |
| **11** | **Customer Notifications** | `/portal/notifications`, `/portal/notifications/preferences` | **VERIFIED WORKING** | In-app notification bell + preference-gated email notifications (`RespectsNotificationPreferences`). |
| **12** | **AI Support Assistance & Escalation** | `/api/ai/*` (`AiChatController`) | **VERIFIED WORKING** | Customer can query own ticket/order/invoice status deterministically, confirm ticket drafts, or escalate to human support. |
| **13** | **Ticket Closure & Support History** | `/portal/history`, `/portal/account`, `/portal/reports/mine` | **VERIFIED WORKING** | Full lifelong timeline of tickets, orders, invoices, payments, receipts, and communications scoped strictly to `auth()->id()`. |

---

## 2. Customer Tenant Isolation & RBAC Verification

- **Customer A vs. Customer B Isolation:** Verified across 10 endpoints in `CustomerIsolationTest` and 21 adversarial scenarios in `AdversarialSecurityTest`. Attempting to view, reply to, download, or pay another customer's ticket, order, invoice, quotation, project, or document returns `403 Forbidden` or `404 Not Found`.
- **Organisation-Level Ticket View:** `/portal/tickets/organisation` scopes tickets by `company_id` only when the customer belongs to a verified `Company` record (`InternationalHelpdeskTest::organisation_tickets_are_company_isolated`).
- **Staff Boundary Enforcement:** Customers are blocked from `/admin/*` (`StaffMiddleware`). Staff are blocked from `/portal/*` (`CustomerMiddleware`). Support agents (`support_agent`) can only view tickets assigned to them or unassigned tickets in their queue (`TicketingTest::agent_cannot_view_another_agents_ticket`) and receive `403 Forbidden` on `/admin/financials/*`, `/admin/invoices/*`, `/admin/users/*`, and `/admin/settings/*`.

---

## 3. Residual Gaps & Recommended Improvements
1. **Admin Ticket Attachment Upload/Download UI (P2):** While customers can upload and download ticket attachments securely, the admin ticket detail view (`resources/views/admin/tickets/show.blade.php`) lacks a dedicated staff attachment upload control.
2. **CSAT Survey UI on Ticket Closure (P2):** The `tickets` table has `satisfaction_rating` and `satisfaction_feedback` columns, but no rating form is presented to the customer when a ticket transitions to `resolved`/`closed`.
