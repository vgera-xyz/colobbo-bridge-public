---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-05-19T14:27:36Z"
total_sessions_since_last_sync: 1
public_sessions_since_last_sync: 1
sessions_omitted_count: 52
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
- Public sessions beyond the N=10 window (older, not mirrored here): 52

## Sessions in this mirror (newest first)

- [`sessions/2026-05-19T1424-CS-351.md`](sessions/2026-05-19T1424-CS-351.md)
- [`sessions/2026-05-19T1342-CS-345.md`](sessions/2026-05-19T1342-CS-345.md)
- [`sessions/2026-05-19T1012-CS-350.md`](sessions/2026-05-19T1012-CS-350.md)
- [`sessions/2026-05-19T0925-COL-79.md`](sessions/2026-05-19T0925-COL-79.md)
- [`sessions/2026-05-19T0550-COL-79.md`](sessions/2026-05-19T0550-COL-79.md)
- [`sessions/2026-05-18T1357-CS-138.md`](sessions/2026-05-18T1357-CS-138.md)
- [`sessions/2026-05-18T0801-CS-138-audit-fix-and-build-discovery.md`](sessions/2026-05-18T0801-CS-138-audit-fix-and-build-discovery.md)
- [`sessions/2026-05-18T0651-CS-342-audit-dogfood-smoke.md`](sessions/2026-05-18T0651-CS-342-audit-dogfood-smoke.md)
- [`sessions/2026-05-18T0450-CS-338.md`](sessions/2026-05-18T0450-CS-338.md)
- [`sessions/2026-05-18T0428-CS-340.md`](sessions/2026-05-18T0428-CS-340.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
