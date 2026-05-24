---
title: "migration-worker wiring update — narrow Linear writes (table cells + comments via split rule) + SP-body single-path lock + separable §Project wiring block"
tags: [bridge, session, adhoc, migration-worker, linear-writes-narrow, sp-path-lock, project-block-separation, table-write-safety, regenerate-and-verify, plan-mode-2-rounds, operator-caught-stale-text, deploy-sh-additive-no-prune, macos-space2-gremlin-recurrence, cs-367-context, cs-368-context, col-205-runway]
last_updated: 2026-05-24
source: memory
confidence: high
lobe_version: 1.0.0
status: active
---

# migration-worker wiring update — Linear writes (narrow split-by-content-shape) + SP-body single-path lock + separable §Project wiring block

## Summary

Wiring-only refinement ship for the throwaway migration-worker agent (shipped 2026-05-23 commit `2077ca145`, ratified by CS-368/ADR-037 as the live `.NET → Node` execution surface through 2026-07-31). Three additive deltas per operator-approved plan at `/Users/vipingera/.claude/plans/task-update-agents-global-migration-work-sharded-curry.md`:

1. **LOCK SP-body + MSSQL schema reads** to a single authoritative source at `Colobbo-Web/db/schema.sql` + `Colobbo-Web/db/procedures/<SpName>.sql`. Drop the deprecated `codebase-maps/db/` cache + broad-Colobbo-Web grep fallbacks. Emit `SP-DUMP-MISSING` + STOP for the batch if the dump is absent at preflight (currently the case — dump landing is a separate operator-owned external dependency).
2. **ADD narrow Linear MCP write surface** with a clean split rule. Worker writes short structured values (M/K verdict, Node Route, PR Link, Status) to the row's table cells in the weekly tracker sub-issue's description via `save_issue` + a regenerate-and-verify safety algorithm (row-count unchanged pre/post; on mismatch → `TABLE-WRITE-UNSAFE` + fall back to comment). Worker writes paragraphs (notes-out, why-safe rationales) + clarification questions as `save_comment` on the same ticket with `list_comments` idempotency. Never edits human-curated cells (`Endpoint` / `Notes-Jignesh-BE` / `Notes-Jaydeep-FE`). Never creates issues. Never touches labels (no status labels — Status is a table cell humans-or-worker can write).
3. **ENCODE all Colobbo-specific wiring** in a single `§Project wiring` block at the top of the file (12 subsections + 1 algorithm subsection `§13 — Table-write safety algorithm`), structurally separable from the MOVE/KEEP methodology. Convention 5 hollowed; methodology references `§Project wiring §N` instead of inline paths. Defensive 3-layer hygiene for a throwaway agent — if revived as a real Lobe post-COL-205, this block lifts out into the LOBE's glaze and the methodology lifts in clean.

Conventions 1-4 verbatim bakes (migration policy, ADR-035 §1.D2, `assertCompanyAccess`, api_node MOVE/KEEP idioms) byte-identical to pre-edit per the build-time fidelity-diff discipline.

Plan-mode session: 2 rounds of clarifying questions + 1 mid-plan structural correction by operator (Linear tracker shape: NOT sub-issue-per-endpoint as CC initially misread, but one-sub-issue-per-WEEK with a markdown table of ~25 endpoint rows — the existing COL-305 rulebook parent + COL-306 weekly-table layout). Executed Phase 1 verification → edit (22 surgical changes) → 8-check verification suite → 2 operator-caught corrections (Operating-rules stale "Read-only Linear" bullet + frontmatter description "read-only Linear (input-source-only)" — both half-true-now-stale after the Linear-write surface addition) → commit + amend (SS→§ glyph correction) + push + deploy + 12-file cleanup of `~/.claude/agents/` (4 CS-368-retired agents + 2 macOS `" 2"` gremlins + their .bak files).

## What shipped

**1 commit + 1 push + 1 deploy + 12-file user-global cleanup.** On `origin/main`.

### Commit `e1fce13e6` — feat(migration-worker): Linear writes + SP path lock + project wiring

1 file changed, **+399 / -101** in `agents/global/migration-worker.md` (732 → 1030 lines). Detailed change list in §Files changed below.

