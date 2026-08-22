---
title: "MCP readiness audit — api_node surface mapped, 6 v1 tools proposed, auth verdict settled, 3 cross-tenant reads filed"
tags: [bridge, session, mcp, api-surface, audit, read-only, api-keys, col-137, col-612, col-649, col-650, col-651, cs-418, tenancy, pii]
last_updated: 2026-08-22
source: memory
confidence: high
lobe_version: 1.0.0
status: active
---

## Summary

Analysis-only audit of the `api_node` API surface ahead of a proposed read-only MCP server for internal ops. Ran end-to-end in one session: verify-by-grep surface map → two-gate selection model → 6 proposed v1 tools → auth assessment against the locked COL-498/COL-499 Cognito architecture → risk register. Deliverable is one wiki page, `wiki/architecture/mcp-readiness-audit.md`, plus three urgent Linear findings and a correction comment on CS-418.

No Linear ticket anchors this work (the brief was operator-issued as "CS [THINK]" with no number). Recorded as an `adhoc-` entry per Step 0; `linked_issue` omitted — Gate 12 pattern-checks the field when present, so omission is correct, never `null` (same reasoning as [[2026-05-29T1422-adhoc-cibo-new-lobe-skill|the cibo-new-lobe entry]]).

**Read-only throughout.** No production or staging system contacted, no credentials used, no live database queried. `api_node` was left byte-identical to session start — same branch (`fix/notification-inapp-template-id`), same 5 untracked files.

## The structural finding

**api_node has two independent authorisation gates, and a route can pass one while failing the other.** This drove every accept/reject in the audit and is the reusable part.

1. **RBAC action check** — fires only for routes in `src/permissions/routeManifest.ts`. Its own header: *"Routes absent from routeManifest skip the permission check entirely — no warning, no 403."* 721 entries against a surface of **855 routes** (416 GET / 248 POST / 86 PUT / 83 DELETE / 22 PATCH).
2. **Tenant row filter** — opt-in per controller via `req.permissionScope` or `assertCompanyAccess()`. Not global middleware; ~14 non-test files consume it. `recordScopeMiddleware` *resolves* scope for every request but explicitly *"never blocks a request"* (`index.ts:182-183`).

Selection rule that fell out: a candidate must pass **both** gates, not be on `publicRouteAllowlist`, and carry no CS-418 injection vector. That cut 416 GET routes to six tools.

The model earned its keep immediately — two drafted tools (`claimprogress`, bill/invoice `payments`) pass Gate 1 and fail Gate 2, and were dropped on verification. Both became filed findings.

## Auth verdict

**Yes — a least-privilege read-only token is mintable today, with no new auth design.** The COL-137 API-key mechanism (`middleware/apiKeyMiddleware.ts`) is more complete than expected: a real `read_only` scope blocking all write methods (Step 9), per-app rate limiting (`rateLimitPerMin` default 60/min, independent of the disabled global limiters), brute-force lockout, expiry + 24h rotation grace, SHA-256 hashing, full audit logging to `api_key_logs`, and a complete management lifecycle that is itself RBAC-gated (`routeManifest.ts:642-651`, `full_access` owner-only at `api-apps.controller.ts:57`).

It **composes with** rather than replaces the locked Cognito direction — that governs human sessions; a machine read-only key is a separate lane.

Four caveats recorded: the `API_KEY_AUTH_ENABLED` flag is blank in `.env.example:211` and set nowhere; `read_only` is **method**-scoped not **endpoint**-scoped; the mechanism was never security-reviewed (the lone API-key mention in [[../../architecture/auth-audit|auth-audit]] is about the booking flow *lacking* one); per-tenant means N keys.

**Prerequisite surfaced, deliberately not filed** (per brief): an `apiApps.allowedEndpoints` column checked alongside the method check, so a key scopes to the six tool paths rather than every GET. Recommended before exposing claims.

## Operator decisions taken mid-session

Two premises in the brief did not survive verification; both were surfaced and settled before writing:

