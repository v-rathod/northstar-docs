# NorthStar — Engineering Best Practices & Agent Guidebook

> **FOR AI AGENTS:** Read this document at the start of every session before making any code changes.
> Cross-reference with the project-specific `copilot-instructions.md` / `ARCHITECTURE.md` in the relevant repo.
>
> **Document purpose:** Captures the evolved technical philosophy, established conventions, and hard-won lessons
> from building the NorthStar program. Every principle here was established through real work — deviate only
> with explicit user instruction, and document the deviation.

---

## 0. Quick Reference — Key Paths

| What | Absolute Path |
|------|--------------|
| Parent (all projects) | `/Users/vrathod1/dev/NorthStar/` |
| P1 Horizon | `/Users/vrathod1/dev/NorthStar/fetch-immigration-data/` |
| P2 Meridian | `/Users/vrathod1/dev/NorthStar/immigration-model-builder/` |
| P3 Compass | `/Users/vrathod1/dev/NorthStar/immigration-insights-app/` |
| Program vision | `/Users/vrathod1/dev/NorthStar/NORTHSTAR_VISION.md` |
| Architecture docs | Each project's `ARCHITECTURE.md` |
| This file | `/Users/vrathod1/dev/NorthStar/BEST_PRACTICES.md` |

---

## 1. Cross-Project Rules (ALL projects)

### 1.1 Git Commit Conventions

Every commit must follow this format:

```
<type>(<scope>): <short summary>

- Detail line 1
- Detail line 2
```

**Type vocabulary:**
| Type | When to use |
|------|------------|
| `feat` | New feature, new dashboard, new component |
| `fix` | Bug fix, data correction |
| `refactor` | Code restructuring without behavior change |
| `docs` | README, PROGRESS.md, copilot-instructions.md updates |
| `test` | Adding or updating tests |
| `chore` | Dependency updates, config changes |
| `perf` | Performance improvements (payload size, load time) |

**Rules:**
- Every feature or fix commit ends with a push to `origin/main`
- Test counts must be correct in the commit message
- Commits that touch multiple files must list all significant changes
- `PROGRESS.md` is updated in the same commit as the work it documents (or a separate `docs:` commit immediately after)

### 1.2 Session Workflow (MANDATORY every session)

At the start of **every coding session:**

1. **Read `NORTHSTAR_VISION.md`** — never forget the program goal
2. **Read `PROGRESS.md`** (last 100 lines) — pick up exactly where the last session left off
3. **Run tests** before making any changes — establish a green baseline
4. **Check git status** — no orphaned uncommitted work

At the end of **every coding session:**

1. **Run tests** — all must pass before committing
2. **Update `PROGRESS.md`** — add a milestone entry for every meaningful piece of work
3. **Update `copilot-instructions.md`** — if file inventory, test counts, or phase status changed
4. **Commit and push** — never leave untracked work; the repo is the source of truth
5. **Update `README.md`** — if status metrics changed (test count, dashboard count, phase)

### 1.3 Documentation Maintenance

| Document | Update trigger |
|----------|---------------|
| `PROGRESS.md` | After every meaningful feature, fix, or data change |
| `copilot-instructions.md` | When file inventory changes, test count changes, or phase advances |
| `README.md` | When product status changes (new dashboards, new features, test count) |
| `ARCHITECTURE.md` | When architectural decisions change (new data sources, new patterns) |
| `BEST_PRACTICES.md` (this file) | When a new convention is established through project work |

### 1.4 Naming Conventions

**Internal codenames (use in code, comments, internal docs):**
- P1 = Horizon = `fetch-immigration-data`
- P2 = Meridian = `immigration-model-builder`
- P3 = Compass = `immigration-insights-app`
- NorthStar = the program (never shown in the Compass web UI)

**Public-facing names (use in user-visible content):**
- The web app is "Compass" — never "NorthStar" in the UI
- "Sponsor Reliability Score" / "SRS" — not "Employer Friendliness Score" / "EFS"
- Priority Date Index / PDC (Priority Date Cortex) — not "forecast" alone

---

## 2. P3 Compass — Next.js / TypeScript / Tailwind

### 2.1 Architecture Constraints (NON-NEGOTIABLE)

