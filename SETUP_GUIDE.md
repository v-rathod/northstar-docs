# NorthStar Setup Guide

**How to set up all 3 NorthStar projects locally for development or contribution.**

---

## Prerequisites

### System Requirements
- **macOS 12+** or **Linux** (Windows with WSL2 supported)
- **Git** 2.30+
- **Python** 3.9+ (for P1 Horizon and P2 Meridian)
- **Node.js** 18+ (for P3 Compass)
- **npm** 9+ (for P3 Compass)

### Required Tools
```bash
# Check installations
git --version
python3 --version
node --version
npm --version
```

---

## Clone All Three Repositories

```bash
# Create a workspace directory
mkdir -p ~/dev/NorthStar && cd ~/dev/NorthStar

# Clone P1 Horizon (Data Collection)
git clone https://github.com/[your-username]/fetch-immigration-data.git

# Clone P2 Meridian (Analytics)
git clone https://github.com/[your-username]/immigration-model-builder.git

# Clone P3 Compass (Web App)
git clone https://github.com/[your-username]/immigration-insights-app.git

# You're now in: ~/dev/NorthStar/
#   ├── fetch-immigration-data/          (P1)
#   ├── immigration-model-builder/       (P2)
#   ├── immigration-insights-app/        (P3)
#   └── northstar-docs/                  (Shared docs)
```

---

## ⚡ Quick Start (Fastest Path)

**Just want to run the web app?** ✅ Takes 2 minutes:

```bash
git clone https://github.com/v-rathod/immigration-insights-app.git
cd immigration-insights-app
npm install
npm run dev
# Open http://localhost:3000 — everything works!
```

✅ All data is pre-built and committed  
✅ No secrets or configuration needed  
✅ 1,265 tests passing  

**Want the full pipeline (P1→P2→P3)?** See below for complete setup.

---

## Setup P1 — Horizon (Data Collection)

```bash
cd fetch-immigration-data

# Copy environment template (optional, but recommended)
cp .env.example .env
# Edit .env with any API keys if you have them (optional)

# Install Python dependencies
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Verify setup
python3 -c "import pandas; import requests; print('✓ P1 Dependencies OK')"

# Read project README for next steps
cat README.md
```

---

## Setup P2 — Meridian (Analytics)

```bash
cd ../immigration-model-builder

# Copy environment template (optional, but recommended)
cp .env.example .env
# Edit .env with any configuration options (optional)

# Install Python dependencies
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Verify setup
python3 -c "import pandas; import pyarrow.parquet as pq; print('✓ P2 Dependencies OK')"

# Read project README for next steps
cat README.md
```

**Note:** P2 expects P1 data in `../fetch-immigration-data/downloads/`. If you haven't run P1 yet, see "Verify End-to-End" section below.

---

## Setup P3 — Compass (Web App)

```bash
cd ../immigration-insights-app

# Install Node dependencies
npm install

# Verify setup
npm run lint
npm test 2>&1 | tail -5

# Start local development server
npm run dev
# Visit http://localhost:3000

# Read project README for next steps
cat README.md
```

---

## Verify End-to-End Integration

After setting up all three projects:

```bash
# From ~/dev/NorthStar/ directory

# P1: Check that Horizon can run
cd fetch-immigration-data && python3 scripts/fetch_visa_bulletin.py --help && cd ..

# P2: Check that Meridian has data (or can generate it)
cd immigration-model-builder && ls artifacts/tables/*.parquet 2>/dev/null || echo "Run P1 first to generate data" && cd ..

# P3: Check that Compass can build
cd immigration-insights-app && npm run build && cd ..

echo "✓ All three projects verified"
```

---

## Development Workflow

### Working on P3 (Compass — Most Common)

```bash
cd immigration-insights-app

# Start dev server
npm run dev
# Visit http://localhost:3000

# In another terminal, run tests
npm test

# Make changes, tests auto-run

# Build for production
npm run build

# Commit and push
git add .
git commit -m "feat: description of changes"
git push origin main
```

### Working on P2 (Meridian — Data/Analytics)

```bash
cd immigration-model-builder

# Activate Python environment
source venv/bin/activate

# Run pipeline
python3 scripts/rebuild_[artifact_name].py

# Generate new artifacts
python3 scripts/rebuild_all.py

# Sync to P3 (if needed)
cd ../immigration-insights-app
python3 scripts/sync_p2_data.py
```

### Working on P1 (Horizon — Data Collection)

```bash
cd fetch-immigration-data

# Activate Python environment
source venv/bin/activate

# Fetch new data
python3 scripts/fetch_visa_bulletin.py

# Parse and save
python3 scripts/parse_bulletins.py
```

---

## Environment Variables

