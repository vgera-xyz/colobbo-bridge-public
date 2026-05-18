---
title: "CS-138 — ADR-012 audit-fix (v2.32.2) + build-path discovery: substrate intelligence (POC epoch 2)"
tags: [bridge, session, cs-138, cs-342, cs-340, cs-343, cs-344, cs-345, cs-346, adr-012, adr-029, capability-profile, audit-operation, build-path, substrate-intelligence, brownfield-greenfield, supported-component-types-gap, architect-cs342-latent, pipeline-orchestration-gap, handover-state-drift, wrong-target-signal, col-274, col-273, n1-existence-proof]
last_updated: 2026-05-18
source: memory
confidence: high
lobe_version: 1.0.0
linked_issue: CS-138
status: active
---

# CS-138 — ADR-012 audit-fix + build-path discovery (combined: Phase 1 + Phase 2)

## Summary

Two-phase session continuing the ADR-012 capability-primitive POC ([[2026-05-18T0651-CS-342-audit-dogfood-smoke|CS-342 audit dogfood]] → this). **Phase 1:** shipped the CS-342 substrate-hygiene fix (Backend Lobe v0.3.0→v0.3.1, the optional-terminal-field emit discipline) surfaced by the [[2026-05-18T0651-CS-342-audit-dogfood-smoke|v2.32.1 audit dogfood]] — tag **v2.32.2**, 18-gate CI ratified, CS-342 → Done. **Phase 2:** attempted a Backend-Lobe **build**-path smoke on COL-274; discovery surfaced **three independent structural blockers** that proved COL-274 was never a viable target. Per operator decision (Option 4) the build smoke was **not forced** — the substrate intelligence is the Phase-2 POC value. Zero substrate edits in Phase 2 (all discovery); v2.32.2 stands as the session's tag ratification. Four follow-up tickets filed (CS-343/344/345/346).

## Phase 1 — CS-342 substrate-hygiene fix (shipped + ratified)

The [[2026-05-18T0651-CS-342-audit-dogfood-smoke|CS-340 audit dogfood]] (v2.32.1) was POC **validated-with-caveat**: all ADR-012 capability mechanics passed + findings 3/3 grounded (incl. a real HIGH security finding), but the captured BackendResult failed schema validation on `$.blocked_reason:null` + `$.error:null` — the substrate defensively null-filled optional terminal fields the "populated only when" contract requires omitted. Operator overrode the locked binary rubric (substrate-hygiene ≠ contract-violation; the validator-as-interim-gate per ADR-012 §10 working as designed).

**Fix shipped:** discovery across all 15 `lobes/backend-lobe/test-fixtures/*/expected.json` proved the omit-unless-applicable contract is **general across both `build` and `audit` emit paths** (every `ready` fixture omits the terminals). One shared *Optional-terminal-field emit discipline* paragraph added to `LOBE.md` §Outputs (cited by both emit paths); deployed agent regenerated same commit (Gate 14 golden); LOBE v0.3.0→v0.3.1 PATCH-clarification (CS-141 `0.2.1` precedent shape); VERSION 2.32.1→**2.32.2**; CHANGELOG `[2.32.2]`. Commit `2de243334`, tag **v2.32.2**, CI run `26018708216` — **success, 26/26 steps, 18 gates green** (Pattern S; sha/tag verified). Sanity-proven: projected v0.3.1 envelope (captured minus null-filled fields) validates exit 0. **CS-342 → Done.** No `<MODE>`/codegen/brownfield touched (ADR-012 §8). No AI attribution (user rule over harness default — prior `5357d196e` carried the Co-Authored-By trailer errantly; corrected forward-only, recorded as a durable feedback memory).

## Phase 2 — build-path discovery: substrate intelligence (the actual POC value)

