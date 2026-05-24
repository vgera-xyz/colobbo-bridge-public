---
title: "active-plan.md re-derive post-ADR-037 + Gate-12 hotfix + cibo-handoff Step 1a hardening (omit-when-multi-track / never-null)"
tags: [bridge, session, adhoc, active-plan, cibo-plan-invocation, cs-367-demotion, col-305-anchor, post-adr-037, linked-issue-rule, gate-12-hotfix, frontmatter-schema-discipline, cibo-handoff-step-1a, omit-when-multi-track, never-null, project-scoped-skill, operator-mandated-revision]
last_updated: 2026-05-24
source: memory
confidence: high
status: active
lobe_version: 1.0.0
---

# active-plan re-derive + Gate-12 hotfix + Step 1a hardening

## Summary

Operator invoked `/cibo-plan` mid-day and surfaced that `wiki/strategy/active-plan.md` was framing migration as a pipeline/Lobe concern and listing CS-367 as program critical-path #1 — both stale post-[[../../decisions/adr-037-cs368-migration-substrate-retirement|ADR-037]] (substrate retired 2026-05-23, v2.44.0). CC re-derived the plan from TIER-1 Linear MCP (`blocks: []` / `blockedBy: []` confirmed on CS-367/CS-368/COL-205/COL-305/CS-322), demoted CS-367 to a §Parallel high-priority decision section, anchored the critical path on COL-305 prereqs (SP-dump → Graphify refresh → smoke → batch-1), demoted PR #246 + CS-322 to §Parallel tracks, and (per operator) flipped `linked_issue: CS-367` → `null`. That `null` value broke Gate 12 (schema-pattern rejects null) — caught in the verify-before-edit pass on a follow-up question about the cibo-handoff Step 1a discipline. Two follow-on commits: a hotfix that omits the `linked_issue` field entirely (schema-Optional, absence is valid), and a durable Step 1a hardening that codifies "omit-when-multi-track, never null" with the frontmatter-validator pattern cited inline.

Three commits, all on `main`:

1. `2ffcc05ea` — `docs(active-plan): re-derive post-ADR-037 — demote CS-367 + anchor on COL-305 prereqs` (1 file, +59/-32)
2. `047af741f` — `fix(active-plan): omit linked_issue line instead of null — Gate 12 hotfix` (1 file, +1/-2)
3. `171f8dfc8` — `docs(cibo-handoff): add linked_issue omit-when-multi-track rule to Step 1a` (1 file, +3/-1)

No Linear ticket touched this session. Operator-driven inline correction + skill hardening discovered downstream. Tag-push wasn't triggered, so the Gate 12 break never reached CI — caught and fixed via local `verify-frontmatter-schema.sh` run within the same conversation.

## What was done — three deltas

### Delta 1 — `active-plan.md` re-derive (commit `2ffcc05ea`)

Triggered by operator reading the plan via `/cibo-plan` and finding the migration framing stale relative to ADR-037 reality. CC verified Linear structured fields:

- **CS-367** — Backlog, due 2026-05-28, High; `blocks: []`, `blockedBy: []`. Description-level claim "Need your call before COL-205 batch-1 starts" reflects convention-default preference, NOT a structural blocking edge.
- **CS-368** — Done, completed 2026-05-23. Substrate retirement ratified.
- **COL-305** — Backlog, parent COL-205. OWN prereqs in body: (1) SP-dump from RDS to `Colobbo-Web/db/`, (2) worker-path reconciliation, (3) Graphify refresh, (4) single-endpoint smoke. `blocks: []`, `blockedBy: []`.
- **COL-205** — Backlog, due 2026-06-30. `blocks: [COL-207]`, `blockedBy: []`. Description names COL-203/COL-204/Agent-Teams as prereqs — not CS-367.
- **CS-322** — Backlog. `blockedBy: [CS-318, CS-146]`. Per ADR-037: wraps BUILD chain only post-retirement; doesn't touch migration runway.

Composed §Critical path as a 4-item operational ladder grounded in COL-305 prereqs (NOT pipeline/Lobe gates); added new §Parallel high-priority decision section for CS-367 with explicit "does NOT block critical path" framing + Linear-field citation; demoted PR #246 + CS-322 + an unscoped feature-builder/brownfield placeholder to §Parallel tracks; rewrote §Blockers around the sole structural blocker (`Colobbo-Web/db/` SP-dump landing); added §Recent corrections entry #1 capturing the demotion rationale; flipped `linked_issue: CS-367` → `null` (operator choice over `COL-305`, reflecting multi-track reality including upcoming unscoped feature-builder/brownfield track).

