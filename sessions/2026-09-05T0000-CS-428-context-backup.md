---
title: "Tier B replacement shipped; COL-682 to PR; seven defects, none in the capability"
tags: [bridge, session, cs-428, cs-425, cs-424, col-682, col-707, brownfield, replacement, gates, context-backup]
last_updated: 2026-09-05
source: memory
confidence: high
linked_issue: CS-428
lobe_version: 1.0.0
status: active
---

## Why this entry exists

Written mid-session as a context backup before compaction, not at `/cibo-handoff`.
It is a continuation brief: enough to resume without re-deriving, and explicit
about what is NOT done.

## The headline

**A brownfield ticket went from Linear to a reviewable PR changing existing code,
with no human writing the change** — [api_node#657](https://github.com/Colobbo/api_node/pull/657),
COL-682, ready for @jthakkar1981.

Read it precisely: **it shipped via INSERTION, not the Tier B replacement this
session was built around.** The codegen emitted `{"insertions": 1, "replacements": 0}`
and routed around the missing capability with `users.splice(0, users.length, ...active)`
— a rebinding expressed as a mutation, so the untouched `.map()` below sees the
filtered set. Functionally correct, all gates green, and **worse code than the
same pipeline produced on runs 5 and 6**, which were blocked for unrelated wiring.

Additive-only did not block the work. It **degraded** it. All four Tier B
guarantees passed and should have — the code is correct. The guarantees are
structural; none can ask "is this what a careful engineer would write". That is
the gap, and widening the guarantees will not close it — it is an argument for
CS-426's ensemble, not another mechanical check.

## Shipped this session

| Thing | Where |
| -- | -- |
| Tier B replacement (G1 confinement, G2 parse via target repo's own tsc, G3 caps, G4 bidirectional-by-count denylist) | `scripts/apply-safe-modify.mjs`, v2.67.0 |
| `replace` opened in `competent_on` | on a 10/10 case against a bar recorded blind at `7b4d719ebd` |
| Regression evidence (stand-in for the absent CS-332) | `scripts/regression-check.mjs`, wired blocking |
| Differential probe (comparative, works on guarded/DB endpoints) | `scripts/differential-probe.mjs` |
| Competence enforcement + 3 bare codegens now refuse | `scripts/check-competence.mjs` (CS-424 item 3) |
| Rebind-as-mutation **advisory** (not a gate — see below) | `apply-safe-modify.mjs` |
| Gate 25 sees silently-skipped assertions AND skipped blocks | `scripts/run-test-suites.sh` |
| Gate 28 verdict capture | `scripts/check-verdict-capture.sh` |
| Gate 29 portability (ambient vs scrubbed comparison) | `scripts/verify-portable.sh` + pre-tag step 6 |
| Emit-collision guard | `scripts/check-emit-collisions.mjs` |
| Graphify: 106-day no-op fixed, 4 graphs rebuilt | v2.65.0 |
| Auth canary (asserts OAuth, not reports) | CS-427, Done |

**Tags: v2.64.0 → v2.68.0. 19 commits since v2.68.0 — a tag is owed.**
Highest gate number in use: **29**.

## Open — item 2, not started

The operator's brief, verbatim in intent:

**2a. Run aggregator — BUILD FIRST.** Over `run.json` across all runs:
runs-to-PR per ticket, which gate failed, defect class. **Plus escape-hatch
usage per run** — `ALLOW_TEST_SKIPS`, `ALLOW_NON_ADDITIVE`, any force path.
Each hatch is fine once; nobody can currently see accumulation, and "used every
run" means it has stopped being an exception. Must exist **before the next
ticket runs or the measurement is lost**. That number decides whether this
capability was worth building. Recommend where it lives and what it records.

*Relevant: I used `ALLOW_NON_ADDITIVE` once today, defensibly, and nothing would
show that to the next session. That is the case for 2a in miniature.*

**2b. Audit every `FAIL:` message** for whether it names what was actually
checked. Two known liars from this session: `"git push failed (token lacks push
rights, or branch protection)"` (it was a branch collision the check above should
have caught) and the typescript-resolution message (true, but the real fault was
dependency ordering). Report which still lie.

**2c. Should a failed run file its own record** so the next session opens with a
diagnosis rather than pasted logs? Recommend the surface — Linear, #colobbo-radar,
or filesystem — with reasoning.

**DO NOT build auto-diagnose-and-repair.** Operator-stated, and I agreed and
would hold it harder: six of seven defects were checks that looked right and
found nothing, and **three of those I wrote myself today**. An agent repairing
its own verification machinery applies exactly the judgement that produced them,
with no reader. Memory before autonomy.

## The seven defects — none in the replacement logic

1. Planner grounding — the runner knew the target repo and never told the Planner
2. Dependency ordering — `npm ci` scheduled after the applier, so G2 could never resolve a compiler
3. Emission bookkeeping — `assemble-pr` counted `files_generated[]` only, so a replacement was invisible
4. **Mine:** vacuous differential — full URL where the harness wanted a path; five empty captures compared as "unchanged"
5. Glaze key convention — `apiNode` (key) vs `api_node` (name); I then "fixed" the wrong map and broke `runtime-verify`
6. Branch-collision check read a LOCAL tracking ref for a REMOTE fact
7. **Mine:** `no()` helper undefined — the assertion that mattered never ran and the suite reported 101 passed

## COL-682 review findings (all in #657, all found in review, none by the pipeline)

- Invite-pending users appearing — the reported defect
- **Federated (`EXTERNAL_PROVIDER`) users silently excluded** by the old CONFIRMED-only rule. Fail-open fixed this **by accident**, recorded as luck not foresight
- **Deleted (`User not found`) users appearing** — introduced by my fail-open rule; `'Error'` (no answer) vs `'User not found'` (Cognito said no) are opposites
- Cognito outage would have emptied the picker — `getCognitoUserStatuses` never throws, returns `{}`
- **My outage tests were fake**: `makeCognitoStatuses({})` throws `UserNotFoundException` per email, so it modelled *every user deleted*. Fixed with `makeCognitoUnreachable()`
- Test file promoted out of `generated/` — the generator has **no** overwrite guard; `output_exists` on one run was model judgement, not contract

**COL-707 filed**: disabled accounts (`Enabled:false`) still appear — `'DISABLED'` is unreachable in `getCognitoUserStatuses`. Predates COL-682. Four consumers listed.

## Decisions that must not be re-litigated

- **Bar for `replace` set blind** at `7b4d719ebd` before any score: `differential_behaviour` + `no_unactioned_skips` + `regression_survived` at max, replacing `runtime_behaviour` (unanswerable on DB-backed paths). One out, two in — a net tightening.
- **Rebind-as-mutation is an ADVISORY, not a gate.** The broad signature flags `app.use(...)`, the canonical proven case. The narrow one is evadable. A gate that is trivially evaded gives false assurance.
- **`feature_id` namespacing removes cross-ticket collisions for tracker-shaped ids only** — it is charset-validated, not uniqueness-enforced. `check-emit-collisions.mjs` is what actually holds.
- Eval row for `replace` is **builder-authored and builder-scored, `ensemble: false`**. It proves the machinery, not that a model writes good replacement plans. CS-426 remains the fix.

## Gotchas that cost time

- `agents/global/*.md` are **compiled**; edit `lobes/*/LOBE.md` and run `compose.sh deploy --all`
- Glaze changes drift fixture snapshots → `resnap-additive-fixtures.sh`, which now **classifies and refuses** non-additive drift (it caught my own wrong "additive-only" claim)
- macOS bash is **3.2.57** — no `command_not_found_handle`
- Actions runs `run:` as `bash -e {0}`; the swallow risk is only inside a deliberate `set +e`
- Never symlink `node_modules` in a worktree (api_node #576/#583)
