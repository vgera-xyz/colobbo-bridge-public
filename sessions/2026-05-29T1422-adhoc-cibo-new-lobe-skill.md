---
title: "cibo-new-lobe skill shipped — investigation → design → test-run → main; thin orchestration over new-lobe.sh, 2 bugs caught by the dummy run"
tags: [bridge, session, adhoc, cibo-new-lobe, skill, lobe-scaffolder, new-lobe-sh, layer-0, adr-032, cs-348, test-run-caught-bugs, registry-regen-ordering, rm-rf-hook-block, parallel-session-origin-moved, commit-msg-hook-scan, no-ticket-adhoc]
last_updated: 2026-05-29
source: memory
confidence: high
lobe_version: 1.0.0
status: active
---

# cibo-new-lobe skill shipped (adhoc — no anchoring ticket)

## Summary

Built and shipped a new project-scoped skill, `cibo-new-lobe`, that scaffolds a compliant greenfield Lobe from a name + plain-English purpose. The skill is a **thin orchestration layer over `scripts/new-lobe.sh`** (CS-348) — it never reimplements the scaffold; its value-add is LLM-filling the scaffold's prose TODOs from the purpose, a Lobe-vs-freeform routing guard, structural + 3-layer-portability validation, and a plain-English review card (approve/revise/abort). One commit (`209a59d84`), pushed direct to `main` (this private repo allows it).

The session ran in four operator-gated phases: (1) a read-only 7-point investigation, (2) a recommendation round on 3 design questions, (3) writing the SKILL.md against locked decisions, (4) a test-run on a dummy Lobe then abort. **The test-run caught two real bugs in the skill that reading could not** — the honest payoff of dogfooding before trusting it.

No Linear ticket anchors this work (closest: CS-348 `new-lobe.sh`, Done; CS-349 `--gated` smoke, Backlog — neither is "the cibo-new-lobe skill"). Recorded as an `adhoc-` bridge entry per `/cibo-handoff` Step 0; `linked_issue` omitted (no valid ticket — Gate 12 pattern-checks the field if present, so omission is correct, not `null`).

## What was done

- **7-point investigation (read-only).** Mapped existing cibo-* skills, the Lobe structural template (code-reviewer-lobe; regression-lobe is retired per ADR-037), the frontmatter validators (Gate 12 is wiki-only — Lobe frontmatter is gated by `verify-lobe-registry.sh` + `compose-lobes.mjs`, NOT `verify-frontmatter-schema.sh`), the registry (`lobes/_registry.json`, NOT `agents/global/_registry.json` which is absent), the Layer-0 `_shared/` fragments, the compose/deploy pipeline, and the Lobe-vs-freeform-agent naming fork. Key find: **`scripts/new-lobe.sh` already exists** as a full scaffolder — so the skill is a wrapper, not a from-scratch generator.
- **3 design decisions locked by operator:** (1) fragments = all 4 (the `new-lobe.sh` default; the "fleet norm of 1" is legacy-retrofit minimalism per `lobe-migration-manifest.json`, not the greenfield target — capability-primitive is effectively mandated for new Lobes by CLAUDE.md Hard Rule 11); (2) ungated on create (`--gated false`); (3) keep the `-lobe` suffix; plus hybrid input flow + a plain-English review card with no raw JSON/YAML.
- **Out of scope (locked):** no auto-generation of `output-schema.json` bodies / `test-fixtures/expected.json` / `validate-<lobe>.mjs` / `run.sh` (false-confidence risk — these encode semantic contract that must be hand-derived; named on a human checklist instead); no editing of existing Lobes.
- **Wrote** `.claude/skills/cibo-new-lobe/SKILL.md` (209 lines, 8-step flow + review-card template + abort-cleanup + scope boundaries).
- **Test-ran** it end-to-end on dummy `scratch-lobe` (scaffold → prose-fill → validate → abort), then patched the two bugs the run exposed.

## The two bugs the test-run caught (would not have been found by reading)