1. **Static export only** — `output: 'export'` in `next.config.ts`. Zero server-side code at runtime.
2. **Zero backend** — No API routes, no Lambda, no database. All data is pre-built JSON.
3. **Client-side interactivity only** — Search, filtering, personalization all run in the browser via Fuse.js.
4. **No heavy compute at runtime** — all ML models, forecasts, and aggregations live in P2 Meridian.
5. **AWS cost < $5/month** — S3 static hosting + CloudFront + Route 53 + ACM. That's the entire infrastructure.

### 2.2 TypeScript Rules

```typescript
// ✅ Correct — strict types throughout
interface EmployerScore { employer_id: string; srs: number | null; srs_tier: string; }

// ❌ Wrong — no any, no type assertions without comment
const data: any = fetch(...);
const score = result as EmployerScore;  // only if unavoidable, document why
```

- TypeScript **strict mode** always on — no `any`, no unexplained type assertions
- All P2 artifact schemas typed in `src/types/p2-artifacts.ts`
- Interface names use PascalCase; all exported
- Prefer `interface` over `type` for object shapes; use `type` for unions/primitives

### 2.3 Component Architecture

```
Server Component (default)
  └── "use client" Client Island (only when needed)
        └── Reason: event handlers, useState, useEffect, Framer Motion
```

- **Add `"use client"` only** for: event handlers, state, effects, Framer Motion, Fuse.js
- **Co-locate data loaders**: one `src/lib/data/[topic].ts` per dashboard
- **Naming**: PascalCase files exactly matching the exported component name
- **Barrel exports** via `index.ts` in every component directory
- **shadcn/ui in `src/components/ui/`** — never modify these directly; wrap them

### 2.4 Styling Rules

- **Tailwind only** — no CSS modules, no styled-components, no inline style objects (except for dynamic values)
- **Design tokens via CSS variables** — all colors from `var(--foreground)`, `var(--accent-blue)` etc.
- **Glassmorphic card pattern**:
  ```tsx
  "backdrop-blur-xl bg-white/[0.03] border border-white/[0.08] rounded-2xl"
  ```
- **Gradient text** (headlines only):
  ```tsx
  "bg-gradient-to-r from-blue-400 to-purple-400 bg-clip-text text-transparent"
  ```
- Never use Tailwind `bg-blue-500` for backgrounds — use `bg-[var(--accent-blue)]` so dark/light themes work

### 2.5 Framer Motion Animation Standards

```typescript
const EASING = [0.25, 0.1, 0.25, 1] as const;  // always use this easing

// Stagger: 50ms between sequential card reveals
// Micro-interactions: 200ms duration
// Page transitions: 400ms duration
// Number tickers: spring animation on viewport entry
// NEVER block interaction for animation completion
```

### 2.6 Data Handling Rules

```typescript
// ✅ Number formatting — always use Intl.NumberFormat with commas
Intl.NumberFormat("en-US").format(1234567)   // "1,234,567"

// ✅ Dates — display as "Month YYYY", store as ISO-8601
// ✅ UTC timezone for all date formatters (prevents test failures)
new Intl.DateTimeFormat("en-US", { month: "long", year: "numeric", timeZone: "UTC" })

// ✅ P2 field remap at load boundary (data loaders, never in components)
// employer_friendliness_scores uses "efs" — remap to "srs" in srs.ts
// NaN in P2 JSON → null (browser JSON.parse doesn't handle bare NaN)
```

### 2.7 PostHog Analytics — ALWAYS Update on UI Changes

Every UI change must keep analytics in sync:

```typescript
// All tracking goes through analytics.* — never call posthog.capture() directly
analytics.dashboardViewed("employer");
analytics.filterChanged({ dashboard: "visa-bulletin", filter: "category", value: "EB2" });
analytics.contactSubmitted("Feature Request");

// NEVER include PII: no email, no employer names, no priority dates in events
// Use buckets/tiers/counts instead
```

**When to add analytics:**
| Change | Required action |
|--------|----------------|
| New dashboard page | `analytics.dashboardViewed('name')` in data-load `.finally()` |
| New filter/toggle | `analytics.filterChanged({ dashboard, filter, value })` |
| New page route | Add to `PageName` union type in `analytics/index.ts` |
| New form submission | Add typed event function |
| New entity selection | Add typed event function |

### 2.8 Security Rules

