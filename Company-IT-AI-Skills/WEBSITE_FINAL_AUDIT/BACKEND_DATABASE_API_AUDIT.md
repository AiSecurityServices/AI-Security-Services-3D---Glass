# Backend, Database & API Architecture Audit Report
**Audit Date:** 2026-11-02  
**Framework & Runtime:** Laravel 9.52.22, PHP 8.4.27 (codebase targets `^8.0.2`), SQLite 3.46.1 (local/testing), MySQL/MariaDB (production target)  
**Total Migrations:** 61 (100% applied cleanly in 1.2s)  
**Total Models:** 92 Eloquent models  
**Total Routes:** 653 (`routes/web.php` + `routes/api.php`)

---

## 1. Database Schema & Migration Integrity

- **61 Sequential Migrations:** All 61 migrations from `2014_10_12_000000_create_users_table` through `2026_10_15_000004_widen_ticket_status_check` run cleanly with zero errors.
- **Authoritative Single-Ledger Financial Schema:**
  - `financial_transactions` is the sole source of truth for P&L income/expense reporting (`FinancialService`).
  - `payments`, `receipts`, and `cash_memos` maintain 1:1 linked integrity with unique reference numbers and idempotency keys (`2026_09_13_000004_payment_refunds_idempotency`).
  - `wallets` and `wallet_transactions` use `DB::transaction()` + `lockForUpdate()` row locking to prevent concurrent race-condition overdraws (`WalletTest::concurrent_payments_cannot_overdraw`).
- **Role & Status Check Constraints:**
  - `2026_09_19_000006_widen_users_role_check` synchronizes the `users.role` constraint with `RoleRegistry::definitions()`.
  - `2026_10_15_000004_widen_ticket_status_check` widens `tickets.status` to include `reopened`, resolving the earlier MySQL/SQLite status constraint mismatch.
- **Foreign Keys & Soft Deletes:** Core business records (`Ticket`, `CustomerDocument`, `User` with wallet history) prevent destructive deletion so financial and audit histories remain intact (`WalletTest::user_with_wallet_history_cannot_be_deleted`).

---

## 2. API & Endpoint Audit

| Endpoint Group | Path Prefix | Auth / Middleware | Rate Limit | Audit Verdict |
|---|---|---|---|---|
| **Sanctum User API** | `GET /api/user` | `auth:sanctum` | `throttle:api` (60/min) | **PASS** — Hides password, remember_token, two_factor_secret, OTP hashes. |
| **Unread Notifications API** | `GET /api/notifications/unread` | `auth:sanctum` | `web` | **NEEDS IMPROVEMENT (BUG-05)** — Calls `$user->notifications()->unread()`, which conflicts with Laravel's built-in `Notifiable` trait (`DatabaseNotification`) unless resolved via `App\Models\Notification::where('notifiable_id', $user->id)`. |
| **AI Chat Endpoints (10 routes)** | `/api/ai/*` | `web` + `authorizeConversation()` | `throttle:60,1` + internal `RateLimiter` (10 start/min, 30 msg/min) | **PASS** — Enforces owner or constant-time `hash_equals` guest `X-Session-ID`. |
| **Frontend Error Telemetry** | `POST /api/health/frontend-error` | `web` (CSRF protected) | `throttle:30,1` | **PASS** — Logs truncated JS errors to `system_error_logs`. |
| **Inbound Email Webhook** | `POST /api/ingress/email` | HMAC-SHA256 signature + timestamp window (`InboundEmailController`) | `throttle:60,1` (CSRF exempt) | **PASS** — Disabled by default (`INBOUND_EMAIL_ENABLED=false`); idempotent threading via `external_id`. |
| **Stripe & Payment Webhooks** | `POST /stripe/webhook`, `POST /payments/webhook/{provider}` | Provider HMAC signature verification | `throttle:60,1` (CSRF exempt) | **PASS** — Rejects unsigned/forged payloads (`StripeWebhookSecurityTest`). |
| **Public QR Verification** | `GET /verify/member/{token}`, `/verify/certificate/{token}` | Signed token lookup | `throttle:30,1` | **PASS** — Minimal public disclosure. |
| **Health Probe** | `GET /healthz` | Public | `web` | **PASS** — Returns HTTP 200 `{"status":"ok","db":"ok"}` or 503 on DB failure; leaks no config. |

---

## 3. Background Jobs, Scheduler & Queue Reliability

- **13 Scheduled Commands (`app/Console/Kernel.php`):**
  1. `sla:check-deadlines` (every 5 minutes, `withoutOverlapping()`)
  2. `contracts:expire` (daily 01:00, `onOneServer()`)
  3. `quotes:expire` (daily 01:30, `onOneServer()`)
  4. `invoices:send-overdue-reminders` (daily 09:00)
  5. `maintenance:remind-due` (daily 08:00)
  6. `billing:process-subscriptions` (daily 06:00, `onOneServer()`)
  7. `fx:refresh` (daily 06:30, `onOneServer()`)
  8. `auth:clear-resets` (every 15 minutes)
  9. `queue:prune-failed --hours=168` (weekly)
  10. `backup:run --type=full` (daily 02:00)
  11. `backup:run --type=db` (every 6 hours)
  12. `backup:prune` (daily 03:30)
  13. `backup:monitor` (hourly)
- **Operational Requirement:** Local `.env` defaults to `QUEUE_CONNECTION=sync`. Production deployment **must** configure `QUEUE_CONNECTION=database` or `redis`, run `php artisan queue:work` under Supervisor, and install the system cron entry `* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1`.

---

## 4. Technical Debt & Code Quality Risks
1. **Framework & PHP Version Compatibility (High):** `composer.json` locks `laravel/framework` at `v9.52.22` and `nette/schema` at `v1.2.5` (`php 7.1 - 8.3`). Running on PHP 8.4 emits nullable parameter deprecation warnings unless `E_DEPRECATED` is suppressed. Upgrade to Laravel 11/12 + PHP 8.2/8.3 LTS is required.
2. **Services Catalog Eager-Load Payload (Medium):** `Public\ServiceController@index` loads all 237 active services and 694 country prices on a single page (795 KB HTML, ~677 ms render time).
