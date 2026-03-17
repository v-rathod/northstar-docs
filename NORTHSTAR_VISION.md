# NorthStar — Vision & Architecture

> **Last Updated:** March 10, 2026

---

## What is NorthStar?

**NorthStar** is an end-to-end data intelligence platform that transforms raw U.S. immigration data into personalized, actionable insights for immigrants navigating the employment-based green card process. It answers the questions that keep hundreds of thousands of skilled immigrants awake at night:

- *How long will I wait for my green card?*
- *Is my employer a reliable sponsor?*
- *Am I being paid fairly compared to market rates?*
- *Should I stay in EB2 or downgrade to EB3?*
- *Which occupations and locations have the strongest visa demand?*

NorthStar does this by collecting data from 15+ official government sources (DOL, DOS, USCIS, BLS, DHS), curating it into analytical tables, training forecasting models, and serving everything through a beautiful, zero-cost web application.

---

## The Three Pillars

NorthStar is composed of three independent but tightly connected projects that form a data pipeline:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   P1: HORIZON                                                   │
│   fetch-immigration-data                                        │
│   "Scans the horizon for new data"                              │
│                                                                 │
│   • Downloads from 15 government sources                        │
│   • 1,230+ files across 207 folders (3-5 GB)                    │
│   • Incremental: only downloads what's new                      │
│   • Outputs: raw PDFs, Excel, CSV in downloads/                 │
│                                                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │ reads from (never copies)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   P2: MERIDIAN                                                  │
│   immigration-model-builder                                     │
│   "The analytical backbone — curates, measures, models"         │
│                                                                 │
│   • 4-stage pipeline: curate → features → models → export      │
│   • 46+ Parquet tables (18.5M+ rows)                            │
│   • 6 dimensions, 18 fact tables, 14 feature tables             │
│   • 3 model outputs (PD forecasts, SRS scores, salary profiles) │
│   • RAG: 341 chunks + 684 QA pairs for search                  │
│   • Outputs: artifacts/ (Parquet, JSON, YAML)                   │
│                                                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │ exports artifacts via sync script
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   P3: COMPASS                                                   │
│   immigration-insights-app                                      │
│   "Guides users with insights"                                  │
│                                                                 │
│   • Next.js 16 static export (zero runtime compute)             │
│   • 8 interactive dashboards + personalized insights            │
│   • RAG-powered Q&A with LLM fallback                           │
│   • AWS S3 + CloudFront ($1-3/month hosting)                    │
│   • TypeScript strict, Tailwind CSS, Recharts                   │
│   • Outputs: static HTML/CSS/JS in out/                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Directory Layout

All three projects live under a single parent directory:

```
/Users/vrathod1/dev/NorthStar/
├── fetch-immigration-data/          ← P1 Horizon
├── immigration-model-builder/       ← P2 Meridian
├── immigration-insights-app/        ← P3 Compass
└── NORTHSTAR_VISION.md              ← This file (shared context)
```

---

## Naming Convention

| Context | Use | Examples |
|---------|-----|---------|
| Internal code, comments, logs | P1, P2, P3 | `# P2 reads P1 data directly` |
| Public-facing docs, UI, README | Codenames | Horizon, Meridian, Compass |
| Internal program name | NorthStar | "NorthStar program", "NorthStar platform" |
| User-visible UI (web app) | Compass only | No NorthStar branding in the app |

**NorthStar** is the internal program name. It should never appear in the public web application UI. The app is called **Compass**.

---

## Data Pipeline — How It All Connects

### Step 1: Collection (P1 Horizon)

Horizon runs `fetch_latest.py` with a manifest-based incremental download system. It collects:

| Source | Data | Format |
|--------|------|--------|
| Dept. of Labor | PERM cases, LCA filings, record layouts | Excel, CSV |
| State Dept. | Visa Bulletin (monthly), visa issuances, waiting list | PDF, CSV |
| USCIS | I-140/I-485/I-765 approvals, H-1B employer hub | Excel, CSV |
| BLS | OEWS wage percentiles, employment statistics | Excel |
| DHS | Admissions by class of admission | CSV |

All raw data lands in `downloads/` (~3-5 GB, gitignored). The manifest tracks URLs, file hashes, and timestamps to avoid re-downloading.

### Step 2: Curation & Modeling (P2 Meridian)

Meridian reads directly from P1's `downloads/` directory (never copies). Its 4-stage pipeline:

1. **Curate**: Parse raw files → canonical dimension + fact tables (Parquet)
2. **Features**: Engineer derived metrics (employer features, salary benchmarks, demand metrics)
3. **Models**: Train forecasting models (priority date regression, employer scoring, salary profiles)
4. **Export**: Generate RAG chunks + QA pairs for Compass search

Key artifacts:
- **Dimensions**: employer (243K), SOC (1,801), country (249), area (587), visa_class (6)
- **Facts**: PERM (1.7M rows), LCA (9.6M), OEWS (446K), cutoffs (14K), approvals (1K+)
- **Models**: PD forecasts (56 series × 24 months), SRS scores (70K rules + 1,695 ML), salary profiles

### Step 3: Delivery (P3 Compass)

Compass runs `sync_p2_data.py` which converts P2 Parquet → optimized JSON slices, then `npm run build` generates a static site:

```bash
# Sync and build
python3 scripts/sync_p2_data.py    # Parquet → JSON
npm run build                       # Next.js static export → out/

# Deploy
aws s3 sync out/ s3://BUCKET --delete
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/*"
```

