# ADR 0001 — Three-tier repo split (frontend / backend / admin)

**Status:** Accepted (2026-05-23) — migration in progress
**Decider:** Stuart French (comfybear71)
**Supersedes:** N/A
**Superseded by:** N/A

## Context

AIG!itch began as a single Next.js monolith (`aiglitch`) serving both the public frontend and a 170+ route admin/API surface. By early 2026, this had grown to:

- Large `vercel.json` cron schedule (~22 crons)
- ~85 admin routes mixed in with consumer routes
- Solana / trading code coupled with persona content code
- Vercel cold-start latency hurting user-facing pages because the same Lambda also loaded heavy backend deps
- Single-Claude sessions struggled with the codebase size — context loss, fix spirals, repeated cost

A second repo `aiglitch-api` was started to host the new headless backend, with a strangler-pattern rewrite (`beforeFiles` in the sister repo's `next.config.ts`) flipping traffic per-endpoint.

By May 2026 the strangler was largely complete for non-admin routes. The remaining question: **what does the final state look like?**

## Decision

Split into **three production repos** plus **one coordination repo**:

| Repo | Concern |
|---|---|
| `aiglitch` (existing) | Public UI shell only after migration. Permanent home for Instagram proxy endpoints (cannot move per CLAUDE.md migration rule #5). |
| `aiglitch-api` (existing) | All business logic, crons, platform endpoints, database schema owner. |
| `admin-aiglitch` (new) | Admin UI shell. Served at `admin.aiglitch.app`. Consumes `aiglitch-api` for all data + actions. |
| `aiglitch-meta` (new) | Coordination layer — playbook, ADRs, cross-repo project board, automation scripts. No production traffic. |

## Consequences

### Positive

- **Smaller, faster repos.** Frontend cold start improves once heavy backend deps are out. Admin can deploy independently without rebuilding the public site.
- **Clearer Claude sessions.** Smaller context windows mean less drift. Each repo's CLAUDE.md and code is focused on one concern.
- **Independent deploy cadence.** Admin can ship a UI change without redeploying public traffic. Backend can deploy without touching frontend.
- **Safer auth boundaries.** Admin auth fully isolated at the `admin.aiglitch.app` domain — separate cookie scope, separate failure mode, separate CORS.
- **Easier mobile / future-consumer addition.** Glitch-app iOS becomes just another consumer of `aiglitch-api`. No coupling to web frontend code.

### Negative

- **More repos to maintain.** Four total (three production + one meta). More PRs to coordinate, more `package.json` files to keep in sync.
- **Cross-repo handoffs add overhead.** Mitigated by the project board + structured handoff format in `MIGRATION-PLAYBOOK.md`.
- **Initial extraction cost.** Moving admin pages from `aiglitch/src/app/admin/*` to `admin-aiglitch` is non-trivial — auth cookie sharing, env var migration, DNS coordination.

### Neutral

- **Shared database stays shared** during migration; `aiglitch-api` becomes sole owner only after Phase 10 cleanup.
- **Permanent Instagram proxy exception** documented and accepted — `aiglitch` repo retains `/api/image-proxy` and `/api/video-proxy` indefinitely.

## Implementation plan

See `docs/architecture.md` for the current vs target state. The migration phases (3, 4, 5, 6, 7, 8, 9, 10) are tracked on the GitHub Project board in this repo.

## Open questions

- Will the `aiglitch` repo eventually be renamed to clarify its reduced role (e.g., `aiglitch-web`)? **Decision deferred to Phase 10.**
- Does `admin-aiglitch` need its own database read-replica for heavy queries? **Probably no** — `aiglitch-api` proxies all DB access. Re-evaluate if admin load grows.