Commit originally pushed as `890a541a8` with "SS" placeholders in the body (over-cautious shell-escaping for `§` glyphs). Operator caught the cosmetic flaw before push; CC amended in place to `e1fce13e6` (safe — unpushed local commit; force-push avoided by using `--amend` not `--force`).

### `bash deploy.sh` — 129 deployed, 97 backed up, 0 skipped

Standard re-sync across `~/.claude/` + 4 product repos. Post-deploy `diff ~/.claude/agents/migration-worker.md agents/global/migration-worker.md` → empty (byte-identical with source).

### `~/.claude/agents/` cleanup — 12 files removed (no repo commit; user-global path)

Post-deploy operator-detected drift: 44 .md files in `~/.claude/agents/` vs 38 in source (deploy.sh additive — never prunes deleted-source files). Investigation surfaced 6 deployed-but-not-source extras:

- **4 CS-368-retired agents** (deleted from source `agents/global/` on 2026-05-23 by ec606ba8b but still in deployed dir): `migration.md`, `migration-lobe-codegen.md`, `regression-lobe.md`, `regression-lobe-codegen.md`.
- **2 macOS iCloud `" 2"` gremlins** (sync-corrupted duplicates per the prior `feedback_macos_space2_duplicate_corrupts_git_refs` auto-memory entry, both dated 19 May 00:01, same size as live versions): `backend-lobe-codegen 2.md`, `doc-writer-lobe-codegen 2.md`.

Operator approved deletion of all 6 .md files + the 6 corresponding `.md.bak` rotation backups from this morning's `deploy.sh` run. Total 12 files removed. Final state: `~/.claude/agents/` has 38 .md (parity with source) + 38 .md.bak (live-agent backups from today's deploy) + `memory/` directory.

## Decisions resolved this session

### Plan-mode operator-approved scope (3 additive deltas)

Plan refined across 2 rounds of clarifying questions (Q1-Q3 then Q4-Q6 after a structural correction). Locked design captured in 660-line plan file at `/Users/vipingera/.claude/plans/task-update-agents-global-migration-work-sharded-curry.md` (not in repo working tree). Three deltas all wiring-only — no methodology change; no verbatim-bake modifications.

### Mid-plan variance: Linear tracker structure misread (R1 → R2 correction by operator)

Operator's first Q2 answer ("Neither exactly. ... switch to sub-issue-per-endpoint") was misread by CC as a restructure proposal. CC drafted sub-issue-per-endpoint design + asked Q4 (endpoint→sub-issue mapping) + Q5 (sub-issue creation responsibility) + Q6 (status labels). Operator REVISED: *"Final structure — one sub-issue PER WEEK with a markdown table of ~25 endpoints (COL-305 rulebook + COL-306 'Week 1' table — already built, don't restructure to per-endpoint sub-issues). ... Worker may WRITE the weekly ticket: update its row in the table ... AND/OR post comments ... Single operator edits the table — no concurrent-writer race."*

CC re-planned to the comment+table-cell-with-split-rule design. The TABLE-WRITE-UNSAFE / TABLE-NOT-FOUND / TRACKER-ROW-MISSING / CELL-CONTENT-UNSAFE / LINEAR-WRITE-DEGRADED safety token set + the §13 regenerate-and-verify algorithm emerged from this revised design. Q5 + Q6 turned into "no sub-issue creates ever" + "no status labels at all" (operator confirmed implicitly via the revised structure — humans + worker both write the Status cell directly; no separate label surface).

Lesson: when an operator answer starts "Neither exactly" + describes a revision, do NOT lock the revision into the plan via downstream clarifying questions until the structural shape is confirmed. The Q4-Q6 round was wasted thought-work on a shape that wasn't going to be built.

### Verification check #1 — 6 inline-token leaks accepted as-is

8-check verification suite ran post-edit. Check #1 (§Project wiring sole source of Colobbo tokens) surfaced 6 inline-token mentions outside §Project block + Convention 3/4 verbatim bakes:

| Line | Context | Token |
|---|---|---|
| 13 | Purpose paragraph 1 | "api_node integration branch" |
| 15 | Purpose paragraph 2 | "the api_node codegen idioms" |
| 757 | Step 4h (end-of-paragraph parenthetical) | "(Convention 4 api_node convention)" |
| 856 | Step 7b self-review checklist | "the api_node block-console-logs hook will fail otherwise" |
| 1028 | §Convention re-bake trigger (pre-existing; not touched by this edit) | bake source-file mentions (`api_node` glaze path + `riskThresholdRoutes.ts`) |
| 1030 | §Convention re-bake trigger (CC-added sentence) | "when COL-305 status vocabulary..." |

