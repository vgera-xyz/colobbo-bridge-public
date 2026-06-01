---
title: "migration-worker COL-306 emit-fidelity lessons baked + cibo-migrate batch-orchestration skill shipped + dotnet-to-node-migration deprecated; CS-393 filed (Planner pipeline stale vs ADR-037)"
tags: [bridge, session, adhoc, migration-worker, cibo-migrate, dotnet-to-node-migration, deprecation, col-306, cs-393, emit-fidelity, read-one-real-row, identity-carve-out, pr-as-checkpoint, glaze-by-reference, planner-pipeline-stale, adr-037, no-ticket-adhoc]
last_updated: 2026-06-01
source: memory
confidence: high
lobe_version: 1.0.0
status: active
---

# migration-worker COL-306 lessons + cibo-migrate skill + dotnet-to-node-migration deprecation (adhoc — no single anchoring ticket)

## Summary

Two-part agent-system tooling session on `main` (direct-push), both parts verify-first → propose → STOP → operator-approved → execute. **Part 1:** baked the six COL-306 silent-bug lessons (each tsc-clean but behaviourally wrong) into `agents/global/migration-worker.md` as gated rules + enforcement hooks (commit `95395db757`), then deployed. **Part 2:** scoped and shipped a new project-scoped skill `/cibo-migrate` — thin CC-as-runtime orchestration over the worker — deprecated the trace-era `dotnet-to-node-migration` predecessor with a redirect banner, and filed **CS-393** for a dangling-ref finding (the Planner migration-intent pipeline still dispatches the now-deprecated skill, stale vs ADR-037) (commit `dee0fab862`).

`linked_issue` omitted (matches the `2026-05-29T1422-adhoc-cibo-new-lobe-skill` precedent + schema marks it optional): the session spans worker hardening (COL-306-derived) + new tooling (adhoc) + a filed follow-up (CS-393), with no single anchor. COL-306 is provenance, not work-completed-this-session.

## What was done

### Part 1 — migration-worker COL-306 emit-fidelity lessons (`95395db757`, then deployed)
Operator-approved 5 decisions, then applied Edits A–F to `migration-worker.md` (+116/−1):
- **Step 4j** MOVE-emit fidelity: pagination metadata (totalRecords/totalPages) reproduction · export-vs-list sizing · `SELECT DISTINCT` → `.distinct()` + `count(DISTINCT)` · COL-186 dynamic-filter injection-fix (column allow-list + bound params) by default.
- **Step 4k** `.identity()` fix-on-contact: narrow carve-out — add `.identity()` to the MIGRATING table's PK only in a canonical schema (the one sanctioned canonical-file edit; `IDENTITY-ANNOTATED`-flagged; never a sweep). Boundaries 11/20 amended.
- **Step 6e** read-one-real-row HARD GATE for reconstructed/enriched output (catches the getBOGReport jsonb-flatten class static parity misses); §4 gained a read-only single-row non-prod carve-out to enable it.
- **§Parity** Tier-1/STATIC now a MANDATORY pre-PR gate; Tier-2/LIVE stated required-but-blocked (no seed env).
- **Enforcement wired** (not inert): Step 7 self-review bullets + Step 9 flag surfacing (`IDENTITY-ANNOTATED`, `INJECTION-FIX-APPLIED`, `STRUCTURE-VERIFY-BLOCKED`). Convention 2 (verbatim ADR-035 bake) untouched. Tracker PR-raised writeback already automated (Step 9b) — no change. Deployed to `~/.claude/agents/` (byte-match verified).

