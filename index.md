---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-05-14T14:57:12Z"
total_sessions_since_last_sync: 36
public_sessions_since_last_sync: 36
sessions_omitted_count: 26
source: mirror
window_size: 10
---

# colobbo bridge mirror — index

One-way mirror of the latest 10 public bridge sessions from `colobbo-agent-system` (private). **Read the sentinel discipline in [README.md](README.md) before trusting this content.**

## Sentinel

`mirror_synced_at` above MUST be within 7 days of your current date. If older, abort and report stale to the human operator. Do not pretend to know state.

## Counts (this sync)

- Sessions added or changed since last sync: 36
- Of those, public: 36
- Public sessions beyond the N=10 window (older, not mirrored here): 26

## Sessions in this mirror (newest first)

- [`sessions/2026-05-14T1453-CS-144.md`](sessions/2026-05-14T1453-CS-144.md)
- [`sessions/2026-05-14T1403-CS-282.md`](sessions/2026-05-14T1403-CS-282.md)
- [`sessions/2026-05-14T1200-CS-314-cleanup.md`](sessions/2026-05-14T1200-CS-314-cleanup.md)
- [`sessions/2026-05-14T0941-CS-314.md`](sessions/2026-05-14T0941-CS-314.md)
- [`sessions/2026-05-14T0645-CS-290.md`](sessions/2026-05-14T0645-CS-290.md)
- [`sessions/2026-05-14T0606-CS-290.md`](sessions/2026-05-14T0606-CS-290.md)
- [`sessions/2026-05-14T0404-CS-290.md`](sessions/2026-05-14T0404-CS-290.md)
- [`sessions/2026-05-14T0214-CS-143.md`](sessions/2026-05-14T0214-CS-143.md)
- [`sessions/2026-05-13T1436-CS-143.md`](sessions/2026-05-13T1436-CS-143.md)
- [`sessions/2026-05-13T0339-CS-297.md`](sessions/2026-05-13T0339-CS-297.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
