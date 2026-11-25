# ADR-017 · Cost plan: free tiers while building, Supabase Pro at go-live

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-17 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Service plans and spending controls |
| **Risk tier** | Medium (cost overruns and production availability) |

## Context

The stack (ADR-003, ADR-007, ADR-013) is chosen to cost little. We need a plan for what is free while building, what is paid at go-live, and how to avoid surprise bills. Figures below come from the decision log and must be re-verified against current vendor pricing before go-live.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review. A and B are close; the difference is cost during the build phase.

| Criterion (weight) | A. Free tiers while building; Supabase Pro at go-live | B. Paid plans from day one | C. Stay on free tiers after go-live |
|---|---|---|---|
| Security / privacy gate | Pass | Pass | Pass |
| Security (20%) | 4 | 4 | 3 |
| Maintainability (20%) | 4 | 4 | 4 |
| Reliability (15%) | 4 | 5 | 1 |
| Performance (15%) | 4 | 4 | 3 |
| Scalability (10%) | 4 | 4 | 2 |
| Cost (10%) | 5 | 3 | 5 |
| Integration (10%) | 4 | 4 | 4 |
| **Weighted score** | **4.10** | **4.05** | **3.10** |

## Decision

**Option A.**

| Service | While building | At go-live |
|---|---|---|
| Hostinger | Existing plan | Existing plan |
| Cloud Run | Free tier | Free tier / a few dollars |
| Supabase | Free | **Pro (about $25/month)**: more egress, projects not paused |
| Cloudflare R2 + Images | Free (10 GB, no egress fees, 5,000 transformations) | Free tier; then $0.015/GB-month and $0.50 per 1,000 transformations |
| GitHub + Actions | Free | Free |

Google Cloud budget alerts are set up from day one.

## Why

- Near-zero cost while the product is being built; paid plan only when real users depend on it.
- Free Supabase projects can be paused when inactive, which is unacceptable for production.

## Rejected

- **B. Paid from day one:** better reliability earlier, but pays for capacity before it is needed.
- **C. Free tiers in production:** pausing, limits and no safety margin for a live product.

## Consequences

- Upgrade Supabase to Pro before the first real client uses the web app.
- Budget alerts on Google Cloud (and usage alerts on Cloudflare and Supabase) from day one.
- Keep image widths few and cache well to stay within transformation limits (ADR-013).
- Claude Code usage has its own cost; check `/usage` and set limits for unattended runs (ADR-016).

## Related decisions

ADR-003, ADR-007, ADR-013, ADR-016.

## Open points

- The monthly budget ceiling, set by the business owner.
- Re-verify all prices and free-tier limits at go-live; confirm what backups and point-in-time recovery the chosen Supabase plan includes.
- Define what triggers a plan change (for example, egress or database size thresholds).