Operator-accepted as-is per the morning-review brief at `/tmp/morning-review-brief.md`. None are load-bearing wiring; all are contextual mentions. Load-bearing wiring (SP source, Linear writes, table schema, status vocab, PR target, deprecated list, module index, Tier-1 list) is correctly inside §Project block. Future Lobe-extraction effort (if worker is ever revived) needs the 5 CC-introduced mentions reworded to §N references for strict 3-layer hygiene.

### Operator-caught stale-text class — 3 Linear-related phrases missed by CC

When rewriting Boundary 13 (the explicit "Linear write" boundary) for the new narrow-write surface, CC missed three related stale phrases that survived the original edit pass:

1. **Line 1008 — `§Operating rules` bullet** (operator caught from diff visual): *"Read-only Linear. Never call save_issue / save_comment / save_document."* — directly contradicted the whole point of the edit. Fixed in place to: *"Narrow Linear writes. Worker reads via `get_issue` + `list_comments`. Worker writes ONLY the weekly tracker sub-issue: table cells via `save_issue` (§13 safety pattern) + comments via `save_comment` ... NEVER `save_document`. ..."* (full Boundary-13-mirroring text).
2. **Line 4 — frontmatter `description` field** (CC grep-caught during the line-1008 fix loop): *"... + read-only Linear (input-source-only)."* — registry-facing surface; would have shown the wrong description in any agent enumeration. Fixed to operator-chosen tighter rewrite: *"... + narrow Linear MCP (reads weekly tracker; writes row cells + comments on the same ticket)."*
3. **Line 1002 — `§Operating rules` Zero-assumptions bullet** (CC flagged; operator instructed fix): *"... + grep + filesystem + read-only Linear."* (half-true after the write surface addition). Fixed to: *"... + grep + filesystem + Linear tracker notes (read per §Project wiring §6)."*

Lesson: when editing one occurrence of a capability description (e.g. shifting "Read-only Linear" → "Narrow Linear writes"), grep the ENTIRE file for ALL related phrasings — including frontmatter `description`, other bullet lists, related boundary items — BEFORE assuming the rewrite is complete. Similar to the `feedback_design_lock_stale_text_grep` auto-memory entry but applied to capability descriptions, not design locks.

### Self-caught duplicate Boundary item at mid-edit

When rewriting Boundary 13 + adding new boundary items, CC accidentally introduced a phantom duplicate Tier-1 line at item 17 (planned count was wrong — original file had 16 items, plan-text incorrectly named the new items 18+19 instead of 17+18). CC noticed immediately + fixed mid-flow (this is "editing error" class, not "verification failure" class — fix-before-continue applies); items renumbered to 17 (SP path lock) + 18 (table-write safety). Operator's "no self-fix on verification failures" rule was honoured (this was caught during edit-step, not during verification-step).

### Cosmetic post-commit fix: SS → § glyph (commit amended pre-push)

Original commit message body substituted `SS` placeholders for `§` symbols in 3 spots (paragraph 3 + final paragraph) — over-cautious shell-escaping for the section symbol in a HEREDOC. Operator caught; CC amended commit (safe — unpushed local). Commit moved `890a541a8` → `e1fce13e6` before push. Force-push avoided per `feedback_force_push_regex_spans_compound_command.md` (used `git commit --amend` on an unpushed local commit; no `--force` in any command string).

