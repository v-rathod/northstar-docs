# NorthStar Program Guardrails

> **The Ten Commandments of NorthStar, plus project-specific guardrails.**
> These are non-negotiable architectural constraints derived from the program's vision, cost model, and hard-won lessons.
> Deviate only with explicit user instruction, and document the deviation in ARCHITECTURE_DECISIONS.md.

---

## The Ten Commandments (Program-Wide, Non-Negotiable)

### I. Thou Shalt Not Add a Backend

No Lambda, no database, no API Gateway, no server-side runtime. The entire platform runs on pre-computed data served as static files. Compass (P3) is a static site. Meridian (P2) runs locally. Horizon (P1) runs locally. Nothing runs in the cloud except S3 + CloudFront serving files.

**Why**: This is the foundational architecture decision. It keeps AWS costs under $5/month and eliminates an entire class of security, scaling, and operational concerns.

### II. Thou Shalt Keep AWS Costs Under $5/Month

S3 static hosting + CloudFront CDN + Route 53 DNS + ACM SSL. That is the entire infrastructure. No paid APIs, no managed databases, no compute services. Free-tier services only.

**Why**: This project proves that a production-quality analytics platform can run for less than a cup of coffee per month. Every architectural decision flows from this constraint.

### III. Data Flows Down, Never Up

```
P1 Horizon → P2 Meridian → P3 Compass → User's Browser
```

Each project reads from the one upstream. No project writes back. P2 reads P1's `downloads/` directory (never copies). P3 reads P2's `artifacts/` directory (via sync script). The browser reads P3's static JSON. The reverse never happens.

**Why**: Unidirectional data flow eliminates circular dependencies, simplifies debugging, and guarantees reproducibility.

### IV. Thou Shalt Pre-Compute Everything

All ML models, forecasts, aggregations, statistics, scores, and metrics are computed in P2 Meridian at build time. P3 Compass only reads and renders. The browser never runs calculations that take more than 100ms.

**Why**: Pre-computation means zero latency, zero server costs, and deterministic output. The same P2 artifacts always produce the same P3 website.

### V. Thou Shalt Separate Concerns

P1 owns data collection. P2 owns analytics and modeling. P3 owns user experience. No project does another project's job.

- P1 never computes aggregations (that's P2's job)
- P2 never serves data to users (that's P3's job)
- P3 never fetches raw government data (that's P1's job)
- P3 never trains models or runs analytics (that's P2's job)

**Why**: Separation of concerns keeps each codebase focused, testable, and independently deployable. A P1 change doesn't require P3 changes unless the data contract changes.

### VI. Thou Shalt Test Before Committing

Every project has tests. Tests must pass before every commit. No exceptions.

- **P1**: Manifest integrity + handler tests
- **P2**: `pytest` (562+ tests): schema validation, golden snapshots, data sanity
- **P3**: `npm test` (1,206+ tests): unit, component, integration, security, real-data

A commit with failing tests is a broken contract. Never use `--no-verify` or skip tests to rush a deploy.

**Why**: Tests are the only reliable way to catch regressions across human and agent sessions. They are the institutional memory of the codebase.

### VII. Thou Shalt Add Regression Tests for Every Bug Fix

When you fix broken functionality, you MUST add permanent regression tests that cover the exact scenario that was broken, plus edge cases and related variations. Fix + tests + documentation are committed together in a single unit.

A bug without a regression test will recur. It is not a matter of if, but when.

**Why**: Agents don't remember. Humans forget. Tests don't. A regression test is the only thing that stands between a fixed bug and its silent return.

### VIII. Thou Shalt Not Break the Data Contract

P1→P2 and P2→P3 have explicit data contracts. Column names, primary keys, field types, and file formats are sacred. Changing them requires coordinated updates across all downstream projects.

- P1→P2 contract: `data-dictionary.md` in P1
- P2→P3 contract: `src/types/p2-artifacts.ts` in P3 + `configs/schemas.yml` in P2
- P3→Browser contract: `public/data/` JSON structure

