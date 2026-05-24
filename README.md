# AIG!itch Meta

> Coordination layer for the AIG!itch ecosystem. Nothing here runs in production — this repo's job is to keep the three production repos aligned.

## The three production repos this coordinates

| Repo | Purpose | URL |
|---|---|---|
| **aiglitch** | Frontend — pure UI consumer | https://github.com/comfybear71/aiglitch |
| **aiglitch-api** | Backend — all business logic, crons, endpoints | https://github.com/comfybear71/aiglitch-api |
| **admin-aiglitch** | Admin UI — separate consumer of the backend | https://github.com/comfybear71/admin-aiglitch |

Production domains:
- `aiglitch.app` → public frontend (consumes `api.aiglitch.app`)
- `api.aiglitch.app` → backend API
- `admin.aiglitch.app` → admin UI (consumes `api.aiglitch.app`)

## What lives here

| Path | Purpose |
|---|---|
| `MIGRATION-PLAYBOOK.md` | **Canonical playbook.** Both Claudes (Opus + Haiku) must read this at session start. |
| `docs/architecture.md` | Current state of the 3-tier split + target state |
| `docs/adr/` | Architecture Decision Records — why we made each significant choice |
| `.github/ISSUE_TEMPLATE/` | Cross-repo issue templates *(coming in Phase C setup)* |
| `scripts/` | Cross-repo verification + automation scripts *(future)* |

## The GitHub Project board

The cross-repo project board lives in this repo and aggregates issues + pull requests from all three production repos.

*(URL added once the board is created in Phase C of the meta repo setup.)*

## For Claudes / contributors

Start every session by reading:
1. This repo's `MIGRATION-PLAYBOOK.md`
2. Your repo's `CLAUDE.md` (per-repo project instructions)
3. Your repo's `HANDOFF.md` (session log)
4. The Project board (above link) for current state

Don't write any code until you've stated explicitly that you've done all four reads.

## Owner

Stuart French (comfybear71) — solo developer. Drives all merges and releases via GitHub web UI.
