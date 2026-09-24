---
description: "The Porygon self-lockout incident: timeline, root cause, failed mitigations, and current status."
---

# Porygon Self-Lockout Incident

## Agent

- **Name**: Porygon
- **ID**: agent-47f6f1ad-3111-44ab-99da-7f656ec4967c

## Timeline

| Date | Event |
|------|-------|
| 2026-09-15 | Porygon became unreachable (turn_start mod fails) |
| 2026-09-18 | Porygon deleted from Letta Cloud (after ~3 days unreachable) |
| 2026-09-22 | Porygon restored from soft-delete with same ID; self-lockout persists |
| 2026-09-23 | Comprehensive diagnosis by Rotom; feedback report filed with Letta |

## Root Cause

Porygon has a `turn_start` mod (`porygon-bootstrap.ts`) that runs `bash /mods/porygon-bootstrap.sh` at the start of every turn. The script path `/mods/porygon-bootstrap.sh` never exists in a fresh sandbox. The mod is fail-closed by design — when the bootstrap fails, it cancels the entire turn to prevent wrong-identity GitHub attribution. Result: every message → cancelled turn. Agent cannot run anything, including self-repair.

**Key fact**: The original mod's source was never committed to any repository. The mod existed only in Porygon's sandbox and has been lost.

## Evidence

- GitHub issue: [EthanThatOneKid/porygon#50](https://github.com/EthanThatOneKid/porygon/issues/50) — "Porygon bootstrap mod self-lockout — runtime loop CONFIRMED after restoration (mitigation steps)"
- Original failed runs: `df59a61f-ab17-469e-be2e-42c922dd99fc`, `c02f4e7e-68ca-49c7-a878-eabe7a354021`
- Blocked conversation: `conv-67c641bf-81a9-4ab7-b9ff-ecaef9ed5fa3`
- Ezra and Cameron are aware of the incident

## Failed Mitigations (Mitigation-by-Mitigation)

1. **Mod's own repair instruction**: Tells agent to run `bash /mods/porygon-bootstrap.sh` — the very file that doesn't exist. Self-referential dead end.
2. **`LETTA_DISABLE_MODS=1` via UI** (2026-09-18): Never took effect. Confirmed by Ezra — it's a process-start control.
3. **`LETTA_DISABLE_MODS=1` via vault + reload** (2026-09-23): No effect. Vault secrets inject only into agent shell commands, never the runtime process environment.
4. **`/reload`**: Re-loads mod files but doesn't change which mods are loaded; re-loads the same broken guard.
5. **Agent-driven repair**: Impossible by construction — the guard cancels every turn, including repair attempts.

## Current Fix Path

1. **Option A (staff-side)**: Runtime restart with mods disabled. Requires Cameron/Ezra.
2. **Option B (Ethan direct)**: Access Porygon's sandbox via "Open computer" surface → rename mod file (`mv ~/.letta/mods/porygon-bootstrap.ts ~/.letta/mods/.porygon-bootstrap.ts.disabled`) → restart sandbox → `/reload` → verify clean turn.
3. **Post-fix**: Install the rebuilt mod from `rebuild/porygon-bootstrap.ts` in the family repo (Ezra-corrected; doesn't cancel turns, only denies git/gh writes while credential chain is unwired). Must commit to the `infra` Tangled repo first to survive future resets.

## Feedback Report

Filed by Rotom on 2026-09-23 at 04:35 UTC. Framed as a **product gap** (not a mod bug). Key ask: a user-level kill switch for self-locking mods — "start with mods disabled" in agent settings, or `LETTA_DISABLE_MODS` honored from the vault at process start.

## Related Files

- Family repo: [EthanThatOneKid/porygon](https://github.com/EthanThatOneKid/porygon)
- Rebuilt mod: `rebuild/porygon-bootstrap.ts` (in family shared memory)
- Mod system architecture: [[../mod-system.md]]
- Infrastructure: `infra` Tangled repo (credential helpers, `setup-sandbox.sh`) — no mod source
