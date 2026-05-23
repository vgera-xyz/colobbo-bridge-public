---
title: "ADR-037: Migration substrate retirement — execution moves to standalone migration-worker; Migration Lobe + Regression Lobe + glaze.migration.codegen + glaze.test.regression retired from active pipeline; ADR-036 superseded; migration excluded from Raku Phase 2 extraction"
tags: [decisions, adr, adr-037, cs-368, cs-367, migration-worker, substrate-retirement, regression-lobe-retired, migration-lobe-retired, adr-036-supersession, raku-phase-2-scope-exclusion, standalone-throwaway-agent, source-fidelity-bake-precedent, security-sentinel-always-on, dev-owned-decommission]
last_updated: 2026-05-23
source: manual
confidence: high
lobe_version: 1.0.0
linked_issue: CS-368
status: active
consumer_contract: true
---

# ADR-037: Migration substrate retirement — execution moves to standalone migration-worker; Migration Lobe + Regression Lobe retired; ADR-036 superseded; migration excluded from Raku Phase 2

## Status

Accepted 2026-05-23. **Supersedes** [[adr-036-cs294-regression-test-lobe-boundary|ADR-036]] (the Regression Test Lobe substrate decision) — ADR-036's structural decisions (D1–D16) become historical record; the Lobe retires from the active pipeline. **Partially supersedes** [[adr-035-cs357-migration-lobe-read-first-body-analysis|ADR-035]] — the body-analysis MOVE/KEEP rule + verbatim source-fidelity contract STAY (now baked verbatim into `agents/global/migration-worker.md` Convention 2 + 3); only the Migration Lobe substrate that originally instantiated the rule retires. **Partially supersedes** [[adr-033-cs330-migration-lobe-codegen-boundary|ADR-033]] — the builder-tier codegen primitive (migration-lobe-codegen) retires; the generic builder-tier contract (ADR-029) stays intact for Backend/Frontend/Database codegen primitives.

