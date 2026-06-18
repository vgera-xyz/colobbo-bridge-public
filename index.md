---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-06-18T07:13:18Z"
total_sessions_since_last_sync: 101
public_sessions_since_last_sync: 101
sessions_omitted_count: 91
source: mirror
window_size: 10
---

# colobbo bridge mirror — index

One-way mirror of the latest 10 public bridge sessions from `colobbo-agent-system` (private). **Read the sentinel discipline in [README.md](README.md) before trusting this content.**

## Sentinel

`mirror_synced_at` above MUST be within 7 days of your current date. If older, abort and report stale to the human operator. Do not pretend to know state.

## Counts (this sync)

- Sessions added or changed since last sync: 101
- Of those, public: 101
- Public sessions beyond the N=10 window (older, not mirrored here): 91

## Sessions in this mirror (newest first)

- [`sessions/2026-06-18T0707-CS-147.md`](sessions/2026-06-18T0707-CS-147.md)
- [`sessions/2026-06-18T0413-CS-50.md`](sessions/2026-06-18T0413-CS-50.md)
- [`sessions/2026-06-17T1609-CS-388.md`](sessions/2026-06-17T1609-CS-388.md)
- [`sessions/2026-06-17T1518-CS-147.md`](sessions/2026-06-17T1518-CS-147.md)
- [`sessions/2026-06-05T1202-COL-307.md`](sessions/2026-06-05T1202-COL-307.md)
- [`sessions/2026-06-04T2339-COL-307.md`](sessions/2026-06-04T2339-COL-307.md)
- [`sessions/2026-06-04T0523-COL-307.md`](sessions/2026-06-04T0523-COL-307.md)
- [`sessions/2026-06-04T0330-CS-397.md`](sessions/2026-06-04T0330-CS-397.md)
- [`sessions/2026-06-03T1041-COL-307.md`](sessions/2026-06-03T1041-COL-307.md)
- [`sessions/2026-06-03T0715-COL-307.md`](sessions/2026-06-03T0715-COL-307.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
- [`decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md`](decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md)
- [`decisions/adr-037-cs368-migration-substrate-retirement.md`](decisions/adr-037-cs368-migration-substrate-retirement.md)
