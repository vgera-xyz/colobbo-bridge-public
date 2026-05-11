# colobbo-bridge-public

One-way public mirror of `wiki/bridge/` from the private `colobbo-agent-system` repo. Read-only artifact for cross-project AI consumers (currently raku-brain, a separate Claude.ai chat with no filesystem access to colobbo-agent-system).

**This repo is NOT a source of truth.** The authoritative bridge is in the private repo. This mirror exists to give filesystem-isolated consumers an autonomous read path.

## Purpose

Pre-2026-05-11, cross-project consumers fetched session context from a Linear document. That document stranded silently when the canonical bridge moved to filesystem (CS-208). Eight session updates went unrecorded between 2026-04-25 and 2026-05-10. CS-283 / ADR-017 (in the private repo) replaces the Linear-doc approach with this filesystem-canonical mirror.

## Sentinel discipline

Consumers MUST read `index.md` frontmatter before trusting any session data:

1. Fetch `https://raw.githubusercontent.com/vgera-xyz/colobbo-bridge-public/main/index.md`.
2. Parse YAML frontmatter.
3. Assert `mirror_synced_at` is within **7 days** of the consumer's current date. If older, **abort and report stale** to the human operator. Do not pretend to know state.
4. Read the count fields below to detect filtering or truncation before fetching individual sessions.

The mirror sync runs at the end of every `/cibo-handoff` session-close in the private repo. A healthy cadence shows `mirror_synced_at` updating daily or near-daily.

## Frontmatter counts

`index.md`'s frontmatter includes three count fields, updated every sync:

| Field | Meaning |
|---|---|
| `total_sessions_since_last_sync` | Total bridge files (regardless of public flag) added/changed since the last sync. |
| `public_sessions_since_last_sync` | Subset of the above that were public (i.e. did not have `public: false` in frontmatter). |
| `sessions_omitted_count` | Number of public sessions that fell outside the N=10 window (i.e. older sessions exist on the private side that aren't in this mirror). |

If `total > public`, sessions were filtered via `public: false`. If `sessions_omitted_count > 0`, older sessions exist beyond the mirror window — request via Vipin if needed.

## Opt-out

Individual sessions can be excluded from this mirror by adding `public: false` to their frontmatter on the private side. Default is public. Operators on the private side are responsible for marking sensitive sessions before they enter the mirror window.

## Window

The mirror surfaces the **newest 10** bridge sessions by filename timestamp (filenames sort alphabetically = chronologically by design). Older sessions remain in the private repo only. This is bounded to keep the public surface small; `sessions_omitted_count` flags weeks where the window is insufficient.

## Consumer-contract ADRs

`decisions/` contains ADRs flagged `consumer_contract: true` on the private side. These document contracts that cross-project AI consumers (currently raku-brain) MUST follow. The same `mirror_synced_at` sentinel discipline applies — verify freshness in `index.md` BEFORE fetching any ADR by raw URL pattern `https://raw.githubusercontent.com/vgera-xyz/colobbo-bridge-public/main/decisions/<filename>`.

**Currently mirrored:**

- [`decisions/adr-017-public-bridge-mirror-pattern.md`](decisions/adr-017-public-bridge-mirror-pattern.md) — the public mirror pattern itself (consumer contract for fetching from this mirror).

Default is opt-out. ADRs without `consumer_contract: true` on the private side are NOT mirrored here. No omit-window applies — the full set of flagged ADRs is mirrored on every sync, since consumers need the CURRENT contract state, not a sliding window. (Per CS-288.)

## Layout

```
colobbo-bridge-public/
├── README.md      ← this file
├── index.md       ← frontmatter sentinel + count fields + cross-link list
├── sessions/      ← latest N=10 session entries, immutable filenames
└── decisions/     ← ADRs flagged consumer_contract: true (full set, no window)
```

## Pattern reference

See `wiki/decisions/adr-017-public-bridge-mirror-pattern.md` in the private repo (`colobbo-agent-system`) for the full decision record: pattern shape, consumer-contract rationale, default-public discussion, N=10 sizing, sentinel staleness threshold.

## Out of scope for this mirror

- Bidirectional sync. Mirror is read-only by design.
- Historical archive. Older sessions stay in the private repo.
- Per-consumer auth. Public means public; sensitivity managed at source via `public: false`.
- Source-of-truth claims. If this mirror ever disagrees with the private bridge, the private bridge wins.
