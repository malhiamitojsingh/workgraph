# Plan: India Tech Hubs Job Board (Zero‑Framework, Black‑and‑White, Semantic HTML)

## Phase 1: Strategic & Geographic Foundation

- **City Scope – Six Indian Tech Hubs Only**
  - Bengaluru (highest AI/startup/GCC density)
  - Hyderabad (enterprise tech, pharma‑tech)
  - Delhi NCR (AI, consulting, public sector)
  - Pune (manufacturing and engineering R&D)
  - Mumbai (BFSI and corporate headquarters)
  - Chennai (established IT and GCC talent)
  - **Constraint:** Crawler and UI will strictly filter to these six cities. No other locations appear anywhere.

- **Data Acquisition Model – Crawler‑First Aggregation**
  - No user‑submitted job postings.
  - Node.js worker script runs every 3–6 hours on a free cron service (GitHub Actions, Vercel Cron Jobs, or a small VPS).
  - Primary Sources
    - Applicant Tracking System sitemaps: Greenhouse, Lever, Ashby.
    - Public RSS feeds of major Indian job portals.
    - Monthly Hacker News "Who is Hiring?" threads.
    - Relevant GitHub remote‑job repositories.

- **Technology Stack Constraints**
  - Frontend: Pure HTML, vanilla JavaScript, and a single `<style>` tag in the `<head>`. No separate CSS files. No frameworks (React, Vue, Angular, etc.).
  - Backend: Node.js or Bun for the crawler and a simple REST API.
  - Database: PostgreSQL for structured storage.
  - Search Engine: Meilisearch (self‑hosted, open‑source, Rust‑based, instant‑as‑you‑type).
  - Deployment: Vercel for static assets and serverless API endpoints, or a Hetzner VPS with Coolify.

---

## Phase 2: Semantic HTML Shell & Accessibility Baseline

- **Objective**
  - Build a page that is fully functional and navigable before any CSS loads.
  - Ensure perfect keyboard and screen‑reader support using native HTML elements and ARIA.

- **Font Strategy – Zero Network Overhead**
  - Use the system UI font stack.
  - No external font files (no Google Fonts, no Adobe Fonts).
  - Font list: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif`.

- **Landmark Structure (Native HTML Tags)**
  - `<header>` contains the main navigation and search form.
  - `<nav>` wraps the city list, using `<ul>` and `<li>`.
  - `<form role="search">` with a visible `<label>` and `<input type="search">`.
  - `<main>` holds the primary content and has `tabindex="-1"` for focus management.
  - `<footer>` contains copyright and legal text.

- **Accessibility Enhancements**
  - Skip‑to‑content link at the top of the page.
  - ARIA labels on navigation (`aria-label="Main navigation"`) and search (`aria-label="Job search"`).
  - `aria-current="page"` on the active navigation link.
  - Live region (`aria-live="polite"`) for announcing search result counts.

- **Command Palette Modal – Native `<dialog>`**
  - Use the HTML `<dialog>` element for the advanced filter overlay.
  - Benefits: automatic focus trapping, backdrop handling, and ESC key dismissal.
  - Trigger button includes `aria-keyshortcuts="Meta+K"`.

- **Job Card Structure – Semantic Articles**
  - Each job listing is an `<article>` inside a list container (ordered or unordered list).
  - Job title wrapped in an `<h2>` with a hyperlink.
  - Location and salary inside a `<p>` with inline `<span>` elements.
  - Posting date uses the `<time>` element with a valid `datetime` attribute.

---

## Phase 3: Design Engineered Black‑and‑White UI

- **Constraint**
  - All CSS resides in a single `<style>` block in the `<head>`.
  - No external stylesheets, no preprocessors, no CSS‑in‑JS.

- **Monochrome Color Palette**
  - Background: white (`#ffffff`).
  - Primary text: near‑black (`#111111`).
  - Borders and dividers: light gray (`#e5e5e5`).
  - Secondary text (timestamps, metadata): medium gray (`#666666`).
  - Hover and focus states use black (`#000000`) combined with opacity transitions.

- **Layout – CSS Grid**
  - Job cards use `display: grid` with two columns (content and optional metadata).
  - Page layout uses a centered single‑column grid with max‑width and generous padding.
  - No float‑based or absolute‑positioning hacks.

- **Minimal Animations (Performance‑First)**
  - Only `opacity` and `transform` properties are animated.
  - Link and button hover states use `transition: opacity 0.15s ease`.
  - Dialog opening animation uses a subtle `fade-in` keyframe.
  - No animation on page load, width, height, or layout‑inducing properties.

- **Focus Indicators**
  - Preserve browser default `:focus-visible` outline.
  - Enhance with `outline: 2px solid #000` and `outline-offset: 2px`.

- **Responsive Behavior**
  - Grid and flex containers wrap naturally.
  - Font sizes use relative units (`rem`) with a base of 16px.
  - Touch targets are at least 44×44 pixels.

---

## Phase 4: Crawler & Data Pipeline (The Engine)

- **Crawler Architecture – Node.js Worker**
  - Single script runs on a schedule.
  - Uses `axios` for HTTP requests, `xml2js` for parsing sitemaps, and `cheerio` for HTML extraction.
  - City Filter: Checks job location text against the six city names and known suburbs. Only matching jobs are processed.