### Delta 2 — Gate-12 hotfix (commit `047af741f`)

Discovered downstream when operator asked "make `linked_issue: null` durable in cibo-handoff Step 1a" and CC verified the schema before drafting the edit. Ran `bash scripts/verify-frontmatter-schema.sh`:

```
FAIL: wiki/strategy/active-plan.md — linked_issue='null' must match (CS|COL|RKU|TD)-NNN
Gate 12 failed: 1 frontmatter violation(s) across 195 page(s).
```

Schema at `schemas/wiki-frontmatter-v1.json:60-64` declares `linked_issue` as `type: string` with pattern `^(CS|COL|RKU|TD)-\d+$`. The field is Optional (not in `required[]`) but when present must conform. The shell validator (`scripts/verify-frontmatter-schema.sh`) parses YAML `null` as the literal string `"null"`, which fails the regex.

CI consequence (verified pre-hotfix): Gate 12 runs on tag-push, NOT main-push (per `feedback_tag_is_the_ci_gate_for_agent_system`). The 3 commits on `main` since v2.44.0 wouldn't trigger CI. But the next tag-cut would RED.

Fix: omit the `linked_issue` line entirely. Schema-Optional → absence is the canonical "no single anchor" representation. Also updated the body-paragraph explanation to cite the schema rule for future readers. Re-ran the validator: `PASS: Gate 12 — 195 page(s) conform to wiki-frontmatter-v1`.

### Delta 3 — Step 1a hardening (commit `171f8dfc8`)

Codified the omit-when-multi-track rule into `.claude/skills/cibo-handoff/SKILL.md` Step 1a so future handoffs don't repeat the null trap. Before the edit, Step 1a's frontmatter template gave only one form:

```yaml
linked_issue: <current primary ticket>
```

No guidance for multi-track plans → operators (or CC composing the would-be content) had no rule to consult.

Added a comment annotation on the template line + a new paragraph immediately after the frontmatter block:

> **`linked_issue` selection rule (hard constraint first, then guidance):** **Omit the field entirely** when the plan spans multiple independent program-level tracks; **never set `null`** — the frontmatter validator requires a string conforming to `^(PREFIX)-NNN$` when the field is present, so `null` fails Gate 12 (`scripts/verify-frontmatter-schema.sh`) and a tag-push would RED. Schema marks the field Optional, so absence is the valid expression of "no single anchor". When to omit vs include: include with a single ticket ID when one tracker honestly anchors the plan's scope (the common case — most plans cohere around a primary delivery); omit when the plan's ticket trees span multiple independent track hierarchies and forcing a single ticket would re-narrow the plan's actual scope.

Ware-agnostic — uses generic terms (track hierarchies, primary delivery, ticket trees, PREFIX). Post-write Lobe-extractability grep at SKILL.md:102-104 only scans `head -10` + H2 headers, unaffected by this body addition. The cibo-handoff skill is project-scoped (`.claude/skills/cibo-handoff/`, NOT in `skills/global/`) so deploy.sh is a no-op — verified via `ls skills/global/ | grep cibo` (empty) + `ls ~/.claude/skills/ | grep cibo` (empty). Edit takes effect immediately when CC runs in this repo.

## Files changed

- `wiki/strategy/active-plan.md` — commits `2ffcc05ea` + `047af741f` (re-derive + Gate-12 hotfix)
- `.claude/skills/cibo-handoff/SKILL.md` — commit `171f8dfc8` (Step 1a hardening)

## Verification trail

- **Pre-hotfix Gate 12 run**: FAIL (`linked_issue='null'`). Documented in the hotfix commit body.
- **Post-hotfix Gate 12 run**: PASS — 195 pages conform.
- **TIER-1 verification of CS-367 demotion**: 5 Linear MCP `get_issue` calls (CS-367, CS-368, COL-205, COL-305, CS-322) with `includeRelations: true` — `blocks/blockedBy` arrays explicitly read and cited in the active-plan §Parallel high-priority decision section.
- **deploy.sh scope verification**: read deploy.sh end-to-end (367 lines, 10 sections). Zero references to `wiki/`. Confirms `wiki/strategy/active-plan.md` edits don't need deploy.
- **cibo-handoff skill scope verification**: `find . -name SKILL.md -path "*cibo-handoff*"` → only `./.claude/skills/cibo-handoff/SKILL.md`. Project-scoped per CLAUDE.md ("Project-scoped lifecycle skills carry `cibo-` prefix"). No deploy needed.