Lesson: bash HEREDOC handles UTF-8 (§, em-dash, etc.) fine; preemptive substitution is over-cautious and ugly in archaeology. Only escape `$`, `` ` ``, `\` in HEREDOC bodies — leave Unicode glyphs alone.

## Pre-flight + verification gates

### Phase 1 verification (read-and-report BEFORE any edit, per operator instruction)

Direct answers to operator's 4 verification questions documented in chat + plan file:

- **1a (SP bodies?):** YES — Step 3b (classification) + §Parity STATIC Step 1 (parity column comparison). Body-content drives MOVE-vs-KEEP signals; path-lock is load-bearing on both stages.
- **1b (current SP paths):** quoted from Step 3b L422-424 + §Parity STATIC L219 — two-source lookup pre-edit (`codebase-maps/db/` cache + `Colobbo-Web` general grep). Step 4b L450-452 declares target Drizzle schema paths (separate concern, co-mentioned).
- **1c (Linear today?):** YES, read-only narrow scope. Pre-edit `tools:` had `mcp__linear__list_issues` + `mcp__linear__get_issue` only. Used in invocation mode 3 + Step 0c to parse ticket body for endpoint URLs. No per-endpoint reads, no writeback, no clarification posts. Boundary 13 pre-edit stated *"Does NOT write to Linear."*
- **1d (caller discovery today?):** Already Graphify+grep, repo-fanout — exactly the shape operator wanted reaffirmed. Step 0a probes per-repo `graphs/<repo>/graphify-out/graph.json`; Step 2a walks inbound call/reference edges; Step 2b always-runs grep verify for HTML templates + dynamic-access. Graphify is TS-only — T-SQL is never indexed.

Plus 6 supplementary filesystem findings via Explore subagent:
- ⚠ `~/Desktop/Colobbo/repos/Colobbo-Web/db/` is **ABSENT** on local + `origin/dev` — the "fresh dump" the user wants locked doesn't exist yet. Operator confirmed **lock the path anyway** — worker emits `SP-DUMP-MISSING` + STOP if absent at invocation.
- `codebase-maps/db/` is **already absent** — removing the fallback is dead-text cleanup.
- **Linear MCP write tools available** in this CC env: `mcp__linear__save_issue`, `mcp__linear__save_comment`, `mcp__linear__list_comments`. All three used.
- Deployed copy `~/.claude/agents/migration-worker.md` byte-identical to source pre-edit.
- `glaze.migration.*` already absent per ADR-037. Worker stays glaze-independent.
- All 5 Graphify graphs present (api_node, web-angular, Colobbo-Mobile, Colobbo-Web, cibo).

### Verification suite — 8 checks per plan

| # | Check | Result |
|---|---|---|
| 1 | §Project wiring is sole source of Colobbo tokens | ⚠ 6 leaks (operator-accepted; see §Decisions above) |
| 2 | SP-body single-source (deprecated tokens only in deprecation declarations) | ✅ PASS |
| 3 | Graphify-for-TS-only guard present | ✅ PASS |
| 4 | Linear tools: `save_issue` + `save_comment` + `list_comments`; no `create_issue_label` / `list_issue_labels` | ✅ PASS |
| 5 | Table-write safety tokens (5 tokens × multiple sites) | ✅ PASS |
| 6 | 4 verbatim bakes byte-identical pre/post | ✅ PASS (4/4) |
| 7 | Only `agents/global/migration-worker.md` modified by this work | ✅ PASS |
| 8 | Diff captured for operator review | ✅ PASS |

### Post-deploy verification

- `diff ~/.claude/agents/migration-worker.md agents/global/migration-worker.md` → empty (byte-identical).
- `grep -E 'mcp__linear__(save_issue|save_comment|list_comments)'` on deployed copy → all 3 tools confirmed present.
- `~/.claude/agents/` cleanup: post-delete `comm -23` + `comm -13` both empty (full parity with source `agents/global/`).

## What's next

### Critical path (largely unchanged; only #2 enhanced by this session)

1. **Jignesh's decision on CS-367** (due 2026-05-28; today is 2026-05-24 — 4 days remaining). UNCHANGED. CC recommendation: Option A native-where-possible + COL-296 JOIN fix.
2. **migration-worker live smoke — ENHANCED this session.** Now has narrow Linear awareness (read tracker notes + write row cells + post comments) + single-source SP path lock + separable §Project wiring block. Still parallel-ready post-CS-367 + first-batch endpoint pick + `Colobbo-Web/db/` dump landing. Worker's first preflight on a real endpoint will surface `SP-DUMP-MISSING` until the dump lands. Operator confirms parity mode (STATIC default) + first-batch endpoint at invocation time.
3. **PR #246 conflict resolution** — depends on CS-367. UNCHANGED.
4. **COL-205 batch-1** (~2026-05-29 kickoff per prior daily-plan reference). Depends on (1) + (2) + dump landing.
5. **CS-322 headless CC pipeline orchestration** — wraps BUILD chain only post-ADR-037. Lower priority since migration-worker covers per-endpoint runway.

### Followups filed this session

NONE in Linear. This was a scoped wiring update per the operator-approved plan; no tickets created per the plan's explicit Out-of-scope item (*"no agent-system VERSION bump, no tag, no Linear ticket required"*).

### Followups for operator (NOT executed this session)

1. **`Colobbo-Web/db/` dump landing** — operator-owned external dependency. Worker emits `SP-DUMP-MISSING` + STOP until landed. Migration runway blocks at SP-source read for any endpoint requiring SP-body classification (most KEEP candidates).
2. **6 inline-token leak rewording** — IF the worker is ever revived as a real Lobe post-COL-205, the 5 CC-introduced contextual mentions (lines 13, 15, 757, 856, 1030) + 1 pre-existing bake-provenance mention (line 1028) need rewording to use `§Project wiring §N` references for strict 3-layer hygiene. Operator-accepted as-is for the throwaway lifecycle.
3. **`deploy.sh` additive-only cleanup pattern** — 4 CS-368-retired agents lingered in `~/.claude/agents/` for 2 days post-retirement because deploy.sh never prunes. Future substrate retirements should include an explicit `rm ~/.claude/agents/<retired>.md` step in the ship plan, OR a `deploy.sh --detect-stale` enhancement (flag-only, not auto-prune — too risky). File as CS follow-up if pattern recurs.

### Bridge-entry watch-points (NO TICKET SPAWN)

1. **Plan file location:** `/Users/vipingera/.claude/plans/task-update-agents-global-migration-work-sharded-curry.md` (NOT in repo working tree — in user-global `~/.claude/plans/`). Survives session but isn't versioned. Reference for archaeology only; do not treat as canonical.
2. **macOS `" 2"` gremlin pattern confirmed (again).** 2 instances at `~/.claude/agents/` this session. The `feedback_macos_space2_duplicate_corrupts_git_refs` auto-memory entry remains accurate. Pattern: file with literal " 2" before extension, same size as live version, created during iCloud/Finder sync events. Safe to delete on inspection.
3. **deploy.sh additive behavior** established as known limitation; no fix planned this session.
4. **MEMORY.md regenerator-bump committed in this handoff.** `M MEMORY.md` was carried over from CS-368's wiki/strategy/ updates (raku-roadmap.md + migration-runway.md content delta caught by `/cibo-start-session` Step 0 regenerator earlier today). Committed alongside this handoff to lock MEMORY.md state aligned with wiki/strategy/ committed state.

## Files changed

### This session — 1 commit + Step-1a active-plan + Step-2 daily-log + Step-4 canonical_pages

**Commit `e1fce13e6` (the migration-worker wiring update):** 1 file changed, +399 / -101.

- MODIFIED: `agents/global/migration-worker.md` (732 → 1030 lines, +298 net)
  - Frontmatter `description` field (line 4) — Linear surface description corrected (CC original miss; operator + CC grep caught during fix-loop)
  - Frontmatter `tools:` line (line 5) — added `mcp__linear__save_issue`, `mcp__linear__save_comment`, `mcp__linear__list_comments`
  - NEW §Project wiring section (lines 19-237) — 13 subsections: §1 legacy source repo, §2 modern target repo, §3 consumer repos, §4 SP-source single-authoritative-path lock, §5 caller-discovery source (Graphify TS-only guard), §6 Linear tracker layout (read+write split rule + table-column schema), §7 COL-305 status vocabulary, §8 PR target + reviewer, §9 operating model (read-vs-produce + deploy-then-web gating + mobile-later), §10 deprecated-list path, §11 module-index path, §12 Tier-1 module list, §13 table-write safety algorithm (regenerate-and-verify with 5 abort/warn tokens)
  - Boundaries: §13 rewritten (Linear scope narrow-by-design); NEW §17 (SP-source single-path lock); NEW §18 (table-write safety constraint)
  - Invocation mode 3 reworded — Linear weekly tracker ticket (sub-issue of rulebook, NOT rulebook parent itself)
  - Convention 5 hollowed — content moved to §Project wiring §10/§11/§12; convention becomes single-line redirect
  - §Methodology surgical rewrites: Steps 0a (Graphify availability → §3+§5), 0c (INPUT resolution — major rewrite for Linear-driven mode), 0d (middleware/utils paths → §2), 0g (Tier-1 gate → §11+§12), NEW 0h (Linear write probe), 1a/1c (deprecated list → §10; api_node check → §2), 2b (consumer repos → §3), 3a (legacy method paths → §1), NEW 3a.1 (Status=migrating writeback via §13), 3b (SP source — LOAD-BEARING REWRITE: deprecated 2-source fallback removed; reads only `<§4 procedures path>/<SpName>.sql`), NEW 3e (clarification escalation — Status=blocked write + comment post + STOP), §Parity STATIC Step 1 (SP source — REWRITTEN: drop `codebase-maps/db/` fallback), 4a/b/c/d (paths → §2), 5a (test framework path → §2), 8e (PR creation flags → §8), NEW 9b (Linear writeback: table cells + comment in one step, per §13)
  - §Operating rules: rewrote stale "Read-only Linear" bullet → "Narrow Linear writes" (Boundary-13-mirror); added "Project wiring separation is load-bearing" bullet; fixed "Zero assumptions" Linear sub-mention
  - §Conditional behaviour summary: added 2 rows (`linear notes inline:` for offline/test mode, `linear writeback: off` for force-fallback mode)
  - §Convention re-bake trigger: added sentence about `§Project wiring` being hand-authored (no fidelity-diff)

### Step 1a — active-plan.md update

Updated `wiki/strategy/active-plan.md` `last_updated → 2026-05-24` + critical-path #2 expanded with new worker capabilities (Linear awareness + SP path lock + project block separation) + Colobbo-Web/db/ dump dependency surfaced as new structural blocker. Recent corrections rolling-window updated with this session's 5 corrections (Linear-tracker-structure R1→R2 misread, Operating-rules stale bullet, frontmatter description stale phrase, Zero-assumptions Linear sub-mention, SS-for-§ over-cautious shell escape). Last session delta paragraph replaced.

### Step 2 — daily log append

`memory/2026-05-24.md` appended with `## Session 0030-adhoc-migration-worker-wiring` section pointing at this bridge entry.