This ADR is **consumer_contract: true** — affects: dispatcher (`cibo-pipeline` skill drops `--chain=migration`), CI (`.github/workflows/lobe-regression.yml` drops Gate 9 + loop entries), glaze (`glaze.lobes[]` drops both entries; `glaze.migration.*` + `glaze.test.regression` blocks retired), and `feature-pipeline.md` (§5 sentinel becomes always-on for Security's MigrationResult input; §10 marks both Lobes Retired).

The decision is **NOT** "migration is unimportant" or "the methodology was wrong." The methodology IS the contract — see ADR-035 + the migration-worker file. The decision is "the substrate pattern (Lobe + codegen + Glaze block) is too heavy a wrapper for a per-endpoint judgement-call exercise with a Jul 31 deadline. A standalone throwaway worker captures the same IP with lower coupling cost." See [[../strategy/active-plan|active-plan]] + [[../bridge/sessions/2026-05-23T0050-CS-367|bridge 2026-05-23T0050-CS-367]] for surfacing evidence.

## Context

Three things changed across the last two sessions:

1. **CS-367 surfaced a convention fork** (PR #246 KEEP-bridge vs PR #247 native-Drizzle) on `apps/api-node/src/modules/work-order-type/work-order-type.service.ts`. Migration becomes per-endpoint judgement-call exercise depending on Jignesh's deciding convention, not a methodology to standardise upstream-first.

2. **`agents/global/migration-worker.md` shipped 2026-05-23** (commit `2077ca145`, 732 lines, single file, throwaway). Bakes the decided convention verbatim: ADR-035 §1.D2 MOVE-or-KEEP rule (32 lines from `wiki/decisions/adr-035-...md:67–98`); `assertCompanyAccess` (33 lines from `apps/api-node/src/routes/riskThresholdRoutes.ts:39–71`); api_node MOVE + KEEP idiom strings (both from glaze). Zero drift across 4 verbatim blocks. Tier-1 HARD GATE + STATIC parity default + filesystem-probe sub-agent reuse with baked self-check fallback — does NOT depend on Migration Lobe OR Regression Lobe being present.

3. **The Migration Lobe substrate (CS-140, CS-357) + Regression Lobe substrate (CS-294, CS-366)** carry meaningful surface area (Lobe + composed agent + L2 codegen + output-schema + validator + glaze block + Test Writer parity branch + Security 4th-upstream pin + cibo-pipeline migration chain + CI gates + pipeline-fixtures partition + glaze invariant (aa) + 4 immutable ADRs). None of this surface is reused by the worker.

The migration runway (660 endpoints by Jul 31, 2026 per COL-205) is **dev-owned** — Jignesh + Jaydeep + Madhuresh own the cutover; agent-system role is providing the worker as throughput multiplier, not standardising the pipeline.

## Decision

**1. Migration Lobe substrate retires from the active pipeline.** Delete `lobes/migration-lobe/`, `agents/global/migration.md`, `agents/global/migration-lobe-codegen.md`, `scripts/validate-migration.mjs`. Drop `glaze.migration.codegen` + `glaze.migration.{module_status_overrides, architectural_assumption, bridge_strategies, empty_list_convention, mount_pattern}` from both glazes. **Note (CS-368 mid-execution reclassification 2026-05-23):** `scripts/lobe-migration-manifest.json` + `schemas/lobe-migration-manifest-v1.json` + `scripts/bulk-migrate-lobes.sh` + `scripts/migrate-lobe-to-layer0.sh` are **NOT** retired — they belong to **ADR-032 Layer-0 substrate tooling** (Lobe-to-Layer-0 substrate-evolution migration), distinct from the .NET-to-Node migration concern. The name "lobe-migration-manifest" is a confusingly-similar but architecturally-distinct artifact; feature-builder-relevant; KEEP. Drop `"migration-lobe"` from `glaze.lobes[]`. Drop the Migration entry from `lobes/_registry.json` (regenerate via `compose.sh registry-regen`). Drop Gate 9 + `migration-lobe` loop entry from `.github/workflows/lobe-regression.yml`. Drop Migration partition from `lobes/pipeline-fixtures/run.sh`.

**2. Regression Lobe substrate retires from the active pipeline.** Delete `lobes/regression-lobe/`, `agents/global/regression-lobe.md`, `agents/global/regression-lobe-codegen.md`, `scripts/validate-regression.mjs`. Drop `glaze.test.regression` block entirely from both glazes. Drop `"regression-lobe"` from `glaze.lobes[]`. Drop Regression entry from `lobes/_registry.json`. Drop `regression-lobe` from the lobe-regression.yml loop. Drop invariant (aa) from `scripts/verify-glaze-shape.sh` + mark retired in `wiki/raku-meta/glaze-shape-invariants.md`.

**3. cibo-pipeline drops the `--chain=migration` branch.** Skill version 0.2.0 → 0.3.0. Validation tightens to `<chain> ∈ {build}`. Build chain unchanged. Description + Steps S4-S6 migration extension + run.json `"chain"` field + Step S6 migration-lobe-codegen section all removed.

**4. ADR-036 is marked Superseded by ADR-037 (this ADR).** ADR-036 file is NOT edited (immutable-Accepted-ADR convention). Supersession recorded here + in `wiki/decisions/index.md` cross-link. ADR-036's D1–D16 stay valid as historical record.

**5. ADR-035 + ADR-033 are partially superseded.** ADR-035's body-analysis MOVE/KEEP rule + source-fidelity-bake discipline stay (migration-worker's contract surface). ADR-033's generic builder-tier contract clarifications stay (preserved for Backend/Frontend/Database codegen inheritance). Only the Migration-specific instantiations retire.

**6. Migration is excluded from Raku Phase 2 extraction.** Update `wiki/strategy/raku-roadmap.md` + `wiki/strategy/migration-runway.md` per CS-368 execution. Phase 2 portability surface (Gate 5) iterates only over the surviving 11 Lobes.

**7. Security Lobe consumes the §5 sentinel for MigrationResult always.** No edit to Security needed — existing sentinel pattern handles absence. The retired MigrationResult v2.0.0 schema is preserved verbatim in Appendix A so Security's `consumes_migration_schema` semver pin remains anchored. The sentinel shape is `{schema_version: "2.0.0", status: "blocked", migration_run_at: "<iso>", feature_id: "<id>", intent: "<intent>", migration_targets: [], advisories: []}` — `status: "blocked"` is the in-enum value from the schema's `status` enum (`["ready", "needs_clarification", "blocked", "error"]`; `not_invoked` is NOT a valid enum value despite being intuitive shorthand for the semantic intent). The synthetic sentinel at `lobes/pipeline-fixtures/synthetic/migration.json` + `pipeline-fixtures/run.sh` smoke gate both use `"blocked"` end-to-end.

**8. Test Writer Lobe keeps its parity branch dormant.** Schema v1.2.0 fields, parity `test_categories` enum value, Step 3.6.5 RegressionResult consumption, and parity-related advisory types all stay in place. Non-migration intents (feature/bugfix/refactor/docs/security/infra) function unchanged. v0.4.0 cleanup that removes the dormant parity branch is deferred to post-COL-205 follow-on.

**9. The migration-worker file is the live migration substrate.** `agents/global/migration-worker.md` is the canonical execution surface for COL-205 endpoint migrations. Self-deletes when COL-205 completes (operator-managed).

**10. Migration-related ADRs stay on disk as historical record.** ADR-033, ADR-035, ADR-036 are NOT edited. ADR-037 + `wiki/decisions/index.md` provide the supersession map. Same pattern ADR-035 used for ADR-033 §1.D2.

## Consequences

**Positive:**

- Surface area drops: ~3,500 lines of LOBE.md + schema + validator + codegen prose retire from the active maintenance burden. Migration-worker's 732 lines + ADRs are the new ratification surface.
- Coupling drops: Regression's 4-upstream pin + Migration's 4-upstream pin + Test Writer's parity-mode + Security's MigrationResult dependency all collapse to "Security always reads sentinel; nothing else cares."
- Gate count drops: Gate 9 + invariant (aa) + cibo-pipeline branch + 2 lobe-regression loop entries.
- Decision velocity goes up for migration: per-endpoint judgement-calls live in operator + worker, not in upstream Lobe coordination.
- Jul 31 .NET decommission deadline unaffected: dev-owned + worker-mediated.
- Raku Phase 2 portability surface shrinks: 13 Lobes → 11 Lobes.

**Negative:**

- CS-294, CS-318, CS-357, CS-366 represent substantial recent investment (Apr–May 2026). Retirement writes off the substrate work (the methodology itself is preserved via ADR-035 + worker bake).
- The "first cross-Lobe handoff at spec stage" architectural precedent (Regression → Test Writer via `--regression-input`, documented at feature-pipeline.md:208) becomes a vestigial pattern.
- The post-merge drift-detector primitive idea (deferred backlog) becomes a future ADR exercise without a working substrate to evolve from.
- Test Writer carries dormant parity code; cleanup deferred to v0.4.0 follow-on.

**Neutral:**

- ADRs 033, 035, 036 stay on disk as historical record. The supersession chain is explicit.
- migration-worker is throwaway; when COL-205 completes the operator deletes it. Agent-system has zero migration substrate at that point — ADRs are the only trace.

### Downstream consequences discovered at execution (2026-05-23 post-Phase-3 audit)

The §Decision items above name the plan-time retirement targets. Three classes of downstream consequence surfaced during execution + were retired in the same ship — all are within the §Decision-1/2 blast radius (deletion of `glaze.migration.*` + `glaze.test.regression` blocks structurally invalidates anything depending on those keys), but are listed here so a future reader of the ADR sees the full surface change without having to reconstruct it from the CHANGELOG or the commit diff:

1. **Cascading invariant retirement.** `scripts/verify-glaze-shape.sh` invariants `(o)` `migration.cutover_deadline`, `(p)` `migration.default_bridge_strategy ∈ supported_bridge_strategies[]`, `(q)` `migration.module_status_overrides` keys-exist-in-modules, and `(s)` `migration.architectural_assumption.shape ∈ supported_shapes[]` — all four retired alongside `(aa)` (the only invariant the §Decision-2 plan-time list named). Each was a sub-key dependency on the retired `glaze.migration.*` block; deletion of the block made the invariants structurally inapplicable. Retired labels `(o)`/`(p)`/`(q)`/`(s)`/`(aa)` are NOT re-used (immutable-label convention, mirroring the documented `(r)` gap discipline). The ADR-026 two-letter notation rule is preserved — next net-new invariant will pick up at `(ab)`.
2. **REGISTRY entry purge in `scripts/verify-lobe-compat.sh`.** 8 entries dropped: 3 migration-* (`migration-arch`, `migration-backend`, `migration-database`) + 1 `security-migration` (Security's `consumes_migration_schema` pin still exists in Security's own schema but no longer has a runtime Lobe schema to compat-check against; sentinel-input contract preserved per §Decision 7 + Appendix A) + 4 regression-* (`regression-migration`, `regression-arch`, `regression-backend`, `regression-review`).
3. **61 per-Lobe test-fixture `glaze-snapshot.json` files mechanically resnapped** via `scripts/resnap-additive-fixtures.sh` after the `glaze.migration.*` + `glaze.test.regression` + `glaze.lobes[]` deletes — the snapshots are byte-identity-checked against the live fintech glaze (invariant `(w)`) and the live glaze shape changed under them. The resnap is mechanical drift-propagation (not a semantic test change); each Lobe's `expected.json` output assertions stay untouched.
4. **Wiki integrity catch — `wiki/patterns/cross-db-bridge.md` stale wikilink.** Plan-time sweep missed an internal `[[../../lobes/migration-lobe/LOBE]]` wikilink in the bridge-patterns domain page (the file itself was correctly KEEP-classified as a domain-knowledge page, but the internal link to the now-deleted Lobe LOBE.md broke). Gate 13 (CS-208 wikilink integrity) caught it on the first tag-run attempt; repaired in a post-tag commit (the link now points at this ADR + the migration-worker as the active execution surface). The bridge-pattern taxonomy itself is preserved as a domain-knowledge reference for any future cross-dialect bridge work.

None of the four are new scope; all are mechanical downstream consequences of §Decision 1 + §Decision 2. Captured here so the ADR is self-contained for archaeology purposes.

## Alternatives considered

- **Soft-retire (keep files, mark deprecated, remove from registry).** Rejected — dormant Lobe files on disk become a "what is this for" question for future contributors; ADRs are the cleaner historical pointer.
- **REWORK Regression Lobe as a post-merge drift-detector primitive.** Deferred to a separate ADR + ticket when post-merge drift-detection becomes a real need. Migration-worker's STATIC parity + operator PR review covers the equivalence-at-migration-time concern with lower coupling.
- **Keep the Migration Lobe + retire only Regression.** Rejected — Lobe's only structural consumers post-retirement would be Security (via sentinel) + Code Reviewer (enum value) + Test Writer (dormant) + cibo-pipeline migration chain. Lobe becomes a registry-listed orphan with no portability benefit.
- **Retire the Lobes but keep glaze.migration.codegen + glaze.test.regression as Ware data blocks.** Rejected — worker reads its bake source from canonical filesystem locations, NOT from glaze. The blocks would be unread.

## Verification (RUN-and-pass — see CS-368 plan Section H)

1. `bash scripts/verify-glaze-shape.sh` — passes; invariant (aa) removed; remaining (a)–(z) + other two-letter invariants clean.
2. `bash compose.sh registry-regen` — `lobes/_registry.json` regenerates without migration-lobe + regression-lobe entries.
3. `bash compose.sh frontend-lobe glaze/fintech.glaze.example.json` — Gate 5 portability passes against post-retirement glaze.
4. `git grep -n "migration-lobe\|regression-lobe\|glaze.migration\|glaze.test.regression" -- ':!graphs/' ':!wiki/bridge/' ':!wiki/decisions/adr-03[3-7]*' ':!CHANGELOG.md' ':!memory/' ':!agents/global/migration-worker.md' ':!wiki/raku-meta/migration-lobe-history.md' ':!wiki/migration/' ':!wiki/patterns/cross-db-bridge.md'` — empty (no active Lobe/script references).
5. Build chain end-to-end smoke (the load-bearing GOAL gate): `bash lobes/pipeline-fixtures/run.sh` — passes; build-chain partition + Backend/Frontend/Database codegen partitions clean.
6. Security Lobe with §5 sentinel MigrationResult input dispatched — emits cleanly; SecurityResult `status: "ready"` (or legitimate `non_security_components_only`); no missing-schema error.
7. `--chain=build` cibo-pipeline invocation — works end-to-end. `--chain=migration` returns the validation error (`<chain> ∈ {build}` rejection).
8. CI tag-run GREEN on the version bump (Gate 19 stamps post-retirement VERSION into system_version; 18-gate sweep clean).

## Appendix A — frozen MigrationResult v2.0.0 schema (Security's pin anchor)

VERBATIM CONTENT of `lobes/migration-lobe/output-schema.json` at ADR-037 acceptance time. 788 lines. `_metadata.current_version: "2.0.0"`. Anchors Security Lobe's `consumes_migration_schema` semver pin (the range covering v2.x) to a textual contract preserved in this ADR, surviving the post-retirement deletion of the source file.

<!-- APPENDIX-A-START -->
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://raku.so/schemas/migration-lobe-output-v1.json",
  "_metadata": {
    "current_version": "2.0.0"
  },
  "title": "MigrationResult",
  "description": "Output payload of migration-lobe v0.2.x. Emitted by the Migration Lobe (third multi-upstream consumer in the Motor Lobe pipeline; consumes ArchResult AND BackendResult AND DatabaseResult per ADR-007 §Generalisation). Consumed by downstream code-writer / test-writer / doc-writer sub-agents as the per-migration_unit transition spec. Pinned to architect-schema 1.x AND backend-schema 1.x AND database-schema 1.x per ADR-007. At v2.0.0+ (ADR-035) the per-target shape carries the body-analysis output: `code_translation_mode` (move|keep) is the new primary selector axis; `bridge_strategy` is derived; `legacy_method_signature` + `legacy_source_ref` + `legacy_method_body_summary` capture the structured body-read.",
  "$comment": "Current version: see _metadata.current_version. Design notes: (1) migration_targets is an array of per-(module, source_unit) entries — M-IDs sequential across the whole MigrationResult. (2) substitutions[] per ADR-009 Decision 1 — capability gating records every documented bridge_strategy substitution. (3) capability_flags[] declarations advertise feature availability without forcing adoption (force_default vs match_adjacent semantics). (4) legacy_context_advisories[] separate from advisories[] — documents observations from legacy-corpus reads without competing for the max_advisories scope cap. (5) error.type enum mirrors Database's plus four additions (incompatible_database_schema, databaseresult_parse_failed, no_upstream_context for the all-skipped guard) per ADR-007 generalisation. (6) advisories[].type extends Database's with cutover_deadline_passed, module_status_override, shape_unresolvable, non_migration_components_only. v2.0.0+ adds advisory types from Step 5: source_unit_unresolvable, default_to_keep_unverified_sp, body_size_exceeded, overload_disagreement, dynamic_dispatch_observed. (7) decommission_target_date and parallel_consumers[] are mutually exclusive at the per-target level — fixture-asserted; legacy_context_status: decommissioned implies parallel_consumers[].length === 0. (8) v2.0.0 (ADR-035): code_translation_mode (move|keep) is the new primary selector axis (REQUIRED per migration_target). bridge_strategy semantics shift from classifier-output to derived-output (move → dark-launch, keep → parallel-sp). Four new REQUIRED per-target fields capture the body-read provenance + structured analysis: code_translation_mode, legacy_method_signature, legacy_source_ref, legacy_method_body_summary. shape_breakdown lookup at Step 2.2 is no longer consulted (deprecated for Lobe consumption). Bump discipline: MAJOR on field rename/removal AND on classifier-semantics shift; MINOR on new optional field, new enum value, new substitution rule, new advisory type; PATCH on doc/comment changes only. Version history: 1.0.0 (CS-140 initial ship); 1.1.0 (CS-143 — MINOR: new optional field feedback_iteration_context for loop-back ingestion per ADR-021); 2.0.0 (CS-357 — MAJOR: classifier-semantics shift, 4 new REQUIRED per-target fields, name-based shape→strategy heuristic deleted; ADR-035).",
  "type": "object",
  "required": [
    "migration_version",
    "schema_version",
    "migration_run_at",
    "status",
    "feature_id",
    "intent",
    "migration_targets",
    "advisories"
  ],
  "properties": {
    "migration_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$",
      "description": "Semver version of the migration-lobe methodology. Hardcoded in Lobe invocation code; 0.1.0 at initial ship. Bumped on methodology changes."
    },
    "schema_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$",
      "description": "Semver version of THIS output schema. Mirrors _metadata.current_version. Independent of migration_version."
    },
    "migration_run_at": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp (UTC) of MigrationResult generation."
    },
    "status": {
      "type": "string",
      "enum": [
        "ready",
        "needs_clarification",
        "blocked",
        "error"
      ],
      "description": "ready = transition spec is actionable (may include empty migration_targets if all resolved modules are modern-only — that case is signalled by advisory non_migration_components_only); needs_clarification = input insufficient (see clarification_questions); blocked = known-fixable condition (see blocked_reason); error = unknown-unknown runtime failure (see error object)."
    },
    "feature_id": {
      "type": "string",
      "description": "Tracker ticket ID (e.g. COL-186, CS-140). Copied verbatim from --feature-id CLI arg. Should match upstream ArchResult.feature_id AND BackendResult.feature_id AND DatabaseResult.feature_id; divergence emits advisory but proceeds with CLI value."
    },
    "intent": {
      "type": "string",
      "enum": [
        "feature",
        "bugfix",
        "migration",
        "refactor",
        "docs",
        "security",
        "infra"
      ],
      "description": "Canonical 7-value intent classification. Propagated verbatim from ArchResult.intent when present, or set to 'migration' when running corpus-only (--no-arch-context). Migration does NOT re-classify."
    },
    "migration_targets": {
      "type": "array",
      "description": "Array of per-(module, source_unit) transition specs. M-IDs sequential across the whole MigrationResult.",
      "items": {
        "type": "object",
        "required": [
          "id",
          "source_unit_id",
          "source_unit_type",
          "module_resolved",
          "code_translation_mode",
          "legacy_method_signature",
          "legacy_source_ref",
          "legacy_method_body_summary",
          "bridge_strategy",
          "cutover_mode",
          "rollback_strategy",
          "legacy_context_status",
          "depends_on"
        ],
        "properties": {
          "id": {
            "type": "string",
            "pattern": "^M\\d+$",
            "description": "Sequential ID across the MigrationResult: M1, M2, ..."
          },
          "source_unit_id": {
            "type": "string",
            "description": "Identifier of the legacy unit being migrated (e.g. an endpoint route, an SP name, a table name). Sourced from the legacy corpus per glaze.migration.legacy_module_index."
          },
          "source_unit_type": {
            "type": "string",
            "description": "Source unit type (one of glaze.migration.supported_migration_unit_types — typically endpoint, procedure, table, module)."
          },
          "target_unit_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Identifier of the target unit (when known — sourced from BackendResult.components[] or DatabaseResult.components[]). Null when running corpus-only or when migration is decommission-only."
          },
          "target_unit_type": {
            "type": [
              "string",
              "null"
            ],
            "description": "Target unit type (same enum as source_unit_type). Null when target_unit_id is null."
          },
          "module_resolved": {
            "type": "string",
            "description": "The module slug matched from glaze.modules.module_to_repo. Required for downstream module-level dispatch."
          },
          "code_translation_mode": {
            "type": "string",
            "enum": [
              "move",
              "keep"
            ],
            "description": "NEW REQUIRED at v2.0.0 (ADR-035 §1.D2). The MOVE-or-KEEP verdict from Step 5 Body Analysis. move = the legacy source-unit body is structurally eligible for rewrite into the modern stack's typed query-builder (single data-source, non-procedural, field-to-field result mapping, no service composition, no per-row enrichment, no post-loop attachments). keep = at least one structural KEEP signal present (multi-SP/multi-service composition, per-row N+1, procedural T-SQL, non-standard joins, post-loop attachments, conditional-flow SP dispatch, transitive delegate-is-KEEP) OR ambiguous case (default-to-KEEP). This is the primary selector axis for codegen idiom-map keying (glaze.migration.codegen.framework_specific_hints[<target_stack>][<code_translation_mode>])."
          },
          "legacy_method_signature": {
            "type": "string",
            "description": "NEW REQUIRED at v2.0.0 (ADR-035 §1.D4). The source-unit signature as read from the body during Step 5 Body Analysis (e.g. the method-level signature line including parameter list + return type, captured verbatim from the legacy source)."
          },
          "legacy_source_ref": {
            "type": "object",
            "required": [
              "file_path",
              "line_range"
            ],
            "additionalProperties": false,
            "properties": {
              "file_path": {
                "type": "string",
                "description": "Absolute or repo-relative path to the legacy source file containing the source unit."
              },
              "line_range": {
                "type": "string",
                "pattern": "^\\d+-\\d+$",
                "description": "Line range as 'start-end' (inclusive) identifying the source unit within the file."
              }
            },
            "description": "NEW REQUIRED at v2.0.0 (ADR-035 §1.D4). Body-read provenance — downstream codegen + Code Reviewer can re-read the body for grounding if needed."
          },
          "legacy_method_body_summary": {
            "type": "object",
            "required": [
              "primary_sp",
              "pre_loop_service_calls",
              "in_loop_service_calls",
              "in_loop_filter_attachments",
              "post_loop_attachments",
              "empty_recordset_branch",
              "tsql_procedural_signals",
              "dto_field_list"
            ],
            "additionalProperties": false,
            "properties": {
              "primary_sp": {
                "anyOf": [
                  {
                    "type": "object",
                    "required": [
                      "name",
                      "params"
                    ],
                    "additionalProperties": false,
                    "properties": {
                      "name": {
                        "type": "string",
                        "description": "The stored-procedure name (verbatim from the body, NOT from any catalog)."
                      },
                      "params": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "required": [
                            "name",
                            "value_expr"
                          ],
                          "additionalProperties": false,
                          "properties": {
                            "name": {
                              "type": "string"
                            },
                            "value_expr": {
                              "type": "string",
                              "description": "The argument expression as written in the body (e.g. a literal, a variable name, or '@input.<field>' for caller-supplied)."
                            }
                          }
                        }
                      }
                    }
                  },
                  {
                    "type": "null",
                    "description": "Null when the body has no SP invocation (e.g. direct ORM/query-builder root)."
                  }
                ],
                "description": "Primary stored-procedure invocation as observed in the body. Source of truth for the SP name; downstream codegen does NOT consult any SP-name catalog (catalogs may be stale)."
              },
              "pre_loop_service_calls": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": [
                    "service",
                    "method",
                    "args",
                    "result_var"
                  ],
                  "additionalProperties": false,
                  "properties": {
                    "service": {
                      "type": "string"
                    },
                    "method": {
                      "type": "string"
                    },
                    "args": {
                      "type": "array",
                      "items": {
                        "type": "string"
                      }
                    },
                    "result_var": {
                      "type": "string",
                      "description": "Local variable name the result is bound to in the body."
                    },
                    "dependent_sp": {
                      "anyOf": [
                        {
                          "type": "object",
                          "required": [
                            "name",
                            "params"
                          ],
                          "additionalProperties": true,
                          "properties": {
                            "name": {
                              "type": "string"
                            },
                            "params": {
                              "type": "array"
                            }
                          }
                        },
                        {
                          "type": "null"
                        }
                      ],
                      "description": "Dependent-SP body-read result for this service-method call (depth-1 recursion per ADR-035 §1.D2 CS-357 Round-3 extension). Mirrors primary_sp shape. Codegen uses dependent_sp.name verbatim. null if depth exceeded or method body unresolvable."
                    }
                  }
                },
                "description": "Service-method invocations occurring BEFORE the row-mapping loop. Each entry is a separate data fetch the composition shell must orchestrate at emit time."
              },
              "in_loop_service_calls": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": [
                    "service",
                    "method",
                    "args",
                    "result_field",
                    "row_ref_arg"
                  ],
                  "additionalProperties": false,
                  "properties": {
                    "service": {
                      "type": "string"
                    },
                    "method": {
                      "type": "string"
                    },
                    "args": {
                      "type": "array",
                      "items": {
                        "type": "string"
                      }
                    },
                    "result_field": {
                      "type": "string",
                      "description": "DTO field the result is assigned to inside the row-mapping loop."
                    },
                    "row_ref_arg": {
                      "type": "string",
                      "description": "Argument expression referencing the row (e.g. 'item[\"id\"]', '@item.id')."
                    },
                    "dependent_sp": {
                      "anyOf": [
                        {
                          "type": "object",
                          "required": [
                            "name",
                            "params"
                          ],
                          "additionalProperties": true,
                          "properties": {
                            "name": {
                              "type": "string"
                            },
                            "params": {
                              "type": "array"
                            }
                          }
                        },
                        {
                          "type": "null"
                        }
                      ],
                      "description": "Dependent-SP body-read result for this service-method call (depth-1 recursion per ADR-035 §1.D2 CS-357 Round-3 extension). Mirrors primary_sp shape. Codegen uses dependent_sp.name verbatim. null if depth exceeded or method body unresolvable."
                    }
                  }
                },
                "description": "Service-method invocations INSIDE the row-mapping loop. Each entry is an N+1 antipattern instance — codegen preserves it at v0.1.0 (with a TODO comment for batch optimisation; followup ticket)."
              },
              "in_loop_filter_attachments": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": [
                    "result_var",
                    "filter_predicate",
                    "target_field"
                  ],
                  "additionalProperties": false,
                  "properties": {
                    "result_var": {
                      "type": "string",
                      "description": "Pre-loop-fetched result variable being filtered per row."
                    },
                    "filter_predicate": {
                      "type": "string",
                      "description": "Filter expression (e.g. 'o => o.workOrderTypeId == item.id' or the equivalent verbatim from the body)."
                    },
                    "target_field": {
                      "type": "string",
                      "description": "DTO field the filtered subset is assigned to."
                    }
                  }
                },
                "description": "In-loop filter-attachments that wire pre-loop batches onto each row. Preserves the legacy 'fetch-then-filter' composition pattern."
              },
              "post_loop_attachments": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": [
                    "target_path",
                    "service",
                    "method",
                    "args"
                  ],
                  "additionalProperties": false,
                  "properties": {
                    "target_path": {
                      "type": "string",
                      "description": "Assignment target (e.g. 'lst[0].fieldX')."
                    },
                    "service": {
                      "type": "string"
                    },
                    "method": {
                      "type": "string"
                    },
                    "args": {
                      "type": "array",
                      "items": {
                        "type": "string"
                      }
                    },
                    "dependent_sp": {
                      "anyOf": [
                        {
                          "type": "object",
                          "required": [
                            "name",
                            "params"
                          ],
                          "additionalProperties": true,
                          "properties": {
                            "name": {
                              "type": "string"
                            },
                            "params": {
                              "type": "array"
                            }
                          }
                        },
                        {
                          "type": "null"
                        }
                      ],
                      "description": "Dependent-SP body-read result for this service-method call (depth-1 recursion per ADR-035 §1.D2 CS-357 Round-3 extension). Mirrors primary_sp shape. Codegen uses dependent_sp.name verbatim. null if depth exceeded or method body unresolvable."
                    }
                  }
                },
                "description": "Assignments to specific elements of the result list AFTER the row-mapping loop. Each entry is a single-row attachment (typically attaching to lst[0])."
              },
              "empty_recordset_branch": {
                "type": "object",
                "required": [
                  "present",
                  "legacy_returns"
                ],
                "additionalProperties": false,
                "properties": {
                  "present": {
                    "type": "boolean"
                  },
                  "legacy_returns": {
                    "type": "string",
                    "enum": [
                      "empty-array",
                      "stub-with-attachments"
                    ],
                    "description": "What the LEGACY method returns on empty primary recordset. 'stub-with-attachments' is recorded-and-overridden per ADR-035 §3 — codegen IGNORES it and emits the modern stack's empty-list convention (typically bare empty array)."
                  },
                  "stub_fields": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "required": [
                        "target",
                        "service",
                        "method",
                        "args"
                      ],
                      "additionalProperties": false,
                      "properties": {
                        "target": {
                          "type": "string"
                        },
                        "service": {
                          "type": "string"
                        },
                        "method": {
                          "type": "string"
                        },
                        "args": {
                          "type": "array",
                          "items": {
                            "type": "string"
                          }
                        }
                      }
                    },
                    "description": "Populated only when legacy_returns='stub-with-attachments'. The list of legacy stub field-attachments. RECORDED-AND-OVERRIDDEN — codegen does NOT emit these (ADR-035 §3 deliberate divergence)."
                  }
                },
                "description": "Empty-recordset branch provenance. Captures what the legacy method does on empty primary recordset, for downstream traceability. Codegen overrides stub-with-attachments cases per the modern empty-list convention."
              },
              "tsql_procedural_signals": {
                "type": "array",
                "items": {
                  "type": "string"
                },
                "description": "Disqualifying T-SQL signals found in SP body (e.g. 'cursor-declared', 'multi-recordset', 'parametric-dynamic-sql', 'outer-apply'). Or ['sp-source-unavailable'] if the SP source could not be read. Drives the MOVE-or-KEEP rule criterion #2."
              },
              "dto_field_list": {
                "type": "array",
                "items": {
                  "type": "string"
                },
                "description": "Per-row field assignments observed in the result-mapping loop. The downstream codegen consumes this as the canonical DTO shape (instead of inferring from a separate model file)."
              }
            },
            "description": "NEW REQUIRED at v2.0.0 (ADR-035 §1.D4). Structured analysis output from Step 5 Body Analysis. Downstream codegen consumes this verbatim to emit the multi-file scaffold mechanically (no re-reading of the legacy body)."
          },
          "bridge_strategy": {
            "type": "string",
            "description": "DERIVED at v2.0.0 (ADR-035 §1.D3) from code_translation_mode + consumer-context: 'move' → 'dark-launch'; 'keep' → 'parallel-sp'. One of glaze.migration.supported_bridge_strategies (the enum range is retained for documentation completeness; only the two derived values are produced at v0.2.0). Downstream test-writer + doc-writer consume this for runbook framing; their contract is preserved across the v1.x → v2.0 boundary."
          },
          "cutover_mode": {
            "type": "string",
            "description": "Cutover mode (one of glaze.migration.supported_cutover_modes — typically dual-write, single-write-modern, read-only-legacy, decommissioned)."
          },
          "rollback_strategy": {
            "type": "string",
            "description": "Rollback strategy (one of glaze.migration.supported_rollback_strategies — typically snapshot, replay-log, manual-runbook, none). Required for cutover_mode != decommissioned."
          },
          "legacy_context_status": {
            "type": "string",
            "description": "Current legacy state (one of glaze.migration.supported_legacy_context_statuses — typically active, dual-running, verified-orphan, decommissioned)."
          },
          "up_sql": {
            "type": [
              "string",
              "null"
            ],
            "description": "Spec-level pseudo-SQL describing the forward migration step. Null for non-SQL migrations (e.g. endpoint route repointing). Dialect-natural rendering is code-writer's job."
          },
          "down_sql": {
            "type": [
              "string",
              "null"
            ],
            "description": "Spec-level pseudo-SQL describing the rollback step. Required when cutover_mode != decommissioned. Optional when cutover_mode == decommissioned (one-way migration with no rollback)."
          },
          "depends_on": {
            "type": "array",
            "items": {
              "type": "string",
              "pattern": "^M\\d+$"
            },
            "description": "Peer M-ids this target requires first (cutover ordering — e.g. tables before procedures that reference them)."
          },
          "decommission_target_date": {
            "type": [
              "string",
              "null"
            ],
            "format": "date",
            "description": "Optional ISO date. When set, indicates the legacy unit will be removed from production by this date. Mutual exclusion with parallel_consumers[].length > 0."
          },
          "parallel_consumers": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Array of consumer identifiers (other modules / external callers) that depend on the legacy unit during dual-running. Empty/absent when legacy_context_status: decommissioned."
          },
          "estimated_hours": {
            "type": [
              "number",
              "null"
            ],
            "description": "Effort estimate. Propagated from upstream Architect / Backend / Database when available; null when corpus-only."
          },
          "test_requirements": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Migration-specific test scenarios (e.g. dual-write consistency checks, rollback validation). Propagated from ArchResult.components[].test_requirements when present, expanded with migration-specific scenarios."
          },
          "runbook_requirements": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "List of runbook concerns for doc-writer (e.g. document cutover-day go/no-go checks for module X)."
          }
        }
      }
    },
    "interface_contracts": {
      "type": [
        "object",
        "null"
      ],
      "description": "Opaque pass-through from ArchResult.interface_contracts when present. Null when running with --no-arch-context."
    },
    "advisories": {
      "type": "array",
      "description": "Non-blocking observations. types: scope_caveat | capability_substitution | shape_unresolvable | module_status_override | non_migration_components_only | missing_optional_input | tooling_resolution_required | pending_human_review | module_unresolvable.",
      "items": {
        "type": "object",
        "required": [
          "type",
          "message"
        ],
        "properties": {
          "type": {
            "type": "string",
            "enum": [
              "scope_caveat",
              "capability_substitution",
              "shape_unresolvable",
              "module_status_override",
              "non_migration_components_only",
              "missing_optional_input",
              "tooling_resolution_required",
              "pending_human_review",
              "module_unresolvable",
              "body_analysis_overrode_upstream_sp_name",
              "code_translation_mode_keep_required",
              "code_translation_mode_move_eligible",
              "deliberate_empty_recordset_divergence",
              "source_unit_unresolvable",
              "default_to_keep_unverified_sp",
              "body_size_exceeded",
              "overload_disagreement",
              "dynamic_dispatch_observed",
              "dependent_sp_depth_exceeded",
              "dependent_is_keep_composition"
            ]
          },
          "message": {
            "type": "string",
            "description": "Human-readable description of the advisory."
          },
          "path": {
            "type": "string",
            "description": "Optional. M-id, module slug, or glaze path the advisory applies to."
          }
        }
      }
    },
    "substitutions": {
      "type": "array",
      "description": "Capability-gating substitution records (per ADR-009 Decision 1). Each entry records a requested-capability → substituted-capability swap with rationale.",
      "items": {
        "type": "object",
        "required": [
          "module",
          "requested_shape",
          "substituted_shape",
          "reason"
        ],
        "properties": {
          "module": {
            "type": "string",
            "description": "Module slug the substitution applies to."
          },
          "requested_shape": {
            "type": "string",
            "description": "Originally-requested shape from glaze.migration.supported_shapes."
          },
          "substituted_shape": {
            "type": "string",
            "description": "Substituted shape used for actual migration_target emission."
          },
          "reason": {
            "type": "string",
            "description": "Substitution reason (e.g. bridge_driver_capability_missing, dialect_unsupported)."
          }
        }
      }
    },
    "capability_flags": {
      "type": "array",
      "description": "Capability declarations exposed by Migration for downstream consumers (per ADR-009). Advertises capability availability without forcing adoption.",
      "items": {
        "type": "object",
        "required": [
          "capability",
          "force_default",
          "match_adjacent"
        ],
        "properties": {
          "capability": {
            "type": "string",
            "description": "Capability name (e.g. escrow_migration_enabled, dual_write_verification, snapshot_rollback)."
          },
          "force_default": {
            "type": "boolean",
            "description": "When true, downstream MUST adopt this capability default. When false, capability is advertised but optional."
          },
          "match_adjacent": {
            "type": "boolean",
            "description": "When true, downstream should match adjacent module convention. When false, downstream uses its own default."
          }
        }
      }
    },
    "skipped_modules": {
      "type": "array",
      "description": "Modules where the feature has no migration impact (with rationale).",
      "items": {
        "type": "object",
        "required": [
          "module",
          "reason"
        ],
        "properties": {
          "module": {
            "type": "string",
            "description": "Module slug skipped from migration_targets."
          },
          "reason": {
            "type": "string",
            "description": "Why this module was skipped (e.g. modern_only, verified_not_built, decommissioned)."
          }
        }
      }
    },
    "legacy_context_advisories": {
      "type": "array",
      "description": "Observations from legacy-corpus reads (e.g. cutover_deadline_passed, verified_not_built_module). Separate from advisories[] — these document corpus observations without competing for the max_advisories scope cap.",
      "items": {
        "type": "object",
        "required": [
          "type",
          "message"
        ],
        "properties": {
          "type": {
            "type": "string",
            "enum": [
              "cutover_deadline_passed",
              "verified_not_built_module",
              "decommission_target_passed",
              "audit_artifact_stale",
              "trace_unmapped_endpoint"
            ]
          },
          "message": {
            "type": "string"
          },
          "path": {
            "type": "string",
            "description": "Optional. Glaze path or corpus path the advisory applies to."
          }
        }
      }
    },
    "clarification_questions": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Populated only when status === 'needs_clarification'. Specific questions the human must answer to unblock the migration spec."
    },
    "feedback_iteration_context": {
      "type": "object",
      "description": "Populated only when invoked with --prior-findings CLI flag (loop-back from CS-143 Code Reviewer Lobe per ADR-021). Optional; absent on first-run invocations.",
      "required": [
        "is_loop_back",
        "prior_findings_consumed_count"
      ],
      "properties": {
        "is_loop_back": {
          "type": "boolean",
          "description": "True iff this run was triggered by CS-143 loop-back."
        },
        "prior_findings_consumed_count": {
          "type": "integer",
          "minimum": 0,
          "description": "Number of prior findings ingested in Step 1 (Perceive)."
        },
        "iteration_id": {
          "type": "string",
          "description": "Optional. CS-143 run_id. Empty string permitted."
        },
        "locked_context_ref": {
          "type": "string",
          "description": "Optional. Path to γ-payload locked_context. Empty string permitted."
        }
      },
      "additionalProperties": false
    },
    "blocked_reason": {
      "type": "string",
      "description": "Populated only when status === 'blocked'. Specific, fixable description of the blocker."
    },
    "error": {
      "type": "object",
      "description": "Populated only when status === 'error'.",
      "required": [
        "type",
        "message"
      ],
      "properties": {
        "type": {
          "type": "string",
          "enum": [
            "missing_feature_id",
            "invalid_intent",
            "archresult_parse_failed",
            "backendresult_parse_failed",
            "databaseresult_parse_failed",
            "glaze_parse_failed",
            "schema_validation_failed",
            "incompatible_arch_schema",
            "incompatible_backend_schema",
            "incompatible_database_schema",
            "missing_required_input",
            "no_upstream_context",
            "internal_error"
          ]
        },
        "message": {
          "type": "string"
        },
        "partial_state": {
          "type": "object",
          "description": "Best-effort partial MigrationResult if some processing completed before the error."
        }
      }
    }
  }
}
```
<!-- APPENDIX-A-END -->
