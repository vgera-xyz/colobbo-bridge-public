---
title: "CS-342 — ADR-012 audit dogfood smoke: backend audit operation invoked against real api_node (POC validated-with-caveat)"
tags: [bridge, session, cs-342, cs-138, cs-340, adr-012, capability-profile, audit-operation, produce-findings, dogfood, poc-validated-with-caveat, rubric-override, substrate-hygiene, frontend-lobe-migration-signals, validator-as-interim-gate, cs-324, n1-existence-proof]
last_updated: 2026-05-18
source: memory
confidence: high
lobe_version: 1.0.0
linked_issue: CS-342
status: active
---

# CS-342 — ADR-012 audit dogfood smoke (backend audit op against real api_node)

## Summary

The deferred empirical validation of [[../../decisions/adr-012-lobe-operating-modes|ADR-012]]'s capability primitive. CS-340 shipped the POC *architecture* (backend-lobe v0.3.0 build+audit) but the `audit` operation had **never been invoked against real Colobbo code** — CS-340's "audit dogfood smoke" was only a reachability check. This session ran the real thing: dispatched the deployed `backend` sub-agent with the structured invocation `--operation audit --audit-target=apiNode --feature-id=CS-138-AUDIT-DOGFOOD --intent=refactor` against `~/Desktop/Colobbo/repos/api_node` (READ-ONLY), captured the emitted BackendResult, schema-validated it, ran semantic assertions, and spot-checked findings for hallucination. **Outcome: POC validated-with-caveat** (operator override of the locked binary rubric). N=1 existence proof of the ADR-012 capability primitive working end-to-end on real code; reliability claim explicitly deferred to the CS-324 deterministic gate. One substrate-hygiene defect filed as **CS-342** (CS-138 child). No code ship — READ-ONLY smoke, no VERSION/tag/resnap/product-repo write.

## What was done

- **Discovery-first (zero-assumptions):** read ADR-012 full, schema (`lobes/backend-lobe/output-schema.json` v1.3.0), LOBE.md, the compose-golden deployed agent `agents/global/backend.md`, all 4 audit fixtures, the CS-340 bridge, and ADR-029/030/031 intros. **Step-8 blocker check PASSED**: `glaze.backend.enabled_capabilities = [read_code, produce_plan, produce_findings]` ⇒ predicate `produce_findings ∈ declared∩enabled` VALID.
- **Pre-flight de-risk** (rthink Pass 1): node v22.14.0; `validate-backend.mjs` schema-path is `scriptDir`-relative (cwd-independent, Pattern T cleared); both audit control fixtures validate exit 0; negative control exits 4 — the schema-validation half proven before dispatch.
- **rthink Pass 1 → draft → rthink Pass 2 (adversarial):** Pass 2 *reversed* a Pass-1 fix (spoon-feeding the contract into the dispatch defeats the dogfood) and made the rubric crisply falsifiable (separating substrate-hygiene from contract-violation). Two operator checkpoints honoured uncollapsed (approach approval, then apply approval).
- **Dispatch** carried only the structured invocation + smoke posture (no contract restatement — the deployed agent produced its own envelope from its composed §Operation: audit).
- **Capture:** byte-exact post-`SCOPE:` JSON → `memory/incoming/cs-138-audit-dogfood-backendresult-2026-05-18.json` (immutable; the review artifact).
- **CS-342 filed** (Pattern G: `list_issues parentId=CS-138` first — only prior child was CS-340, no duplicate). CS-138 child, Medium, Colobbo Strategic. Requested labels (`backend-lobe`, `adr-012`, `audit-operation`, `substrate-hygiene`) do **not** exist in the workspace and were **not auto-created** (unscoped schema mutation declined; categorization carried by title/description/this bridge).

## POC outcome: validated-with-caveat (rubric override — documented)

The locked rubric was **binary on schema-validity** ("schema fails → STOP, no ticket, no close"). The envelope failed schema validation on exactly two points — `$.blocked_reason: null` and `$.error: null` (schema: optional, `type: string`/`object`, "populated only when" blocked/error; both audit fixtures omit the keys entirely). The deployed substrate null-fills them defensively.

Operator override (ratified): in retrospect the rubric should have distinguished a **substrate-hygiene defect** (null-vs-omit on optional terminal fields — the validator-as-interim-gate per ADR-012 §10 doing exactly its job) from a **contract-violation** (capability mechanics broken). All mechanics passed; the audit reasoning is sound. **Pass-3 rubric refinement noted for future smokes: schema-fail must sub-classify hygiene vs contract-violation, not treat them identically.** Filed CS-342 + closed despite the locked rubric's "no ticket, no close" — explicitly an operator-authorised deviation, recorded here not silently baked.

