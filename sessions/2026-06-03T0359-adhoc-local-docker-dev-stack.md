---
title: "Local Docker dev stack built + shipped to Colobbo/colobbo-devstack (NEW private repo) — compose + dual-DB seed + onboarding docs + secret-free env template; legacy .NET found Windows-only (remote pointer, not a container); notification-service ESM/tsx fix"
tags: [bridge, session, adhoc, local-dev-stack, colobbo-devstack, docker-compose, db-seed, mssql, postgres, redis, notification-service, onboarding, env-template, secrets-hygiene, three-bucket-env-merge, dotnet-framework-4.8-windows-only, tsx-esm, design-tokens, verify-first, no-ticket-adhoc]
last_updated: 2026-06-03
source: memory
confidence: high
lobe_version: 1.0.0
status: active
---

# Local Docker dev stack — built, verified, shipped to `Colobbo/colobbo-devstack` (adhoc — no anchoring ticket)

## Summary

Multi-day verify-first session (spanning 2026-06-01 → 2026-06-03) that scoped, built, and shipped a complete **local Docker dev stack** so the team can run the Colobbo product on their own machines for free — one `docker compose up` brings up web-angular + api-node + Postgres + MSSQL + Redis + notification-service. Work lives in a **NEW private repo `Colobbo/colobbo-devstack`** (created via `gh`, pushed) plus **local-only Dockerfiles** in the product repos. **Zero work touched colobbo-agent-system source** (this bridge entry is the only agent-system artifact) — the session is recorded here because cibo-handoff is the operator's session journal. No single Linear ticket anchors it (adhoc; `linked_issue` omitted per the adhoc-bridge precedent).

Every phase ran **verify-first → propose → operator-approved → execute**, with the operator repeatedly requesting "verify against the actual files, don't guess."

## What was done (by phase)

1. **Phase-0 verification + Phase-1 proposal (scoping).** Read the real repos instead of trusting the brief. Corrected a load-bearing premise: **api_node is NOT an Nx monorepo** (no nx.json/project.json; plain two-app folder under `apps/`). Reconstructed the full env contract from source (`grep process.env` + configs): two DB sets (`DB_*`=MSSQL, `POSTGRES_*`=Postgres), Redis, Cognito (api-node verifies tokens with `AWS_REGION`+`COGNITO_USER_POOL_ID` JWKS), S3/SendGrid/OpenAI/Sentry/Intercom/Pipedream-Xero. Flagged: web-angular `src/app/environments/` is **gitignored/absent**; api-node `pgDatabase.ts` **hardcodes PG SSL on** (local-PG blocker); committed live secrets (RDS creds in api_node BACKEND-SETUP.md + DATABASE-SCHEMA.md; AWS key id in web-angular PLACEHOLDER-SUMMARY.md).

