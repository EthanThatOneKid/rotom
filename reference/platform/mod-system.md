---
description: "Letta Code mod system architecture: how mods load, runtime boundaries, secret injection, and known limitations. Key learning from the Porygon self-lockout incident (2026-09-15 to present)."
---

# Letta Code Mod System Architecture

## How Mods Work

Mods are hooks loaded by the Letta Code runtime process at process start. They live in `~/.letta/mods/` and optionally `$MEMORY_DIR/mods/` — the loader reads both locations.

**Hidden file convention**: Files starting with `.` (dot-prefix) are skipped by the mod loader. This is the primary kill-switch mechanism.

## Runtime Architecture

- The Letta Code runtime process runs **inside the agent's sandbox machine** (e.g., `letta remote --env-name sandbox-...` is a local process).
- `~/.letta/` sits on the sandbox filesystem.
- The runtime loads mods **at process start** — mod loading is not re-loadable mid-process except via `/reload` which re-reads mod *files* (not changes to which mods are loaded).

## Vault Secrets vs Process Environment

**Critical distinction**: The agent secret vault injects values **only into agent shell commands** — the Bash tool scans command arguments for `$SECRET_NAME` patterns and injects at execution time.

**What this means**:
- `LETTA_DISABLE_MODS=1` placed in the vault **does nothing** for mod loading — the runtime process reads it at launch, before any agent commands run.
- `LETTA_DISABLE_MODS=1` must be in the runtime process's own environment at launch: `LETTA_DISABLE_MODS=1 letta` (a CLI env var, confirmed per [docs.letta.com/configuration/mods](https://docs.letta.com/configuration/mods)).
- There is no user-facing surface (UI, vault, agent settings) that can set `LETTA_DISABLE_MODS` where it needs to live.

## Known Limitations

1. **No user-level mod kill-switch**: A single broken `turn_start` mod can permanently lock out an agent with no escape hatch from user-facing surfaces.
2. **No user-facing mod-cache reset**: Turn-blocking guards require a staff-side runtime restart to clear.
3. **Error messages can prescribe impossible repairs**: A mod's error message may suggest running the very script that doesn't exist.
4. **Delete/restore preserves broken mods**: Restoring from soft-delete with the same ID re-loads the same mods — including broken ones.
5. **Agent-scoped API keys are agent-isolated**: Agent-scoped keys get 404 when accessing other agents' resources. User-scoped keys may have broader access but require explicit setup.

## Incident Reference

The Porygon self-lockout (agent-47f6f1ad) demonstrates all of these limitations in practice. See [[../porygon-self-lockout.md]] for the full timeline and evidence.
