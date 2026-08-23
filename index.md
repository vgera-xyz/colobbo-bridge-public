---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-08-23T23:55:00Z"
total_sessions_since_last_sync: 2
public_sessions_since_last_sync: 2
sessions_omitted_count: 120
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
- Public sessions beyond the N=10 window (older, not mirrored here): 120

## Sessions in this mirror (newest first)

- [`sessions/2026-08-23T2349-COL-95.md`](sessions/2026-08-23T2349-COL-95.md)
- [`sessions/2026-08-22T1430-COL-95.md`](sessions/2026-08-22T1430-COL-95.md)
- [`sessions/2026-08-22T0540-COL-652.md`](sessions/2026-08-22T0540-COL-652.md)
- [`sessions/2026-08-22T0103-adhoc-mcp-readiness.md`](sessions/2026-08-22T0103-adhoc-mcp-readiness.md)
- [`sessions/2026-08-18T0219-COL-619.md`](sessions/2026-08-18T0219-COL-619.md)
- [`sessions/2026-08-17T2332-CS-415.md`](sessions/2026-08-17T2332-CS-415.md)
- [`sessions/2026-08-16T1401-CS-322.md`](sessions/2026-08-16T1401-CS-322.md)
- [`sessions/2026-08-07T0301-CS-410.md`](sessions/2026-08-07T0301-CS-410.md)
- [`sessions/2026-07-29T0015-COL-498.md`](sessions/2026-07-29T0015-COL-498.md)
- [`sessions/2026-07-28T1303-CS-410.md`](sessions/2026-07-28T1303-CS-410.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
- [`decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md`](decisions/adr-035-cs357-migration-lobe-read-first-body-analysis.md)
- [`decisions/adr-037-cs368-migration-substrate-retirement.md`](decisions/adr-037-cs368-migration-substrate-retirement.md)