2. **Dual-DB seed mechanism (the #1 risk, retired).** Verified the MSSQL schema source EXISTS on disk: git-tracked `Colobbo-Web/IPMDatabase/` SSDT project (337 tables + ~700 procs + 3 functions + 1 type); the untracked `Colobbo-Web/db/` dump is dup-polluted/local-only (not used). Built loaders: **Postgres** via `drizzle-kit push` (30-table api-node drizzle model) → synthetic edge-case seed; **MSSQL** via multi-pass FK-ordered load (TYPE → tables → functions → procs; **337/337 tables, 674/700 procs** — the 26 skips are linked-server/SSDT-drift, expected) → synthetic seed. Synthetic data (sentinel tenant 9001) covers the 4 named edge cases (DISTINCT fan-out, multi-page paging, null contractor, missing `ans`). Verified end-to-end on real containers (Postgres arm64-native; MSSQL 2019 amd64-emulated on this Apple Silicon Mac).

3. **Full compose stack.** `docker-compose.yml` (7 services incl. one-shot `mssql-seed`; Postgres self-seeds via initdb; MSSQL seeds via compose-native `mssql-seed` entrypoint reading the `/ipmdb` mount), per-repo `Dockerfile.dev`s, `env.example`, and a `gen-web-env.sh` that generates web-angular `environment.dev.ts` from `.env` at container start. **Key finding: the legacy .NET API (`IPMProj`/`IPMMasterAPI`) targets .NET Framework v4.8 → Windows-only → NOT containerisable on macOS.** Did not fabricate a broken container; wired it as a **remote-dev-.NET pointer** (`DOTNETAPI_URL`/`WEB_DOTNET_API_URL`). Verified: `docker compose config` valid; data tier self-seeds via pure `docker compose up`; all 3 app images build (fixed `npm ci`→`--legacy-peer-deps` for api-node/web-angular, `npm install` for lockfile-less notification-service).

4. **`.env` 3-bucket merge (Jignesh backend + Jaydeep frontend).** Merged real secrets WITHOUT spreading them or breaking local: **local-pinned** DB/Redis/host values + DB creds (Jignesh's remote DB creds deliberately ignored — local containers were initialised with the throwaway creds); **took** Jignesh's real Cognito/WOMS/AWS/SendGrid/OpenAI/Sentry/Pipedream secrets; **filled** the 4 frontend Cognito endpoints from Jaydeep's `cognitoConfig.awsBaseURL` (`https://demoid.colobbo.com`) — verified in code that web-angular *derives* token/userInfo/login from the domain (`endpoints.ts`), so only the domain is load-bearing. All via key-name-only diffing + automated leak-checks; secrets never printed.

5. **notification-service ESM/tsx fix.** Diagnosed (not guessed): the service is ESM (`"type":"module"` + `NodeNext`) so plain `ts-node` fails on Node 20 (`ERR_UNKNOWN_FILE_EXTENSION`); api-node works only because it's CommonJS. Fixed in `Dockerfile.dev` (install `tsx`, `CMD npx tsx watch src/app.ts`) — booted clean on :4000. Also caught + fixed a stale anonymous `node_modules` volume shadowing the baked tsx (`--renew-anon-volumes`).

6. **Onboarding docs.** `docs/ONBOARDING.md` (source-of-truth, every command/number verified against the actual files) + a self-contained `docs/onboarding.html` for the team portal, styled with the **real web-angular design tokens** (verified: `styles.scss :root --primary-color #7C5CFF` + accents, `tailwind.config.js` Inter/Poppins, `design-system.ts` neutrals/radius) — contents nav, copy-to-clipboard, prominent ⚠️ "known current gaps" callout. Rendered + screenshot-verified in-browser. README links the guide.

7. **Secret-free distributable env template.** `dev.env.template` (committed, placeholders only — leak-checked twice, zero real secrets): safe-local verbatim, `__FILL__` for the 11 dev secrets, blank for high-value secrets (AWS/SendGrid/OpenAI/Sentry/Intercom/Pipedream/Xero/HubSpot), `JWT_SECRET`→`local-dev-secret`. `.gitignore` now ignores `.env`+`dev.env`, keeps `dev.env.template` tracked. Docs document "ask your manager for their filled dev `.env`" or self-serve from the template.

## Files / commits

- **`Colobbo/colobbo-devstack`** (new private repo, 6 commits this session): `380cf61` scaffold (compose + seed) · `68948ee` POSTGRES_SSL→DATABASE_SSL rename · `b6fb2e0` README onboarding link · `cc76a31` ONBOARDING.md · `e8d4ffd` onboarding.html · `cfba93d` dev.env.template + manager-env flow. Repo: https://github.com/Colobbo/colobbo-devstack
- **Local-only, intentionally uncommitted** (held until the stack boots clean E2E): `Dockerfile.dev` + `.dockerignore` in `repos/api_node/apps/api-node`, `repos/api_node/apps/notification-service`, `repos/web-angular`; the merged `colobbo-devstack/.env` (gitignored).
- **No colobbo-agent-system source changed** (besides this handoff's bridge/plan/glaze records).

## Verification state

- ✅ Data tier (postgres :5433, mssql :1434, redis :6379) + **notification-service :4000** — up and verified; both DBs seeded.
- ✅ `docker compose config` valid; all 3 app images build.
- ⛔ **api-node ↔ local Postgres** blocked on Jignesh's `pgDatabase.ts` `DATABASE_SSL` toggle PR (not merged).
- ⛔ **web login** blocked on registering `http://localhost:4200/auth/login` as a Cognito callback URL.
- (Per operator: don't boot api-node until the SSL PR lands — no new info otherwise.)

## What's next

- **2 gaps to close** (then remove the gaps box from BOTH `docs/ONBOARDING.md` and `docs/onboarding.html` + the `#gaps` nav link): (1) Jignesh `DATABASE_SSL` PR merged; (2) Cognito `localhost:4200/auth/login` callback registered. Trigger = a clean `docker compose up` confirming api-node connects + web login works.
- **Dockerfile PRs** to the 3 product repos held until the stack boots clean E2E (api_node → development/@jthakkar1981; web-angular → development-stage/@pjtech18).
- **Parked cleanup:** gitignore product-repo `CLAUDE.md` in web-angular (it tracks CLAUDE.md so deploy.sh writes show as drift; api_node already gitignores `CLAUDE*.md` at `.gitignore:24` — mirror that). Spot-check Colobbo-Mobile/Colobbo-Web for the same.
- **Security watchpoints (rotate):** committed live RDS creds in `api_node/BACKEND-SETUP.md` + `DATABASE-SCHEMA.md`; AWS access-key id in `web-angular/PLACEHOLDER-SUMMARY.md`; Google Maps dev key + Metabase secret + Cognito client secret in the Jaydeep env file (delete `~/Desktop/Colobbo/jaydeep.dev.ts` + `jignesh.env` if not already).
- **First real onboarding run** by a new dev is the true test of the docs + dev.env.template.

## See also

- [[2026-06-01T0706-adhoc-migration-worker-and-cibo-migrate|prior bridge entry — migration-worker lessons + cibo-migrate skill]]
- [[../../strategy/active-plan|active-plan]] — updated this session (local-dev-stack added as a parallel track)
- [https://github.com/Colobbo/colobbo-devstack](https://github.com/Colobbo/colobbo-devstack) — the shipped repo (private)