- **Data Normalization and Deduplication**
  - Location Normalization: Map variations to canonical city names (e.g., "BLR" → "Bengaluru").
  - Tech Stack Normalization: Simple mapping object to unify synonyms (e.g., "React.js" → "React").
  - Duplicate Detection: Compare title, company, and location within a three‑day window before inserting into the database.

- **Database Schema (PostgreSQL)**
  - Table `jobs` with columns: `id`, `title`, `company`, `location` (enum or text constrained to six cities), `description_text`, `posted_date`, `source_url`, `tech_stack` (array), `created_at`.
  - Indexes on `location`, `posted_date`, and `tech_stack` for fast queries.

- **Execution Environment**
  - GitHub Actions workflow with a cron trigger (`0 */6 * * *`).
  - Or a lightweight VPS with a systemd timer.

---

## Phase 5: Meilisearch Integration & Instant Search

- **Meilisearch Setup**
  - Self‑hosted on the same VPS as the API.
  - Run as a systemd service with a master key for security.
  - Create an index named `jobs` with searchable attributes: `title`, `company`, `location`, `tech_stack`, `description_text`.

- **Indexing from the Crawler**
  - After saving a job to PostgreSQL, push the same document to Meilisearch using the official JavaScript client.
  - Include a unique `id` field matching the database primary key.

- **Frontend Search Implementation**
  - Use the official `instantsearch.js` library (vanilla JavaScript version).
  - Connect to Meilisearch using the `instant-meilisearch` adapter.
  - Widgets configured:
    - `searchBox` bound to the main search input.
    - `hits` to render job cards with a custom template.
    - `refinementList` for city filtering (attribute: `location`).
    - `pagination` for navigating results.

- **Accessibility Integration with InstantSearch**
  - Listen for the `render` event.
  - Update a dedicated ARIA live region element with the number of results found.
  - Manage focus when filters are applied or cleared.

---

## Phase 6: SEO Domination

- **Structured Data – JSON‑LD JobPosting**
  - Every job detail page includes a `<script type="application/ld+json">` block.
  - Contains all required Google properties: `title`, `description`, `datePosted`, `validThrough`, `hiringOrganization`, `jobLocation` with `addressLocality` set to the Indian city.

- **Programmatic SEO (pSEO) Pages**
  - Generate static HTML pages for every combination of city and major tech skill.
  - Example URLs: `/bengaluru-react-jobs`, `/hyderabad-python-jobs`, `/pune-ai-ml-jobs`.
  - Each page features:
    - Unique `<h1>` and `<meta name="description">`.
    - A short, AI‑generated market analysis (200‑300 words) specific to that city and skill.
    - A list of the 20 most recent matching jobs.

- **Technical SEO Foundations**
  - Dynamic Sitemap: `/sitemap.xml` split into multiple files when exceeding 50,000 URLs.
  - Canonical Tags: Self‑referencing `<link rel="canonical">` on every page.
  - Mobile‑First Design: The minimalist grid and system fonts guarantee excellent rendering on all devices.
  - Page Speed: Target Google Lighthouse score of 100/100 (achievable due to zero framework overhead).
  - Google Indexing API: Notify Google instantly when new jobs are added or old ones expire.

- **Semantic HTML as an SEO Signal**
  - Proper use of `<article>`, `<h2>`, `<time>`, and `<address>` (or location spans) provides strong contextual clues to search engines.

---

## Phase 7: Monetization (Optional, UX‑Friendly)

- **Sponsored Listings Model**
  - Free tier: Jobs appear in the feed in chronological order.
  - Paid tier: A company can pay a flat monthly fee to have one or more jobs pinned to the top of relevant city or skill pages.
  - Visual Distinction: Sponsored jobs receive a subtle visual treatment (e.g., a thin black left border) and a small "Sponsored" label.

- **No Traditional Display Ads**
  - Avoid banner ads, pop‑ups, or intrusive overlays to maintain the clean, high‑trust design.

---

## Phase 8: Launch & Monitoring

- **Deployment**
  - Frontend: Vercel connected to a GitHub repository for automatic deployments on push.
  - Backend API and Meilisearch: Deployed on a single VPS (e.g., Hetzner) using Coolify for easy management or a simple Docker Compose setup.

- **Monitoring & Alerts**
  - Use Cronitor or Healthchecks.io to ping the crawler endpoint after each scheduled run.
  - Alert via SMS or email if the crawler fails or the total job count drops by more than 10%.

- **Analytics**
  - Integrate Plausible Analytics (privacy‑focused, cookie‑less).
  - No need for a cookie consent banner, further improving page speed and user trust.

- **Maintenance Checklist**
  - Weekly review of crawler logs for new ATS domains or broken selectors.
  - Monthly Meilisearch version updates.
  - Quarterly content review of pSEO pages for freshness.

---

## Phase 9: Final Deliverable Summary

- The site is a single, fast HTML document with embedded styles.
- It lists only jobs from the six designated Indian tech hubs.
- It uses semantic HTML and ARIA to be fully accessible.
- It features a black‑and‑white, system‑font, grid‑based design with minimal, purposeful animations.
- It is powered by a scheduled Node.js crawler that populates PostgreSQL and Meilisearch.
- It offers instant, typo‑tolerant search via Meilisearch.
- It is optimized to rank #1 in Google for niche Indian tech job queries through rigorous technical SEO and structured data.
