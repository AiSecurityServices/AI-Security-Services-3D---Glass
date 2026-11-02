# Phased Remediation Plan
**Audit Date:** 2026-11-02  
**Platform:** AISecurity / PerfectITSecurity

---

## Phase A — Completed Safe Fixes (Implemented & Verified During This Audit)

In strict compliance with Phase 18 (*"Implement only approved, non-destructive changes. Preserve all existing functionality and data. Run regression tests after each significant change"*), three safe, non-destructive fixes were applied and verified:

1. **Restored 20th Industry on `/industries` (`BUG-04`):**
   - *File:* `resources/views/public/industries.blade.php`
   - *Change:* Added the 20th industry record (`Aerospace, Aviation & Defence Supply Chain`) with challenges, solutions, and tags while preserving all 19 existing industries.
2. **Fixed Broken Sitemap URLs & Missing Public Pages (`BUG-05`):**
   - *File:* `app/Http/Controllers/Public/SitemapController.php`
   - *Change:* Added `/ai-services`, `/get-quote`, `/faq`, `/portfolio`, `/useful-links`, and `/legal/accessibility` to `/sitemap.xml`, and routed AI division service slugs to `/ai-services/{slug}` instead of `/services/{slug}` (preventing 9 broken 404 URLs in `/sitemap.xml`).
3. **Connected Broken FAQ Live Search (`BUG-06`):**
   - *File:* `resources/views/public/faq.blade.php`
   - *Change:* Merged the disconnected Alpine `x-data` search scope with the FAQ category panels and added `x-show` filtering across all 27 FAQ items.

---

## Phase B — Pre-Production Blockers Requiring Owner Input / Approval (P0)

1. **Business Identity, Contact & Office Claims Cleanup (`LEG-01`, `LEG-02`, `BUG-03`):**
   - *Owner Action Required:* Provide the official registered company name, registration number, real office address(es), verified phone number(s), and official support/billing/SLA email addresses.
   - *Engineering Action:* Replace hardcoded `techsupport.com` / `techsupportsolutions.com` / `555-TECH-SOC` / `123 Tech Avenue` strings across `/contact`, `/get-quote`, `/faq`, `/offices`, `/legal/refund-policy`, and `/legal/service-level-agreement`; remove unverified `Tier IV data centers`, `24/7 Global SOC`, `500+ Team Members`, and `250+ Clients` claims.
2. **Framework & Dependency Security Upgrade (`SEC-F01`, `SEC-F02`, `BUG-01`):**
   - *Proposal (Requires Approval):* Create a dedicated upgrade branch to migrate `laravel/framework` from `9.52.22` -> `11.x`/`12.x`, `symfony/*` -> `6.4`/`7.x`, `dompdf/dompdf` -> `>=3.1.6`, and `vite` -> `>=5.x`/`6.x` on PHP 8.2/8.3 LTS.
   - *Verification Gate:* `composer audit --locked` = 0 advisories, `npm audit --omit=dev` = 0 advisories, and all 395+ PHPUnit tests green.
3. **20-Role AI Employee Workforce Decision (`BUG-02`):**
   - *Decision Required:* Confirm whether the business intends to launch with the current **AI Customer Support Agent (with 20 deterministic AI Skills) + 9 AI Consulting Services**, or whether a backend multi-agent **20-Role AI Employee Workforce** (`AiEmployee` model, task queue, inter-agent collaboration, and Workforce Manager UI) should be architected and built as a new feature phase.

---

## Phase C — Performance, Security & UX Hardening (P1 / P2)

1. **Optimize `/services` Page Payload (`BUG-07`):** Paginate or cap per-category cards on `/services` (reducing HTML size from 795 KB to ~220 KB and response time from 677 ms to <80 ms) and cache navigation categories.
2. **Sanitize Rich-Text HTML on Save (`SEC-F03`, `BUG-08`):** Add allowlist HTML sanitization in Admin controllers for Services, KB articles, and Blog posts.
3. **Fix `/api/notifications/unread` Query (`BUG-09`):** Use `App\Models\Notification` query builder directly.
4. **Update Cookie Policy & Privacy Policy AI Disclosure (`LEG-03`, `LEG-04`, `LEG-05`):** Align cookie names (`aisecurity_session`, `locale`) and document AI chat data handling.
5. **Seed Initial `/useful-links` and `/blog` Entries (`BUG-10`):** Add curated public cybersecurity resources (NCSC, CISA, OWASP, NIST, MITRE) and initial technical blog posts.

---

## Phase D — Staging & Production Infrastructure Gates

1. Provision managed MySQL/MariaDB database, Redis cache/queue, and Supervisor worker (`php artisan queue:work`).
2. Install system cron for Laravel scheduler (`* * * * * php artisan schedule:run`).
3. Configure production `.env` (`APP_ENV=production`, `APP_DEBUG=false`, `SESSION_SECURE_COOKIE=true`, live SMTP, Stripe/payment credentials, and private Ollama or cloud AI key).
4. Execute a full staging backup -> restore swap drill (`php artisan backup:run --type=full` + `backup:restore`).
