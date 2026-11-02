# Performance & Speed Audit Report
**Audit Date:** 2026-11-02  
**Environment:** Linux Sandbox (`127.0.0.1:8000`, PHP 8.4.27 CLI Server, SQLite 3.46.1, Vite 4.5.14 Production Bundle)

---

## 1. Measured HTTP Response Times & Payload Sizes (30 Public & Auth Endpoints)

All measurements below were captured live against the running application (`http://127.0.0.1:8000`) after Vite production build:

| URI | HTTP Status | Response Time (ms) | HTML/Body Size (Bytes) | Performance Verdict |
|---|---:|---:|---:|---|
| `/` (Homepage - cold boot) | 200 | 384.1 ms (warm: 28.5 ms) | 126,804 B (123.8 KB) | **PASS** |
| `/about` | 200 | 27.1 ms | 100,090 B (97.7 KB) | **PASS** |
| `/services` (Full Catalog) | 200 | **676.7 ms** | **795,105 B (776.5 KB)** | **NEEDS OPTIMIZATION (High Impact)** |
| `/services/penetration-testing` | 200 | 35.6 ms | 103,022 B (100.6 KB) | **PASS** |
| `/ai-services` | 200 | 31.3 ms | 141,315 B (138.0 KB) | **PASS** |
| `/ai-services/custom-ai-agents` | 200 | 28.9 ms | 108,122 B (105.6 KB) | **PASS** |
| `/industries` (All 20 Industries) | 200 | 32.4 ms | 149,133 B (145.6 KB) | **PASS** |
| `/knowledge-base` | 200 | 35.4 ms | 109,010 B (106.5 KB) | **PASS** |
| `/knowledge-base/what-is-managed-it-support` | 200 | 33.6 ms | 85,728 B (83.7 KB) | **PASS** |
| `/contact` | 200 | 25.8 ms | 90,626 B (88.5 KB) | **PASS** |
| `/get-quote` | 200 | 30.3 ms | 124,094 B (121.2 KB) | **PASS** |
| `/pricing` | 200 | 29.8 ms | 100,138 B (97.8 KB) | **PASS** |
| `/faq` | 200 | 24.4 ms | 115,451 B (112.7 KB) | **PASS** |
| `/case-studies` | 200 | 28.4 ms | 119,571 B (116.8 KB) | **PASS** |
| `/portfolio` | 200 | 27.2 ms | 103,695 B (101.3 KB) | **PASS** |
| `/careers` | 200 | 25.1 ms | 97,500 B (95.2 KB) | **PASS** |
| `/offices` | 200 | 26.0 ms | 99,695 B (97.4 KB) | **PASS** |
| `/useful-links` | 200 | 25.2 ms | 93,306 B (91.1 KB) | **PASS** |
| `/blog` | 200 | 27.7 ms | 88,404 B (86.3 KB) | **PASS** |
| `/legal/privacy-policy` | 200 | 24.3 ms | 86,194 B (84.2 KB) | **PASS** |
| `/legal/terms-of-service` | 200 | 24.2 ms | 85,539 B (83.5 KB) | **PASS** |
| `/legal/cookie-policy` | 200 | 24.1 ms | 84,030 B (82.1 KB) | **PASS** |
| `/legal/refund-policy` | 200 | 24.7 ms | 86,486 B (84.5 KB) | **PASS** |
| `/legal/service-level-agreement` | 200 | 25.0 ms | 92,439 B (90.3 KB) | **PASS** |
| `/legal/accessibility` | 200 | 24.1 ms | 83,862 B (81.9 KB) | **PASS** |
| `/login` | 200 | 16.7 ms | 9,497 B (9.3 KB) | **EXCELLENT** |
| `/register` | 200 | 23.4 ms | 42,899 B (41.9 KB) | **EXCELLENT** |
| `/password/reset` | 200 | 18.3 ms | 5,623 B (5.5 KB) | **EXCELLENT** |
| `/sitemap.xml` | 200 | 23.4 ms | 33,755 B (33.0 KB) | **EXCELLENT** |
| `/healthz` | 200 | 22.0 ms | 60 B | **EXCELLENT** |

---

## 2. Frontend Asset Bundle Audit (`npm run build`)

| Asset File | Minified Size | Gzipped Size | Assessment |
|---|---:|---:|---|
| `public/build/manifest.json` | 0.86 KB | 0.26 KB | Optimal |
| `public/build/assets/app-3d0580a5.css` | 271.96 KB | 41.74 KB | Acceptable; contains global terminal/3D/dark-mode utility rules |
| `public/build/assets/hero-globe-54c134e3.js` | 11.50 KB | 4.28 KB | Optimal; lazy-loaded only when `[data-globe]` exists |
| `public/build/assets/global-3d-e9c9cac1.js` | 26.42 KB | 8.26 KB | Optimal; pauses automatically when tab is hidden or mobile cost cap triggers |
| `public/build/assets/app-5374e343.js` | 57.79 KB | 20.69 KB | Optimal (Alpine.js + app store) |
| `public/build/assets/three.module-ea499613.js` | **516.09 KB** | **132.05 KB** | **Medium Impact:** Three.js core chunk exceeds Vite's 500 KB warning threshold |

---

## 3. Highest-Impact Performance Improvements (Prioritized)

1. **Paginate or Lazy-Render `/services` Catalog (High Impact):**
   - *Evidence:* `/services` renders all 14 categories and all 237 services with 694 `<x-service-price>` components in a single DOM tree, producing a **795 KB HTML document** and taking **676.7 ms** server time.
   - *Fix:* Show top 6 services per category on `/services` when no `?category=` filter is selected, with a `"View all X services in {Category}"` link to `/services?category={slug}`. This cuts HTML size by ~70% and server render time to `<80 ms`.
2. **Eliminate Uncached Layout Queries on Every Public Request (Medium Impact):**
   - *Evidence:* `resources/views/layouts/public.blade.php` executes `ServiceCategory::where('is_active', true)->orderBy('sort_order')->take(9)->get()` in the navbar (line 133) and `take(6)->get()` in the footer (line 311) plus `Setting::get()` calls on every page load.
   - *Fix:* Cache active navigation categories in `Cache::remember('public.nav.categories', 3600, ...)` via a View Composer.
3. **Defer Three.js Loading Until Idle / Intersection (Medium Impact):**
   - *Evidence:* `three.module-ea499613.js` is 516 KB minified (132 KB gzip).
   - *Fix:* Load the 3D scene module via `requestIdleCallback` or `IntersectionObserver` after Largest Contentful Paint (LCP) completes so initial paint is never blocked on mobile networks.
