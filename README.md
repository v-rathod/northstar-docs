# 🌐 NorthStar — Shared Documentation

**Shared documentation hub for the NorthStar Immigration Analytics Program.**

> **Version:** 1.0 | **Last Updated:** Mar 17, 2026

---

## 📋 Documentation Index

### Core Program Documents

1. **[GUARDRAILS.md](GUARDRAILS.md)** — **The Ten Commandments** (READ FIRST)
   - Non-negotiable architectural constraints for the entire program
   - No-backend architecture, $5/month cost constraint, unidirectional data flow
   - Mandatory testing, regression test requirements, data contract protection
   - Program-level guardrails: architecture, data quality, testing, security, git workflow
   - Cross-references to project-specific guardrails in P1, P2, P3

2. **[NORTHSTAR_VISION.md](NORTHSTAR_VISION.md)** — Program architecture, vision, and design principles
   - Overview of Horizon (P1), Meridian (P2), Compass (P3)
   - Data flow: collection → analytics → web
   - Design guardrails and non-negotiables
   
3. **[BEST_PRACTICES.md](BEST_PRACTICES.md)** — Engineering conventions across all 3 projects
   - TypeScript strict mode, security standards
   - Testing strategy, code review checklist
   - Agent workflow guidelines
   - Data validation rules

4. **[SETUP_GUIDE.md](SETUP_GUIDE.md)** — How to set up all 3 projects locally

---

## 🏗️ NorthStar Program Structure

```
NorthStar Immigration Analytics Program
│
├─ Horizon (P1): Data Collection
│  Repository: fetch-immigration-data
│  Role: Fetches immigration PDFs, parses tables, stores raw data
│  Output: Raw CSV/Parquet files → P2 Meridian
│
├─ Meridian (P2): Analytics & ML Pipeline
│  Repository: immigration-model-builder
│  Role: Builds forecasts, computes scores, generates artifacts
│  Output: Parquet artifacts + JSON exports → P3 Compass
│
└─ Compass (P3): Web Dashboard & UI
   Repository: immigration-insights-app
   Role: Static Next.js site with interactive dashboards
   Output: Static HTML/CSS/JS → AWS S3/CloudFront
```

### Data Flow
```
P1 Horizon         P2 Meridian           P3 Compass
(Raw Data) ——→ (Analytics) ——→ (Web App)
   │                  │              │
   ├ PDFs          ├ Forecasts    ├ 16 pages
   ├ Tables        ├ Scores       ├ 9 dashboards
   └ CSV           ├ Metrics      └ ~100K users
                   └ Export
```

---

## 🚀 Quick Start

### For Contributors (All Projects)

1. **Read the vision first:**
   ```bash
   cat NORTHSTAR_VISION.md
   ```

2. **Set up your environment:**
   ```bash
   # See SETUP_GUIDE.md for detailed instructions
   cd ../fetch-immigration-data        # P1
   cd ../immigration-model-builder     # P2
   cd ../immigration-insights-app      # P3
   ```

3. **Follow engineering standards:**
   ```bash
   # Review BEST_PRACTICES.md before writing code
   ```

### For Each Project

- **P1 (Horizon):** See `fetch-immigration-data/README.md`
- **P2 (Meridian):** See `immigration-model-builder/README.md`
- **P3 (Compass):** See `immigration-insights-app/README.md`

---

## 📊 Current Status (as of Mar 25, 2026)

| Component | Status | Version | Tests |
|-----------|--------|---------|-------|
| P1 Horizon | ✅ Live | Apr 2026 VB ingested | N/A |
| P2 Meridian | ✅ Live | April 2026 artifacts | N/A |
| P3 Compass | ✅ Live | Milestone 21.0 | **1,265 passing** (42 files) |

**Highlights:**
- ✅ P1→P2→P3 end-to-end pipeline operational
- ✅ All 9 dashboards live on Compass
- ✅ 1,265 regression + live-data tests passing
- ✅ SEO optimized (favicon, OG image, JSON-LD, canonical URLs)
- ✅ Deployed to AWS CloudFront (~$1-3/month)
- ✅ Prod: immigrationcompass.fyi | Stage: stage.immigrationcompass.fyi

---

## 🔗 Reference Links

- **P1 Repository:** https://github.com/[you]/fetch-immigration-data
- **P2 Repository:** https://github.com/[you]/immigration-model-builder
- **P3 Repository:** https://github.com/[you]/immigration-insights-app
- **Compass Live:** https://d10immmzyp7xgr.cloudfront.net

---

## ✏️ Contributing

All contributors must:

1. ✅ Read [GUARDRAILS.md](GUARDRAILS.md) — **The Ten Commandments** (non-negotiable rules)
2. ✅ Read [NORTHSTAR_VISION.md](NORTHSTAR_VISION.md) — understand the big picture
3. ✅ Read [BEST_PRACTICES.md](BEST_PRACTICES.md) — follow engineering standards
4. ✅ Read the project-specific `GUARDRAILS.md` — understand your component's rules
5. ✅ Read the project-specific README — understand your component
6. ✅ Follow the agent workflow checklist — if using AI agents

For Pull Requests: See [BEST_PRACTICES.md#code-review](BEST_PRACTICES.md) for pull request requirements.

---

## 🆘 Support & Questions

- **Technical questions:** See project-specific documentation in each repo
- **Architecture questions:** See [NORTHSTAR_VISION.md](NORTHSTAR_VISION.md)
- **Code style questions:** See [BEST_PRACTICES.md](BEST_PRACTICES.md)
- **Setup help:** See [SETUP_GUIDE.md](SETUP_GUIDE.md)

---

## 📝 License

All NorthStar documentation is shared across P1, P2, P3 projects. Each project has its own LICENSE file for code; these documentation files are shared under the same terms.

---

**NorthStar Program** — Building the future of immigration analytics with data-driven insights.
