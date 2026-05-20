---
title: "ADR-035: CS-357 Migration Lobe read-first body analysis — MOVE-or-KEEP contract supersedes name-based bridge_strategy heuristic"
tags: [decisions, adr, adr-035, cs-357, cs-318, migration-lobe, body-read, move-or-keep, code-translation-mode, read-first-contract, drizzle-pg-query-builder, sp-call-shell, mssql-beta-guard, empty-recordset-divergence, adr-033-supersession-the-classifier]
last_updated: 2026-05-20
source: manual
confidence: high
lobe_version: 1.0.0
linked_issue: CS-357
status: active
consumer_contract: true
---

# ADR-035: CS-357 Migration Lobe read-first body analysis — MOVE-or-KEEP contract supersedes name-based bridge_strategy heuristic

## Status

Accepted 2026-05-20 (substrate redesign — the redesigned MigrationResult v2.0.0 schema + Body Analysis Step 5 + `code_translation_mode` selector axis + the route-mount codegen capability + the empty-recordset modern-convention divergence ship together as the CS-357 substrate). Supersedes the **classifier step** of [[adr-033-cs330-migration-lobe-codegen-boundary|ADR-033]] §1.D2 — the *bridge-strategy selector axis* on the codegen idiom map. ADR-033's other decisions (§1.D1 absent-`arch_component_id` resolution chain, §1.D3 parsed `__skip__` sentinel pattern for unwired keys, §1.D4 greenfield-emit-only operating mode, §1.D5 reworked source-grounding HIGH-D guard, §1.D6 glaze namespace + tier-aware §10.2 gate posture) **stay intact** — the new contract preserves them and extends them. ADR-033 the file is **not edited** (immutable-ADR precedent); this ADR documents the supersession explicitly.

This ADR is **consumer_contract: true** — downstream consumers of the Migration Lobe (codegen sub-agent, test-writer-lobe, doc-writer-lobe, code-reviewer-lobe) MUST adapt to MigrationResult v2.0.0; the schema bump is MAJOR.

## Context

[[adr-033-cs330-migration-lobe-codegen-boundary|ADR-033]] §1.D2 anchored the Migration codegen idiom-map selector on `bridge_strategy`, resolved by Migration Lobe Step 5 ("Bridge Resolution"). Step 5's resolution chain — `migration_targets[].module_resolved` → `glaze.migration.architectural_assumption.shape_breakdown[<module>]` → shape token → `bridge_strategy` — **never reads the legacy method body**. The codegen sub-agent reads the body later, but by then the SPEC is locked to a strategy that may not match the body's actual structure.

**CS-318 (Migration Lobe live integration test, started 2026-05-20) proved this is a structural defect, not an edge case.** Validation target `api/WorkOrderType/GetAll/{companyId}` resolved `bridge_strategy=parallel-sp` from the module-shape lookup. The actual legacy method (`/Users/vipingera/Desktop/Colobbo/repos/Colobbo-Web/IPMMasterAPI/Services/WorkOrderTypeService.cs:399-485`) is a **7-source N+1 aggregation**:

- `SP_WorkOrderTypeGet` primary recordset (note: NOT `SP_WorkOrderTypeGetByStatus` which `StoredProcedures.WorkOrderType.Get` in the modern catalog points at — the catalog has stale entries; the body-read is the only authoritative source)
- 4 pre-loop batch service calls (`woTypeUDFSer.GetWorkOrderTypeUDFDetailsAsync`, `chkListRulesSer.GetCheckListRulesDetailsAsync`, `wkFlowSer.GetWorkOrderTypeWorkFlowDetailsAsync`, `wkFlowDtlsSer.GetWorkOrderTypeWorkFlowDetailsDetailsAsync`)
- 2 per-row SP-bearing service calls inside the row-mapping loop (`GetWorkOrderTypeSkillDetailsAsync`, `GetWorkOrderTypeMilestoneDetailsAsync`) — a true N+1 antipattern
- 4 in-loop `.Where()` filter-attachments wiring the pre-loop batches onto each row
- 2 post-loop single-row attachments to `lst[0]` (`hseFormList`, `menuDetails`)
- An empty-recordset branch returning a 1-element stub with the same `hseFormList` + `menuDetails` attachments

The Code Reviewer Lobe correctly halted CS-318 at `status: "escalated_to_architect"` with substantive findings — DTO parity gap, empty-recordset semantic inversion, file-decomposition gap, authz gap, route-unregistered, SP-registry bypass, console.error usage. **All trace back to the upstream substrate fault: the Lobe classified by name, not by structure.**

