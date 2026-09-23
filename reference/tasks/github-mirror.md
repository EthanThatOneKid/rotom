---
description: Weekly mirror of Rotom's agent memory to github.com/ethanthatonekid/rotom; access gap as of 2026-09-22.
---
# GitHub Mirror Task (requested by Ethan, 2026-09-22)

**Goal**: Mirror my agent files (MemFS memory) to `github.com/ethanthatonekid/rotom`, synced at least weekly.

**Schedule**: `github-mirror-sync` cron — Sundays 18:00 UTC (11am PDT), `0 18 * * 0`, cloud runner, fresh conversation per fire.

**Access gap (2026-09-22)**: The sandbox's GitHub integration is connected to the `wazootech` org only. `ethanthatonekid/rotom` returns 404 via API (repo doesn't exist or is private + inaccessible). To unblock: Ethan must either install the Letta GitHub App for the `ethanthatonekid` owner and create the repo, or make the repo public so it can be cloned without auth (push still needs credentials though — so App install is the real fix).

**Mirror mechanics when unblocked**: MemFS dir is a git repo; add remote `mirror` → https://github.com/ethanthatonekid/rotom.git, commit pending changes, push main. Shared `porygon-family` memory is a separate repo — ask Ethan whether it should be mirrored too (it has its own remote already).