GOAL was an end-to-end build POC on COL-274 (pre-resize images on upload + S3 variant cache) → Backend Lobe build → codegen → DRAFT PR. Discovery (zero-assumptions, ADR-029 + codegen sub-agent + Architect pre-flight + COL-274/COL-273 Linear + schema/glaze verification) surfaced three independent structural blockers. Per Option-4 operator decision the smoke was banked, not forced. **Findings:**

### (a) Build path requires a 4-stage chain, not from-ticket dispatch

Faithful chain: **Planner → (PlanResult) → Architect → (ArchResult) → Backend → (BackendResult) → backend-lobe-codegen → files → PR**. Each stage hard-requires its upstream artifact (Architect §Inputs: PlanResult per `lobes/planner-lobe/output-schema.json`; Backend §Operation:build Step 1.2: ArchResult required else `error.type:archresult_parse_failed`; codegen ADR-029 §1.D1: `--arch-result` required). At v0.1.0 every stage is **hand-dispatched**; no primitive auto-chains the four with inter-stage validation. ADR-029 §2 defers L2 dispatch to the unshipped CS-311 contract. → **CS-345** (High; gates all future build smokes).

### (b) Codegen v0.1.0 is greenfield-emit-only by design (ADR-029 §4)

`backend-lobe-codegen` v0.1.0 skips any component whose `ArchResult.file_mapping[].change_type != "new"` with `brownfield_modify_deferred`. ADR-029 §4 pins this as **correct v0.1.0 behaviour, not a defect**. Empirically modelled during dispatch-plan construction (not just doc-read): COL-274's core (items 1/2/4 = service *modifications*) would correctly defer. → **CS-346** (High; brownfield-modify substrate v0.2.0).

### (c) Backend `supported_component_types` vocabulary gap

`[express-endpoint, express-middleware, drizzle-schema, drizzle-migration, stored-procedure, shared-type]` — exclusively HTTP/persistence. COL-274's only greenfield-`new` deliverable (a standalone S3/Sharp **backfill script**) fits **none** → Backend Step-2 type-filter skips it `non_backend_components_only`. Real codebases routinely ship ops scripts / backfill runners / cron / one-shot migration tools — the substrate has no vocabulary for them. **This is the decisive blocker**: COL-274 has *no* component that is simultaneously Backend-supported-type AND greenfield-`new`, so Options 1b *and* 2 were both blocked regardless of upstream-seeding strategy. → **CS-343** (Medium; operational-component category).

### (d) Architect Lobe carries the latent CS-342-class defect (unfixed)

`lobes/architect-lobe/output-schema.json` has the **same** `clarification_questions`/`blocked_reason`/`error` "populated-only-when" optional terminals as Backend pre-CS-342. Architect did NOT get the CS-342 fix (Backend-only). A real Architect dispatch would null-fill → `validate-arch.mjs` rejection on `status:ready`, exactly as the CS-138 audit envelope failed pre-CS-342. Sibling-Lobe fix needed before Architect can be smoke-tested end-to-end. → **CS-344** (Medium; mirror the CS-342 fix shape).

## Methodology lessons (durable)

### (e) Handover state claims must be Linear-verified at discovery

User CONTEXT asserted *"COL-273 shipped, no longer active"*; live Linear showed COL-273 `status:QA`, `completedAt:null`, PRs #229/#230 unmerged. ("Jignesh has done 273" meant code-complete-in-QA, not shipped; the prompt encoded the weaker claim incorrectly. Linear is authoritative.) Pattern: any dispatch prompt asserting ticket state must carry an explicit *"verify <ticket> current status via Linear"* discovery step rather than encoding it as ground truth. Canonical example. (Pairs with [[../../raku-meta/state-drift-patterns|state-drift patterns]] / `feedback_handover_state_drift`.)

### (f) Two distinct "greenfield/brownfield" vocabularies — disambiguate

**Substrate-mechanic vocabulary:** greenfield = `change_type:"new"` (codegen-emittable) / brownfield = `modify` (deferred at v0.1.0) — the codegen gate. **Conceptual-test vocabulary:** greenfield = no-existing-deps / brownfield = grounded-in-existing-code. A new file CAN be substrate-greenfield AND conceptually-brownfield (e.g. a backfill script that reads existing S3 conventions). Future POC framings must disambiguate or they mask what is actually being tested.

