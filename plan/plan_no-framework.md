# India Tech Hubs Job Board: Enhanced Aggregator Plan

## Phase 1: Strategic Foundation & Source Prioritization

- **Geographic Focus – Six Indian Tech Hubs Only**
  - Bengaluru
  - Hyderabad
  - Delhi NCR
  - Pune
  - Mumbai
  - Chennai
  - All crawlers and UI filters will strictly enforce this list. Jobs from other cities are discarded.

- **Data Source Strategy – Prioritized for India Relevance**
  - Primary Tier (High-Volume India Platforms)
    - Naukri (India's largest job portal)
    - Shine (HT Media)
    - TimesJobs (Times Group)
    - Foundit (Monster India)
    - Instahyre (AI-powered, tech focus)
    - Hirist (exclusive tech jobs in India)
    - CutShort (startup and tech roles)
  - Secondary Tier (Global Platforms with Strong India Presence)
    - Indeed
    - Glassdoor
    - Monster
    - SimplyHired
    - LinkedIn (via public feeds or partner API)
    - StackOverflow Jobs
    - GitHub Jobs/Careers
  - Tertiary Tier (Remote and Niche Tech)
    - RemoteOK
    - WeWorkRemotely
    - AngelList/Wellfound
    - Dice
    - Turing
    - Arc
    - FlexJobs
  - **Source Count:** 24 total platforms identified; initial launch will focus on Primary Tier and top Secondary sources.

- **Technology Stack Constraints (Unchanged)**
  - Frontend: Pure HTML, vanilla JavaScript, single `<style>` tag. No frameworks.
  - Backend: Node.js or Bun for crawler and API.
  - Database: PostgreSQL.
  - Search Engine: Meilisearch (self-hosted).
  - Deployment: Vercel + VPS.

---

## Phase 2: Semantic HTML Shell & Accessibility (Unchanged)

- **Objective**
  - Fully functional page before CSS loads.
  - Perfect keyboard and screen-reader navigation.

- **Font Strategy – System UI Font Stack Only**
  - No external font downloads.

- **Landmark and ARIA Structure**
  - Semantic elements: `<header>`, `<nav>`, `<main>`, `<footer>`, `<article>`, `<time>`.
  - Explicit ARIA labels for navigation and search.
  - Skip-to-content link.
  - Live region for search result announcements.

- **Command Palette Modal – Native `<dialog>` Element**
  - Handles focus trapping and backdrop automatically.

- **Job Card Structure**
  - Each job is an `<article>` inside an ordered or unordered list.

---

## Phase 3: Design Engineered Black‑and‑White UI (Unchanged)

- **Constraint**
  - Single `<style>` block in `<head>`.

- **Monochrome Palette**
  - Background: `#ffffff`
  - Text: `#111111`
  - Borders: `#e5e5e5`
  - Secondary text: `#666666`

- **Layout – CSS Grid**
  - Clean, responsive, no floats.

- **Minimal Animations**
  - Only `opacity` and `transform` transitions.
  - Hover effects: subtle opacity change.

- **Focus Indicators**
  - Enhanced `:focus-visible` outline.

- **Responsive Behavior**
  - Relative units, touch targets minimum 44×44 pixels.

---

## Phase 4: Enhanced Crawler & Data Pipeline (Revised with Source Strategy)

- **Crawler Architecture – Node.js Worker**
  - Single script triggered by cron schedule.
  - Uses `axios`, `xml2js`, `cheerio`.
  - City Filter: Checks location text against six canonical city names and known suburbs.

- **Source-Specific Handling – Tiered Approach**
  - **Primary Tier (India Portals)**
    - Scraping required for most (Naukri, Shine, TimesJobs, Foundit).
    - Implement lightweight, targeted scrapers with selectors that are regularly maintained.
    - Use Apify or custom scripts with caching to reduce redundant requests.
  - **Secondary Tier (Global Platforms)**
    - Use official APIs where available:
      - Indeed Job Sync API (requires partner agreement, fallback to scraping if needed)
      - LinkedIn Partner Program API (if accessible)
      - RemoteOK public API
      - WeWorkRemotely READ API
    - For platforms without APIs (Glassdoor, SimplyHired), use scraping with careful rate limiting.
  - **Tertiary Tier (Niche Tech)**
    - Most offer public RSS/Atom feeds or simple HTML.
    - Crawl once daily due to lower volume.

- **Smart Scheduling Strategy**
  - **High-Volume Sources:** Naukri, Indeed, Shine → every 2-4 hours.
  - **Medium-Volume Sources:** TimesJobs, Foundit, Instahyre, Hirist → every 6-8 hours.
  - **Low-Volume Sources:** CutShort, RemoteOK, WeWorkRemotely, StackOverflow → once daily.
  - **Real-Time Updates:** Implement webhook listeners where platforms support them to avoid polling.

- **Data Normalization and Deduplication (Enhanced)**
  - **Location Normalization:** Map variants like "Bengaluru", "Bangalore", "BLR" to "Bengaluru".
  - **Company Name Normalization:** Remove suffixes like "Pvt Ltd", "Inc." for matching.
  - **Job Title Normalization:** Map synonyms (e.g., "Frontend Developer" ↔ "Front End Engineer").
  - **Deduplication Engine:**
    - Generate a hash based on: normalized title + normalized company + city + first 150 chars of description.
    - Use fuzzy matching for slight variations (Levenshtein distance on title and company).
    - Store canonical job entry in PostgreSQL; subsequent duplicates from other sources are ignored or logged.

- **Database Schema (PostgreSQL)**
  - Table `jobs`:
    - `id` (UUID)
    - `title` (text)
    - `normalized_title` (text)
    - `company` (text)
    - `normalized_company` (text)
    - `location` (text, constrained to six city values)
    - `description_text` (text)
    - `posted_date` (timestamp with time zone)
    - `source_url` (text, unique)
    - `tech_stack` (text array)
    - `created_at` (timestamp)
    - `source_platform` (text)
  - Indexes on `location`, `posted_date`, `tech_stack`, `normalized_title`, `normalized_company`.

- **Execution Environment**
  - GitHub Actions with cron triggers for each tier.
  - Fallback VPS cron jobs for reliability.

---

## Phase 5: Meilisearch Integration & Instant Search (Unchanged)

- **Meilisearch Setup**
  - Self-hosted on VPS with master key.
  - Index `jobs` with searchable attributes: `title`, `normalized_title`, `company`, `normalized_company`, `location`, `tech_stack`, `description_text`.

- **Indexing from Crawler**
  - After PostgreSQL insert, push document to Meilisearch using JavaScript client.

- **Frontend Search**
  - `instantsearch.js` with `instant-meilisearch` adapter.
  - Widgets: `searchBox`, `hits`, `refinementList` (city), `pagination`.
  - Accessibility: ARIA live region announces result count on render.

---

## Phase 6: SEO Domination (Unchanged)

- **Structured Data – JSON-LD JobPosting**
  - Every job detail page includes complete schema.

- **Programmatic SEO (pSEO) Pages**
  - Static pages for city–skill combinations.
  - Unique content and 20 most recent matching jobs.

- **Technical SEO**
  - Dynamic sitemap, canonical tags, mobile-first, target Lighthouse 100.
  - Google Indexing API for instant updates.

- **Semantic HTML as SEO Signal**
  - Proper use of `<article>`, `<h2>`, `<time>`, and location markup.

---

## Phase 7: Monetization (Optional, Unchanged)

- **Sponsored Listings Model**
  - Free tier: chronological feed.
  - Paid tier: pinned jobs with subtle black left border and "Sponsored" label.
  - No banner ads.

---

## Phase 8: Launch & Monitoring (Enhanced with Source Health)

- **Deployment**
  - Frontend: Vercel (GitHub integration).
  - Backend API and Meilisearch: VPS with Coolify or Docker Compose.

- **Monitoring & Alerts**
  - Cronitor or Healthchecks.io pings crawler after each run.
  - Alert if job count drops more than 10% overall or per primary source.
  - Source-specific health checks: Monitor selector changes for scraped platforms; trigger manual review if failure persists.

- **Analytics**
  - Plausible for privacy-first, cookie-less insights.

- **Maintenance Checklist (Updated)**
  - Weekly: Review crawler logs for new domains or broken selectors.
  - Monthly: Update Meilisearch and dependencies.
  - Quarterly: Audit pSEO page content freshness.
  - Bi-annual: Re-evaluate source list for new or deprecated job boards.

---

## Phase 9: Final Deliverable Summary (Updated)

- **Scope:** Six Indian tech hubs only.
- **Sources:** 24 prioritized platforms, with a focus on India-specific boards.
- **Frontend:** Single HTML file, embedded styles, system fonts, semantic and accessible.
- **Design:** Black‑and‑white, grid‑based, minimal animations.
- **Backend:** Node.js crawler with tiered scheduling, deduplication, and API fallbacks.
- **Search:** Meilisearch with instant, typo‑tolerant results.
- **SEO:** Optimized for #1 ranking via structured data, pSEO, and speed.
- **Compliance:** Respects `robots.txt`, rate limits, and platform terms; adds value through aggregation and enhanced search.