### Step 4 — canonical_pages registration

`glaze.wiki.canonical_pages` extended with one entry for this bridge file (`bridge/sessions/2026-05-24T0030-adhoc-migration-worker-wiring.md`) + topic keywords (migration-worker, narrow-linear-writes, sp-path-lock, project-wiring-block, table-write-safety, regenerate-and-verify, plan-mode, operator-caught-stale-text, deploy-sh-additive, macos-space2-gremlin).

### MEMORY.md (carry-over from prior session)

`MEMORY.md` regenerator-bump from `/cibo-start-session` Step 0 earlier today — wiki/strategy/raku-roadmap.md + migration-runway.md content delta from CS-368 ship caught by regenerator. Committed in this handoff to lock MEMORY.md aligned with committed wiki/strategy/ state.

### Cleanup (user-global `~/.claude/agents/` — no commit, no repo touch)

12 files removed (6 stale .md + 6 corresponding .md.bak):
- `migration.md` + `migration.md.bak` (CS-368-retired)
- `migration-lobe-codegen.md` + `.md.bak` (CS-368-retired)
- `regression-lobe.md` + `.md.bak` (CS-368-retired)
- `regression-lobe-codegen.md` + `.md.bak` (CS-368-retired)
- `"backend-lobe-codegen 2.md"` + `.md.bak` (macOS gremlin)
- `"doc-writer-lobe-codegen 2.md"` + `.md.bak` (macOS gremlin)