### (g) Multiple structural blockers on one target = wrong-target signal

Three *independent* blockers on COL-274 (4-stage-chain requirement, brownfield-modify deferral, supported_component_types vocabulary gap) cumulatively prove COL-274 was never a viable Backend-Lobe build-smoke target — not because of Linear-state drift, because of **substrate-vocabulary fit**. The discipline: when ≥2 independent structural blockers stack on a single target, that is a target-selection signal, not a forcing-function problem. Banking the findings (Option 4) was higher-value than a forced empty/mistyped PR.

## Follow-up tickets filed (Pattern G — `list_issues parentId=CS-138` first; CS-138 children = CS-340/CS-342 only, no dup)

| Ticket | Title | Priority | Note |
|---|---|---|---|
| **CS-343** | Backend `supported_component_types` — operational-component category | Medium | finding (c); relatedTo CS-340/342/215 |
| **CS-344** | Architect Lobe — apply CS-342 optional-terminal-field discipline | Medium | finding (d); mirror CS-342 shape; relatedTo CS-342/340 |
| **CS-345** | Pipeline orchestration — formalize Planner→Architect→Backend→codegen chain | High (Medium-High intent, body-noted) | finding (a); relatedTo CS-322/325/327/335 |
| **CS-346** | backend-lobe-codegen v0.2.0 — brownfield-modify emit | High | finding (b); verified-absent before filing; relatedTo CS-327/215/335/334 |

## AF2 security finding (separate track — NOT closed by this session)

The [[2026-05-18T0651-CS-342-audit-dogfood-smoke|audit dogfood]] surfaced a genuine HIGH security finding: `approveBillController` derives actor `userId` from `req.body?.userId` when `req.user` absent (client-assertable identity bypass on bill approval), recurring at `apps/api-node/src/modules/bill/bill.controller.ts:65,603,738,888`. Jignesh is actively triaging on the api_node side — **independent ownership, this session's close does not close that thread.**

## Versioning

- v2.32.2 (CS-342, Phase 1) is the session's tag ratification. **No Phase-2 VERSION bump** — Phase 2 was all discovery, zero substrate edits (inline-fix bound: 0/4 distinct substrate edits this session; only Phase-1's single CS-342 edit + companions).

## What's next

- CS-344 (Architect CS-342-sibling fix) is the lowest-effort highest-leverage next substrate ship — unblocks Architect end-to-end smokeability, same shape as CS-342.
- CS-345 (pipeline orchestration) gates all future build-path POCs.
- CS-346 (brownfield codegen v0.2.0) gates production-shape build validation.
- CS-343 (component-type vocabulary) unblocks ops-script/backfill-class tickets.
- A genuinely greenfield, Backend-typed ticket (new express-endpoint) remains the right future build-smoke target (Option 3, deferred).
- No open blockers on shipped artifacts. ADR-012 capability primitive: audit path POC-validated (N=1, CS-342); build path prerequisites mapped (this session).

## See also

- [[2026-05-18T0651-CS-342-audit-dogfood-smoke|CS-342 audit dogfood bridge]] — Phase 1's origin (the validated-with-caveat audit POC)
- [[2026-05-18T0428-CS-340|CS-340 bridge]] — the ADR-012 capability-profile POC architecture ship
- [[../../decisions/adr-012-lobe-operating-modes|ADR-012]] — the capability primitive · [[../../decisions/adr-029-cs327-backend-lobe-codegen-boundary|ADR-029]] — the builder-tier codegen boundary (greenfield-only v0.1.0)
- [Linear CS-138](https://linear.app/colobbo/issue/CS-138) (parent) · CS-342 (Done) · CS-343/344/345/346 (filed) · COL-274 (the unsuitable target) · COL-273 (QA, not shipped)
