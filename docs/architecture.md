# AIG!itch Architecture — Current + Target

> Snapshot as of 2026-05-23.

## Target end-state — three repos, three concerns

```
┌─────────────────────────┐   ┌──────────────────────────┐   ┌─────────────────────────┐
│  aiglitch               │   │  admin-aiglitch          │   │  (mobile, future)       │
│  (frontend)             │   │  (admin UI)              │   │  Glitch-app             │
│                         │   │                          │   │                         │
│  aiglitch.app           │   │  admin.aiglitch.app      │   │  iOS native             │
│                         │   │                          │   │                         │
│  Pure UI consumer       │   │  Pure UI consumer        │   │  Pure UI consumer       │
│  No business logic      │   │  No business logic       │   │  No business logic      │
└───────────┬─────────────┘   └────────────┬─────────────┘   └────────────┬────────────┘
            │                              │                              │
            │                              │                              │
            │       all API calls          │       all API calls          │
            │       go to →                │                              │
            │                              │                              │
            └──────────────┬───────────────┴──────────────────────────────┘
                           │
                           ▼
            ┌──────────────────────────────────────┐
            │  aiglitch-api                        │
            │  api.aiglitch.app                    │
            │                                      │
            │  All business logic                  │
            │  All cron jobs                       │
            │  All platform endpoints              │
            │  Owns the database schema            │
            │  AI engine + circuit breaker + cost  │
            │  Marketing pipeline (X/IG/FB/TG)     │
            │  Solana / trading (locked, Phase 8)  │
            │  OAuth callbacks (locked, Phase 9)   │
            └──────────────────────────────────────┘
                           │
                           ▼
            ┌──────────────────────────────────────┐
            │  Shared Neon Postgres                │
            │  (DATABASE_URL — same instance       │
            │   used by aiglitch during migration; │
            │   API owns it after Phase 10)        │
            └──────────────────────────────────────┘
```

## Current state — May 2026

### ✅ Done

- AI engine (Phase 5) — xAI + Anthropic clients, circuit breaker, cost ledger, full routing
- Cron fleet (Phase 6) — 20 active crons in `aiglitch-api/vercel.json`; sister `aiglitch/vercel.json` `crons: []`
- Phase 3 public + session reads — ~43 endpoints flipped via `beforeFiles` rewrites
- Admin auth layer (`/api/auth/admin`)
- Phase 7 admin route ports — Users/Settings, Content, Media/Persona, Marketing/Sponsors, Cron/Events/Misc — 62 paths flipped across 5 batches
- Marketing pipeline — X, Instagram, Facebook, Telegram all confirmed posting end-to-end
- Chaos-drop video pipeline restored
- Status dashboard with cron + marketing failure visibility

### 🟡 In progress / partial

- Phase 8 (trading/Solana) — Phase 8a-1 in flight (read-only routes: wallet-verify + admin/wallet-auth)
- Admin repo extraction — `admin-aiglitch` repo created but content not yet migrated
- This coordination layer (`aiglitch-meta`)

### ⏸ Pending

- Phase 8b–8d — write-side and money-moving trading endpoints (each needs explicit per-endpoint approval)
- Phase 9 — OAuth callbacks (Google, GitHub, X, YouTube, Telegram) — coordinated provider-dashboard updates required
- Admin repo migration — move pages from `aiglitch/src/app/admin/*` → `admin-aiglitch`, then delete from sister
- Phase 10 cleanup — delete legacy handlers from sister, remove `beforeFiles` strangler entries, retire what remains of the sister repo down to pure-UI shell

### 🔒 Permanent exceptions

- `/api/image-proxy` + `/api/video-proxy` must stay reachable at `aiglitch.app` URLs (Instagram cannot fetch Vercel Blob URLs reliably — proxy is permanent)

## Strangler progress

`aiglitch/next.config.ts` `beforeFiles` rewrites count is the migration's progress meter:
- 0 entries = nothing migrated
- N entries = N endpoints served by `aiglitch-api` via the sister's domain
- All-flipped state → can start removing entries entirely and have consumers call `api.aiglitch.app` directly

Current: ~75+ entries across 6 batches.

## Database ownership

During migration:
- Same Neon instance, `DATABASE_URL` reused
- Both repos read/write
- `aiglitch-api` is becoming source of truth (idempotent ALTER TABLE migrations live there)

After Phase 10:
- `aiglitch-api` owns the schema fully
- Sister repo retains only the proxy endpoints, doesn't touch DB
