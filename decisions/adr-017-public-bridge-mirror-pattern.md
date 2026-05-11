---
title: "ADR-017 — Public bridge mirror for cross-project AI consumption"
tags: [decisions, adr, architecture, bridge, cross-project, raku, mirror, cs-283]
last_updated: 2026-05-11
source: manual
confidence: high
lobe_version: 1.0.0
linked_issue: CS-283
status: active
consumer_contract: true
---

# ADR-017 — Public bridge mirror for cross-project AI consumption

## Status

Accepted 2026-05-11 (during CS-283 ship session).

## Context

[[adr-013-2026-agent-memory-pattern|ADR-013]] / [[../../CLAUDE|CLAUDE.md]] made `wiki/bridge/sessions/` in this private repo the canonical surface for session bridges. Cross-project AI consumers — currently raku-brain on Claude.ai chat — have no filesystem access to this repo. Pre-CS-208 they read a Linear document (`677f561accd9`) updated by `/end-session-v2`. That doc stranded silently when canonical state moved to filesystem on 2026-04-22; eight session updates between 2026-04-25 and 2026-05-10 were unrecorded there.

The Linear-as-bridge approach failed structurally on two prior incidents (`save_document` silent-truncation at ~16K chars; `save_issue` table-corruption; both captured in `feedback_save_document_truncation.md`). CS-208 did not replace it with a cross-project read path — it simply made filesystem canonical, leaving consumers without one. CS-283 closes that gap.

The decision is constrained by:

- Consumers run in environments with no filesystem access to colobbo-agent-system.
- Consumers cannot be granted write access to the private repo without coupling their session security to ours.
- Consumers need a freshness signal — silent staleness (the Linear-doc failure mode) is the primary anti-pattern to avoid.
- Sensitive session content must remain server-side; not every bridge entry is safe to publish.

## Decision

Knowledge in private git → mirror to public git → consumers fetch public mirror.

**Mechanics:**

- Public repo at `github.com/vgera-xyz/colobbo-bridge-public` (created 2026-05-11; empty until first sync).
- Sync script `scripts/sync-bridge-public.sh` in this repo, invoked by `cibo-handoff` Step 6 after every session-close.
- One-way only. Public is read-only. Disagreement between sides → private wins.
- **N=10 session window.** Newest 10 by filename timestamp (filenames sort chronologically by construction).
- **Default-public, per-entry opt-out** via `public: false` in session frontmatter. Opt-out is checked on every sync — flipping a previously-mirrored session to `public: false` removes it from the mirror on next run.

**Public `index.md` frontmatter sentinels:**

| Field | Purpose |
|---|---|
| `mirror_synced_at` | ISO 8601 UTC timestamp. Consumer aborts if older than 7 days. |
| `total_sessions_since_last_sync` | Sessions added or content-changed since prior sync. |
| `public_sessions_since_last_sync` | Subset of above that were public (i.e. opt-out detection). |
| `sessions_omitted_count` | Public sessions beyond the N=10 window (truncation flag). |

## Consumer contract

A cross-project consumer MUST follow this sequence before trusting any session content from the mirror:

1. Fetch `https://raw.githubusercontent.com/vgera-xyz/colobbo-bridge-public/main/index.md`.
2. Parse the YAML frontmatter.
3. **Sentinel check 1 — freshness.** Assert `mirror_synced_at` is within 7 days of the consumer's current date. If older, abort and report stale to the human operator. Do not pretend to know state.
4. **Sentinel check 2 — counts.** If `total_sessions_since_last_sync > public_sessions_since_last_sync`, sessions were filtered via opt-out — consumer knows some session activity is not visible. If `sessions_omitted_count > 0`, older sessions exist on the private side beyond the mirror window — request via the human operator if needed.
5. Only after both sentinel checks pass: fetch individual session entries by URL pattern `https://raw.githubusercontent.com/vgera-xyz/colobbo-bridge-public/main/sessions/<filename>`.

## Consequences

**Positive:**

- Cross-project consumers have an autonomous read path without filesystem access.
- Freshness is mechanical (sentinel timestamp), not eyeball (Linear-doc last-updated visible in UI but never enforced).
- Failure modes are visible: sync failure logs to `memory/incoming/sync-failures.md` and the mirror stops freshening — the sentinel catches it.
- Pattern extensible to future consumers with no script changes; each consumer just follows the contract.

