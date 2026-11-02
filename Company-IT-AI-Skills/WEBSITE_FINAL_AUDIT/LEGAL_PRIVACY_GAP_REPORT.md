# Legal, Privacy & Customer Trust Gap Report
**Audit Date:** 2026-11-02  
**Platform:** AISecurity / PerfectITSecurity  
**Disclaimer:** This audit identifies factual, technical, and transparency gaps in the website's legal and trust disclosures. All legal documents must be reviewed and approved by qualified legal counsel before production launch.

---

## 1. Existing Legal Pages Inventory

All 6 legal routes exist, return HTTP 200, and are linked in the global footer (`resources/views/layouts/public.blade.php:436-441`):
1. **Privacy Policy:** `/legal/privacy-policy` (`resources/views/public/legal/privacy.blade.php`)
2. **Terms of Service:** `/legal/terms-of-service` (`resources/views/public/legal/terms.blade.php`)
3. **Cookie Policy:** `/legal/cookie-policy` (`resources/views/public/legal/cookies.blade.php`) + `<x-cookie-consent />` banner
4. **Refund Policy:** `/legal/refund-policy` (`resources/views/public/legal/refund.blade.php`)
5. **Service Level Agreement (SLA):** `/legal/service-level-agreement` (`resources/views/public/legal/sla.blade.php`)
6. **Accessibility Statement:** `/legal/accessibility` (`resources/views/public/legal/accessibility.blade.php`)

---

## 2. Critical Trust, Legal & Factual Accuracy Gaps

| Gap ID | Page / Location | Issue Description | Severity | Required Action (Before Production) |
|---|---|---|---|---|
| **LEG-01** | **Company Identity & Contact Inconsistency** (`/contact`, `/get-quote`, `/faq`, `/offices`, `/legal/refund-policy`, `/legal/service-level-agreement`, Footer) | The website displays conflicting email domains, phone numbers, and physical addresses across pages: Footer uses `info@AISecurity.com`; `/contact` uses `ops@techsupportsolutions.com`, `+1 (800) 555-TECH-SOC`, and `Level 42, Cyber Tower One, Technology Corridor`; `/get-quote` and `/legal/refund-policy` use `info@techsupport.com`, `billing@techsupport.com`, `complaints@techsupport.com`, `+1 (555) 123-4567`, and `123 Tech Avenue, San Francisco, CA 94105`; `/offices` lists 6 landmark skyscrapers (`1 Canada Square London`, `1 World Trade Center NYC`, `Raffles Place Singapore`, `DIFC Dubai`, `Macquarie Place Sydney`, `Gulshan Avenue Dhaka`). | **HIGH (P0 Trust Blocker)** | Replace all placeholder emails, phone numbers, and fictitious skyscraper addresses with the company's **actual registered legal entity name, company registration number, verified business address(es), and active support email/phone channels** driven from `Setting::get()`. |
| **LEG-02** | **Unverified Marketing Claims on `/about`, `/pricing`, `/faq`, `/get-quote`** | `/about` claims `"Tier IV certified data centers with N+2 power"`, `"Our Global Security Operations Center (SOC) operates 24/7/365 with active telemetry analysis"`, and `"ISO 27001, SOC 2 Type II, HIPAA, and GDPR aligned security governance"`. `/get-quote` claims `"250+ Clients worldwide"`. `/offices` claims `"500+ Team Members"` across `"15+ Countries"`. Meanwhile, `/ai-services` and the homepage stats strip were already cleaned to be honest and non-fabricated. | **HIGH (P0 Commercial / Legal Risk)** | Remove or qualify any unverified headcount, client count, Tier-IV data center, 24/7 live telemetry SOC, or compliance claims on `/about`, `/offices`, `/get-quote`, `/pricing`, and `/faq` so they match the honest, scope-based language already used on `/ai-services`. |
| **LEG-03** | **Cookie Policy Technical Mismatch** (`/legal/cookie-policy`) | `resources/views/public/legal/cookies.blade.php:48,55` states that the platform sets `techsupport_session` and `NEXT_LOCALE`. In reality, `config/session.php` sets `aisecurity_session` (from `Str::slug(env('APP_NAME', 'laravel')).'_session'`), `XSRF-TOKEN`, and `locale` (plus `localStorage` keys `theme`, `selected_country`, `cookie_consent`). | **MEDIUM (P1)** | Update `cookies.blade.php` to list the actual cookies and `localStorage` keys used by the Laravel + Alpine application. |
| **LEG-04** | **AI Assistance Disclosure in Privacy Policy** (`/legal/privacy-policy`) | While the AI chat widget (`ai-chat-widget.blade.php`) and `/ai-services` clearly disclose that the user is interacting with an AI assistant and label non-KB answers as general guidance, the Privacy Policy (`/legal/privacy-policy`) does not explicitly explain how AI chat transcripts (`ai_conversations`, `ai_messages`) are stored, whether local vs. cloud LLM providers process prompts, or how long chat logs are retained. | **MEDIUM (P1)** | Add Section 6 *"AI Support Assistant & Automated Processing"* to `/legal/privacy-policy` disclosing transcript storage, human escalation review, and local vs. external model processing. |
| **LEG-05** | **Missing Governing Law, Jurisdiction & DPO Contact** (`/legal/terms-of-service`, `/legal/privacy-policy`) | `/legal/terms-of-service` does not specify the governing law and dispute jurisdiction (e.g., England & Wales, Delaware/California, or Bangladesh), and `/legal/privacy-policy` mentions contacting the "Data Protection Officer" without providing a dedicated `dpo@` or `privacy@` email address and postal address. | **MEDIUM (P1)** | Have legal counsel supply the governing law clause, registered company number, and DPO contact details. |