1. **Step 7 was missing a `registry-regen`.** The prose-fill in Step 6 changes the include `params:`, which changes each fragment's `param_hash` and the Lobe's `composite_signature`. `new-lobe.sh` only regenerated the registry at scaffold time (against the TODO params), so **Gate 19 failed on drift** after the fill. Fixed: Step 7 now runs `bash compose.sh registry-regen` before the Gate 19 check, with a load-bearing-ordering note. (Verified: hashes changed `862df1…`→`bd2290…` / `3c8fd7…`→`31a0d7…`; regen made Gate 19 pass.)
2. **The abort path used `rm -rf`, which the `block-dangerous-commands.py` hook blocks.** A real abort would have stalled mid-cleanup. Fixed: abort uses `rm -r` / `rm` (verified to pass the hook); `-f` is both blocked and unnecessary (Step 4 already proved the paths are this-run-only).

## Verification (all green; tree returned to before-state after abort)

- Scaffold: 4 files created; composed agent at `agents/global/scratch-lobe.md`; registry → 12 lobes with all-4-fragment composite signature.
- Prose-fill: `PROVISIONAL:` prefix correctly carried on the schema-derived params.
- Portability: compiled against BOTH wares (`colobbo` + `fintech`); 3-layer leak grep clean on each.
- Gate 19: green after `registry-regen`.
- Abort: `lobes/scratch-lobe` + `agents/global/scratch-lobe.md` removed; registry back to 11; Gate 19 green; `git status` identical to the captured before-state. No stray `LOBE.compiled.md`.

## Commits this session (1 mine)

- `209a59d84` — feat(skills): add cibo-new-lobe Lobe scaffolder skill (rebased onto the parallel-session origin/main `f9e80c285` graphify refresh, then fast-forward pushed to main — no force-push).

## Files changed

- `.claude/skills/cibo-new-lobe/SKILL.md` (NEW, 209 lines). Project-scoped; `deploy.sh` does NOT deploy `.claude/skills/` (auto-loaded when CWD is this repo, like the 9 sibling cibo-* skills).

## Environmental frictions handled (not chat-side prior errors)

- **origin/main advanced mid-session** (parallel graphify push `f9e80c285`); rebased onto it rather than force-pushing.
- **`block-dangerous-commands.py` fired on the commit MESSAGE** — the body literally contained `rm -rf` (describing the fixed bug); the hook scans the whole command string including the heredoc. Same class as the force-push-regex-spans-command gotcha. Reworded to avoid the literal.

## What's next

- **cibo-new-lobe is ready for first real use** — invoke `/cibo-new-lobe` when the next greenfield Lobe is authored (e.g. the Phase-2 Performance or Design Lobes from the CS-373 derivation). First real run is the true integration test of the prose-fill quality.
- **CS-349** (`new-lobe.sh --gated true` end-to-end smoke, Backlog) is now partially exercised — the ungated path is proven via this test-run; the gated path still unrun.
- Migration critical path + agent-system M5/M6 are unchanged by this session (parallel ergonomics work).

## See also

- [[../../strategy/active-plan|active-plan]] — cibo-new-lobe added as a parallel track
- [[2026-05-29T0504-CS-391-CS-390|prior session — migration-worker Pattern-A rebake + MSSQL-MOVE idiom]]
- [[../../decisions/adr-032-lobe-substrate-layering|ADR-032]] — Layer-0 fragments + registry + Gate 19 (the substrate this skill scaffolds against)
- [[../../decisions/adr-013-2026-agent-memory-pattern|ADR-013]] — handoff protocol this skill close implements
- `scripts/new-lobe.sh` — the wrapped scaffolder (CS-348)
- `.claude/skills/cibo-new-lobe/SKILL.md` — the skill shipped this session
- [CS-348](https://linear.app/colobbo/issue/CS-348) · [CS-349](https://linear.app/colobbo/issue/CS-349) — related Layer-0 scaffolding tickets
