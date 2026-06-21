# ADR-001: PT-PF Reusable Workflows Foundation + Sonar Portfolio Health Automation

**Date:** June 2026
**Status:** Proposal — Awaiting Lead Review
**Author:** Akhil Mandala

---

## Problem

### Immediate: Sonar 90-Day Purge

SonarQube purges project analysis data after 90 days of inactivity. When this happens, the project silently disappears from portfolio metrics. The Monday morning portfolio report then shows an incorrect picture — missing apps reduce coverage %, quality gate totals, and reliability ratings without any alert.

**Validated findings:**
- Issue #22 in sonar-utilities (`recompute_portfolio.sh`) cannot restore purged projects — it recomputes from existing analysis data. Once purged, there is nothing to recompute.
- Portfolio recomputation only accepts portfolio keys (e.g. `PT___Product_Foundation`), not project keys.
- Dormant repos exist in the portfolio (integration adapters, sync tools) with legitimate gaps of 60–90+ days between PRs.

### Underlying: No Reusable Workflow Foundation for PT-PF

PT-PF has 40+ app repos, each maintaining its own `build-and-deploy-to-dev.yaml`. There is no shared workflow layer — every repo duplicates the same pipeline logic. Changes (token migrations, new tooling, policy updates) must be made repo-by-repo with no single source of truth.

---

## Why Now

Two migrations are already in flight across PT-PF repos:

| Migration | Skill |
|---|---|
| Artifactory → OIDC `mint-jfrog-token` | `/artifactory-token-refactor` (exists in skill-depot) |
| Sonar → OIDC `mint-sonar-token` | `/sonar-token-refactor` (exists in skill-depot) |

Both migrations require touching `.github/workflows/` in every repo. This is the right moment to establish the reusable workflow foundation for PT-PF — not as a separate initiative, but as the correct way to do the migration once.

---

## Options Considered

### Option A: Fix sonar purge only — add `sonar-refresh.yml` per repo
- Minimal change. Dedicated analysis-only workflow per repo.
- **Why rejected:** Solves the immediate problem but misses the bigger opportunity. Still requires touching every repo. Does nothing to address the underlying duplication problem.

### Option B: Extend SonarQube retention policy org-wide
- Ask SonarQube admin team to extend the global purge threshold.
- **Why rejected:** Admin team owns retention policy across all teams — org-wide cost for one team's benefit. Doesn't fix the duplication problem either.

### Option C: Add `sonar-only` flag to existing `build-and-deploy-to-dev.yaml`
- Optional input that skips build/test/deploy steps.
- **Why rejected:** Pollutes the deploy pipeline with health-check logic. Conditional steps inside a job named `test-quality-checks` that silently skips its own tests is confusing. Also doesn't solve the underlying duplication.

### Option D (Proposed): New PT-PF reusable workflows repo as the foundation
- Build `reusable-deploy.yml` — the full configurable pipeline with a `sonar-only` input.
- Sonar refresh becomes a mode of the deploy workflow, not a separate tool.
- Token migration PRs migrate each repo to call the reusable workflow — one change removes duplication and solves the sonar purge simultaneously.
- Build `/sonar-portfolio-refresh` skill in skill-depot — generic, usable by any team.

---

## Decision

**Option D** — new PT-PF reusable workflows repo as the platform foundation.

**Key insights:**
1. Don't build a narrow sonar-only workflow and rebuild later. Build the reusable deploy workflow once, correctly, with `sonar-only` as one of its modes.
2. The token migrations are a forcing function — every repo gets touched anyway. Use that moment to migrate to the reusable workflow pattern, not just update tokens.
3. The portfolio key in `sonar-utilities/Portfolios/` is the correct team boundary for the skill — the regExp defines exactly which project keys belong to PT-PF.
4. The skill should be generic — any of the 50+ product teams can use it via skill-depot without modification.

---

## Architecture

### Reusable Workflows Repo (new — PT-PF)

**`reusable-deploy.yml`** — full configurable pipeline:

```
inputs:
  sonar-only: boolean (default: false)

jobs:
  set-short-sha         → always runs
  test-quality-checks   → always runs (sonar-only stops here)
  image-build-scan-push → if: !sonar-only
  tag-repo              → if: !sonar-only
  deploy-to-dev         → if: !sonar-only
  update-chartis-file   → if: !sonar-only
```

**Why `update-chartis-file` is gated:** Chartis is the deployment manifest (stored in `product/product-chartis-deploy`). The `dockerImageTag` field is updated after every build with the new git SHA. In `sonar-only` mode, no image is built — writing a stale tag to the chartis file would cause an unintended deploy. The gate is essential.

