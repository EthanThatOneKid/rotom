---
description: Weekly mirror of Rotom's agent memory to github.com/ethanthatonekid/rotom; access gap as of 2026-09-22.
---
# GitHub Mirror Task (requested by Ethan, 2026-09-22)

**Goal**: Mirror my agent files (MemFS memory) to `github.com/ethanthatonekid/rotom`, synced at least weekly.

**Schedule**: `github-mirror-sync` cron — Sundays 18:00 UTC (11am PDT), `0 18 * * 0`, cloud runner, fresh conversation per fire.

**Status: LIVE (2026-09-23 04:50 UTC)**. Initial mirror pushed — full MemFS history to `EthanThatOneKid/rotom` main (a5ab90d). Remote `mirror` configured in MemFS repo (https://github.com/EthanThatOneKid/rotom.git — note capital E/T/O/K; lowercase URL triggers a redirect warning).

**Failed sync (2026-09-27 18:00 UTC)**: Integration connected for `wollacksystems`, not `ethanthatonekid` — `gh repo view` returned not_connected. MemFS was clean (no uncommitted changes to lose). No push attempted. Next attempt: next scheduled fire (2026-10-04 18:00 UTC).

**Failed sync (2026-10-04 18:00 UTC)**: Same owner gap — broker reports connected account is `wollacksystems`, requested `ethanthatonekid` (not_connected). Repo itself is reachable unauthenticated (`git ls-remote` returns HEAD a310993), so it exists and is non-empty. Push attempt failed (credential rejected; broker retry hit a 429). MemFS was clean — nothing uncommitted. Next attempt: next scheduled fire (2026-10-11 18:00 UTC).

**Known fragility**: The Letta GitHub integration flips connected owners (observed 2026-09-22/23: `wazootech` → `EthanThatOneKid` → `wollacksystems` → `EthanThatOneKid` → `wollacksystems` → `EthanThatOneKid`). When it's not on `EthanThatOneKid`, token minting for the mirror fails. If the weekly push fails with "not_connected for this owner", that's why — retry later, don't spam Ethan. Also: the token broker rate-limits (429) on rapid retries; wait 60s+ between attempts.

**Mirror mechanics**: MemFS dir is a git repo; commit pending changes, then `git push mirror main`. Repo creation is NOT possible via the App token (403 — no Administration permission); the repo already exists now, so this doesn't matter. Push capability verified (test branch created+deleted on 100daysofdesign earlier).

**Open question**: Shared `porygon-family` memory is a separate repo with its own remote — Ethan hasn't answered whether it should be mirrored into rotom too. Currently excluded.