Before renaming a column, removing a field, or changing a file format: check all downstream consumers, update their types/loaders/tests, and commit everything together.

**Why**: A silent contract break (renamed column, changed key type) produces zero errors at build time but broken dashboards at runtime. It is the hardest class of bug to diagnose.

### IX. Thou Shalt Keep Compass Branding Correct

The web application is called **Compass**. "NorthStar" is the internal program name and must never appear in the user-facing UI. Use P1/P2/P3 in code. Use Horizon/Meridian/Compass in documentation. The Employer Friendliness Score is "Sponsor Reliability Score" / "SRS" in P3.

Additionally, never use em-dashes (`—`) in UI text. Never use AI marketing filler words (unlock, discover, journey, empower, leverage, seamless, comprehensive, cutting-edge, revolutionize, delve, holistic, supercharge).

**Why**: Consistent branding builds user trust. Marketing language erodes credibility with a technical audience.

### X. Thou Shalt Document Every Session

After every meaningful piece of work:

1. Update `PROGRESS.md` with a timestamped milestone entry
2. Update the relevant project's context file (copilot-instructions, NEXT_AGENT_CONTEXT, etc.)
3. Commit and push. Never leave the repo in a dirty state.

The repo is the single source of truth. If it isn't committed, it didn't happen.

**Why**: AI agents are stateless. Humans switch context. The documentation chain is the only thread of continuity between sessions.

---

## Program-Level Guardrails (All Projects)

### Architecture Guardrails

| # | Guardrail | Scope | Enforcement |
|---|-----------|-------|-------------|
| A1 | Static export only for P3 (`output: 'export'`) | P3 | `next.config.ts` |
| A2 | No runtime server computation anywhere | P3 | Architecture review |
| A3 | All data in Parquet (P2 internal) or JSON (P3 serving) | P2, P3 | File format conventions |
| A4 | Deterministic builds: same input → same output | P2, P3 | No randomness, no external API calls during build |
| A5 | Incremental by design (manifest-based in P1, change-detection in P2, S3 sync in P3) | All | Build scripts |
| A6 | Privacy first: zero cookies, zero tracking pixels, zero user accounts | P3 | Code review |
| A7 | All personalization in localStorage, never leaves device | P3 | Security review |

### Data Quality Guardrails

| # | Guardrail | Scope | Enforcement |
|---|-----------|-------|-------------|
| D1 | NaN → null at every serialization boundary (P2 JSON export, P3 loader) | P2, P3 | `to_json()` handlers, loader transforms |
| D2 | Primary keys must be unique in every table | P2 | pytest PK assertions |
| D3 | Referential integrity ≥95% across dimension/fact joins | P2 | pytest RI assertions |
| D4 | Row count assertions in build scripts to catch regressions | P2 | `assert len(df) > N` |
| D5 | Data provenance: every number traces to a government source file | P2 | Column lineage in schemas.yml |
| D6 | Employer IDs use SHA1 hash of normalized name (not raw strings) | P2, P3 | `dim_employer.employer_id` |

### Testing Guardrails

| # | Guardrail | Scope | Enforcement |
|---|-----------|-------|-------------|
| T1 | All tests pass before every commit (zero failures) | All | `npm test` / `pytest` pre-commit |
| T2 | Bug fix = regression test (committed together in atomic unit) | All | Code review |
| T3 | Test real data when possible (not synthetic mocks) | P2, P3 | `describe.skipIf(!DATA_AVAILABLE)` pattern |
| T4 | UTC timezone for all date assertions | P3 | Vitest `timeZone: "UTC"` |
| T5 | Mock Framer Motion and next/navigation in component tests | P3 | `setup.ts` standard mock |
| T6 | Clear localStorage in beforeEach to prevent state leakage | P3 | Test hygiene |
| T7 | Coverage ≥95% per dataset in P2 | P2 | Coverage gates |
| T8 | Post-deploy smoke tests (238+ checks) must pass | P3 | `scripts/smoke-test.mjs` |

