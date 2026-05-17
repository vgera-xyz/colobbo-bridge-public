---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-05-17T05:16:32Z"
total_sessions_since_last_sync: 1
public_sessions_since_last_sync: 1
sessions_omitted_count: 38
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
- Public sessions beyond the N=10 window (older, not mirrored here): 38

## Sessions in this mirror (newest first)

- [`sessions/2026-05-17T0508-CS-327.md`](sessions/2026-05-17T0508-CS-327.md)
- [`sessions/2026-05-17T0339-CS-336.md`](sessions/2026-05-17T0339-CS-336.md)
- [`sessions/2026-05-17T0219-CS-325.md`](sessions/2026-05-17T0219-CS-325.md)
- [`sessions/2026-05-16T1617-CS-146.md`](sessions/2026-05-16T1617-CS-146.md)
- [`sessions/2026-05-16T1306-CS-294.md`](sessions/2026-05-16T1306-CS-294.md)
- [`sessions/2026-05-16T0749-CS-317.md`](sessions/2026-05-16T0749-CS-317.md)
- [`sessions/2026-05-16T0055-CS-311.md`](sessions/2026-05-16T0055-CS-311.md)
- [`sessions/2026-05-15T1311-CS-316.md`](sessions/2026-05-15T1311-CS-316.md)
- [`sessions/2026-05-15T1214-CS-316.md`](sessions/2026-05-15T1214-CS-316.md)
- [`sessions/2026-05-15T0643-CS-316.md`](sessions/2026-05-15T0643-CS-316.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