### Part 2 — cibo-migrate skill + dotnet-to-node-migration deprecation (`dee0fab862`)
- **NEW `.claude/skills/cibo-migrate/SKILL.md`** (project-scoped, not deploy.sh; live immediately). Thin CC-as-runtime dispatcher over `migration-worker` — the migration sibling of `cibo-pipeline`. 4 operator-confirmed design locks: **PR-as-checkpoint** (no worker change — the opened PR + surfaced report is the review; worker never merges) · **per-batch dispatch, one PR per module** (worker groups internally) · **skill owns branch lifecycle** (capture/create/restore; tells the worker "branch exists, use it" via the worker's obey-additional-instructions clause — confirmed Step 8b is naming-only, no `git checkout -b`) · **v0.1.0 single-module-per-invocation** (multi-module deferred). Glaze-by-reference for all Ware values (`repos.apiNode.path` · `branches.apiNode` · `reviewers.apiNode` · `conventions.branchFormat`); `COL`/`api_node`/`development`/reviewer literals scrubbed from skill prose.
- **Deprecated `dotnet-to-node-migration`** with a redirect banner → `/cibo-migrate` (kept, not deleted). It is the trace-era predecessor (stale `codebase-maps/traces/*` source; craft predates the worker).
- **Filed CS-393** (Colobbo Strategic · Medium · Backlog · related → CS-372/CS-368/CS-281): the Planner `migration`-intent pipeline still names `dotnet-to-node-migration` as executor (`glaze.sub_agents` + planner-lobe + planner.md + module-migration-playbook), stale vs ADR-037 (migration is operator-invoked via the worker, not pipeline-dispatched). Rewire scoped as its own ship (3-layer + recompose blast radius); explicitly NOT folded into this build.

## Files changed
- `agents/global/migration-worker.md` (+116/−1; Part 1) — deployed to `~/.claude/agents/`.
- `.claude/skills/cibo-migrate/SKILL.md` (NEW, 126 lines; Part 2).
- `skills/global/dotnet-to-node-migration/SKILL.md` (+2; deprecation banner — reaches `~/.claude/skills/` on next deploy).
- Linear: CS-393 created.

## Verification notes
- Part 1: fence balance unchanged (28, even); all 16 rule + 3 flag markers present once; deployed copy byte-matches source (1366 lines). Trailing-whitespace of the Flags-row table inspected before the Edit (anchored on the unique `SCHEMA-AUTHORED / MD-FALLBACK / SCHEMA-MISSING` line — `TIER-1-REFUSAL` line is duplicated in the Classification row).
- Part 2: pre-draft grep confirmed worker Step 8b does NOT hard-create the branch (naming-only) → skill-owns-branch needs no worker change. Dangling-ref grep surfaced the Planner-pipeline staleness → CS-393. Dup-check (list_issues) confirmed CS-393 distinct from CS-372 (trace-staleness vs dispatch-pipeline-staleness).
- Both pushes required a rebase over parallel graphify auto-commits (clean fast-forward both times); no force-push.

## What's next
- **CS-393** — rewire Planner migration-intent off `dotnet-to-node-migration` (separate ship; Backlog).
- **dotnet-to-node-migration banner reaches `~/.claude` only on next `deploy.sh`** (global skill) — this handoff's Step 5 deploy closes that gap.
- **First real `/cibo-migrate` run** is the true test of the skill (this session only built + reviewed it). The next COL-205 single-module batch is the natural first invocation.
- COL-306 (QA, PR #261 awaiting Jignesh review) + COL-205 batch-1 fan-out unchanged on the critical path — this session hardened the *tooling*, not the migration itself.

## See also
- [[../../strategy/active-plan|active-plan]] — updated this session (Part 1 + Part 2 ships + CS-393)
- [[2026-06-01T0226-COL-306|prior bridge — COL-306 BootsOnGround migration (the lessons' source)]]
- [[../../decisions/adr-037-cs368-migration-substrate-retirement|ADR-037]] — why migration is operator-invoked, not pipeline-wired (the CS-393 finding's basis)
- [[../../decisions/adr-035-cs357-migration-lobe-read-first-body-analysis|ADR-035]] — the MOVE/KEEP rule the worker bakes verbatim (untouched this session)
- `agents/global/migration-worker.md` — Part 1 target (all migration craft)
- `.claude/skills/cibo-migrate/SKILL.md` — Part 2 new skill · `skills/global/dotnet-to-node-migration/SKILL.md` — deprecated predecessor
- [COL-306](https://linear.app/colobbo/issue/COL-306) · [CS-393](https://linear.app/colobbo/issue/CS-393) · [api_node PR #261](https://github.com/Colobbo/api_node/pull/261)
