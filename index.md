---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-07-09T03:42:27Z"
total_sessions_since_last_sync: 2
public_sessions_since_last_sync: 2
sessions_omitted_count: 104
source: mirror
window_size: 10
---

# colobbo bridge mirror — index

One-way mirror of the latest 10 public bridge sessions from `colobbo-agent-system` (private). **Read the sentinel discipline in [README.md](README.md) before trusting this content.**

## Sentinel

`mirror_synced_at` above MUST be within 7 days of your current date. If older, abort and report stale to the human operator. Do not pretend to know state.

## Counts (this sync)

- Sessions added or changed since last sync: 2
- Of those, public: 2
- Public sessions beyond the N=10 window (older, not mirrored here): 104

## Sessions in this mirror (newest first)

- [`sessions/2026-07-09T0339-CS-406.md`](sessions/2026-07-09T0339-CS-406.md)
- [`sessions/2026-07-09T0224-CS-373.md`](sessions/2026-07-09T0224-CS-373.md)
- [`sessions/2026-07-02T1331-CS-404.md`](sessions/2026-07-02T1331-CS-404.md)
- [`sessions/2026-06-23T1352-CS-373.md`](sessions/2026-06-23T1352-CS-373.md)
- [`sessions/2026-06-23T1209-CS-404.md`](sessions/2026-06-23T1209-CS-404.md)
- [`sessions/2026-06-23T1129-CS-373.md`](sessions/2026-06-23T1129-CS-373.md)
- [`sessions/2026-06-22T1600-CS-373.md`](sessions/2026-06-22T1600-CS-373.md)
- [`sessions/2026-06-22T1510-CS-360.md`](sessions/2026-06-22T1510-CS-360.md)
- [`sessions/2026-06-22T1408-CS-373.md`](sessions/2026-06-22T1408-CS-373.md)
- [`sessions/2026-06-22T0942-CS-373.md`](sessions/2026-06-22T0942-CS-373.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
- [`decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md`](decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md)
- [`decisions/adr-037-cs368-migration-substrate-retirement.md`](decisions/adr-037-cs368-migration-substrate-retirement.md)
