# AIG!itch Migration Playbook v1

> Canonical playbook for the dual-repo + admin-extraction migration. Both Claude sessions (Opus on backend, Haiku on frontend, and whatever runs the admin repo) MUST treat this as mandatory reading at session start.
>
> Per-repo `CLAUDE.md` files are still authoritative for repo-specific paths and conventions; this playbook is the cross-repo discipline that overrides nothing but supplements everything.

---

## Goal

End-state of the migration:

- **aiglitch** = pure UI only. No business logic, no crons, no auth endpoints. Calls `api.aiglitch.app` for everything.
- **aiglitch-api** = all business logic, all crons, all platform endpoints. Owns the database schema.
- **admin-aiglitch** = admin UI only, separate Vercel project, separate domain (`admin.aiglitch.app`). Calls `api.aiglitch.app/api/admin/*`.

Everything not aligned with that end-state is migration debt.

---

## Mandatory session start (every Claude, every session)

1. **Read this playbook** (the one in `aiglitch-meta`, not a stale copy in your repo).
2. **Read your repo's `CLAUDE.md`** — project-specific instructions take precedence on repo-local details.
3. **Read your repo's `HANDOFF.md`** — latest session log, what was left in flight.
4. **Pull the sister repos locally** — for cross-repo work, `git pull --ff-only` the other clones inside your working directory (you already have them set up).
5. **Check the GitHub Project board** in `aiglitch-meta` — see current state of every active workstream, what's blocked, what's ready.
6. **State explicitly in your thinking** that you've done steps 1–5 before writing any code.

---

## Core principles (non-negotiable)

### 1. Never guess about another repo

If you need to know what's in `aiglitch` while you're working in `aiglitch-api` (or vice versa), READ THE LOCAL CLONE. Don't speculate. Don't write code based on memory of how it "probably" works.

### 2. Cross-repo handoffs go through the project board first

When your work creates a task for the other side (e.g., backend ships a new route → frontend needs to add a `beforeFiles` entry), the primary handoff is **a card on the shared project board**, not a free-form prompt.

The board card is the source of truth. Any prompt you write for the other Claude is a derivative artifact summarizing the card.

### 3. Cost awareness

Context loss and fix spirals have cost real money on this project. Symptoms to watch for:
- Repeated "let me re-read the file" cycles
- Multiple consecutive "final" deploys (the X media upload saga shipped 4 commits titled "final" in 37 minutes — anti-pattern)
- Long natural-language prompts describing the other repo's state instead of just asking the other Claude to check the project board

### 4. Spiral protocol (from CLAUDE.md history)

Stop after **3 attempts** on the same problem. The system has 6+ cache layers, 2 repos (soon 3), and 2+ Claude sessions. Each fix can reveal a deeper layer. Going past 3 attempts without external help (sister Claude reads your code; or you escalate to user) wastes money fast.

---

## Session end requirements (every time)

Before closing your turn:

1. **Update the project board** — move cards, set status, add blockers, assign next-action owner.
2. **Write a short structured handoff for the other Claude** (max 15 lines):
   - What was completed (with PR/commit links)
   - What was left in a clean state
   - Open questions / decisions needed from user
   - Suggested next batch for the other side
3. **Commit + push your branch.**
4. **Update HANDOFF.md** in your repo with the session entry.
5. **Tag releases per Rule 5** (see your repo's CLAUDE.md for the exact format).

---

## Batch discipline

- Prefer **thematic batches** (e.g., "Phase 7 admin Marketing/Sponsors group") over single endpoints.
- Every batch must have: tests + manual verification + rollback path documented before consumer flip.
- **Trading (Phase 8) and OAuth (Phase 9)** require explicit user written approval **per endpoint** before any work begins. Locked decisions per project history.

---

## Automation & efficiency

- Prefer **updating the project board** over writing long prompts for the other Claude.
- Write **small verification scripts** in `aiglitch-meta/scripts/` instead of "manually test by curling these URLs" instructions.
- Use **issue templates** in `aiglitch-meta/.github/ISSUE_TEMPLATE/` for cross-repo work — they enforce the right fields.

---

## Anti-patterns (do not do)

- ❌ Long natural-language prompts describing the other repo's state. Update the board instead.
- ❌ Starting work on the other side's files without a clear handoff card.
- ❌ Skipping board updates "because it's faster."
- ❌ Multiple consecutive "final" PRs on the same problem (spiral protocol violation).
- ❌ Generating code from memory of the sister repo without reading the local clone.
- ❌ Adding speculative features, columns, or routes "for later." Trim to what's needed today.

---

## Success metric

Migration is complete when:

- `aiglitch` repo has zero business logic, zero crons, and only calls `aiglitch-api`.
- All traffic (except permanent Instagram proxies) goes through `api.aiglitch.app`.
- Admin UI is fully extracted to `admin-aiglitch` repo and `admin.aiglitch.app` serves it independently.
- The `aiglitch` repo can be reduced to a pure-UI shell + permanent Instagram proxy and nothing else.

---

## Phase status (high-level)

See `docs/architecture.md` for current state and `docs/adr/` for decision records. The project board has the working catalogue of every remaining task.