- **Input sanitization** on all user text: `sanitizeTextInput()` from `src/lib/security/index.ts`
- **XSS prevention**: `escapeHtml()` before rendering any user-supplied text
- **LocalStorage access** always via `secureGet/Set/Remove/ClearAll` (with `compass_` prefix)
- **URL validation**: `sanitizeUrl()` blocks `javascript:`, `data:`, `vbscript:` protocols
- **No secrets in client code** — zero API keys, tokens, or credentials in the codebase

### 2.9 Smart Visibility Principle (MANDATORY)

> **Never render a widget whose only possible output is "please provide input".**

```tsx
// ❌ BAD — widget that shows "enter your data above"
<PredictionCard hasPriorityDate={!!pd} />

// ✅ GOOD — hide widget, show CTA, reveal on input
{!pd && (
  <div className="rounded-2xl border border-dashed border-blue-500/[0.15] py-8 text-center">
    <p>Enter your priority date to see predictions</p>
  </div>
)}
{!!pd && <PredictionCard priorityDate={pd} />}
```

### 2.10 Testing Strategy

```bash
npm test              # single run (use before committing)
npm run test:watch    # dev mode
npm run test:coverage # coverage report
```

- **Every** component, utility, and data loader must have tests
- **Framer Motion** must be mocked in component tests (use the standard mock in setup.ts)
- **`next/navigation` + `next/link`** must be mocked
- **localStorage cleared** in `beforeEach` to prevent test state leakage
- **UTC timezone** for all date assertions
- **`happy-dom`** (not jsdom) — lighter, ESM-compatible
- Test file naming: `src/__tests__/[feature].test.tsx` (colocated by feature)

### 2.11 P2 Data Sync Workflow

```bash
# 1. Sync P2 artifacts → public/data/
python3 scripts/sync_p2_data.py

# 2. Run tests (data integrity checks included)
npm test

# 3. Build static export
npm run build

# 4. Deploy
aws s3 sync out/ s3://BUCKET_NAME --delete
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/*"
```

---

## 3. P2 Meridian — Python / Pandas / Analytics

### 3.1 Architecture Philosophy

```
Raw files (P1)
  └── Parser script (scripts/build_fact_X.py)
        └── Parquet artifact (artifacts/tables/X.parquet)
              └── Sync script in P3 (scripts/sync_p2_data.py)
                    └── JSON (public/data/...)
```

- **One script per table** — `scripts/build_fact_X.py` builds `artifacts/tables/X.parquet`
- **Parquet is the canonical format** — never commit raw CSV/XLSX as artifacts
- **Artifacts are versioned** in git; raw source files (downloads/) are gitignored
- **Idempotent scripts** — running a build script twice must produce the same result

### 3.2 Python Code Conventions

```python
# ✅ Type hints on all public functions
def compute_srs(df: pd.DataFrame, *, min_filings: int = 10) -> pd.DataFrame:

# ✅ Docstrings on all non-trivial functions
"""
Compute the Sponsor Reliability Score for each employer.
Returns a DataFrame with columns: employer_id, srs, srs_tier, srs_ml.
"""

# ✅ Constants at module top, UPPER_SNAKE_CASE
MIN_FILINGS_FOR_RATING = 10
SCORE_TIERS = {"Elite": (80, 100), "Strong": (60, 80), "Average": (40, 60)}

# ❌ No bare except — always catch specific exceptions
try: ...
except (ValueError, KeyError) as e: ...
```

### 3.3 Data Quality Rules

```python
# ✅ NaN → null at serialization boundary (P3 JSON.parse doesn't handle bare NaN)
df.to_json(orient="records", default_handler=lambda x: None if pd.isna(x) else x)

# ✅ Dedup with explicit strategy — always document which key wins
df.drop_duplicates(subset=["employer_id", "fy"], keep="last")  # last = latest data

# ✅ Row count assertions in build scripts — catch regressions
assert len(df) > 1000, f"Expected >1000 rows, got {len(df)}"
```

### 3.4 Testing

```bash
CHAT_TAP_DISABLED=1 python3 -m pytest tests/ -q --tb=short
```

- Tests live in `tests/` mirroring `scripts/` structure
- Every build script must have corresponding tests
- Fixtures use real data slices (not synthetic) where possible
- Use `conftest.py` for shared fixtures

### 3.5 Artifact Export Contract to P3