## Operator-caught corrections this session

1. **Stale migration framing in active-plan** (the trigger for the whole session) — operator caught at `/cibo-plan` invocation. CC re-derived from TIER-1.
2. **Frontmatter Gate 12 break introduced by CC** — would have shipped a broken frontmatter without the operator's follow-up question about Step 1a discipline. Caught only because CC ran `verify-frontmatter-schema.sh` before drafting the skill edit. Lesson: when composing/proposing a value for a frontmatter field, dry-run the validator against the proposed value (not just inspect the schema declaration prose).
3. **Operator framing premise for Q3** ("cibo-handoff is a SKILL — one of deploy.sh's 10 sections — so this edit DOES need deploy.sh") — CC flagged the misalignment: `cibo-*` skills are project-scoped (live ONLY in `.claude/skills/`), not in `skills/global/`, so deploy.sh is a no-op for this skill. Operator acknowledged.

## Lessons (process — not codified as feedback memory entries)

- **Schema-pattern fields take values literally** — when the schema says `type: string`, the YAML parser's `null` becomes the literal string "null" downstream; the validator's regex sees "null" and rejects it. Omit rather than null when "no value" is the intended semantics.
- **Operator premise verification is cheap insurance** — Q3 in the brief assumed deploy.sh propagates this skill; CC's two-line `ls`-based verification flipped the premise within seconds and avoided a wasted explanation. Worth doing on every multi-question follow-up brief.
- **The pre-edit dry-run pattern paid off twice this session** — once for `bash deploy.sh` (no-op confirmed before commit), once for `verify-frontmatter-schema.sh` (Gate 12 break caught locally, never shipped to tag-CI). Pattern: when about to amend a contract-bound surface (schema, deploy, gate), run the contract-check first.

## What's next (deferred, not actioned this session)

- **Feature-builder / brownfield agent-system build track scoping** — operator's explicit note: "next session's job, starting with discovery of what those terms map to in `agents/` + `wiki/`." Active-plan carries the placeholder in §Parallel tracks + §Open decisions + §Watchpoints.
- **CS-367 still open** — Jignesh decision due 2026-05-28; PR #246 fate + worker MOVE/KEEP default depend on it. Worker functions without it via STATIC parity + ADR-035 body-analysis rule.
- **`Colobbo-Web/db/` SP-dump landing** — sole structural blocker on the migration program; operator/Jignesh-owned external dep; worker emits `SP-DUMP-MISSING` + STOP at preflight until landed.
- **CS-321 MEMORY.md compaction** — backlog watchpoint; MEMORY.md still over 200 soft cap (the start-of-session reminder showed a 24.8KB / 24.4KB warning).
- **CS-320 macOS gremlin** — backlog; new sightings this session in `wiki/bridge/sessions/` + `memory/` ("2026-05-24T0030-adhoc-migration-worker-wiring 2.md" + "memory/2026-05-24 2.md"). Not actioned — class is well-documented and pending systemic cleanup.

## See also

- [[../../strategy/active-plan|active-plan]] — the re-derived plan-state surface (current frontmatter omits `linked_issue` per the multi-track rule)
- [[../../decisions/adr-037-cs368-migration-substrate-retirement|ADR-037]] — the substrate retirement that makes the prior plan framing stale
- [[../../decisions/adr-034-cs353-plan-state-persistence-boundary|ADR-034]] — plan-state ownership boundary (CC = sole writer; chat = research/draft only)
- [[../../decisions/adr-013-2026-agent-memory-pattern|ADR-013]] — the 3-layer memory pattern this handoff implements
- [[2026-05-24T0030-adhoc-migration-worker-wiring|prior bridge — migration-worker wiring refinement]]
- [[2026-05-23T0940-CS-368|earlier bridge — CS-368 + ADR-037 substrate retirement ship]]
- [[2026-05-23T0050-CS-367|earlier bridge — CS-367 surfacing + migration-worker ship]]