1. **Claims are not injury-related.** `modules/claim/claim.schema.ts` is purely financial (`totalClaimAmt`, `totalGstAmt`, retention, approve/reject). A grep for `injury|incident|hazard|swms|near.?miss|workers.?comp` across `apps/api-node/src` returns 9 hits, all the bare word "incident" in error contexts. **Decision: keep ELEVATED, reclassify as commercial-financial.** Separately flagged `modules/inspection/inspection.schema.ts:17` — `templateJson` is an unstructured `json` blob whose per-tenant contents cannot be classified statically; new **UNCLASSIFIABLE** category, excluded from v1.
2. **Tenancy.** Decision: **per-tenant keys, one tenant at a time.** Keeps `scope='organisation'` so the existing tenant predicate applies; cross-tenant `scope='all'` out of scope for v1.

## Proposed v1 tools (6)

| # | Tool | Endpoint | Gates |
|---|---|---|---|
| 1 | `find_jobs` | `GET /api/workorder/getWorkOrderList` | manifest `:28` · JWT-derived scope `workOrderController.ts:888` · ⚠️ `sortDataField` raw |
| 2 | `resolve_job` | `GET /api/workorder/getWorkOrderListForDropdown` | manifest `:29` · JWT-only, **zero query params** |
| 3 | `get_job_summary` | `GET /api/work-orders/summary` | manifest `:41` · `assertCompanyAccess` `:139` · pure Drizzle |
| 4 | `get_job_claims` | `GET /api/claim/:workOrderId/getclaims` · `/claim/get` | manifest `:226-227` · tenant predicate `claim.service.ts:2026`, `:1841` |
| 5 | `list_job_types` | `GET /api/work-order-type/all` | manifest `:84` · `getCompanyId(req)` |
| 6 | `get_customer` | `GET /api/Customer/getCustomerById` | manifest `:355` · JWT companyId — **conditional on PII policy** |

Tool 3 is the cleanest endpoint in the API — `drizzle-orm-mssql` predicates throughout, Zod-validated, COL-307-built. Tool 1 carries a CS-418 vector via `sortDataField`; **mitigation is not to expose it as a tool input**. That generalises: an MCP tool's input schema is itself a control surface — wrapping an endpoint does not oblige exposing its query string.

## Rejected, with evidence

- **Scheduler — the biggest gap.** Nine routes on `publicRouteAllowlist.ts:20-40` bypass apiKey + Cognito + recordScope entirely (`index.ts:161-198`). **An API key is never inspected on them**, so "what's scheduled this week" is not buildable. Needs a new authenticated read endpoint or dual-auth on `Scheduler/Get` — which [[../../architecture/auth-modernization-plan|the modernization plan]] §A8 bend #4 already flags for the staff caller.
- **Activity log** — cross-tenant readable, zero manifest entries → COL-649.
- **Dashboards/reporting** — 3 raw `additionalCriteria` splices, 58 interpolation sites in one file.
- **Customer list / address search** — `filterCriteria` + MSSQL-resident address columns.
- **Inspections/templates** — unclassifiable payload.
- **`work-orders/summary-grid`** — no manifest entry (Gate 1 fails, Gate 2 passes).

## Findings filed