All artifacts exported to P3 must:
1. Use `pd.DataFrame.to_json(orient="records")` — not `json.dump()` (NaN safety)
2. Include `employer_id` as the stable join key (not raw name string)
3. Use field names matching the TypeScript interfaces in `src/types/p2-artifacts.ts`
4. Be documented in `artifacts/metrics/FINAL_SINGLE_REPORT.md`

---

## 4. P1 Horizon — Python / Requests / Data Collection

### 4.1 Architecture Philosophy

- **`sources.yaml`** is the single config file — add/modify sources here, not in code
- **Manifest-based incremental** — never re-download files already in `_manifest.json`
- **15 specialized handlers** — each source has its own parser; don't generalize prematurely
- **Downloads are gitignored** — only code and manifest are version-controlled

### 4.2 Handler Conventions

```python
# ✅ Each handler returns a list of DownloadResult objects
# ✅ Each handler respects the existing manifest (skip already-downloaded)
# ✅ Each handler logs clearly: what was found, what was skipped, what failed
# ✅ Build-in retry with exponential backoff for transient HTTP failures
```

### 4.3 Data Dictionary Maintenance

- `data-dictionary.md` is P2's contract — update it whenever a new source is added or a schema changes
- Run `scripts/update_data_dictionary.py` after adding a new source
- Always commit `data-dictionary.md` alongside the handler change

---

## 5. Aurora Design System — P3 UI

### 5.1 Core Aesthetic

**Linear / Vercel / Raycast-inspired** — dark-first, glassmorphism, precision, restraint.
Every pixel must justify its existence. The UI should feel like it was crafted by Apple's design team.

### 5.2 Color Tokens (use CSS variables, not hardcoded hex)

```css
/* Dark theme defaults */
--background:      #09090b    /* near-black zinc */
--foreground:      #fafafa    /* near-white */
--card:            rgba(255,255,255,0.03)
--accent-blue:     #3b82f6
--accent-purple:   #8b5cf6
--accent-emerald:  #10b981
--accent-amber:    #f59e0b
--accent-rose:     #f43f5e
--gradient-primary: linear-gradient(135deg, #3b82f6, #8b5cf6)
```

### 5.3 Component Patterns

| Pattern | Class string |
|---------|-------------|
| Glassmorphic card | `backdrop-blur-xl bg-white/[0.03] border border-white/[0.08] rounded-2xl` |
| Gradient text | `bg-gradient-to-r from-blue-400 to-purple-400 bg-clip-text text-transparent` |
| Muted label | `text-[var(--muted-foreground)] text-xs font-mono uppercase tracking-widest` |
| Input field | `rounded-xl border border-[var(--border)] bg-[var(--muted)]/30 px-3.5 py-2.5` |
| Action button | `bg-gradient-to-r from-blue-500 to-purple-500 text-white rounded-xl hover:opacity-90` |
| Tab active | `border-b-2 border-[var(--accent-blue)] text-[var(--foreground)]` |

### 5.4 Typography

- **UI text**: Geist Sans (auto-loaded via `next/font`)
- **Data/numbers**: Geist Mono — use `font-mono` class for all numeric data cells
- **Hierarchy**: `text-2xl font-bold` (page title) → `text-lg font-semibold` (section) → `text-sm` (body) → `text-xs` (labels)
- **Gradient text for heroes only** — not for body copy

### 5.5 Chart Theming (Recharts)

```typescript
// Standard chart colors — always use these, not arbitrary tailwind colors
const CHART_COLORS = {
  primary:  "#3b82f6",   // blue
  secondary:"#8b5cf6",   // purple
  success:  "#10b981",   // emerald
  warning:  "#f59e0b",   // amber
  danger:   "#f43f5e",   // rose
  neutral:  "#6b7280",   // gray
};

// Grid lines — always subtle
<CartesianGrid strokeDasharray="3 3" stroke="rgba(255,255,255,0.05)" />

// Axis tick styling — always muted
<XAxis tick={{ fill: "var(--muted-foreground)", fontSize: 11 }} />
```

---

## 6. Environment Variables

### P3 Compass `.env.local`

| Variable | Purpose | Required? |
|----------|---------|-----------|
| `NEXT_PUBLIC_GROQ_API_KEY` | Groq cloud LLM (Llama 3.3 70B) for `/ask` page | Optional — falls back to mock |
| `NEXT_PUBLIC_POSTHOG_KEY` | PostHog product analytics | Optional — silent if absent |
| `NEXT_PUBLIC_POSTHOG_HOST` | PostHog ingest host | Required if PostHog key set |
| `NEXT_PUBLIC_FORMSPREE_ID` | Formspree form ID for Contact Us → email delivery | Optional — falls back to mailto: |

