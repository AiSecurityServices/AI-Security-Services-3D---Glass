# Knowledge Hub & Learning Resources Audit Report
**Audit Date:** 2026-11-02  
**Platform:** AISecurity / PerfectITSecurity  
**Audited Components:** `/knowledge-base` (`Public\KnowledgeBaseController`), `/useful-links` (`Public\UsefulLinksController`), `/admin/knowledge-base` (`Admin\KnowledgeBaseController`), `/admin/help/*` (`HelpCenterController`), `/admin/academy/*` (`LearningController`), `AiKnowledgeSeeder`

---

## 1. Public Knowledge Base (`/knowledge-base`)

- **Content Inventory:** Seeded via `AiKnowledgeSeeder` with **21 categories** and **41 published articles** (100% verified in SQLite database and HTTP 200 responses).
- **Categories Covered (21):**
  1. `About AISecurity` (3 articles)
  2. `IT Support` (2 articles)
  3. `Microsoft 365` (3 articles: Outlook not opening, Email sync problems, M365 login problems)
  4. `Networking` (3 articles: No internet quick checks, What is DNS, Wi-Fi slow or dropping)
  5. `Website Security` (3 articles: HTTPS/TLS, Vulnerability assessments, Passwords & MFA)
  6. `Web Development` (3 articles)
  7. `Cloud & Server` (2 articles)
  8. `Backup & Disaster Recovery` (2 articles: 3-2-1 strategy, RPO/RTO)
  9. `Digital Marketing` (2 articles)
  10. `Managed IT Services` (1 article)
  11. `Pricing & Quotes` (2 articles)
  12. `Orders & Billing` (2 articles)
  13. `Projects` (1 article)
  14. `Support Tickets` (2 articles)
  15. `Customer Portal` (2 articles)
  16. `AI Assistant` (2 articles)
  17. `Frequently Asked Questions` (1 article)
  18. `Security Awareness` (2 articles: Phishing, Ransomware)
  19. `Troubleshooting Guides` (1 article)
  20. `Policies` (1 article)
  21. `Contact & Escalation` (1 article)
- **Search & Filtering:** Keyword search (`?search=`) and category pill filtering (`?category=`) work cleanly and preserve query strings across pagination.
- **Helpful Voting & Versioning:** `POST /knowledge-base/{slug}/vote` (throttled `20,1`) increments `helpful_yes` / `helpful_no` counters via AJAX. Admin KB edits track `version` and `ai_readable` flags (`2026_09_08_120000_add_versioning_and_votes_to_kb_articles`).
- **Safety & Attribution:** All 41 articles provide safe, defensive guidance and explicitly distinguish between safe first checks a user can perform and actions that require a professional support ticket. Zero fabricated certifications or copyright violations exist.

---

## 2. Curated Cybersecurity Resources (`/useful-links`)

- **Functionality:** `Public\UsefulLinksController@index` renders curated external resources with `target="_blank" rel="noopener noreferrer"`, category badges, and host domain extraction, plus a throttled (`10,1`) "Suggest a Resource" form (`LinkSubmission` moderation queue in `/admin/link-submissions`).
- **Gap Identified (BUG-06 / P2):** No seeder populates the `useful_links` table by default, so `/useful-links` currently displays `"No resources available yet."` Seeding 10–15 authoritative public cybersecurity resources (e.g., CISA, NCSC UK, OWASP, NIST NVD, MITRE ATT&CK, SANS) is recommended.

---

## 3. Internal Staff Knowledge Hub & Academy (`/admin/help/*` & `/admin/academy/*`)

- **HTML Training & Problem Center (`/admin/help/training`, `/admin/help/problems`):** Backed by `App\Support\TrainingCenterLessons` and `TrainingCenterProblems`, providing role-specific operational runbooks for staff.
- **Training Academy (`/admin/academy`):** Seeded via `TrainingAcademySeeder` with courses, modules, lessons, quizzes, practical assessments, and signed completion certificates (`TrainingAcademyTest` PASS).