Pattern G before filing surfaced **[COL-612](https://linear.app/colobbo/issue/COL-612)** ("Enforce companyId scoping at middleware on the node API", High/Backlog, Jignesh, project *Security Stuff*) as the correct parent — it explicitly records that *no audit of endpoints added in the last 6 months exists*, and that its staging probe `e2e/cross-company.spec.ts` *"has never returned a verdict — every run reports INCONCLUSIVE because no probe endpoint has yet qualified."* All three findings qualify as probe endpoints. Filed under COL-612, **not** COL-498, per operator instruction.

| Ticket | Finding | Evidence |
|---|---|---|
| [COL-649](https://linear.app/colobbo/issue/COL-649) | Activity-log feed takes `companyId` from the URL path unchecked; **zero** `routeManifest` entries for the whole router | `activity-log.routes.ts:126-129` |
| [COL-650](https://linear.app/colobbo/issue/COL-650) | Claim-progress queries take no `companyId` at all; `SELECT *` | `claim.service.ts:2258` · `claim.controller.ts:165-166` |
| [COL-651](https://linear.app/colobbo/issue/COL-651) | Bill/invoice payment reads ignore the `companyId` column they already **write** | `integrationPaymentService.ts:71,140` · write path at `:82-83` |

All Urgent, assigned Jignesh, labels `api`/`backend`/`security`. COL-649 additionally `relatedTo` COL-139 (analytics company-scoping, Urgent/Triage since March — the three `/analytics/*` routes overlap it).

**[CS-418](https://linear.app/colobbo/issue/CS-418) correction comment posted.** Status is **Backlog, `startedAt: null`** — never started, not "in flight". Contamination map is ~2× the documented surface: `filterCriteria` in **15+** controllers not 8 (adds `rateCard`, `account`, `sorGroup`, `sor`, `currency`, **`user`**, **`scheduler`**); `additionalCriteria` at **3** sites not 1; and **`sanitiseLike()` is not the defence** — it lives only in `modules/skill/skill.service.ts` and no controller in the chain calls it, so that chain has *no* sanitiser. Also named a vector the ticket omits: `sortDataField` on otherwise well-scoped endpoints.

## Verification

- Gates green on every commit: **Gate 12** (frontmatter, 269 pages) · **Gate 13** (wikilinks, 2027 links) · **glaze-shape 26/26 exit 0**.
- Page registered in `glaze.wiki.canonical_pages`; added to [[../../architecture/index|architecture index]].
- Rajeev proposal section grepped clean of internal vocabulary (`raku|lobe|glaze|cibo-|ADR-0|CS-|COL-|routeManifest|permissionScope|drizzle|IDOR|api_node|MSSQL` → zero hits across 55 lines).
- Branch-integrity check before any claim: `git diff --stat origin/development..HEAD` on route files confirmed drift limited to an unrelated notification fix, so the audited surface equals `development`.
- `api_node` working tree unchanged from session start (same branch, same 5 untracked files).

## Incidental fix

`wiki/security/secret-rotation-list-2026-08.md` has existed since before this branch and was never registered in `canonical_pages`, so **Gate 1 invariant (k) was already failing on `main`** — and because the script halts at (k), invariants (l)–(z) had not been running. Registered it; all 26 now pass. Verified pre-existing via `git show HEAD:glaze/colobbo.glaze.json` and `git cat-file -e HEAD:<page>` before touching it.

## What's next

1. **Operator decision on the endpoint-allowlist prerequisite** (§5 of the audit page) — recommended before tool 4 ships. Not filed; Pattern G check against COL-478 / CS-415 first if it goes ahead.
2. **Build order if approved:** tools 2, 3, 5 first (low-PII, fully gated) → tool 1 with `sortDataField` pinned → tool 4 after the allowlist → hold tool 6 pending a PII policy call.
3. **Enable `API_KEY_AUTH_ENABLED` in dev** and security-review the COL-137 mechanism (N6) before it becomes an external surface.
4. **Jignesh:** COL-649/650/651 triage. COL-649 should give COL-612's staging probe a real verdict immediately.
5. **Doc drift to fix separately:** [[../../architecture/auth-modernization-plan|auth-modernization-plan]] still lists COL-498 A1 as open; `cognitoService.ts:25-28` shows it landed (`aws-jwt-verify` pinning iss/aud/token_use/RS256).
6. **Scheduler MCP access** stays blocked until a dual-auth or new authenticated read path exists.

## Commits

`a2aa6620fe` audit page + canonical registration + architecture index · `cc27151365` ticket cross-links + pre-existing invariant-(k) fix. Both on `audit/mcp-readiness`. No product-repo commits.

## See also

- [[../../architecture/mcp-readiness-audit|The audit page]] — full evidence with file:line
- [[../../architecture/auth-audit]] · [[../../architecture/auth-modernization-plan]] — the locked Cognito architecture this composes with
- [[../../architecture/col-307-work-order-summary-aggregation]] — how tool 3's endpoint was built
- [[../../raku-meta/state-drift-patterns]] — Pattern G, applied before filing
