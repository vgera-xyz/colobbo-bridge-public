---
title: "colobbo bridge mirror — index"
mirror_synced_at: "2026-05-13T14:40:58Z"
total_sessions_since_last_sync: 5
public_sessions_since_last_sync: 5
sessions_omitted_count: 18
source: mirror
window_size: 10
---

# colobbo bridge mirror — index

One-way mirror of the latest 10 public bridge sessions from `colobbo-agent-system` (private). **Read the sentinel discipline in [README.md](README.md) before trusting this content.**

## Sentinel

`mirror_synced_at` above MUST be within 7 days of your current date. If older, abort and report stale to the human operator. Do not pretend to know state.

## Counts (this sync)

- Sessions added or changed since last sync: 5
- Of those, public: 5
- Public sessions beyond the N=10 window (older, not mirrored here): 18

## Sessions in this mirror (newest first)

- [`sessions/2026-05-13T1436-CS-143.md`](sessions/2026-05-13T1436-CS-143.md)
- [`sessions/2026-05-13T0339-CS-297.md`](sessions/2026-05-13T0339-CS-297.md)
- [`sessions/2026-05-12T1528-CS-142.md`](sessions/2026-05-12T1528-CS-142.md)
- [`sessions/2026-05-12T1424-CS-141.md`](sessions/2026-05-12T1424-CS-141.md)
- [`sessions/2026-05-11T1334-CS-284.md`](sessions/2026-05-11T1334-CS-284.md)
- [`sessions/2026-05-11T0522-CS-255.md`](sessions/2026-05-11T0522-CS-255.md)
- [`sessions/2026-05-11T0119-CS-283.md`](sessions/2026-05-11T0119-CS-283.md)
- [`sessions/2026-05-10T0755-cs237-diagnostic.md`](sessions/2026-05-10T0755-cs237-diagnostic.md)
- [`sessions/2026-05-09T0052-CS-281-CS-194-verify-followup.md`](sessions/2026-05-09T0052-CS-281-CS-194-verify-followup.md)
- [`sessions/2026-05-08T1508-CS-194-verify-extended.md`](sessions/2026-05-08T1508-CS-194-verify-extended.md)

## Consumer-contract ADRs

These are architecture decision records that consumers MUST read for the contracts they describe. The same `mirror_synced_at` sentinel rules apply — verify freshness before fetching by URL.

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md)
