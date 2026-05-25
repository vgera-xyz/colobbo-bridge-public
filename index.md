---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-05-25T00:41:19Z"
total_sessions_since_last_sync: 1
public_sessions_since_last_sync: 1
sessions_omitted_count: 65
source: mirror
window_size: 10
---

# colobbo bridge mirror — index

One-way mirror of the latest 10 public bridge sessions from `colobbo-agent-system` (private). **Read the sentinel discipline in [README.md](README.md) before trusting this content.**

## Sentinel

`mirror_synced_at` above MUST be within 7 days of your current date. If older, abort and report stale to the human operator. Do not pretend to know state.

## Counts (this sync)

- Sessions added or changed since last sync: 1
- Of those, public: 1
- Public sessions beyond the N=10 window (older, not mirrored here): 65

## Sessions in this mirror (newest first)

- [`sessions/2026-05-25T0031-CS-37.md`](sessions/2026-05-25T0031-CS-37.md)
- [`sessions/2026-05-24T1454-CS-371.md`](sessions/2026-05-24T1454-CS-371.md)
- [`sessions/2026-05-24T1350-CS-370.md`](sessions/2026-05-24T1350-CS-370.md)
- [`sessions/2026-05-24T0202-adhoc-active-plan-rederive-step1a-fix.md`](sessions/2026-05-24T0202-adhoc-active-plan-rederive-step1a-fix.md)
- [`sessions/2026-05-24T0030-adhoc-migration-worker-wiring.md`](sessions/2026-05-24T0030-adhoc-migration-worker-wiring.md)
- [`sessions/2026-05-23T0940-CS-368.md`](sessions/2026-05-23T0940-CS-368.md)
- [`sessions/2026-05-23T0050-CS-367.md`](sessions/2026-05-23T0050-CS-367.md)
- [`sessions/2026-05-21T1604-CS-366.md`](sessions/2026-05-21T1604-CS-366.md)
- [`sessions/2026-05-21T0847-CS-294.md`](sessions/2026-05-21T0847-CS-294.md)
- [`sessions/2026-05-21T0537-CS-318.md`](sessions/2026-05-21T0537-CS-318.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
- [`decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md`](decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md)
- [`decisions/adr-037-cs368-migration-substrate-retirement.md`](decisions/adr-037-cs368-migration-substrate-retirement.md)
