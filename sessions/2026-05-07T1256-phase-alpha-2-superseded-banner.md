---
title: "Session 2026-05-07T1256 — Phase α.2 SUPERSEDED banner on sp-inventory-summary"
tags: [bridge, session, phase-alpha-2, cs-158, sp-inventory, superseded, frontmatter-v1, migration]
last_updated: 2026-05-07
source: memory
confidence: high
lobe_version: 1.0.0
linked_issue: CS-158
status: active
---

# Session 2026-05-07T1256 — Phase α.2 SUPERSEDED banner on sp-inventory-summary

## What was done

Phase α.2 of the 2026-05-07 evening knowledge-layer integrity plan. Added a SUPERSEDED banner + frontmatter status to `wiki/migration/sp-inventory-summary.md` because its orphan-SP claims (dated 2026-04-22) were contradicted by the CS-158 audit (`wiki/migration/colobbo-web-audit/orphan-sp-detection.md`) which found the documented-orphans list stale. CS-158's ship comment had earlier claimed a banner was added; filesystem verification (`grep -i superseded`) returned empty — banner had been missed during CS-158 closeout. This session closes that loose end.

Two file-state changes to `wiki/migration/sp-inventory-summary.md`:

1. **Frontmatter:** `last_updated: 2026-04-22 → 2026-05-07`; added `status: superseded`; added `supersededBy: migration/colobbo-web-audit/orphan-sp-detection.md`; appended `superseded` to `tags`.
2. **Body:** inserted SUPERSEDED blockquote banner directly under frontmatter, above the `# SP inventory summary` heading. Banner includes the wikilink `[[colobbo-web-audit/orphan-sp-detection]]` (resolves correctly from the sibling-subfolder context).

## Schema notes

- `status: superseded` is one of the 4 enum values permitted by `schemas/wiki-frontmatter-v1.json` (line 55: `["active", "stub", "superseded", "archived"]`). Schema also documents that `superseded` exempts the page from Gate 11 size-delta — confirmed by the local Gate 11 pass on this small addition.
- `supersededBy` is NOT in the schema's named properties — only `supersedes` exists (the inverse-direction field). Schema's `additionalProperties: true` (line 15) permits `supersededBy` as a custom field. **This is the first page in the corpus to use `supersededBy`** (corpus-wide grep was empty pre-edit). If schema-tightening lands as a future ticket, `supersededBy` should be added as a sibling enum-typed string field.
- This is also the first page in the corpus with `status: superseded` in its frontmatter (the two prior matches for `status: superseded` text in `wiki/architecture/agent-memory-pattern.md` and `wiki/bridge/sessions/2026-05-05T1330-CS-208.md` are in body text, not frontmatter).

## Verification

- Gate 12 (frontmatter-schema): `PASS — 90 page(s) conform to wiki-frontmatter-v1`
- Gate 13 (wikilinks): `PASS — 582 wikilink(s) across 90 page(s) all resolve` (was 581 pre-edit; +1 from the new banner wikilink)
- Gate 11 (CLAUDE.md size-delta): `PASS — no wiki/**/*.md changes between origin/main and HEAD` (the banner addition is a content-only change; size-delta gate scopes to CLAUDE.md and the `superseded` status would have exempted this page anyway)

## Commits

- `7e0e335 docs(migration): SUPERSEDED banner on sp-inventory-summary per CS-158 audit` (rebased over `6eaca29 chore(graphify): refresh cibo from d8b78f4 [CS-240]` which landed remote-side via the self-graph workflow during the prior cibo-handoff)

## Files changed

- `wiki/migration/sp-inventory-summary.md` (1 file, 6 insertions, 2 deletions)

## What's next

- **Phase β cleanup audit** — separate, future work. The user flagged it as out-of-scope for Phase α.2; banner was the loose-end-closure.
- **Schema tightening (low-priority)** — could add `supersededBy` as a named property in `schemas/wiki-frontmatter-v1.json` v1.1.0 once the field has 2+ uses in the corpus. Currently single-use.
- **Audit prior CS-158 closeout** — CS-158 ship-comment claim of "banner added" was inaccurate (banner was missed). Could be a generic discipline lesson (verify ship-claims against filesystem before marking ticket Done) but the lesson is already captured in this session's earlier CS-279 phantom-execution event in `feedback_branch_grep_pre_flight.md` — same root pattern (stale-claim drift between ticket prose and filesystem state).

## See also

- [[2026-05-07T1244-CS-279]] — earlier this session, related discipline lesson on filesystem-grounded verification
- [[../../decisions/adr-013-2026-agent-memory-pattern]] — pattern this handoff implements
- CS-158 — original audit that produced `orphan-sp-detection.md`
- [[../../migration/sp-inventory-summary]] — the page that received the banner
- [[../../migration/colobbo-web-audit/orphan-sp-detection]] — the canonical replacement page