### Security Guardrails

| # | Guardrail | Scope | Enforcement |
|---|-----------|-------|-------------|
| S1 | No API keys, tokens, or credentials in source code | All | Git pre-commit, code review |
| S2 | `sanitizeTextInput()` on all user-provided text | P3 | `src/lib/security/` |
| S3 | `escapeHtml()` before rendering user-supplied text | P3 | XSS prevention |
| S4 | `secureGet/Set/Remove/ClearAll()` for all localStorage access | P3 | `compass_` prefix namespace |
| S5 | URL validation blocks `javascript:`, `data:`, `vbscript:` | P3 | `sanitizeUrl()` |
| S6 | `.env.local` always gitignored, never committed | All | `.gitignore` |
| S7 | Only permitted env vars: `NEXT_PUBLIC_POSTHOG_*`, `NEXT_PUBLIC_GROQ_API_KEY`, `NEXT_PUBLIC_FORMSPREE_ID` | P3 | Code review |

### Git & Workflow Guardrails

| # | Guardrail | Scope | Enforcement |
|---|-----------|-------|-------------|
| G1 | Commit messages follow `<type>(<scope>): <summary>` format | All | Convention |
| G2 | Every commit pushed to `origin/main` (no orphaned local commits) | All | Workflow habit |
| G3 | `PROGRESS.md` updated with every meaningful change | All | Session discipline |
| G4 | Deploy only via `bash scripts/deploy.sh` (never raw `aws s3 sync`) | P3 | Deploy script |
| G5 | No destructive operations without user confirmation (force push, drop table, rm -rf) | All | Agent safety |
| G6 | Cross-project changes require coordinated commits | All | Data contract awareness |

### UI & Design Guardrails (P3 Only)

| # | Guardrail | Scope | Enforcement |
|---|-----------|-------|-------------|
| U1 | Tailwind only: no CSS modules, no styled-components | P3 | ESLint, code review |
| U2 | Dark-first design with CSS variable tokens (not hardcoded hex) | P3 | `globals.css` tokens |
| U3 | Framer Motion with standard easing `[0.25, 0.1, 0.25, 1]` | P3 | Convention |
| U4 | Smart Visibility: never render empty "please provide input" widgets | P3 | Component review |
| U5 | Number formatting via `Intl.NumberFormat` with commas | P3 | Convention |
| U6 | Dates displayed as "Month YYYY", stored as ISO-8601 | P3 | Convention |
| U7 | `font-mono` for all numeric data cells | P3 | Aurora design system |
| U8 | Server components by default; `"use client"` only for interactivity | P3 | Next.js architecture |

---

## Changing a Commandment

The Ten Commandments are not absolute. They can change when the program makes a deliberate architectural shift. But the bar is high:

1. **Propose** the change in conversation with the user
2. **Document** the rationale in the relevant `ARCHITECTURE_DECISIONS.md`
3. **Update** this file and all downstream references
4. **Update** the parent `NORTHSTAR_VISION.md` if the change affects the program vision
5. **Commit** the change with a clear `docs: update commandment X — [reason]` message

Commandments should never be silently broken. If you find yourself working around one, stop and discuss.

---

## Cross-References

- **Parent vision & architecture**: `/Users/vrathod1/dev/NorthStar/northstar-docs/NORTHSTAR_VISION.md`
- **Engineering best practices**: `/Users/vrathod1/dev/NorthStar/northstar-docs/BEST_PRACTICES.md`
- **P1 project-specific guardrails**: `/Users/vrathod1/dev/NorthStar/fetch-immigration-data/.github/GUARDRAILS.md`
- **P2 project-specific guardrails**: `/Users/vrathod1/dev/NorthStar/immigration-model-builder/.github/GUARDRAILS.md`
- **P3 project-specific guardrails**: `/Users/vrathod1/dev/NorthStar/immigration-insights-app/.github/GUARDRAILS.md`
