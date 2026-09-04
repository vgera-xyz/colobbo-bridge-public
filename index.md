---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-09-04T23:12:00Z"
total_sessions_since_last_sync: 3
public_sessions_since_last_sync: 3
sessions_omitted_count: 129
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
- Public sessions beyond the N=10 window (older, not mirrored here): 129

## Sessions in this mirror (newest first)

- [`sessions/2026-09-05T0000-CS-428-context-backup.md`](sessions/2026-09-05T0000-CS-428-context-backup.md)
- [`sessions/2026-09-04T2303-CS-428.md`](sessions/2026-09-04T2303-CS-428.md)
- [`sessions/2026-08-31T1356-CS-425.md`](sessions/2026-08-31T1356-CS-425.md)
- [`sessions/2026-08-30T0034-CS-423.md`](sessions/2026-08-30T0034-CS-423.md)
- [`sessions/2026-08-28T1427-CS-423.md`](sessions/2026-08-28T1427-CS-423.md)
- [`sessions/2026-08-27T0721-CS-412.md`](sessions/2026-08-27T0721-CS-412.md)
- [`sessions/2026-08-27T0427-COL-668.md`](sessions/2026-08-27T0427-COL-668.md)
- [`sessions/2026-08-27T0315-COL-668.md`](sessions/2026-08-27T0315-COL-668.md)
- [`sessions/2026-08-24T1521-CS-416.md`](sessions/2026-08-24T1521-CS-416.md)
- [`sessions/2026-08-23T2349-COL-95.md`](sessions/2026-08-23T2349-COL-95.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
- [`decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md`](decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md)
- [`decisions/adr-037-cs368-migration-substrate-retirement.md`](decisions/adr-037-cs368-migration-substrate-retirement.md)
