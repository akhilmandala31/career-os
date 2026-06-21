# Engineering Note: Sonar Portfolio Refresh — From Manual Fix to Platform Primitive

**Date:** June 2026
**Area:** Platform Engineering / AI Skills / GitHub Actions
**GAP Context:** PT — Product Foundation, Sonar portfolio for 40+ app repos

---

## What Problem Was I Actually Solving?

Monday morning: team lead opens the Sonar portfolio report. A few apps are missing. No alert. No error. Just gone.

SonarQube silently purges project analysis data after 90 days of inactivity. Dormant repos — integration adapters, sync tools, apps that don't change for months — hit this threshold without anyone noticing. The portfolio recomputation utility (Issue #22 in sonar-utilities) can't fix it, because there's no analysis data left to recompute.

The real failure isn't the 90-day limit. It's that the system gives no warning, and the fix requires pushing fresh analysis data before the purge, not after.

---

## What I Built and Why Each Layer Exists

This is a platform engineering problem, not a DevOps task. The solution has three layers because the failure has three distinct scopes.

### Layer 1: `sonar-refresh.yml` — Trigger per repo

A lightweight, analysis-only GHA workflow. No build, no test, no deploy. Just:
1. Checkout
2. Mint credentials (Artifactory OIDC + Sonar OIDC — both via mint composite actions)
3. `./gradlew sonar`

Why not reuse the existing `build-and-deploy-to-dev.yaml`?

Because that workflow is **coupled**: build → test → Docker → deploy all run as one pipeline. Triggering it on a dormant repo causes an unintended deployment to dev. The blast radius is wrong for a data hygiene concern. The existing workflow has the right behavior for CI — the wrong behavior when the only goal is "keep the Sonar data alive."

This is a core platform engineering principle: **don't extend a workflow beyond its intended scope**. Write a new primitive with a single responsibility.

### Layer 2: `reusable-sonar-analysis.yml` — Orchestration layer

Hosted in `product/reusable-workflows` (already used for `sync-main-into-feature-branch`). This is what makes the solution reusable across teams.

Logic at this layer:
- Check last analysis date via Sonar API
- If > 75 days (15 days before 90-day purge), trigger refresh
- After analysis completes, post Issue #22 comment in sonar-utilities with the portfolio key to trigger recomputation

Why 75 days and not 90? **Defence in depth.** GHA workflows can fail on transient errors. 15 days of buffer means a retry window before the actual purge.

Why Issue #22 instead of calling Sonar API directly?
- `recompute_portfolio.sh` in sonar-utilities is the org-wide approved primitive for this operation
- Calling it via issue comment keeps the audit trail in the right place (sonar-utilities, not scattered across team repos)
- When/if Sonar MCP is approved (PSEC-9238), replace with direct API call — but don't wait for it

### Layer 3: `/sonar-portfolio-refresh` skill — Cross-team adoption

The reusable workflow exists but not every team will add `sonar-refresh.yml` to their repos. Adoption friction is the most common way platform tools fail.

The skill has two modes:
- **Workflow exists:** trigger it via GitHub MCP
- **Workflow missing:** create a PR to add it, with the correct configuration for that repo

This is the "make the right thing the easy thing" principle. The team doesn't have to think about the architecture — they review a PR that was already written correctly.

---

## The Portfolio Key Insight

Biggest design decision: what identifies a "team's apps" across repos?

The git org is too broad — PT-PF has 100 repos but only 40 are apps. Individual project keys are too narrow — you'd need to maintain a list. The answer is already in `sonar-utilities/Portfolios/`:

```yaml
portfolios:
  - name: PT - Product Foundation
    regExp: com.gap.product.productfoundation:.*
    portfolioKey: PT___Product_Foundation
```

The `regExp` field is a project key pattern — it defines exactly which Sonar projects belong to a team. The `portfolioKey` is what Issue #22 needs. The skill reads this YAML to get both without hardcoding anything.

**Architecture lesson:** before building new configuration, look for the source of truth that already exists. sonar-utilities YAML was already org-wide authoritative for this mapping.

---

## What Breaks at 10x Scale

- **100+ teams using the skill:** the Issue #22 comment mechanism in sonar-utilities becomes a bottleneck — all teams append comments to a single issue. At scale, the issue becomes unreadable and the recomputation script needs to be triggered by multiple simultaneous comments. Proper fix: sonar-utilities exposes a REST endpoint or a dedicated per-team issue.
- **1000+ repos:** the skill's discovery phase (find repos with > 75-day analysis gaps) needs pagination and batching. GitHub MCP search API has rate limits; naive parallelism hits them.
- **Analysis failures on dead repos:** if a repo's `./gradlew sonar` was never properly configured, the refresh workflow will fail silently. The skill needs a failure-report mode, not just a trigger mode.

---

## Platform Engineering Vocabulary Mapping

| What was built | Platform Engineering name |
|---|---|
| `sonar-refresh.yml` | Paved road workflow — correct default behavior encoded once |
| `reusable-sonar-analysis.yml` | Platform primitive — reusable across all teams |
| `/sonar-portfolio-refresh` skill | IDP tool — reduces adoption friction to a PR review |
| Issue #22 recomputation | Existing platform API — reuse before building new |
| Portfolio YAML as source of truth | Single source of truth — config, not code |

---

## What I'd Say in an Interview

**"Tell me about a platform automation you've built."**

> "I identified a silent data integrity problem in our Sonar portfolio reporting — dormant repos were being purged after 90 days with no alert, corrupting the Monday team report. Before designing a fix, I tested the existing recomputation utility to confirm it couldn't restore purged data — so the only real solution was proactive prevention. I designed a three-layer system: a minimal analysis-only workflow per repo, a reusable orchestration layer shared across 40+ apps, and an AI skill that handles adoption friction by creating the PR for teams that haven't onboarded. The key architectural insight was using the sonar-utilities portfolio YAML as the source of truth for the team-to-project-key mapping — that already existed org-wide, so we didn't need new configuration. Implementation is gated on two OIDC token migrations completing first; I made that call explicitly to avoid building on a foundation we knew was changing."

That answer hits: problem identification, hypothesis testing before designing, platform thinking (not just fixing the symptom), adoption friction as a first-class concern, build vs. reuse decision, timing judgment.

---

## Three Interview Questions This Note Answers

1. **"How do you approach automation that needs to be adopted by teams you don't control?"** — Make the right thing the easy thing: generate the PR, don't send a doc.
2. **"What's the difference between a DevOps fix and a platform primitive?"** — Scope and reusability. A DevOps fix solves your team's problem. A platform primitive encodes the correct behavior once and reduces every team's cognitive load.
3. **"How do you decide when to build vs. when to wait?"** — When the foundation is actively changing (OIDC migration), build on the new foundation once, not twice. The cost of doing it wrong is rebuilding, not just delaying.

---

## Revision Needed

No