**Negative / drift surfaces:**

- Mirror can lag silently for up to 7 days before the sentinel fails. Acceptable: 7 days is wider than a normal session cadence (currently ~daily), so genuine outages surface quickly while transient blips (one missed handoff) don't false-alarm.
- Public surface contains session bodies. Operators are responsible for marking sensitive sessions `public: false` BEFORE they enter the mirror window. Once mirrored and published, retroactive `public: false` removes from current mirror but does NOT erase git history on the public side. Treat as published-once.
- N=10 cap means deep history is inaccessible to consumers; request via human operator. Bounded surface is a feature (smaller blast radius if mirror is compromised, smaller fetch for consumers).
- Sync runs inside `cibo-handoff` after the commit + push of bridge entry. If the sync push to the public repo conflicts with another writer (rare; only Vipin pushes), the script logs and exits 0; next session picks it up.

## Alternatives considered

1. **Keep Linear-as-bridge.** Rejected. Failed structurally twice (silent truncation, table corruption). No freshness sentinel. CS-208 already moved canonical state off Linear.
2. **GitHub Pages from private repo.** Rejected. Requires paid Pages plan for private-source-public-output, complicates the auth surface. Public-repo + raw URL is free and mechanically simpler.
3. **Bidirectional sync (consumer pushes back).** Out of scope by design. Consumers are read-only; pushing back into colobbo-agent-system would require write auth, session conflict-resolution, and rate-limit considerations. None justified.
4. **Per-consumer auth on the public repo.** Rejected. Public means public; sensitivity is managed at source via `public: false`. Adding auth would re-introduce the Linear-doc failure mode (a separate surface that can desync).
5. **Mirror the entire `wiki/bridge/sessions/` history.** Rejected. Surface grows unbounded; sensitive content can never be retroactively pulled (git history); useful older context for consumers is a request, not a default. N=10 covers heavy weeks; `sessions_omitted_count` flags the cap.

## Rationales — specific knobs

### Default-public, opt-in opt-out

Considered the inverse (default-private, opt-in opt-in). Rejected:

- Opt-in creates a silent failure mode where "no recent sessions in mirror" is ambiguous (none happened? all opted out? sync broken?). Default-public makes "missing session" anomalous and visible.
- The audit cost is symmetric: with opt-in, every session needs a `public: true` marker; with default-public, only sensitive sessions need `public: false`. The current bridge has 21 sessions and zero sensitive ones — default-public is right-sized for now. If the ratio inverts later, flip via a separate ADR.

### N=10 window

- A heavy session week has been ~12 sessions (the CS-194 verify week, 2026-05-08 to 2026-05-09). N=10 covers a normal week without becoming a historical archive.
- Larger N (say 50) creates a public surface big enough to be misread as "the bridge" rather than "the recent slice"; the truncation flag (`sessions_omitted_count`) is intentionally visible so consumers know there's more behind.
- Smaller N (say 5) drops fresh context too aggressively — a Tuesday session would lose visibility of the prior Friday's context by Wednesday.

### 7-day staleness threshold

- Cadence baseline: handoffs run daily on most days, with multi-day gaps occasionally.
- 7 days is wider than typical gaps so non-events don't false-alarm. Genuine outages (script broken, repo deleted, network) surface within a week.
- If outages are short (< 1 day), the next handoff naturally re-syncs and the sentinel never trips — the path of least drift is correct-by-default.

## Cross-references

- [[../bridge/index|wiki/bridge/index]] — the canonical bridge surface this mirror reflects
- [[adr-013-2026-agent-memory-pattern|ADR-013]] — the filesystem-canonical pattern this builds on
- [[adr-015-memory-incoming-contract|ADR-015]] — `memory/incoming/` contract that governs `sync-failures.md` (status: operational)
- [[../../scripts/sync-bridge-public.sh|scripts/sync-bridge-public.sh]] — the implementation
- [[../../.claude/skills/cibo-handoff/SKILL|.claude/skills/cibo-handoff/SKILL.md]] — the skill that invokes the sync (Step 6)