### P3 (Compass)

Create `.env.local` in `immigration-insights-app/`:

```bash
# PostHog Analytics (optional)
NEXT_PUBLIC_POSTHOG_KEY=your_key_here
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# Feedback Form (optional)
NEXT_PUBLIC_FORMSPREE_ID=your_formspree_id
```

### P2 (Meridian)

Create `.env` in `immigration-model-builder/` (if needed for API keys):

```bash
# Example: if fetching from external APIs
API_KEY=your_key_here
```

---

## Troubleshooting

### "Module not found" errors (Python)

```bash
# Ensure you're in the virtual environment
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

### "Cannot find module" errors (Node)

```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Tests failing

```bash
# P3 (Compass) — clear build cache
npm run build  # or npm run dev
npm test

# P2/P1 — check data files exist
ls artifacts/tables/
```

### Port 3000 already in use (Compass dev server)

```bash
# Kill the process using port 3000
lsof -i :3000
kill -9 <PID>

# Or use a different port
npm run dev -- -p 3001
```

---

## ✅ First-Time Setup Checklist

Use this checklist when setting up on a new machine:

### Pre-Setup
- [ ] Verify system requirements (Git, Python 3.9+, Node.js 18+)
- [ ] Read [NORTHSTAR_VISION.md](NORTHSTAR_VISION.md) (5 min)
- [ ] Create workspace: `mkdir -p ~/dev/NorthStar && cd ~/dev/NorthStar`

### P3 Only (Quickest)
- [ ] Clone P3: `git clone https://github.com/v-rathod/immigration-insights-app.git`
- [ ] Enter dir: `cd immigration-insights-app`
- [ ] Install: `npm install`
- [ ] Run: `npm run dev`
- [ ] Verify: Visit http://localhost:3000 ✅

### Full Setup (All 3)
- [ ] Clone all 3 repos (see "Clone All Three Repositories" section)
- [ ] Setup P1: Python venv + `pip install -r requirements.txt`
- [ ] Setup P2: Python venv + `pip install -r requirements.txt`
- [ ] Setup P3: `npm install`
- [ ] Verify: `npm test` in P3 (should pass all 1,265 tests)
- [ ] Verify: P1 script runs: `python3 scripts/fetch_visa_bulletin.py --help`
- [ ] Verify: P2 data exists: `ls artifacts/tables/*.parquet`

### After Setup
- [ ] Read [BEST_PRACTICES.md](BEST_PRACTICES.md) before writing code
- [ ] Jot down your GitHub fork URLs for easy reference
- [ ] Consider adding `.env` files (copy from `.env.example`—optional)
- [ ] Join contribution workflow (see project README)

---

## 📦 Large Artifacts & GitHub Releases

### The Problem
P2 Meridian generates large Parquet files (500MB+) that shouldn't be in Git history. Currently:
- ✅ P3 (`public/data/`) has pre-built JSON files (~28 MB) → works fine, committed
- ⚠️ P2 (`artifacts/tables/`) has Parquet files → **should be in GitHub Releases**, not Git

### The Solution
Large artifacts are stored as GitHub Releases, not committed to the repo:

```bash
cd immigration-model-builder

# Download latest artifacts from GitHub Release
./scripts/download-artifacts.sh
# or manually from: https://github.com/v-rathod/immigration-model-builder/releases

# This pulls ~500 MB of Parquet files locally without bloating the Git repo
```

### For Contributors Updating Artifacts
```bash
# After rebuilding P2 (rebuild_all.py)
cd immigration-model-builder

# Create new GitHub Release (requires GitHub CLI)
gh release create v1.1 --notes "April 2026 artifacts"
gh release upload v1.1 artifacts/tables/*.parquet

# Tag the commit
git tag v1.1 && git push origin v1.1
```

### Adding to .gitignore (if not already there)
```bash
# immigration-model-builder/.gitignore
artifacts/tables/*.parquet
artifacts/tables/*.csv
artifacts/models/*.pkl
/data/
```

---

## Next Steps

1. ✅ **Read the vision:** `cat northstar-docs/NORTHSTAR_VISION.md`
2. ✅ **Review best practices:** `cat northstar-docs/BEST_PRACTICES.md`
3. ✅ **Pick a project:** P1 (data), P2 (analytics), or P3 (web)
4. ✅ **Read project README:** `cat [project]/README.md`
5. ✅ **Make a change, run tests, commit**

---

## Support

- **Technical issues:** Check project-specific README
- **Architecture questions:** See `NORTHSTAR_VISION.md`
- **Code standards:** See `BEST_PRACTICES.md`
- **Git/GitHub questions:** See project README

---

**Happy contributing to NorthStar! 🚀**
