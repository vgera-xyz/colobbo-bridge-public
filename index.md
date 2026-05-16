---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-05-16T07:53:11Z"
total_sessions_since_last_sync: 3
public_sessions_since_last_sync: 3
sessions_omitted_count: 33
source: mirror
window_size: 10
---

# colobbo bridge mirror — index

One-way mirror of the latest 10 public bridge sessions from `colobbo-agent-system` (private). **Read the sentinel discipline in [README.md](README.md) before trusting this content.**

## Sentinel

`mirror_synced_at` above MUST be within 7 days of your current date. If older, abort and report stale to the human operator. Do not pretend to know state.

## Counts (this sync)

- Sessions added or changed since last sync: 3
- Of those, public: 3
- Public sessions beyond the N=10 window (older, not mirrored here): 33

## Sessions in this mirror (newest first)

- [`sessions/2026-05-16T0749-CS-317.md`](sessions/2026-05-16T0749-CS-317.md)
- [`sessions/2026-05-16T0055-CS-311.md`](sessions/2026-05-16T0055-CS-311.md)
- [`sessions/2026-05-15T1311-CS-316.md`](sessions/2026-05-15T1311-CS-316.md)
- [`sessions/2026-05-15T1214-CS-316.md`](sessions/2026-05-15T1214-CS-316.md)
- [`sessions/2026-05-15T0643-CS-316.md`](sessions/2026-05-15T0643-CS-316.md)
- [`sessions/2026-05-15T0517-CS-316.md`](sessions/2026-05-15T0517-CS-316.md)
- [`sessions/2026-05-14T1626-CS-145.md`](sessions/2026-05-14T1626-CS-145.md)
- [`sessions/2026-05-14T1453-CS-144.md`](sessions/2026-05-14T1453-CS-144.md)
- [`sessions/2026-05-14T1403-CS-282.md`](sessions/2026-05-14T1403-CS-282.md)
- [`sessions/2026-05-14T1200-CS-314-cleanup.md`](sessions/2026-05-14T1200-CS-314-cleanup.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