### Evidence (all assertions ran; full detail in the captured envelope)

- **Schema validation:** `node scripts/validate-backend.mjs <envelope>` → exit 4, 2 violations (`$.blocked_reason` null≠string, `$.error` null≠object). `clarification_questions:[]` is NOT a violation (type array accepts `[]`).
- **Semantic assertions — ALL PASS:** `capability=="produce_findings"`; `components==[]`; `findings` array, not null, len 3; `status=="ready"`; ids `^AF\d+$`; severities in enum; `target_runtimes==["node-express-drizzle"]` (glaze-resolved, NOT the fixtures' fintech values); predicate dogfooded by the agent.
- **Anti-hallucination spot-check — 3/3 GROUNDED, zero fabrication:** AF2 (HIGH) verbatim at `bill.controller.ts:65` (recurs 603/738/888); AF3 (low) exact at `bill.controller.ts:47`; AF1 (medium) real, line-pointer off-by-one (cited 13, `CREATE TABLE` at 14 — quote legitimately spans 13–14). All 7 agent-reported `SCOPE:` paths resolve in api_node (no unresolvable-path signal).

## Three substrate-quality signals for the Frontend Lobe migration prompt

Captured now so the next sibling migration (Frontend, Phase 3) inherits the fixes preemptively:

a. **Optional-terminal-field hygiene** — the ticketed CS-342 defect: null-fill vs omit for `blocked_reason`/`error`/`clarification_questions`-when-empty. Frontend audit-emit needs an explicit "omit optional terminal fields when N/A per schema contract" instruction (or schema accepts `null` — a design decision, not this smoke's).
b. **Output-format adherence** — the agent prepended a prose sentence before the mandated `SCOPE:` line-1 despite an explicit "nothing before line 1" instruction. Strict structured-output framing is not perfectly honoured by the opus substrate — relevant for any deterministic dispatch parser.
c. **Line-pointer precision** — AF1 cited `line:13` for evidence spanning lines 13–14. Multi-line evidence needs a pointer convention (cite the anchor line, or a `line_start`/`line_end`).

## Real production value (surface to Jignesh separately — not blocked by this handoff)

**AF2 is a genuine HIGH security finding in api_node:** `approveBillController` (privileged: bill+claim approve + Xero sync) derives the acting `userId` from `req.body?.userId` when `req.user` is absent — a client-assertable actor-identity bypass for authorisation/attribution on bill approval. Recurs at `apps/api-node/src/modules/bill/bill.controller.ts:65, 603, 738, 888`. This is independent of the POC and should reach the api_node owner directly.

## Commits (this session, colobbo-agent-system main)

- Handoff commit: bridge entry + daily-log append + glaze canonical_pages registration + captured envelope. No VERSION bump, no tag, no Pattern-Q resnap, no product-repo write (READ-ONLY smoke). CS-340 (closed) untouched; pointer → CS-342.

## What's next

- **CS-342** (filed, CS-138 child, Backlog/Medium) — one-line LOBE.md audit-emit fix + preemptive Frontend-prompt fix. The remediation, not done in this smoke (out of scope: LOBE.md/schema edits, ADR-012 amendment, VERSION/tag).
- **CS-324** — deterministic capability-validation gate (ADR-012 §10); the reliability claim remains deferred to it.
- **AF2 security finding** — route to the api_node owner outside this handoff.
- **R1** (open, from CS-340) — `produce_plan`-for-build is an ADR-012 §3 interpretation; remedy if rejected is an ADR-012 amendment, not an atom invented in a consumer ticket. Unaffected by this smoke.
- No open blockers. The ADR-012 capability primitive is empirically demonstrated working end-to-end on real Colobbo code (N=1 existence proof).

## See also

- [[../../decisions/adr-012-lobe-operating-modes|ADR-012]] — the capability primitive this smoke empirically validates
- [[2026-05-18T0428-CS-340|CS-340 bridge]] — the POC architecture ship (this smoke is its deferred empirical validation)
- [[../../raku-meta/backend-lobe-history|Backend Lobe history]] — methodology evolution log
- [Linear CS-342](https://linear.app/colobbo/issue/CS-342) (filed) · [CS-138](https://linear.app/colobbo/issue/CS-138) (parent) · [CS-340](https://linear.app/colobbo/issue/CS-340) (closed POC ship) · CS-324 (gate follow-up)