**The COL-205 migration runway** ([Linear COL-205](https://linear.app/colobbo/issue/COL-205), 660 endpoints, dueDate 2026-06-30) cannot tolerate name-based pre-classification at scale. The defect produces wrong specs upstream which cascade into wrong codegen which the Reviewer must catch every time. Body-read every endpoint, every time. Production (Decon, WHA tenants). Precision over speed.

### What this ADR is grounded in

Pre-flight direct reads (not delegated — the CS-330 session's Pattern G + free-form-Explore-unreliability discipline reaffirmed; two Explore agents this session refused to use tools claiming hallucinated prior constraints):

- `lobes/migration-lobe/LOBE.md` (full) — the current Step 5 logic + the field set.
- `agents/global/migration-lobe-codegen.md` (full) — the bridge_strategy-keyed idiom selection at Step 7g.
- `lobes/migration-lobe/output-schema.json` — the v1.1.0 schema + the bridge_strategy enum.
- `/Users/vipingera/Desktop/Colobbo/repos/Colobbo-Web/IPMMasterAPI/Services/WorkOrderTypeService.cs:390-490` — the validation-case body (the 7-source aggregation directly verified).
- `/Users/vipingera/Desktop/Colobbo/repos/Colobbo-Web/IPMMasterAPI/Services/AccountIndustryService.cs:35-150` — a single-SP CRUD example (MOVE-eligible) + a same-controller paginated overload with `filterCriteria` base64-string param (KEEP via dynamic-SQL signal). Confirms the rule discriminates correctly within the same controller.
- `/Users/vipingera/Desktop/Colobbo/repos/api_node/apps/api-node/src/routes/riskThresholdRoutes.ts` — the canonical Express router pattern + the `assertCompanyAccess` helper (lines 39-71) — the modern auth/authz contract the redesigned codegen MUST emit into KEEP-emit route files.
- `/Users/vipingera/Desktop/Colobbo/repos/api_node/apps/api-node/src/modules/risk-thresholds/risk-thresholds.service.ts` — the canonical Drizzle PG service pattern (`db.select().from(...)` + `eq`/`and`/`isNull` predicates) — the MOVE-emit reference shape.
- `/Users/vipingera/Desktop/Colobbo/repos/api_node/apps/api-node/src/utils/utils.ts:16-47` — `executeStoredProcedure` helper signature (mssql driver direct; no Drizzle MSSQL coupling).
- `/Users/vipingera/Desktop/Colobbo/repos/api_node/apps/api-node/src/shared/storedProcedures.ts` (head) — the SP-name catalog (verified stale on the WorkOrderType.Get entry → confirms body-read is canonical, not catalog).
- `glaze/colobbo.glaze.json` `.migration` block (full) — the current shape_breakdown + bridge_strategy enum + framework_specific_hints.
- Drizzle MSSQL beta state verified via Context7 (`/drizzle-team/drizzle-orm-docs`): MSSQL dialect is in `drizzle-orm-v1beta2`; RQBv2 unsupported; recommended path is `db.execute(sql\`...\`)`. The api_node-pinned `drizzle-orm@0.45.1` is pre-v1 → MSSQL dialect not present at all; the KEEP path uses the raw `mssql` driver via `executeStoredProcedure`. No Drizzle MSSQL coupling.
- Jaydeep-confirmed (2026-05-20): api_node GET-list convention is bare `[]` on empty recordset (verified across SORType + Inspection + Customer endpoints). The legacy .NET 1-element-stub-on-empty is **NOT replicated** in Node.

A `/plan` + `/rthink` Pass 1+2 over the redesign plan surfaced 20 gaps; 17 were fixed inline in the plan; 3 deferred to substrate-authoring phases. Five design checkpoints (MOVE-or-KEEP rule, bridge_strategy disposition, glaze key-axis transition, route-mount codegen capability, empty-recordset convention) were operator-approved before this ADR was authored.

---

## §1 — The read-first body-analysis contract

### D1 — Body-read first; no name-based pre-classification

For each migration_target the Migration Lobe **resolves the legacy method's file path + line range** (via `migration_targets[].module_resolved` → `glaze.modules.module_to_repo` → `glaze.repos.<repo>.path` + the legacy-corpus index at `glaze.migration.legacy_module_index` + the source_unit_id), **reads the method body via its Read tool**, **runs the structural-analysis rule (§1.D2)**, and **emits** the verdict (`code_translation_mode`) + the structured analysis (`legacy_method_body_summary`) in MigrationResult.migration_targets[]. The body-read is required for every target. The cost (tokens per endpoint) is accepted: the alternative is wrong specs at scale.

The previous Step 5 (`Bridge Resolution` — name-based shape→strategy heuristic) is **removed**. The replacement Step 5 (`Body Analysis`) is documented in `lobes/migration-lobe/LOBE.md` Step 5 as the new methodology. `glaze.migration.architectural_assumption.shape_breakdown` is **no longer consulted by the Lobe**; it remains in glaze for operator documentation, but its substantive role is supplanted.

### D2 — The MOVE-or-KEEP rule (deterministic, structural, conservative)

The body-analysis verdict (`code_translation_mode: "move" | "keep"`) is determined by a precise rule. Ambiguous cases default to `keep` (safer during dual-running grace window — preserves legacy SP-call path, no false-positive Drizzle-rewrite of a complex body).

**MOVE eligible iff ALL of the following:**

1. **Single data-source**: method body contains exactly one SP invocation (`CommandText = "SP_..."`) or exactly one EF/LINQ root query against a single recognised table. No additional service calls (`_otherService.GetXAsync(...)`) before or after the data fetch.
2. **SP body is non-procedural**: the underlying SP is a single SELECT/INSERT/UPDATE/DELETE statement. Disqualifying T-SQL signals: `IF`/`WHILE`/`DECLARE @var`, cursors (`DECLARE ... CURSOR`), multi-recordset returns, `EXEC sp_executesql` / dynamic SQL, `OUTPUT INTO`, recursive CTEs (except trivial single-recursive WITH), `TRY...CATCH`/`THROW` for control flow (not just error logging). SP source not available for reading → `unverified` → defaults to KEEP (conservative).
3. **Joins are standard**: inner/left/right/cross joins on simple equality predicates. Disqualifying: `OUTER APPLY`, `CROSS APPLY`, `MERGE`, lateral joins, joins to table-valued functions with arguments.
4. **Filters are static expressions**: WHERE/HAVING use column references + literal values + parameters. Disqualifying: procedural pre-computation (WHERE clause built from C# string concatenation that becomes a dynamic-SQL @filterCriteria param, e.g. the AccountIndustryService paginated overload), correlated subqueries with side effects.
5. **Result mapping is field-to-field**: the C# row-mapping loop assigns column values directly to DTO fields. Disqualifying: per-row C# delegate transformations beyond literal-type coercion (Convert.ToInt32/Convert.ToDateTime are fine; service-call transformations are not), per-row sub-queries (`woTypeUDF.Where(o => o.id == item[0]).ToList()` is a KEEP signal — per-row enrichment), conditional row-shape variation.
6. **No additional post-loop attachments**: the method returns the list directly after the row-mapping loop. Disqualifying: `lst[0].fieldX = otherService.GetX(...)` after the loop (the WorkOrderType signal).

**KEEP required if ANY of the following structural signals are present:**

- Multi-SP composition (≥2 SP invocations in the method).
- Multi-service composition (≥2 service-call invocations including the primary SP).
- Per-row N+1 (in-loop service calls or sub-queries against another data source keyed by the row).
- Procedural T-SQL in SP body (any signal from MOVE #2 disqualifying list).
- Non-standard joins (signal from MOVE #3 disqualifying list).
- Post-loop attachments (signal from MOVE #6 disqualifying list).
- Conditional-flow SP dispatch (the method calls SP_A under one condition and SP_B under another, with structurally different result shapes).
- Transitive: the method delegates to another method that is itself KEEP (read the delegate body too).

**"I don't know" cases default to KEEP**:

- SP source not available for reading → cannot verify SP body is non-procedural → KEEP.
- Method body too large to read in one pass (>500 lines or >20k tokens) → KEEP (the size is itself a complexity signal).
- Method body contains overloads or signature variants that disagree on shape → KEEP (require operator disambiguation; emit a `clarification_questions[]` entry asking which overload to migrate).
- Cross-method dynamic dispatch (reflection, runtime-resolved method names) → KEEP (cannot statically analyse).

**Empty-recordset stub is NOT a classifier input** (operator clarification 2026-05-20). The legacy 1-element-stub on empty recordset is a deliberate legacy quirk dropped at emit time, not a structural-complexity signal. See §3 (consequences) for the deliberate-divergence treatment.

### D3 — `code_translation_mode` is the new primary selector axis

The codegen idiom-map key axis shifts: `glaze.migration.codegen.framework_specific_hints[<target_stack>][<code_translation_mode>]` (was: `[<bridge_strategy>]`). For `api_node`, both keys (`move` + `keep`) are WIRED at v0.1.0 of this redesigned substrate — the old `fdw` + `chained-http` enum keys are **dropped entirely** from `framework_specific_hints` (their corresponding shape resolutions no longer occur; the parsed-sentinel skip mechanism from [[adr-031-cs328-database-lobe-codegen-boundary|ADR-031]] §1.D3 is no longer needed in the Migration codegen idiom map).

`bridge_strategy` retains its enum (`["fdw", "chained-http", "parallel-sp", "dark-launch"]`) in MigrationResult v2.0.0 but its semantics shift to **derived output**: it is computed by the Lobe from `code_translation_mode` + glaze consumer-context per:

- `move` → `dark-launch` (modern silently builds against PG via Drizzle query-builder; legacy continues for legacy consumers; consumer cutover is a separate runbook step).
- `keep` → `parallel-sp` (modern calls the shared SP via `executeStoredProcedure`; modern and legacy both read through the same SP during the dual-running grace window).

`fdw` + `chained-http` remain enum values for documentation completeness (a future Lobe extension could re-introduce them as derived outputs for cross-DB FDW reads or chained-HTTP scenarios), but the current Lobe never produces them.

**Downstream consumer compatibility preserved**: test-writer-lobe and doc-writer-lobe continue to read `bridge_strategy` for runbook framing; they do not need to consume `code_translation_mode` directly (though they can — the field is REQUIRED in v2.0.0).

### D4 — New required schema fields (MigrationResult v2.0.0)

MigrationResult `_metadata.current_version` bumps `1.1.0 → 2.0.0` (MAJOR — semantic shift of `bridge_strategy` from classifier-output to derived-output). Per `lobes/migration-lobe/LOBE.md` §schema_version bump discipline: MAJOR on classifier-semantics change.

New REQUIRED fields per `migration_targets[]` item:

- `code_translation_mode`: enum `["move", "keep"]` — the new selector axis per D3.
- `legacy_method_signature`: string — the C# method signature as read from the body (e.g. `public List<WorkOrderType> GetAllWorkOrderTypeDetailsAsync(int companyId)`).
- `legacy_source_ref`: object `{ file_path: string, line_range: string ("start-end") }` — the body-read provenance. Codegen + downstream consumers can re-read the body if needed.
- `legacy_method_body_summary`: object — structured analysis output with the following sub-properties:
  - `primary_sp`: object `{ name: string, params: Array<{name, value_expr}> }` OR `null` if direct LINQ.
  - `pre_loop_service_calls`: array `<{ service: string, method: string, args: string[], result_var: string }>`.
  - `in_loop_service_calls`: array `<{ service: string, method: string, args: string[], result_field: string, row_ref_arg: string }>`.
  - `in_loop_filter_attachments`: array `<{ result_var: string, filter_predicate: string, target_field: string }>`.
  - `post_loop_attachments`: array `<{ target_path: string, service: string, method: string, args: string[] }>`.
  - `empty_recordset_branch`: object `{ present: boolean, legacy_returns: "empty-array" | "stub-with-attachments", stub_fields?: array }` — **provenance only; recorded-and-overridden per §3**.
  - `tsql_procedural_signals`: array — disqualifying T-SQL signals found in SP body (or `["sp-source-unavailable"]` if the SP couldn't be read).
  - `dto_field_list`: array of strings — per-row field assignments observed in the result-mapping loop.

The structured `legacy_method_body_summary` lets the codegen sub-agent emit a composition shell mechanically (Step 7d.5 in the redesigned codegen Methodology) rather than re-reading the body. This is the contract that enables 2-or-3-file emit + the route-mount append from a single body-analysis pass.

### D5 — Codegen multi-file emit + `src/index.ts` mount-line append

The codegen sub-agent (`agents/global/migration-lobe-codegen.md`, Phase B5 of the CS-357 substrate ship) gains TWO new capabilities:

- **Multi-file emit per migration_target**: `keep` mode emits 2 files (service.ts in module dir + route file at the conventional `src/routes/<moduleName>Routes.ts` location); `move` mode emits 3 files when no Drizzle schema exists yet for the module (`<module>.schema.ts` + `<module>.service.ts` + `<moduleName>Routes.ts`). The single-file inline emit (CS-318 v1 substrate behavior) is retired.
- **Route registration in `src/index.ts`**: the codegen appends an `import` + `app.use('/api', <moduleName>Routes)` line to `apps/api-node/src/index.ts` (idempotent — skip with advisory `route_mounted_in_index` if the line already exists; skip with advisory `route_mount_skipped` if `index.ts` cannot be located OR is malformed). This is a small mechanical edit (string-find-and-append-after-the-last-similar-line); same Write-tool tier as the file emit. Eliminates the orphaned-route papercut CS-318 surfaced.

The change to `src/index.ts` is the first capability where the codegen modifies an EXISTING file (the prior contract was greenfield-emit only). This is **scoped to one specific file** (`src/index.ts`) with **append-only semantics** (no edits to existing lines) — not a general `modify_existing_files` capability (that remains deferred per [[adr-033-cs330-migration-lobe-codegen-boundary|ADR-033]] §1.D4 / [[adr-012-lobe-operating-modes|ADR-012]] §8). Documented as a narrow exception to greenfield-emit-only, justified by the orphaned-route papercut.

### D6 — MSSQL-beta guard

Drizzle MSSQL support is in `drizzle-orm-v1beta2` (BETA); RQBv2 unsupported for MSSQL (Context7-verified 2026-05-20). The api_node-pinned `drizzle-orm@0.45.1` is pre-v1 → MSSQL dialect not present at all; the KEEP path uses the raw `mssql` driver via `executeStoredProcedure(...)` (no Drizzle MSSQL coupling).

The Lobe **NEVER emits a Drizzle MSSQL query-builder query**. The contract:

- `move` → always targets PostgreSQL via Drizzle query-builder (stable; `drizzle-orm@0.45+` PG support is production-grade). The forward-state Drizzle schema MUST exist (or be planned by Database Lobe upstream) before MOVE can be emitted.
- `keep` → uses `executeStoredProcedure(StoredProcedures.<X>.<Y>, {<params>})` (mssql driver direct) OR `db.execute(sql\`...\`)` (Drizzle's dialect-agnostic raw-SQL escape hatch; works for PG today, would work for any future MSSQL Drizzle release). The SP-name comes from the body-read (`legacy_method_body_summary.primary_sp.name`), NOT from the `StoredProcedures.<X>.<Y>` catalog (the catalog has stale entries — CS-318 surfaced this on WorkOrderType.Get).

Per COL-206 (MSSQL → PostgreSQL migration), the underlying KEEP-mode SP-call paths will eventually migrate to PG; at that point the codegen can re-emit the same module as `move` against the new PG Drizzle schema. The KEEP path is dialect-agnostic by construction, so the COL-206 timeline does not affect this ADR's correctness.

---

## §2 — Boundary clarifications relative to ADR-029/031/033

The redesigned Migration Lobe + codegen sub-agent are **still** ADR-029-shape builder-tier primitives — pre-Reviewer, no `arch_ref`/`review_ref`/per-component `files_to_modify`, `--arch-result` REQUIRED, no `--review-result`. The new contract does NOT alter the builder-tier boundary; it alters the **selector axis** that keys the idiom map on the codegen side, plus the **methodology** that produces the spec on the Lobe side.

ADR-033 §1.D1 (absent `arch_component_id` resolution chain) is **preserved**. The new `legacy_source_ref` field on `migration_targets[]` is the additional resolution context — the Lobe-side equivalent of the codegen's existing legacy-corpus-index cross-check (the reworked HIGH-D guard from ADR-033 §1.D5).

ADR-033 §1.D2 (the `bridge_strategy` selector axis) is **the supersession point** of this ADR. The new selector axis is `code_translation_mode`. The codegen idiom map is re-keyed accordingly.

ADR-033 §1.D3 (parsed `__skip__` sentinel for unwired Ware-specific tokens, inherited from ADR-031 §1.D3) is **no longer inherited** by the Migration codegen idiom map — the new axis (`move`/`keep`) is Ware-agnostic, so unwired keys don't occur. (Other Lobes — Database codegen — continue to use ADR-031 §1.D3; the supersession is Migration-scoped.)

ADR-033 §1.D4 (greenfield-emit-only `<MODE>` token + ADR-012 §8 capability vocab) is **preserved with a narrow exception** documented at §1.D5 above — the `src/index.ts` append-only capability is a scoped exception, not a general `modify_existing_files` capability.

ADR-033 §1.D5 (reworked source-grounding HIGH-D guard via the legacy_module_index) is **preserved and strengthened** — the new body-read is itself a HIGH-D guard (the SP-name in the catalog is verified to match the body-read SP-name; mismatch → advisory, not silent emit).

ADR-033 §1.D6 (glaze namespace + tier-aware §10.2 builder-tier smoke gate) is **preserved** — the §10.2 smoke gate's Migration builder-tier branch updates its partition assertion from `bridge_strategy` (2 wired / 2 sentinel-skip) to `code_translation_mode` (2 wired / 0 skip). All other §10.2 invariants (no `arch_ref`/`review_ref`, no per-target `arch_component_id`) continue to apply.

---

## §3 — Consequences

### Positive

- **The CS-318 fault disappears by construction.** The Lobe's spec reflects what the code-writer will actually emit because both consume the same `legacy_method_body_summary`. Spec-vs-implementation divergence (the root cause of the Reviewer escalation on WorkOrderType) is no longer possible.
- **The 660-endpoint runway scales.** Body-read every endpoint adds tokens-per-endpoint cost (CS-322 sizing input), but the alternative — name-based misclassification cascading to wrong codegen on hundreds of endpoints — is structurally worse.
- **The classifier is deterministic + auditable.** The MOVE-or-KEEP rule (D2) is a structural-analysis rule with no judgment calls. The rule's output is reproducible per body. Conservative defaults to KEEP eliminate the "false MOVE" risk.
- **Reference-pattern grounding**: the codegen mirrors the canonical api_node module shape (`risk-thresholds` for MOVE-Drizzle; the route-file-in-`src/routes/` convention for both modes; `assertCompanyAccess` for tenant-scope authz) — emitted code lands in-convention, eliminating the file-organization + authz findings the Reviewer surfaced on CS-318.

### Negative

- **Body-read cost**: each migration_target costs N tokens for the body-read + structural-analysis (estimated 5-30k tokens per endpoint depending on body size). For 660 endpoints, this is 3-20M tokens cumulative. The cost is acknowledged + flagged for CS-322 Day-2 sizing decision.
- **KEEP composition-shell emit is genuinely complex.** Some legacy bodies (deeply nested service calls, multi-recordset SPs, dynamic-SQL via @filterCriteria-style string injection at the C# layer) will produce KEEP verdicts that the codegen substrate cannot yet emit cleanly. The 2-round bound (per the redesign plan) captures these as named substrate gaps — follow-on tickets, not blockers.
- **Schema MAJOR bump = downstream coordination cost.** test-writer-lobe + doc-writer-lobe consume MigrationResult; they don't need to change for v2.0.0 (the new fields are additive; `bridge_strategy` semantics shift is transparent to them since they read the field-value, not the field-derivation), but the substrate ship must regen their compat-pin range. Coordinated in CS-357 Phase B.

### Neutral

- **`glaze.migration.architectural_assumption.shape_breakdown` is deprecated for Lobe consumption** but stays in glaze for documentation. Future ticket may remove or repurpose.
- **`fdw` + `chained-http` keys are dropped** from `framework_specific_hints` but remain in `supported_bridge_strategies` enum (preserves the derived-field range). Removable in a future cleanup ticket.

### Deliberate divergence — empty-recordset stub (NOT a defect, an explicit design choice)

The legacy .NET pattern (visible in `WorkOrderTypeService.cs:428-434`) constructs a 1-element stub when the primary SP recordset is empty:

```csharp
if (dtDetails.Rows.Count == 0)
{
    lst.Add(new WorkOrderType());
    lst[0].hseFormList = formSer.GetFormDetailsAsync(0, companyId);
    lst[0].menuDetails = menuSer.GetMenuDetailsAsync("0");
}
```

The api_node convention is bare `[]` on empty primary recordset (no wrapper, no stub). **Verified by Jaydeep against SORType + Inspection + Customer GET-list endpoints (2026-05-20).** The legacy stub is a backward-compat quirk for old web/mobile clients that the modern api_node codebase has deliberately moved past.

**The Migration Lobe captures the legacy stub in `legacy_method_body_summary.empty_recordset_branch` for provenance, but the codegen deliberately drops it.** The KEEP-emit service file:

1. ALWAYS returns `[]` on empty primary SP recordset, regardless of `legacy_returns: "stub-with-attachments"`.
2. Includes a one-line code comment documenting the deliberate divergence: `// Empty-recordset convention: api_node GET-list returns [] (Jaydeep-confirmed, deliberate divergence from legacy stub).`.

Parity tests assert the modern Node `[]` convention, NOT byte-identity with the legacy stub. The `legacy_returns: "stub-with-attachments"` field is provenance-only.

This is a documented, deliberate-not-incidental .NET → Node migration-class divergence. Other similar legacy quirks (e.g. magic-number sentinels, special-case branches) follow the same `recorded-and-overridden` discipline: capture in the structured summary, drop at emit time, document in a one-line code comment. See memory file `feedback_legacy_quirks_dont_propagate.md` for the broader principle.

---

## §4 — Verification

### Substrate ship acceptance (CS-357 Phase B-D)

- [ ] ADR-035 authored (this file) + registered in `glaze.wiki.canonical_pages[]` atomically + cross-linked from `wiki/decisions/index.md` (invariant k).
- [ ] `lobes/migration-lobe/LOBE.md` Step 5 rewritten as "Body Analysis"; §Inputs documents the body-read; §Outputs documents the new required fields; §Changelog appends the redesign entry. **Lobe-extractability discipline**: `bash compose.sh migration-lobe glaze/fintech.glaze.example.json` + forbidden-token grep on `LOBE.compiled.md` → 0 hits.
- [ ] `lobes/migration-lobe/output-schema.json` `_metadata.current_version = "2.0.0"` + new required fields documented + `$comment` block updated with the MAJOR bump rationale.
- [ ] `agents/global/migration-lobe-codegen.md` (recomposed) — Step 7g keys off `code_translation_mode`; Step 7d emits 2-or-3 files per target; Step 7k+ appends the `src/index.ts` mount line. Tools whitelist gains `Edit` (for the append-only `src/index.ts` modification) IF not already present.
- [ ] `glaze/colobbo.glaze.json` + `glaze/fintech.glaze.example.json` `.migration.codegen.framework_specific_hints` updated per the new key axis (`move` + `keep` for `api_node`; the fintech glaze symmetrically with Ware-neutral idiom strings).
- [ ] `lobes/pipeline-fixtures/run.sh` Migration builder-tier branch partition assertion updated (2 wired-idiom on `code_translation_mode` / 0 skip); `arch_ref`/`review_ref`/`arch_component_id` absence assertions retained; smoke gate exits 0.
- [ ] `lobes/pipeline-fixtures/synthetic/codegen-migration-result.json` schema_version 2.0.0 + new field set + new partition shape.
- [ ] 3 Migration Lobe test-fixtures updated (`dual-running-sp-cutover-coordination`, `mobile-api-client-direct-slice`, `custom-fields-eav-no-migration`) — schema 2.0.0 + new fields.
- [ ] `wiki/raku-meta/migration-lobe-history.md` appended with the redesign entry.
- [ ] `wiki/architecture/feature-pipeline.md` §10 Migration→Backend handoff doc updated per the new schema.
- [ ] `.claude/skills/cibo-pipeline/SKILL.md` Step S5 description updated (Migration stage references body-read; S6 codegen references the new selector axis).
- [ ] VERSION bump 2.38.0 → 2.39.0 (MINOR — additive substrate change; not a breaking change to OTHER lobes since downstream consumers read field-values, not the classifier semantics). `lobes/_registry.json` regen via `bash compose.sh registry-regen`. `CHANGELOG.md` `[2.39.0]` `### Added` entry.

### Dry-run + validation acceptance (CS-357 Phase E)

- [ ] Re-run `cibo-pipeline --chain=migration` against CS-318 (WorkOrderType GetAll). Migration Lobe produces MigrationResult v2.0.0 with `code_translation_mode: "keep"` for M1 + structured `legacy_method_body_summary` matching the 7-source aggregation (primary_sp = SP_WorkOrderTypeGet; 4 pre_loop_service_calls; 2 in_loop_service_calls; 2 post_loop_attachments; empty_recordset_branch.legacy_returns = "stub-with-attachments").
- [ ] migration-lobe-codegen (live, no --dry-run) emits: `apps/api-node/src/modules/work-order-type/work-order-type.service.ts` + `apps/api-node/src/routes/workOrderTypeRoutes.ts` + mount-line append in `apps/api-node/src/index.ts`. Service file orchestrates SP_WorkOrderTypeGet + the 4 pre-loop SPs + 2 per-row SPs (with N+1 TODO comment) + post-loop attachments to lst[0]; empty primary recordset returns `[]` (per §3 deliberate divergence).
- [ ] `npx tsc --noEmit` in api_node exits 0; the emitted code compiles clean against the entire TypeScript surface.
- [ ] Code Reviewer Lobe (Round 1) emits `status: "ready"` (0 critical/high findings) OR `status: "escalated_to_architect"` (Round 2 trigger). 2-round bound holds — if Round 2 also escalates → STOP; capture the specific substrate gap as a follow-on ticket; documented failure shape IS the deliverable.
- [ ] Test Writer Lobe parity test runs green — asserts the modern `[]`-on-empty contract, NOT byte-identity with the legacy 1-element-stub.

---

## Alternatives considered

| Option | Verdict | Rationale |
|---|---|---|
| Repurpose `bridge_strategy` semantics in-place (rename meaning, keep field as the move/keep axis) | Rejected (§D3) | Semantically conflates two orthogonal concerns: code translation (move/keep) and consumer coordination (dark-launch/parallel-sp). Cleaner to keep `bridge_strategy` as derived consumer-coordination + add `code_translation_mode` as the new code-translation selector. |
| Body-read in codegen sub-agent, not Lobe (leave the spec name-based) | Rejected (§Context) | The SPEC must reflect what will actually be emitted; spec-vs-implementation divergence is the CS-318 fault. Reading the body twice (Lobe + codegen) is also wasteful — read once, capture in `legacy_method_body_summary`, codegen consumes the structured summary. |
| Static analysis tool integration (parse C# AST via Tree-sitter or Roslyn-via-shell) | Rejected for v1 | CC's Read + Grep + line-counting is sufficient for the deterministic MOVE-or-KEEP rule defined in D2. AST-grade analysis is a future enhancement (CS-357-followup-E) — would extend MOVE eligibility on harder bodies that the conservative rule classifies KEEP. |
| Delete `bridge_strategy` entirely (cleaner schema, single selector axis) | Rejected (operator decision, A4) | Downstream test-writer-lobe + doc-writer-lobe consume `bridge_strategy` for runbook framing. Keeping the field as derived preserves their contract; deleting it forces coordinated updates across the chain. The added schema complexity is small (one derived field). |
| Keep all 4 old `framework_specific_hints` keys (fdw/chained-http/parallel-sp/dark-launch) AND add move/keep — runtime feature flag | Rejected (operator decision, A4) | Larger glaze, dual axes, runtime flag complexity. Clean break is cleaner; the 4 old keys are documentation-only in `supported_bridge_strategies` enum (preserves derived-field range). |
| Defer `src/index.ts` mount-line append capability to a follow-up | Rejected (operator decision, A4) | The orphaned-route papercut would persist across the 660-endpoint runway, requiring operator hand-mount N times. ~30 min substrate cost vs N × hand-mount cost is well-spent. |
| Preserve the legacy 1-element-stub-on-empty (parity) | Rejected (operator decision, A4 + Jaydeep-confirmed) | api_node GET-list convention is bare `[]` on empty. Verified across SORType + Inspection + Customer. Legacy stub is a deliberate quirk dropped at emit time. Recorded-and-overridden pattern. |

---

## Follow-ups (deferred)

1. **CS-357-followup-A**: structured per-stage `tokens_used` telemetry hook in cibo-pipeline run.json (coarse run-level capture in Phase F3 suffices for CS-322 Day-2 input; structured per-stage is downstream wish-list).
2. **CS-357-followup-B**: N+1 batch optimization for KEEP-composition emit — current KEEP idiom preserves N+1 with a TODO comment for code-writer follow-through.
3. **CS-357-followup-C**: structured logger wiring (replace console.error in emit + module-wide pre-existing usage).
4. **CS-357-followup-D**: `src/shared/storedProcedures.ts` catalog hygiene sweep — `WorkOrderType.Get → SP_WorkOrderTypeGetByStatus` is stale vs the legacy method's actual `SP_WorkOrderTypeGet`; the body-read is canonical, but the catalog should be auditing.
5. **CS-357-followup-E**: AST-grade body analysis (Tree-sitter or Roslyn-via-shell) for harder cases — current rule is conservative + may KEEP cases that AST could MOVE.
6. **CS-357-followup-F**: `glaze.migration.architectural_assumption.shape_breakdown` removal (now deprecated for Lobe consumption but still in glaze).
7. **CS-357-followup-G**: cross-Lobe compat-pin range regen for test-writer-lobe + doc-writer-lobe (the MigrationResult v2.0.0 acceptance — additive new fields, transparent shift in `bridge_strategy` semantics).

## See also

- [[adr-033-cs330-migration-lobe-codegen-boundary|ADR-033]] — the supersession parent for the bridge_strategy selector axis (§1.D2). ADR-033 the file is NOT edited (immutable-ADR precedent); this ADR documents the supersession explicitly.
- [[adr-029-cs327-backend-lobe-codegen-boundary|ADR-029]] — the generic builder-tier contract (preserved verbatim).
- [[adr-031-cs328-database-lobe-codegen-boundary|ADR-031]] — sibling skip-mechanism (no longer inherited by Migration codegen since the new axis has no skip; Database continues to inherit).
- [[adr-012-lobe-operating-modes|ADR-012]] §8 — capability vocabulary; `<MODE>` token preserved (greenfield-emit) with the narrow `src/index.ts` append-only exception documented at §1.D5.
- [[adr-009-multi-dialect-tooling-reality|ADR-009]] — capability gating (preserved); the new body-analysis preserves the substitution mechanism for capability-gated upstream features.
- [[adr-007-cross-lobe-consumer-pinning|ADR-007]] — multi-upstream Lobe pattern / cross-Lobe consumer pinning (preserved; Migration continues to consume Arch + Backend + Database).
- [[adr-005-lobe-schema-pin-and-path-discipline|ADR-005]] — path-discipline for the runtime glaze read (preserved).
- [[adr-021-cs143-cs49-boundary|ADR-021]] — Reviewer `escalated_to_architect` is terminal (the precedent that surfaced the CS-318 finding cleanly).
- [[../architecture/feature-pipeline|feature-pipeline.md]] §10 — the substrate → L2 codegen handoff (the §10.2 gate retains its tier-aware shape; the Migration builder-tier branch updates per §4 verification).
- [[../raku-meta/migration-lobe-history|migration-lobe history]] — the CS-357 redesign appends here.
- Memory `feedback_legacy_quirks_dont_propagate.md` (at `~/.claude/projects/.../memory/`) — the broader principle that modern conventions supersede legacy quirks by default.
- Memory `reference_api_node_get_list_empty_convention.md` (at `~/.claude/projects/.../memory/`) — the specific empty-recordset `[]` convention (Jaydeep-confirmed).
- [Linear CS-357](https://linear.app/colobbo/issue/CS-357) — substrate redesign ticket · [Linear CS-318](https://linear.app/colobbo/issue/CS-318) — validation case · [Linear CS-330](https://linear.app/colobbo/issue/CS-330) — substrate parent · [Linear COL-205](https://linear.app/colobbo/issue/COL-205) — downstream runway.