**`reusable-sonar-analysis.yml`** — calls `reusable-deploy.yml` with `sonar-only: true`. This is the trigger used by the `/sonar-portfolio-refresh` skill.

### Per-Repo (each app during token migration)

Each repo's `build-and-deploy-to-dev.yaml` is replaced with a thin caller:

```yaml
jobs:
  deploy:
    uses: product/pt-pf-reusable-workflows/.github/workflows/reusable-deploy.yml@main
    with:
      sonar-only: false
```

### Skill (skill-depot — generic, org-wide)

```
/sonar-portfolio-refresh <org> <team>

1. Read sonar-utilities/Portfolios/<team>.yaml
   → get portfolioKey + regExp
2. For each repo matching regExp:
   → sonar-refresh workflow exists? trigger it
   → doesn't exist? create PR to add it
3. Post Issue #22 comment with portfolioKey
   → trigger portfolio recomputation
```

### Full Flow

```
sonar-utilities/Portfolios/<team>.yaml
  ├── portfolioKey: PT___Product_Foundation
  └── regExp: com.gap.product.productfoundation:.*
                        │
                        ▼
        /sonar-portfolio-refresh skill
        (skill-depot — generic, any team)
        (uses GitHub MCP — approved org-wide)
                        │
          ┌─────────────┴──────────────────┐
          │                                │
   reusable-sonar-analysis.yml     Create PR to add
   triggered → calls                reusable-deploy.yml
   reusable-deploy.yml              (sonar-only: true)
   (sonar-only: true)
          │
          ▼
   Issue #22 comment → portfolio recomputation
   (sonar-utilities repo)
```

---

## Rollout Plan

| Phase | What | Dependency |
|---|---|---|
| Phase 0 | Create new PT-PF reusable workflows repo. Build `reusable-deploy.yml` with `sonar-only` input and `reusable-sonar-analysis.yml` | Lead approval on repo name/ownership |
| Phase 1 | Token migration rollout — each repo PR replaces `build-and-deploy-to-dev.yaml` with thin caller to `reusable-deploy.yml` | `mint-sonar-token` action available |
| Phase 2 | Build `/sonar-portfolio-refresh` skill in skill-depot | Phase 1 stable on at least one repo |
| Phase 3 | Full PT-PF coverage as migration completes across all 40+ repos | Phase 1 complete |

---

## Tradeoffs Accepted

| Tradeoff | Why Accepted |
|---|---|
| Broader scope than a sonar-only fix | The token migration touches every repo anyway — the incremental cost of building the reusable workflow is low; the long-term benefit is high |
| New repo required | PT-PF has not adopted reusable workflows yet — this is the right moment to establish the pattern |
| Implementation gated on both OIDC migrations | Build once correctly on the new foundation, not twice |
| Incremental coverage | Portfolio protection grows repo-by-repo as migration progresses |
| Portfolio recomputation via Issue #22 comment | Interim — Issue #22 is the only available mechanism today |

---

## What This Is Not

- Not a request to change SonarQube's retention policy
- Not a new service or new infrastructure — built on existing GHA, GitHub MCP, and sonar-utilities
- Not a breaking change to any existing workflow — every change goes through normal PR review

---

## Decisions Required from Leads

- [ ] **New PT-PF reusable workflows repo** — approve creation, confirm naming convention and ownership
- [ ] **Token migration sequencing** — confirm PT-PF drives both `/artifactory-token-refactor` and `/sonar-token-refactor` across all 40+ repos
- [ ] **skill-depot contribution** — confirm PT-PF can contribute `/sonar-portfolio-refresh` as a generic org-wide skill

---

## What Would Change This Decision

1. **SonarQube admin team adds a REST endpoint for portfolio recomputation** — replace Issue #22 comment with direct API call.
2. **Sonar retention policy extended to 180+ days org-wide** — sonar purge problem disappears; reusable workflow foundation still valuable.
3. **Sonar MCP approved (PSEC-9238 resolved)** — skill can query project analysis ages directly from Sonar.

---

## Interview Framing

> "I identified a silent data integrity problem in our Sonar portfolio reporting — dormant repos were being purged after 90 days with no alert. Rather than building a narrow fix, I used the problem as the forcing function to propose PT-PF's reusable workflow foundation. Two token migrations were already touching every repo — I proposed building a configurable reusable deploy workflow once, correctly, so the sonar refresh becomes a mode of the pipeline rather than a bolt-on tool. The scope expanded from fixing a symptom to eliminating the underlying duplication across 40+ repos."