**Rules:**
- Never put `NEXT_PUBLIC_*` vars with secrets in any component rendered on the server — they become public
- Add all new env vars to both `.env.local` (with values) and the table above
- `.env.local` is always gitignored — never commit it
- Provide comments in `.env.local` explaining how to obtain each value

---

## 7. AWS Deployment (P3)

```
S3 bucket (static site)  ← out/ from `next build`
    └── CloudFront CDN   ← edge caching, HTTPS, gzip
          └── Route 53   ← DNS
          └── ACM        ← free SSL cert
```

**Estimated cost: $0.52–3.00/month** (S3 $0.02 + CloudFront free tier + Route53 $0.50)

**No Lambda, no database, no API Gateway, no server** — the entire backend is a static file bucket.

**Deploy commands:**
```bash
npm run build                                           # → out/
aws s3 sync out/ s3://BUCKET_NAME --delete             # deploy
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/*"
```

---

## 8. Testing Pyramid Summary

| Project | Framework | Run command | Green baseline |
|---------|-----------|-------------|----------------|
| P3 Compass | Vitest 4 + RTL + happy-dom | `npm test` | 556 tests, 24 files |
| P2 Meridian | pytest | `CHAT_TAP_DISABLED=1 python3 -m pytest tests/ -q` | See PROGRESS.md |
| P1 Horizon | pytest | `python3 -m pytest` | See README |

**Universal testing rules:**
- Run tests before making any changes (green baseline)
- Run tests after every change before committing
- Never commit with failing tests
- Test count must be accurate in commit messages and PROGRESS.md

---

## 9. Common Pitfalls & Lessons Learned

| Pitfall | Lesson | Fix |
|---------|--------|-----|
| `JSON.parse` fails silently on `NaN` | P2 outputs bare `NaN` tokens; browsers don't parse them | Use `pd.DataFrame.to_json()` not `json.dump()` in P2; NaN→null in P3 loaders |
| Wide-format USCIS files cause PK collision | Dedup `keep="last"` kept the wrong row | Parse the explicit `Total` row; never dedup blindly |
| 2-digit FY filenames (`fy23`) not matched | Regex was 4-digit only | Add `FISCAL_YEAR_SHORT` regex + `_short_year_to_full()` |
| `happy-dom` vs jsdom | jsdom pulls ESM-only deps that break Vite 7 via CJS require | Use `happy-dom` everywhere in P3 tests |
| FOUC on theme init | React hydrates after first paint | Add blocking `<script>` in `<head>` before hydration (theme-provider.tsx pattern) |
| Footer is a server component | `ContactButton` needs state | Use self-contained `ContactButton` client island instead of making the whole footer a client component |
| EFS vs SRS naming | P2 uses `efs`; P3 uses `srs` | Remap at the data loader boundary, never in components |
| LocalStorage keys collide in test | Tests share state across runs | Clear localStorage in `beforeEach` |
| Timeline chart shows invisible lines in dark mode | Chart colors hardcoded to dark hex | Always use CSS var tokens so light mode works |
| Font sizes on mobile charts | Recharts defaults are too large | Set `tick={{ fontSize: 11 }}` explicitly |

---

## 10. Agent Instructions

### When starting any session on this codebase:

1. Read `NORTHSTAR_VISION.md` — understand why the program exists
2. Read `BEST_PRACTICES.md` (this file) — internalize the conventions
3. Read the project-specific `copilot-instructions.md` — get current file inventory and state
4. Read the last 100 lines of `PROGRESS.md` — pick up from the last milestone
5. Run the tests — confirm green baseline before touching code

### When making any change:

- **Implement, don't suggest** — make the change, don't describe it
- **Test first** — never commit failing tests
- **Update docs** — PROGRESS.md + copilot-instructions.md + README in the same session
- **Analytics in sync** — PostHog event for every new user interaction
- **Commit and push** — never leave the repo in a dirty state

### When in doubt:

- Prefer the established pattern over a new one
- Check copilot-instructions.md for existing conventions before inventing a new approach
- The codebase consistency matters more than individual elegance
