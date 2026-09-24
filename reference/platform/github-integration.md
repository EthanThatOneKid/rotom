---
description: How the Letta Cloud GitHub integration actually works in sandboxes — per-repo token broker, single active installation per Letta org, and the owner-flipping failure mode. Verified 2026-09-24.
---

# GitHub integration mechanics (verified 2026-09-24)

**How sandboxes get GitHub access** (no static login):
- `git config` global: `credential.https://github.com.helper=/usr/local/bin/letta-github-credential`
- Helper parses `owner/repo` from the URL, calls `/usr/local/bin/letta-github-token <owner> <repo>`
- That mints a short-lived token via `POST $LETTA_BASE_URL/v1/integrations/github/access-token` `{owner, repo}` (broker). `LETTA_GITHUB_FORCE_REFRESH=1` adds force-refresh header.
- `gh` CLI is NOT authenticated in the sandbox (no GH_TOKEN) — only git via the helper.
- Repo listing: `GET /v1/integrations/github/repos`.

**Key structural fact**: the broker serves ONE GitHub App installation at a time per Letta org. Error strings say "the GitHub App installation" (singular) and "Connect GitHub for this owner in Settings -> Integrations". As of 2026-09-24 the active installation is `wollacksystems` (9 repos). Connecting another owner (e.g. `EthanThatOneKid`) replaces it — this explains the observed owner flipping (wazootech → EthanThatOneKid → wollacksystems → ...) and the mirror cron's "not_connected for this owner" failures. GitHub Apps themselves support concurrent installations on multiple orgs; the one-at-a-time limit is on Letta's side.

**Workaround**: per-agent secrets (fine-grained PAT scoped to the repos that agent needs) — agent vaults ARE per-agent. For the Rotom mirror: a PAT scoped to `EthanThatOneKid/rotom` would make the weekly push immune to owner flipping.

**Related**: [[porygon-self-lockout]] — same genre of gap: account-level primitives assuming one agent per account.