Final `~/.claude/agents/` state: 38 .md (parity with source) + 38 .md.bak (live-agent backups from today's deploy) + `memory/` directory.

## Pointers

- **Commit**: `e1fce13e6` on `origin/main` (was `890a541a8` pre-amend; commit amended pre-push to fix SS→§ glyphs)
- **Plan file** (operator-approved, not in repo): `/Users/vipingera/.claude/plans/task-update-agents-global-migration-work-sharded-curry.md`
- **Morning-review brief** (pre-commit verification surfacing): `/tmp/morning-review-brief.md`
- **Pre-edit snapshot**: `/tmp/migration-worker.pre-edit.md` (md5 `fd070a6628d38c019f0fcbbbb74b959c`)
- **Final diff capture**: `/tmp/migration-worker.diff` (666 lines)
- **Bake byte-diff fixtures** (verification check #6): `/tmp/{pre,post}.{adr035,aca,move,keep}.txt`
- **Prior bridge** (substrate retirement context): [[2026-05-23T0940-CS-368|2026-05-23T0940-CS-368]]
- **Earlier bridge** (original migration-worker ship): [[2026-05-23T0050-CS-367|2026-05-23T0050-CS-367]]
- **Related ADRs**: [[../../decisions/adr-037-cs368-migration-substrate-retirement|ADR-037]] (substrate retirement → worker is live execution surface), [[../../decisions/adr-035-cs357-migration-lobe-read-first-body-analysis|ADR-035]] (body-analysis MOVE/KEEP — partial supersession; Convention 2 verbatim bake), [[../../decisions/adr-013-2026-agent-memory-pattern|ADR-013]] (this bridge contract), [[../../decisions/adr-034-cs353-plan-state-persistence-boundary|ADR-034]] (Step 1a ownership)
- **Related Linear**: CS-367 (api_node convention fork — gating worker first smoke), CS-368 (substrate retirement context), COL-205 (migration runway)

## Memory entries to save this session

1. **`feedback_grep_related_phrasings_for_capability_edits.md`** — when editing one occurrence of a capability description (e.g. "Read-only Linear" → "Narrow Linear writes"), grep entire file for ALL related phrasings (frontmatter `description`, other bullet lists, related boundary items, conditional behaviour tables) BEFORE assuming rewrite is complete. Same fix-shape as the prior `feedback_design_lock_stale_text_grep` auto-memory entry but applied to capability descriptions. Surfacing evidence: this session — 3 stale "read-only Linear" mentions (lines 4, 1002, 1008) survived CC's original edit pass; operator caught 1, CC grep-caught 1 during fix-loop, operator-instructed-fix 1.
2. **`feedback_heredoc_handles_utf8_fine.md`** — bash HEREDOC handles UTF-8 glyphs (§, em-dash, ✓, etc.) without special escaping; preemptive substitution (SS for §) is over-cautious and ugly in commit archaeology. Only escape `$`, `` ` ``, `\` in HEREDOC bodies — leave Unicode alone. Surfacing evidence: this session — first commit had "SS" placeholders for §; amended in place to use proper glyphs.
3. **`feedback_deploy_sh_additive_no_prune.md`** — `deploy.sh` is additive-only; never prunes files from `~/.claude/agents/` that exist deployed but not in source. Substrate retirements that delete source files leave deployed copies dangling indefinitely until manual cleanup. Add explicit `rm ~/.claude/agents/<retired>.md` step to substrate-retirement ship plans, OR file `deploy.sh --detect-stale` enhancement (flag-only, not auto-prune — too risky for user-global writes). Surfacing evidence: this session — 4 CS-368-retired agents lingered for 2 days post-retirement until operator-noticed during post-deploy verification.
4. **`feedback_operator_revision_neither_exactly.md`** — when an operator answer starts "Neither exactly" + describes a structural revision, do NOT lock the revision into the plan via downstream clarifying questions until the structural shape is confirmed. The wasted-thought-work cost of asking Q4/Q5/Q6 on the wrong shape was ~1 round of plan-mode iteration. Right move: ask "is this the final shape?" or just propose the new design + confirm-back before asking detail-level questions on it. Surfacing evidence: this session — Q4 (endpoint mapping) + Q5 (sub-issue creation) + Q6 (status labels) all became moot when operator's R2 answer revealed "Don't restructure to per-endpoint sub-issues."

(Memory entries are saved separately to the auto-memory layer outside this repo — bridge entry just records what's worth saving.)

## See also

- [[../../strategy/active-plan|active-plan]] — Step 1a-updated this handoff (critical path #2 enhanced with new worker capabilities)
- [[../../decisions/adr-013-2026-agent-memory-pattern|ADR-013]] — H2 resolution this skill implements
- [[../../decisions/adr-034-cs353-plan-state-persistence-boundary|ADR-034]] — Step 1a ownership boundary
- [[../../decisions/adr-037-cs368-migration-substrate-retirement|ADR-037]] — substrate retirement that makes the worker the live execution surface
- [[2026-05-23T0940-CS-368|2026-05-23T0940-CS-368]] — prior bridge (substrate retirement context)
- [[2026-05-23T0050-CS-367|2026-05-23T0050-CS-367]] — original migration-worker ship bridge
- `agents/global/migration-worker.md` — the agent updated this session