The web app has zero runtime compute. All interactivity (search, filtering, personalization) runs client-side in the browser.

---

## Architectural Principles

### 1. Zero Runtime Cost
No Lambda, no database, no API Gateway. The entire platform runs on pre-computed data served as static files. AWS hosting costs $1-3/month.

### 2. Data Flows Down, Never Up
P1 → P2 → P3. Each project reads from the one above it. No project writes back upstream.

### 3. Separation of Concerns
P1 owns data collection. P2 owns analytics and modeling. P3 owns user experience. No project does another's job.

### 4. Pre-Compute Everything
All ML models, forecasts, aggregations, and statistics are computed in P2 at build time. P3 only reads and renders. The browser never runs expensive calculations.

### 5. Privacy First
Zero cookies, zero tracking pixels, zero user accounts. All personalization (priority date, employer, job title) is stored in the browser's localStorage and never leaves the device.

### 6. Data Provenance
Every curated table tracks its source files, ingestion timestamps, and data quality metrics. Every number in the UI can be traced back to a specific government source file.

### 7. Incremental by Design
P1 incrementally downloads (manifest-based). P2 incrementally rebuilds (change detection). P3 incrementally deploys (S3 sync deltas). Full rebuilds are rare.

### 8. Deterministic & Reproducible
Given the same P1 data, P2 produces identical artifacts. Given the same P2 artifacts, P3 produces an identical website. No randomness, no external API calls during build.

---

## Technology Stack

| Layer | P1 Horizon | P2 Meridian | P3 Compass |
|-------|-----------|-------------|------------|
| Language | Python 3.12 | Python 3.12 | TypeScript 5 (strict) |
| Framework | Scripts | Pipeline CLI | Next.js 16 (App Router) |
| Data Format | Raw (PDF/Excel/CSV) | Parquet + JSON | JSON (static) |
| Testing | pytest | pytest (349+ tests) | Vitest (545+ tests) |
| Key Libs | requests, selenium, PyPDF2 | pandas, pyarrow, pdfplumber | Tailwind, Recharts, Framer Motion |
| Hosting | Local only | Local only | AWS S3 + CloudFront |
| Cost | $0 | $0 | $1-3/month |

---

## Key Metrics

| Metric | Value |
|--------|-------|
| Government data sources | 15 active |
| Raw data files | 1,230+ |
| Curated tables | 46+ |
| Total rows processed | 18.5M+ |
| ML model outputs | 3 (PD forecasts, SRS, salary profiles) |
| Interactive dashboards | 8 |
| RAG search chunks | 341 |
| Pre-computed QA pairs | 684 |
| Test coverage | 900+ tests across all projects |
| Monthly AWS cost | ~$1-3 |

---

## Guardrails for Future Development

### What You MUST NOT Do

1. **Add a backend to P3** — No Lambda, no API routes, no database. Ever. If you need computed data, add it to P2.
2. **Copy P1 data into P2** — P2 reads P1 data in-place. The `downloads/` directory is the single source of truth.
3. **Put API keys in P3 source code** — The only permitted env vars are `NEXT_PUBLIC_POSTHOG_*` (analytics) and `NEXT_PUBLIC_GROQ_API_KEY` (RAG LLM, optional).
4. **Break the P2→P3 data contract** — Don't rename columns, don't change primary keys, don't remove RAG topics without updating P3's types and loaders.
5. **Add heavy client-side computation** — If a calculation takes >100ms, it belongs in P2 pre-computation.
6. **Show NorthStar branding in the app UI** — The web app is "Compass". NorthStar is the internal program name only.
7. **Add paid services** — The $1-3/month constraint is non-negotiable. Free-tier services only.

### What You SHOULD Do

1. **Add new data sources to P1** when government agencies publish new datasets.
2. **Add new feature/metric tables to P2** when you need new analytics for the UI.
3. **Add new dashboards to P3** that visualize P2 artifacts — the architecture supports unlimited dashboards.
4. **Update tests** whenever you change behavior — all three projects have comprehensive test suites.
5. **Update documentation** — copilot-instructions.md, PROGRESS.md, and this vision doc should reflect current state.
6. **Run the full pipeline** after any P1 update: fetch → build → sync → deploy.

---

## For AI Agents

### Starting a Session

When you start a new AI chat session on any NorthStar project:

1. **Read this file first** (`/Users/vrathod1/dev/NorthStar/NORTHSTAR_VISION.md`) to understand the program vision and architecture.
2. **Read the project-specific context**:
   - P1: `.github/copilot-instructions.md` + `.ai-instructions.md`
   - P2: `.github/copilot-instructions.md` + `.copilot-context.md` + `PROGRESS.md`
   - P3: `.github/copilot-instructions.md` + `PROGRESS.md`
3. **Understand the data flow**: P1 → P2 → P3. Changes in upstream projects may require downstream updates.
4. **All three projects share a parent directory** at `/Users/vrathod1/dev/NorthStar/`. You may need to read or modify files across projects.

### Cross-Project Changes

If your change affects the data contract between projects:
- **P1 change** (new data source) → Update P2 builder + P2 tests → Update P3 sync + P3 types
- **P2 schema change** (renamed column) → Update P3 types + P3 data loaders + P3 tests
- **P3 new dashboard** → Verify P2 artifact exists → Add P3 loader + component + tests

### Session Documentation

After every significant session:
1. Update `PROGRESS.md` in the relevant project(s)
2. Update `copilot-instructions.md` if file inventory, test counts, or architecture changed
3. Commit and push changes
