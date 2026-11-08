---
title: 'A librarian signs in at test-admin.piviki.fi on the test environment (tracer bullet)'
type: 'feature'
ticket: '1'
created: '2026-11-03'
status: done
baseline_revision: 'piviki-core@ba75aa21e5bea26fad3b220824da4bbc9dd6b8d7 piviki-infra@186bdfa80429e0ca2b6081cb23fec7a5c9456c9b'
route: 'full'
route_source: 'auto'
risk: 'high'
review: 'quick'
review_source: 'pinned'
lenses_ran: ['quick']
review_loop_iteration: 0
context:
  - '{project-root}/_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The tracer bullet is built and the AWS host is bootstrapped as `test`, but the code and OpenTofu still target production names, OpenTofu has one production root that would own both shared and environment resources, and core does not know which environment it runs in.

**Approach:** Re-target story 1.1 to test on top of the merged code (no revert): production's OpenTofu state holds only the shared resources, a reusable `core-ingress` module gives test its Tunnel, Access and DNS, core and the deploy checks refuse host names that disagree with the host's environment name, and the runbooks walk a founder through test's first deploy to a signed-in librarian at `test-admin.piviki.fi`.

## Decisions

- Re-plan 2026-11-05 (user): keep all merged code; plan only the delta for test. Earlier decisions in this plan's history (repo split, AD-21, Bitwarden, checks token removed, Access 72h) stand.
- Production start belongs to `epic-soft-open`; this story creates production's state with shared resources only.
- Folder naming (2026-11-05, user): production's root stays `tofu/prod`; test's is `tofu/test`.
- Module name (2026-11-05, user): `tofu/modules/core-ingress` — one environment's route in to core's host (Tunnel, Access, DNS); the patron site's Worker is not in it.
- The environment name has one source, the host's `/etc/piviki/environment`: `deploy.sh` passes it to the stack; no env file repeats it.
- Host-name agreement (2026-11-05, user, option a): a naming rule, no host list in code. On `test`, every host in `SERVER_URL`, `CORS_ORIGINS`, `CSRF_ORIGINS` (core) and `ADMIN_HOST`/`API_HOST` (checks) has a first label `test` or starting `test-`; on `production`, none does; on `local`, only `localhost` or `*.localhost`.

## Boundaries & Constraints

**Always:**
- AD-17 as amended 2026-11-05: AWS free plan, test only; IaC split (shared resources only in production's state; test's state only test-named resources); per-environment configuration with one environment name per host; hosts `test-admin`, `test-api`, `ssh-test`, all `noindex`, Access on `test-admin`; GitHub Environment `test`; Bitwarden `test` collection.
- AD-21 holds: the change under `deploy/` ships as a core commit and tag; a founder moves the checkout with `--move-core` before deploying it.
- The coming-soon records are imported into production's state with a no-change plan; `piviki.fi` keeps serving the coming-soon page.
- AD-5 order: first librarian seeded over SSH before `test-admin`/`test-api` records exist.

**Never:**
- No production Tunnel, Access apps, DNS records or host (epic-soft-open).
- No shared resource (zone, coming-soon records, one-time PIN login method) in test's state; test never reads production's state or its passphrase.
- No production secret or value in test's GitHub Environment, Bitwarden collection or host files.

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| Hosts agree | environment `test`, `SERVER_URL=https://test-admin.piviki.fi`, origins on test hosts | core starts | — |
| Production host on test | environment `test`, any origin on `admin.piviki.fi` | core refuses to start, names the value | deploy fails at health |
| Test host on production | environment `production`, any origin on a `test` host | core refuses to start | deploy fails at health |
| Checks on wrong names | `checks.sh test` with `ADMIN_HOST=admin.piviki.fi` | check fails before any request | workflow red |
| Test destroyed | `tofu -chdir=tofu/test plan -destroy` | lists only test-named resources | — |

</frozen-after-approval>

## Code Map

- `piviki-core/src/env.ts` -- `loadEnv()` reads origins; add `ENVIRONMENT` and the host-name agreement check (rule: Decisions).
- `piviki-core/deploy/deploy.sh` -- writes `$STATE_DIR/compose.env` (`CORE_TAG=`, line ~125); add `ENVIRONMENT=$host_env`. `host_env` already read from `/etc/piviki/environment`.
- `piviki-core/deploy/compose.yml` -- `core` (line ~46): pass `ENVIRONMENT: ${ENVIRONMENT:?}`; `compose.local.yml`: `local`.
- `piviki-core/deploy/checks.sh` -- `production | test)` branch: add the host-name agreement check on `ADMIN_HOST`/`API_HOST`; `host-checks.mjs` gets the same rule for `caddy.env` hosts.
- `piviki-core/deploy/test/*.test.sh`, `piviki-core/test/env.test.ts` -- extend; CI already runs them.
- `piviki-infra/tofu/prod/` -- shared-only: keep `versions.tf`, `zone.tf`, the OTP identity provider from `access.tf`; add coming-soon record imports; remove Tunnel, Access apps/policies/token, DNS (they move to the module).
- `piviki-infra/tofu/modules/core-ingress/` (new) -- Tunnel + config, Access policies/apps/deploy token, `ssh`/`admin`/`api` records with `public_records_enabled`; inputs: name prefix, hosts, team, emails, zone id, OTP id. Moved from today's `tunnel.tf`, `access.tf`, `dns.tf`, `outputs.tf`.
- `piviki-infra/tofu/test/` (new) -- own backend key and passphrase, calls the module with test hosts; zone id and OTP id are plain variables (from production's outputs).
- `piviki-infra/.github/dependabot.yml` -- terraform dirs `/tofu/prod`, `/tofu/test`, `/tofu/modules/core-ingress`.
- `piviki-infra/docs/{runbook-first-deploy,secrets-inventory}.md`, `README.md` -- production shared state first, then test; GitHub Environment `test`; Bitwarden `test`; test names throughout. `runbook-host-setup.md` -- already run on test; names only.

## Tasks & Acceptance

**Execution:**
- [x] `piviki-core/src/env.ts`, `test/env.test.ts` -- environment name required (`local`/`test`/`production`), host agreement enforced; matrix rows 1–3 as tests
- [x] `piviki-core/deploy/{deploy.sh,compose.yml,compose.local.yml,checks.sh,host-checks.mjs}` + tests -- pass the name through; checks fail on disagreeing hosts (row 4)
- [x] `piviki-infra/tofu/` -- `prod/` shared-only with coming-soon imports; `modules/core-ingress/`; `test/`; `fmt`/`validate` all three
- [x] `piviki-infra/.github/workflows/host-files.yml` -- add a `tofu` job: pinned OpenTofu, `fmt -check` and `validate` on `prod` and `test`
- [x] `piviki-infra/.github/dependabot.yml`, docs, READMEs -- as Code Map
- [ ] Founder: Cloudflare token, R2 bucket, production shared apply, test apply, GitHub Environment `test`, `--move-core`, deploy, seed, records, Verification

**Acceptance Criteria:**
- Given production's state after import, when planned again, then no changes, and `piviki.fi` serves the coming-soon page.
- Given test's applied state, when `plan -destroy` runs, then no zone, coming-soon record or login method is listed.
- Given a manual deploy naming `test` and a commit tag, then the workflow's checks pass and the tag is recorded for `test`.
- Given a founder through Access at `test-admin.piviki.fi`, then sign-in works; `test-api.piviki.fi/api/v1/health` is 200, other api paths 404; first-register 404; one connector; outside scan shows only SSH; core cannot reach IMDS.

## Design Notes

The module keeps test and production's later resources identical by construction; epic-soft-open adds a `module "core_ingress"` call to `tofu/prod`. Test takes the zone id and the OTP id as inputs, so its state never references a shared resource and never needs production's passphrase.

## Verification

**Commands:**
- `pnpm -C piviki-core test && pnpm -C piviki-core typecheck` -- pass
- `bash piviki-core/deploy/test/deploy.test.sh && bash piviki-core/deploy/test/checks-public.test.sh && bash piviki-core/deploy/test/checks-connector.test.sh` -- pass
- local stack + `./checks.sh local` (piviki-core/deploy/README.md) -- pass
- `tofu fmt -check -recursive piviki-infra/tofu && tofu -chdir=… init -backend=false && tofu -chdir=… validate` for `prod` and `test` -- pass

**Manual checks (founder, test):** the story's verify line, item by item, recorded in Implementation Notes.

## Implementation Notes

**Agent work (2026-11-04), verified locally:**
- `pnpm -C piviki-core test` (4 files, 25 tests, real Postgres 18.6 via `embedded-postgres`) and `pnpm -C piviki-core typecheck`: pass. `next build` with no runtime config: passes (standalone output).
- No Docker daemon access on the dev machine, so the Compose stack and the image were **not** run locally. Substitute run: the standalone server on embedded Postgres behind Caddy 2.11.7 using `caddy/Caddyfile`; `deploy/checks.sh local` and `deploy/host-checks.mjs` both passed every I/O-matrix row (first-register 404, api off-allowlist 404 incl. dot-segment/encoded variants, foreign Origin 403, unknown host 404, health 200). Login through Caddy gave `payload-token` `Secure; HttpOnly; SameSite=Lax`, no `Domain`, ~8h expiry, no token in body; an 11-char librarian create returned a validation error; the admin cookie on `api.*` got 404.
- Image layout simulated without Docker: prod-only deps with the Dockerfile's prune step ran `payload migrate` (as the non-superuser role created by `postgres-init/10-core-role.sh`'s SQL) and `seed-librarian` with and without `--force`.
- `docker compose config` on the merged local files: valid. `tofu validate` and `tofu fmt -check` (cloudflare provider 5.28.1): pass. `caddy validate`/`fmt`: pass. `shellcheck` on all shell scripts: clean.

**Deviations from the plan text, and why:**
- Deploy checks run in two places: `deploy.sh` runs the HTTP checks in-network (`host-checks.mjs`, via Caddy with Host headers) on every deploy, and the workflow runs `checks.sh` (public HTTP checks through Access with a separate checks service token, plus the connector count). Reason: the first deploy precedes seeding, so no public `admin`/`api` record exists yet (AD-5); `PUBLIC_RECORDS_ENABLED` skips only the public HTTP part until then.
- Running tag recorded on the host (`/var/lib/piviki/compose.env`, `deploys.log`) and as a GitHub deployment per environment with `payload.tag`.
- Password minimum is a virtual `password` field override (validated on create, update and reset); no plaintext column exists.
- `core` sits only on internal networks (no egress at all); IMDS hop-limit proof is also run from a container on the `edge` network by `bootstrap.sh --check`.
- Zone imported alone; the coming-soon apex/`www` records are deliberately not managed by OpenTofu.

**Not yet verified (needs a founder or a Docker host):** the Docker image build and the Compose stack itself; the bootstrap on a real Ubuntu host; the deploy workflow end to end; the zone import producing no change; Access/Tunnel origin JWT validation on SSH ingress. Founder steps: `piviki-infra/docs/runbook-host-setup.md`, then `runbook-first-deploy.md` §1-8.

**Decision (2026-11-04, user):** Access session on the admin app and the librarians policy raised from 8h to 72h (`piviki-infra/tofu/prod/access.tf`), to reduce sign-in friction: a one-time PIN about every 3 days per device. The Payload librarian session stays about 8 hours (AD-7). AD-7 sets no Access session length, so no spine change. `tofu fmt -check` and `tofu validate` pass with the user's OpenTofu 1.13.2.

**For review (2026-11-04, user agreed): remove the Access checks token.** The `checks` service token gives the deploy workflow a second way through Access on `admin` besides the librarians' one-time PIN. A leak would put the Payload login in reach of anyone, and the same checks (first-register 404, foreign `Origin` 403) already run behind Access on every deploy in `deploy/host-checks.mjs`. Proposed change:
- `tofu/prod/access.tf`: remove the `checks` service token, its policy and its entry on the admin app. `outputs.tf`: remove `checks_service_token`.
- `piviki-core/deploy/checks.sh` (production/test): no Access headers. An unauthenticated `GET https://<admin>/admin` must be a 302 to `*.cloudflareaccess.com` (proves Access is on), and fail on anything that reaches the app. Keep the api checks and the connector count. Keep the first-register and `Origin` checks for `local`.
- `piviki-core/.github/workflows/deploy.yml`: drop `CF_ACCESS_CHECKS_*`. Docs (`secrets-inventory.md`, `runbook-first-deploy.md` §2.3, §4, §7, §8) and `README.md`: remove the checks token.
- Open point: confirm against the real Access at the first deploy that an unauthenticated `curl` gets the 302.

**Repo split and AD-21 applied (2026-11-04, user approved; agent in session).** Backup of both repos before the move: `scratchpad/backup-2026-11-04-pre-move/`.
- Moved Compose, Caddy, the deploy script, both checks, `piviki-compose`, `postgres-init/` and `local-env/` from `piviki-infra` to `piviki-core/deploy/`; added `deploy/env-manifest`, `deploy/README.md`, `.github/CODEOWNERS` (team `piviki-org/admins`, enforced by a `main` ruleset requiring Code Owner review; runbook-first-deploy §1). `deploy` added to core's `.dockerignore`.
- cloudflared, caddy and postgres pinned by tag and multi-arch index digest (looked up on Docker Hub 2026-11-04). Dependabot: `docker-compose` moved from infra `/compose` to core `/deploy`.
- `deploy.sh` locates the checkout from its own path, takes the deploy lock, then refuses a tag when: the checkout is dirty; the tag's commit is not in the checkout; `deploy/` differs between that commit and HEAD; a manifest env file or key is missing. `status` and `deploys.log` now also record the checkout commit.
- `bootstrap.sh`: `--core-commit` (required on the first run) clones `piviki-core` root-owned to `/opt/piviki/core` and checks out the commit after confirming it is on `origin/main`; `--move-core <commit>` moves it under the same lock as a deploy, touching nothing else; sudoers and `ForceCommand` name `/opt/piviki/core/deploy/deploy.sh`; `/usr/local/sbin/piviki-compose` is a symlink into the checkout; env files are created from the manifest (missing keys appended empty, values never touched). New host checks: checkout root-owned and root-writable only, clean, sudoers names the checkout script, env files equal the manifest.
- `deploy.yml`: checks out core (full history), refuses a tag whose commit is not on `main`, runs `deploy/checks.sh` at that commit. The `piviki-infra` checkout and `INFRA_REF` are gone.
- Runbooks: new host-setup §0 (Bitwarden EU organisation), cross-runbook order (first-deploy §1 before host-setup §4), `--core-commit`/`--move-core`, "Later deploys", rollback with a checkout move. Secrets inventory: Bitwarden as source of truth, manifest reference.
- Verified: `deploy.sh` in a sandbox repo with Docker stubbed (same-`deploy/` older commit deploys; HEAD deploys; `deploy/`-changed commit, unknown commit, dirty checkout, missing env key and wrong environment all refused). Bootstrap's `checkout_core` and manifest loop extracted and run against a local bare origin (first clone, move, side-branch commit refused, short id refused, dirty refused; env files 0600, prefilled value kept, idempotent). Found and fixed: the clean-checkout test ran right after `git clone --no-checkout`, which always reports deletions. `shellcheck` (the scratchpad copy), `node --check`, YAML parse, `tofu fmt -check`, `tofu validate`: pass.
- Not verified: the full bootstrap as root on Ubuntu, sudo matching the checkout path, Compose with digest-pinned images (no Docker access), the workflow's checkout step on GitHub.

**Local Docker run (2026-11-04, after the user granted Docker access; Docker 29.1.3, Compose 5.0.1).** From `piviki-core/deploy`: `docker compose config` valid; `piviki-core:local` built (user `node`, 140 MB); digest-pinned postgres and caddy pulled; migration `20261103_221651_initial` applied as `piviki_core`; stack healthy, only Caddy published (`127.0.0.1:8080`, local override). `checks.sh local` and `host-checks.mjs` (run as `deploy.sh` does, on `piviki_edge`): every I/O-matrix row passes. Seed: first run `Created librarian id=1`, second refused (exit 1). Login through Caddy: 200, `payload-token` HttpOnly, SameSite=Lax, Secure, no Domain, expiry 8h, no token in the body. core: uid 1000, read-only rootfs, no outbound (`EAI_AGAIN`), metadata IP unreachable (`ENETUNREACH`).
- **Bug found and fixed:** Caddy crash-looped with `exec /usr/bin/caddy: operation not permitted`. The official image's binary carries the file capability `cap_net_bind_service`; with `cap_drop: [ALL]` it is outside the bounding set, so the kernel refuses the exec. Added `cap_add: [NET_BIND_SERVICE]` to caddy only; its bounding/effective set is now exactly that one capability (`0x400`).
- **For review:** Payload writes the cookie attributes as `Secure=true; HttpOnly=true` rather than bare `Secure; HttpOnly`. Browsers ignore attribute values for these two flags, so they apply; confirm in the browser at production verification item 1.
- **Connector-count gap closed:** `piviki-core/deploy/test/checks-connector.test.sh` runs the real `checks.sh` (production, `SKIP_PUBLIC=1`) with a fake `curl`: 1 connector passes; 0, 2, `success:false`, non-JSON and a curl failure fail; the request names the account/Tunnel connections endpoint with the bearer token. 7/7 pass; a mutated `checks.sh` accepting 1+ is caught. Runs in `build.yml`'s test job and in `build-local.sh` step 7.

**Agent work for the test re-plan (2026-11-06), uncommitted on `feat/1.1-test-environment` in both repos.**
- Core: `src/env.ts` requires `PIVIKI_ENVIRONMENT` (`local`/`test`/`production`) and refuses any `SERVER_URL`/`CORS_ORIGINS`/`CSRF_ORIGINS` host that breaks the naming rule, naming the value (`hostAgrees`). Vitest runs as `local` on `https://admin.localhost` (was `admin.piviki.test`, which the `local` rule refuses). `deploy.sh` writes `PIVIKI_ENVIRONMENT=$host_env` next to `CORE_TAG` and passes it to `host-checks.mjs`; `compose.yml` passes `${PIVIKI_ENVIRONMENT:?}` to core; `compose.local.yml` sets `local`. `checks.sh` and `host-checks.mjs` (also on `ADMIN_ORIGIN`) fail on a disagreeing host before any request, `checks.sh` also under `SKIP_PUBLIC`. New `deploy/test/host-checks-names.test.sh` (in `build.yml`'s test job).
- Deviation: Compose interpolates `compose.yml` before merging `compose.local.yml`, so the local stack (README, `build.yml` stack job) must export `PIVIKI_ENVIRONMENT=local` beside `CORE_TAG=local`; `compose.local.yml` still pins `local`.
- Infra: `tofu/prod` shared-only (zone import; `coming_soon_records` map imported by `for_each`; One-time PIN created, or imported when `otp_identity_provider_import_id` is set; outputs `zone_id`, `otp_identity_provider_id`; `random` provider pin kept for the later module call). `tofu/modules/core-ingress` from the old `tunnel.tf`/`access.tf`/`dns.tf`/`outputs.tf`, names from `var.name`. `tofu/test` (own backend key and passphrase, test hosts validated against the naming rule, lock file copied from prod). `host-files.yml` `tofu` job; Dependabot dirs; runbooks, secrets inventory and README for test. `runbook-host-setup.md` also drops the "paid plan" line for AD-17's free-plan rule.
- Verified: `pnpm test` 38/38, `pnpm typecheck`; `deploy.test.sh`, `checks-public.test.sh`, `checks-connector.test.sh`, `host-checks-names.test.sh` pass, and a mutated `checks.sh`/`host-checks.mjs` that skips the refusal is caught. Local stack rebuilt and run with Docker: `checks.sh local` and `host-checks.mjs` pass, core gets `PIVIKI_ENVIRONMENT=local`; the image with `PIVIKI_ENVIRONMENT=test` on local hosts, or `production` with a `test-admin` `SERVER_URL`, exits 1 naming the value. `tofu fmt -check -recursive` and `init -backend=false` + `validate` on `prod`, `test` and the module (providers from the local cache, lock unchanged). Conditional `for_each` import to a single resource checked in a scratch config.
- Not verified: shellcheck (not installed); the `tofu` CI job (its `.deb` sha256 is the apt-verified local `tofu_1.13.2_amd64.deb`; that it equals the GitHub release asset is unconfirmed, so the first run may fail at the checksum); any plan against Cloudflare (no-change import, `plan -destroy` list); the coming-soon record values (founder copies them from Cloudflare; the plan must show imports only).
- Resume: show the diffs and commit messages to the user, commit on the branches, the user pushes and merges; then the founder steps (runbook-first-deploy §2 to §8, with `--move-core` to the new core commit in §5).

**Parent session after the implementer (2026-11-06).** Re-ran every check above: all pass (vitest 38/38; the four `deploy/test` scripts; `fmt`/`validate` on both roots). Matrix audit: rows 1–3 `test/env.test.ts`, row 4 `checks-public.test.sh` and `host-checks-names.test.sh`; row 5 had no automated test, so added (all run, all pass):
- `tofu/modules/core-ingress/tests/names.tftest.hcl` (mocked providers, offline): every resource is named `piviki-test…`, every record is on a test host and uses the zone id passed in, admin uses the login-method id passed in, no admin/api record before seeding. Mutation-checked (a `piviki-prod` Tunnel name fails it).
- `tofu/test/tests/only-test-resources.tftest.hcl`: the root plans with mocked providers, with and without public records, and refuses a production admin host.
- `host-files.yml` `tofu` job: runs both `tofu test`s, plus a static check that neither `tofu/test` nor the module declares a zone, login method or ruleset, or any `import` (mutation-checked with a planted `import`).
- `TOFU_DEB_SHA256` confirmed equal to the release's `tofu_1.13.2_SHA256SUMS` on GitHub (2026-11-06): no longer a risk.
- `tofu/prod/terraform.tfvars.example`: runbook reference corrected to §2.2.
- Founder-plan scope note: AD-17 also lists "the rate limit rule" as shared; no story 1.1 text asks for it, so it is not created here (for review).
- **Decision (2026-11-06, user): the core variable is `ENVIRONMENT`, not `PIVIKI_ENVIRONMENT`.** Renamed everywhere in piviki-core (`src/env.ts`, tests, `deploy.sh`, Compose files, `host-checks.mjs`, deploy tests, `build.yml`, `.env.example`, `deploy/README.md`) and in `scripts/story-1.1/build-local.sh`; piviki-infra never named it. Notes above that say `PIVIKI_ENVIRONMENT` predate this. Re-verified: vitest 38/38, typecheck, all four `deploy/test` scripts, `docker compose config` (local merge sets it; `compose.yml` alone still refuses a missing value).

**Where story 1.1 stands (2026-11-06, resume point).** Plan `built`; review (quick) done, patches applied, re-verified. Committed on `feat/1.1-test-environment`, not pushed: piviki-core `45308ea1c80b79a1a98cddd8e10c805f2e12aa57`, piviki-infra `f1ac8979ac5fc9a8bc663fb847570c31f29905be`, then `3f8043958fc3e252cf07225116154bf2d76b5392` and `7d8aae9ade9f303120def6ec327febd413168bc3` (state buckets and Cloudflare tokens per environment), then `5db402942b5d3f83d328d10dc0c97d98977bca59` (runbook: why the tokens are split), then `868203bc480ac7c85a8e132e41cec867f5ea8b89` (runbook: token tables, Zero Trust first-time setup), then `ff905fc05b1fe98cff2dd221f6968ae9e34ecc87` (runbook: current Zero Trust paths), then `3ac32aef13a97b8f4c453d01e398b6d849cc76cc` (R2 key pair in Bitwarden, item types; head).
  - Next (user): push both branches, open PRs (titles: Conventional Commits), merge; core `main` build green gives the new tag for `--move-core` and the first deploy.
  - Then (founder): runbook-first-deploy §2 (Cloudflare tokens, state buckets `piviki-prod-tofu-state-eu` and `piviki-test-tofu-state-eu` in the EU jurisdiction with their R2 tokens, a Cloudflare API token per environment, `tofu/prod` shared-only import with a no-change re-plan, `tofu/test` apply and `plan -destroy` record), §3 to §8 on test; record each verify item here.
  - Before closing: Dependabot ignore for `@types/node` semver-major (`chore/1.1-…`); `AGENTS.md` in both repos via `bmad-project-context`; then Sofia marks the ticket done.
  - Open with the founders: the AWS free plan's end date and kind of credits.

**Decision (2026-11-06, user): one private state bucket and R2 token per environment.** Replaces the single `piviki-private-eu` bucket and shared `r2-tofu-state-token`, because an R2 token cannot be limited to a key inside a bucket, so test's work could overwrite or delete production's state. Now `piviki-prod-private-eu` + `r2-tofu-state-token-prod` (Bitwarden `production`) and `piviki-test-private-eu` + `r2-tofu-state-token-test` (Bitwarden `test`); test's bucket is also AD-17's "own private test bucket" for erasure records. Changed in piviki-infra: both `backend.hcl.example`, both `versions.tf` comments, runbook-first-deploy §2.1–2.3, secrets inventory, README (uncommitted at the time of writing). `fmt`/`validate` pass. Wording to align, not edited here: spine AD-17 "one encrypted state per environment in the private bucket" (via `bmad-architecture`), and story 1.1's ticket description "in the private R2 bucket" (via Sofia).

**Decision (2026-11-06, user, after the spine's review): state-only buckets and a Cloudflare API token per environment.** Supersedes the bucket names in the entry above. The spine review found that AD-17's backup bucket lock (7 days) and expiry rule (30 days) would lock and delete a state kept beside the backups, and that the backup and core R2 tokens could overwrite it. Now the state buckets are `piviki-prod-tofu-state-eu` and `piviki-test-tofu-state-eu`, holding nothing else, with no lifecycle or lock rule; backups and erasure records go to `piviki-<env>-private-eu`, created by the stories that write to them. OpenTofu uses `cloudflare-tofu-api-token-prod` (Bitwarden `production`) and `cloudflare-tofu-api-token-test` (Bitwarden `test`). Test's token can still change production's routing, which the spine records as an accepted risk. The spine's AD-16 and AD-17 were updated and finalized through `bmad-architecture` (memlog 2026-11-06). Still to align: story 1.1's ticket text (Sofia).

- 2026-11-06 — runbook-first-deploy §2.1 done (reported by the user): state buckets `piviki-prod-tofu-state-eu` and `piviki-test-tofu-state-eu` (EU jurisdiction), their R2 tokens, `cloudflare-tofu-api-token-prod` and `-test`, `piviki-test-tunnel-read-token`, Zero Trust team name noted. Values in Bitwarden only.

- 2026-11-06 — §2.2 prep: the live `piviki.fi` records are 4 apex A (GitHub Pages, proxied) and `www` CNAME, which go into `coming_soon_records`, plus 3 Email Routing MX, an apex SPF TXT and an apex TXT `1|www.piviki.fi`, which stay unmanaged (not AD-17 shared resources). Noted on story 9 for its SPF and MX work.

- 2026-11-06 — runbook-first-deploy §2.2 done (reported by the user). First plan: 7 to import (zone, 5 coming-soon records, the existing One-time PIN method via `otp_identity_provider_import_id`), 0 to add, 1 to change, 0 to destroy; the change was the PIN method's `name`, which the user reports as identical in Cloudflare (difference not identified). Deviation, user's decision: applied with that one cosmetic change instead of matching the config first. Second plan: **No changes** (AC: no-change plan after import). Outputs `zone_id` and `otp_identity_provider_id` feed `tofu/test`; their values are not tracked here (read them with `tofu -chdir=tofu/prod output`). Coming-soon page check after apply: `piviki.fi` still shows the coming-soon page (reported by the user, 2026-11-06): **verify item met**.

**Resume point (2026-11-06, user at lunch).** Done: §2.1 (buckets, tokens, Zero Trust), §2.2 (`tofu/prod` applied; second plan "No changes"). Next: confirm the coming-soon page with dig/curl; unset prod's credentials; §2.3 in a fresh terminal with test's credentials (`tofu/test`, expect 8 to add, 0 import/change/destroy), then outputs to Bitwarden `test`, `plan -destroy` recorded. Branches still unpushed: piviki-core `45308ea`, piviki-infra head `3ac32ae`. Open: §1's PRs and merge before §5 (`--move-core` needs the new core commit on `main`).

- 2026-11-06 — runbook-first-deploy §2.3 done (reported by the user): `tofu/test` plan 9 to add (random_bytes, Tunnel `piviki-test` and its config, service token `piviki-test-deploy`, 2 Access policies, 2 Access apps, `ssh-test` CNAME), 0 change, 0 destroy; applied. Outputs stored in Bitwarden `test` (`TUNNEL_TOKEN`, `CF_ACCESS_DEPLOY`). Provider warning: a Tunnel config cannot be destroyed on its own (dropped from state; deleted with its Tunnel). `plan -destroy` (not applied) lists exactly 9, all under `module.core_ingress`: `cloudflare_dns_record.ssh`, access applications `admin` and `ssh`, access policies `deploy` and `librarians`, access service token `deploy`, tunnel `env`, tunnel config `env`, `random_bytes.tunnel_secret`. No zone, coming-soon record or login method: **verify item met** ("a tofu/test destroy plan names no shared resource").

- 2026-11-06 — runbook-first-deploy §3 done (reported by the user): `core.env`, `postgres.env`, `caddy.env`, `cloudflared.env` filled on the test host with test host names; `PAYLOAD_SECRET`, `POSTGRES_PASSWORD`, `PIVIKI_CORE_DB_PASSWORD` generated in Bitwarden `test` (48 letters and digits) and pasted with `sudoedit`; no empty value (`grep -cE '^[A-Z_]+=$'` 0 in each). Found: the runbook's empty-value hint must not match base64 `=` padding.

- 2026-11-06 — runbook-first-deploy §4 done (reported by the user): GitHub Environment `test` on piviki-core (deployment branches `main` only), variables `ADMIN_HOST`, `API_HOST`, `SSH_HOST`, `CF_ACCOUNT_ID`, `CF_TUNNEL_ID`, `DEPLOY_SSH_KNOWN_HOSTS`, `PUBLIC_RECORDS_ENABLED=false`, secrets `DEPLOY_SSH_KEY`, `CF_ACCESS_DEPLOY_CLIENT_ID`, `CF_ACCESS_DEPLOY_CLIENT_SECRET`, `CF_TUNNEL_READ_TOKEN`. Next: push and merge both branches, then §5 (`--move-core` to the merged core commit, deploy its tag).

**Resume point (2026-11-06, later).** Fix on 1.1 built and committed: the host compares the image's `fi.piviki.host-runtime` fingerprint with its checkout, so app-only tags deploy with no `--move-core` (plan `plan-1-1-deploy-tree-label.md`). piviki-core `1e11a5a…` on `fix/1.1-deploy-tree-label`, piviki-infra `9debc52…` on `feat/1.1-test-environment`, both unpushed. Done: §2.1–§4. Next: push, PR and merge both; on `main`'s green build note the tag; on the host `--move-core` to core's merge commit; §5 deploy that tag to `test`. Pre-fix images (e.g. `aa64421…`) are refused.

- 2026-11-06 — First §5 deploy of `f67bc3e8a9c0-20261106` failed with `websocket: bad handshake`. cloudflared is in core's Compose stack, which only a deploy starts, so a fresh host has no Tunnel connector. The workaround is to start cloudflared alone on the host (`docker compose --env-file <CORE_TAG/ENVIRONMENT temp file> -f /opt/piviki/core/deploy/compose.yml up -d --no-deps cloudflared`), then deploy again. **Open, the user's decision: the runbook update comes later.** The lasting fix and the mid-deploy cloudflared recreate are handed to Sofia (`handoff-sofia-2026-11-06.md`) for a new story at the end of the epic.

- 2026-11-06 — runbook-first-deploy §5 done (reported by the user): after the manual cloudflared start, the deploy of `f67bc3e8a9c0-20261106` to `test` is green. That means the in-network checks passed (host names, first-register 404, allowlist, health, Origin, unknown host) and so did the runner's one-connector check; the HTTP checks were skipped (`PUBLIC_RECORDS_ENABLED=false`). Run URL: to record. Next: §6, seed the first librarian.

- 2026-11-06 — §6 done (reported by the user): the first librarian was seeded over SSH. The Payload warning "No email adapter" is expected until story 1.9. The piviki-infra README was updated (status, fingerprint rule, first-deploy gap, a "Running OpenTofu" section) in `584b05ef9d80744fcd0a1f2f1bf752357ad577ab` on `docs/1.1-readme-tofu`, not pushed. Next: §7 (`public_records_enabled = true`, plan 2 to add, then the GitHub `PUBLIC_RECORDS_ENABLED=true` and a redeploy), §8 verification, then closing (AGENTS.md, the Dependabot `@types/node` ignore, Sofia marks the story done).

- 2026-11-06 — §7 done (reported by the user): `public_records_enabled = true` applied in `tofu/test`, `PUBLIC_RECORDS_ENABLED=true` set on Environment `test`, and the redeploy of `f67bc3e8a9c0-20261106` is green with the public checks. **Main AC met: a librarian signs in at test-admin.piviki.fi** (Access one-time PIN, then Payload). Run URLs: to record. Next: §8, the remaining verification items.

- 2026-11-06 — §8 checks run from a founder machine. Item 3 passes: `test-api.piviki.fi/api/v1/health` returns 200, and `/`, `/admin`, `/api/librarians`, `/api/graphql`, `/api/users/first-register` return 404. `test-admin.piviki.fi/admin` without Access returns 302 to `piviki.cloudflareaccess.com` (part of item 4). Item 8 passes: `piviki.fi` returns 200 with the coming-soon page, and the apex A records are Cloudflare's proxied addresses. Still to check by the user: item 1 (cookie flags; the `X-Robots-Tag: noindex, nofollow` that Caddy sets, visible only behind Access), item 2 (deployment record tag), item 4 (run log lines), items 5 and 6 (outside scan, metadata service), item 7 (both plans).

- 2026-11-06 — §8 item 2 met: the latest `deploy:image` deployment for `test` (id 7604844878, 16:13:54Z) has payload `{"tag":"f67bc3e8a9c0-20261106"}` and ref `f67bc3e8a9c0`, read from the public GitHub API. Each run also gets an automatic `deploy` deployment from the job's `environment:`, with ref `main` and no tag. The repo's Deployments page shows that one, which is why the tag isn't visible there.

- 2026-11-06 — §8 item 4 met (reported by the user): the 16:13Z run of `f67bc3e8a9c0-20261106` on `test` shows first-register 404 (GET and POST, in-network), `admin is behind Access` and `exactly one connector`, and the public checks were not skipped. Run URL not recorded; it can be found under the `deploy:image` deployment 7604844878.

- 2026-11-06 — §8 item 5 met (user's scan from outside, 18:39 EET, against the instance's public address): the full TCP SYN scan (`nmap -Pn -sS -p-`) found only `22/tcp` open, with 65534 filtered and no response. The UDP top-50 scan got no response on any port (open|filtered, as expected when a firewall drops traffic).

- 2026-11-06 — §8 item 6 met (reported by the user): `bootstrap.sh --check` on the test host shows the metadata-service probe from core's network as PASS, so core cannot reach the instance metadata service.

- 2026-11-06 — §8 item 7 met (reported by the user): `tofu/prod` plan says No changes, and `tofu/test` `plan -destroy` (not applied) lists only test's `module.core_ingress` resources, with no zone, coming-soon record or login method. **§8 complete: every verify item met on test.**

**Resume point (2026-11-06, end of verification).** Story 1.1 has been verified on test. Left before Sofia marks it done:
1. Push and merge piviki-infra `docs/1.1-readme-tofu` (`584b05ef9d80744fcd0a1f2f1bf752357ad577ab`).
2. Add the Dependabot ignore for `@types/node` majors (`chore/1.1-…`).
3. Write `AGENTS.md` in both repos via `bmad-project-context`.
4. Add a runbook step for the first Tunnel start on a fresh host (the user chose to do it later; the README notes the gap).

Handoff to Sofia for the new end-of-epic story: `handoff-sofia-2026-11-06.md`.

- 2026-11-07 — The user asked that the admin host's `/` redirect to `/admin`. Committed on piviki-core `feat/1.1-admin-root-redirect` (`b567dff417665ec2821f846da0b9f413822138bc`), not pushed. The redirect is a Caddy 302 for GET or HEAD of exactly `/`, asserted by host-checks.mjs and the local checks. It changes the host-runtime fingerprint, so after merge and build the host needs `--move-core` to the merge commit before the new tag deploys.

- 2026-11-07 — Closing items. The Dependabot `@types/node` major ignore is merged (core #8, `3508a5701feda3c48f81cd91890a255682302425`). `AGENTS.md` and `CLAUDE.md` (`@AGENTS.md`) were added via bmad-project-context on `docs/1.1-agents-md`: core `6a2eb03524b10b76a554320a18e4cd7309865c53`, infra `ead0308bb507fa64d2443832959da650d9f3e878`, not pushed. The admin root redirect is deployed (reported by the user). Open, by the user's decision: the runbook step for the first Tunnel start (the README notes the gap). Next: Sofia marks the story done and plans the end-of-epic story from `handoff-sofia-2026-11-06.md`.

- 2026-11-07 — The runbook step for the first Tunnel start is done (option A, the user's choice): `runbook-first-deploy.md` §5 now starts cloudflared alone before the first deploy, and the README points to it. Committed in piviki-infra `6bd595754152a8e460db6ba484ff19bf250c85c3` on `docs/1.1-runbook-first-tunnel`, not pushed. This closes the open runbook item. The lasting fix stays with the new end-of-epic story (`handoff-sofia-2026-11-06.md`).

**Resume point (2026-11-07, story 1.1 complete).** Every verification item is met on test, and every closing item is done. Pushed for PRs, which the user opens and merges: core `docs/1.1-agents-md`, infra `docs/1.1-agents-md` and `docs/1.1-runbook-first-tunnel`. Next, in a fresh session: Sofia (bmad-agent-po) marks story 1.1 done and plans the new end-of-epic story from `handoff-sofia-2026-11-06.md`. Deferred: `deferred-work.md` entry on non-main workflows pushing to GHCR.

## Plan Change Log

- 2026-11-05 — Re-plan for the test environment (user chose "Re-plan, keep code" over the review step's intent-gap revert). Trigger: Sofia's ticket update (story on test at `test-admin.piviki.fi`, AWS free plan, production state shared-only, core refuses disagreeing host names). Amended: frozen Intent, Decisions, Boundaries, I/O matrix, Code Map, Tasks, Verification; the previous plan text is backed up at `/tmp/claude-1000/-home-manhlamt-dev-pikivi/0f705a75-f2f0-4c47-b28d-d64c80f43e29/scratchpad/plan-before-replan-2026-11-05.md`. Known-bad state avoided: reverting merged, host-verified code. KEEP: everything merged in piviki-core `ba75aa2` and piviki-infra `186bdfa` (AD-21 runtime, bootstrap with `bootstrap/files/`, checks, Access token removal, 72h Access session).

## Review Triage Log

Review 2026-11-04, thorough (blind-hunter, edge-case-hunter, verification-gap, intent-alignment). Verified by the parent session against the code, Payload 3.92.1's source and the running local stack.

| # | Lens | Finding | Verdict | Evidence | Route |
|---|---|---|---|---|---|
| 1 | EC, BH, VG | `/admin/login` check expects 200, but before any librarian exists Payload redirects every admin route to create-first-user; the first deploy (before seeding) fails its checks and records no tag | high | `@payloadcms/next/dist/views/Root/index.js:199-200` redirects when `!dbHasUser`; `host-checks.mjs:65` and `checks.sh:109` want 200 and do not follow redirects | patch |
| 2 | BH, VG, EC | `deploy.yml` pipes `ssh … \| tee` and `checks.sh \| tee` under the default `bash -e` without pipefail, so failing checks leave the step green and a success deployment is recorded | high | No `shell:`/`defaults` in `deploy.yml`; GitHub's default for unspecified shell is `bash -e {0}` | patch |
| 3 | BH | `systemctl restart nftables` runs the unit's `ExecStop=/usr/sbin/nft flush ruleset`, wiping Docker's rules that the bootstrap restarted Docker to create | high | `systemctl cat nftables.service` on Ubuntu: `ExecStop=/usr/sbin/nft flush ruleset`; `bootstrap.sh:274` restarts docker, `:310` restarts nftables | patch |
| 4 | BH | `bootstrap.sh` is not executable, but every runbook runs it directly | high | `ls -l`: `-rw-rw-r--` | patch |
| 5 | EC | `deploy.sh` checks only that manifest keys exist; an empty `PIVIKI_CORE_DB_PASSWORD` makes the postgres init script abort after initdb, and init never re-runs on the now-initialised volume | medium | `require_env_files` greps `^KEY=`; `10-core-role.sh` uses `${…:?}`; postgres entrypoint skips init when the data dir exists | patch |
| 6 | EC | First-run clone succeeds but checkout fails: the `--no-checkout` clone stays, later runs die as "already placed" or "local changes" | medium | `checkout_core` clones in place before resolving the commit | patch |
| 7 | BH | SSH rate limit is one global bucket; Tunnel SSH from cloudflared (docker bridge → host) shares it with internet scanners, so deploys can be dropped | medium | `bootstrap.sh` nft rule `tcp dport 22 ct state new limit rate 30/minute accept`; cloudflared reaches sshd via `host.docker.internal:host-gateway` | patch |
| 8 | BH, EC | Access service tokens expire after 8760h with no recorded expiry, reminder or rotation step | medium | `access.tf` `duration = "8760h"`; nothing in docs | patch |
| 9 | VG, BH, IA | No CI job runs the built image or the Compose stack: Caddy rules, the two-host split, entrypoint, migrations as `piviki_core` are only exercised at deploy time on a live host | medium | `build.yml` test job runs vitest and the connector test only; PRs never build the image | patch |
| 10 | VG | No check sends a same-origin POST on admin, so a wrong `ADMIN_ORIGIN` (e.g. trailing slash) 403s every login after a green deploy | medium | `host-checks.mjs`/`checks.sh` only send foreign Origin (403) and first-register (404 regardless) | patch |
| 11 | VG | `deploy.sh` enforcement (checkout match, dirty, record-only-on-success) has no committed test | medium | The sandbox test was not committed; grep finds `require_matching_checkout` only in `deploy.sh` | patch |
| 12 | VG | `deleteOtherLibrarians` (self-delete refused) untested | medium | `test/defineCollection.test.ts:53-63` only anonymous; no delete in `librarians.test.ts` | patch |
| 13 | VG | Deny-by-default tests pass for a collection that bypasses `defineCollection` (Payload's default access is also a function returning false for anonymous) | medium | `test/defineCollection.test.ts:38-63` | patch |
| 14 | VG | Health route 200/503 untested | medium | grep `v1/health` finds no vitest test | patch |
| 15 | VG | Seed CLI entry (`isMain`, stdin, `--force`) never spawned in a test | medium | `test/seed-librarian.test.ts` imports functions only | patch |
| 16 | BH, EC | Deployment record `ref` is `GITHUB_SHA` (workflow ref), not the tag's commit | low | `deploy.yml` `--arg ref "$GITHUB_SHA"` | patch |
| 17 | EC | Re-running the bootstrap with another `--environment` silently flips the host | low | `bootstrap.sh` overwrites `/etc/piviki/environment` | patch |
| 18 | EC | Deploy key file whose first line is not the key writes a non-key line to `authorized_keys` | low | validation greps any line, `head -n1` takes the first | patch |
| 19 | EC | `checks.sh` `status()` prints `000000` on curl failure | low | reproduced: `-w` prints 000, then `\|\| echo 000` | patch |
| 20 | EC | `SERVER_URL` not validated | low | `env.ts:59` `read('SERVER_URL')` only | patch |
| 21 | BH | Caddy `@dotdot` rule never fires for dot segments or `%2f`; comment and README overstate it | low | Local stack: `/admin/%2e%2e/api/librarians/me` → core 200; no block bypassed (first-register, GraphQL, api allowlist all 404 via traversal) | patch (wording) |
| 22 | BH | `build.yml` `cancel-in-progress: true` on `main` can leave a merged commit without an image | low | `concurrency` keyed on ref | patch |
| 23 | BH | CODEOWNERS omits `Dockerfile`, `docker/`, `build.yml` | low | `.github/CODEOWNERS` | patch |
| 24 | BH | No core README for local development; `.env.example` points at a role nothing creates | low | no `piviki-core/README.md` | patch |
| 25 | BH | Tests run as Postgres superuser, not `piviki_core` | low | `test/pg.ts`; covered by #9's stack job, which migrates as `piviki_core` | grouped with #9 |
| 26 | EC | Postgres init failure is permanent without a documented recovery | low | same root as #5 | grouped with #5 (runbook note) |
| 27 | BH | Password reset prints the reset token to logs | false | Local stack: core logs only `Email attempted without being configured. To: '<email>', Subject: 'Reset Your Password'`; no token | — |
| 28 | BH | Forgot-password has no working delivery and logs the librarian's email address | low | Same log line; the email adapter belongs to a later epic story (epic: "Owns … the email adapter and domain") | defer |
| 29 | BH | No database backups or pre-migration dump | false (scope) | Epic decision 2026-11-03: backups are story 7, before the first irreplaceable data | — |
| 30 | BH, EC | Connector count can read 0 or 2 transiently after cloudflared is recreated | maybe-false | Would need the API's behaviour during a connector swap; if true, medium | defer |
| 31 | BH | After failed checks, the running tag differs from the recorded one, so `piviki-compose` uses the previous image | low | By design (`deploy.sh` says "running but NOT recorded, redeploy to roll back"); fix adds state | reject |
| 32 | BH | Any librarian can change another librarian's email or password | low | `update: librarianOnly`; all librarians hold equal rights, and with no email adapter a peer reset is the only recovery path | reject (noted to user) |
| 33 | EC | Two librarians deleting each other concurrently leaves none | low | needs simultaneous deletes; fix adds a hook | reject |
| 34 | EC | No rollback hint when migrate or `up` fails | low | set -e exits with compose's error; fix adds a trap | reject |
| 35 | EC | Quoted `caddy.env` values break the in-network checks | low | unlikely; fix adds parsing | reject |
| 36 | EC | `checks` subcommand runs without the deploy lock | low | founder-run, read-only | reject |
| 37 | BH | LICENSE missing; `.terraform.lock.hcl` not committed | false | Both exist; the review diff excluded them | — |
| 38 | BH | README network diagram shows caddy/core on one network each | low | The text under it states both networks; mermaid cannot put a node in two subgraphs | reject |
| 39 | IA | Diff carries no production evidence | false | Production proof is the open founder task ("Founder runs the runbooks and Verification") | — |
| 40 | IA | Two-hostname risk answered narrowly (Payload kept off `api`) | false | Deliberate in Design Notes: Payload REST only via `admin.*`, `api.*` serves `/api/v1/*` | — |

**Review patches applied (2026-11-04, parent session; the step-03 implementer could not be continued).** Triage rows 1-26 fixed:
- Checks: `/admin/login` accepts 200 or Payload's redirect to create-first-user (first deploy before seeding); new same-origin login POST must reach Payload (401), catching a wrong `ADMIN_ORIGIN`; `checks.sh` curl failure now reads `000`.
- `deploy.yml`: `defaults.run.shell: bash` (pipefail); the deployment `ref` is the tag's commit. `build.yml`: no cancelling on `main`; new `stack` job (image + Compose stack, unseeded, `checks.sh local`, `host-checks.mjs`, core non-root/read-only/no egress) gating `image`; `deploy/test/deploy.test.sh` in the test job. CODEOWNERS adds `Dockerfile`, `docker/`, `build.yml`.
- `deploy.sh`: refuses empty values except in the optional `ghcr.env?` (manifest marker); `PIVIKI_*` test overrides, stripped because sudo always resets the environment (sudo-rs cannot disable `env_reset`; the sudoers file holds only the rule, see the 2026-11-05 lockout note).
- `bootstrap.sh`: executable; nftables reloaded/started, never restarted (Ubuntu's ExecStop flushes Docker's rules); Tunnel SSH from docker bridges (`iifname "br-*"`) exempt from the global rate limit; first checkout placed via a temp clone moved into place only on success; deploy key line taken by pattern; refuses to switch an existing host's environment.
- `env.ts`: `SERVER_URL` must be an exact origin. `defineCollection` stamps `custom.deniedByDefault`.
- Tests added: self-delete refused / other delete allowed (mutation-checked), every app collection carries the marker, ungranted `readVersions` refused for a librarian, health 200/503, seed CLI spawned (exit codes, no email/password in output), `SERVER_URL` validation, `deploy.test.sh` (16 cases). Docs: token expiry and rotation, failed Postgres first start recovery, Caddy traversal wording, core README, local-stack migrate step.
- Verified after patching: `pnpm typecheck`; `pnpm test` 32/32; `deploy.test.sh` 16/16; `checks-connector.test.sh` 7/7; `checks.sh local` and `host-checks.mjs` on the running stack (and the new origin check fails with a wrong origin); pre-seed login redirect accepted / other redirects refused (fake server); `caddy validate`; shellcheck clean; bootstrap `checkout_core` re-run (failed first placement leaves nothing).
- Not verified: the `stack` CI job itself (runs on GitHub), the unseeded redirect against real Payload (CI covers it), the nftables file syntax (needs root: `sudo nft -c -f`), the bootstrap end to end on Ubuntu.

**Decision (2026-11-04, user): Access checks token removed** (the "For review" item above; the review lenses did not see plan notes, so it was decided after the review). `access.tf`: `checks` service token, its policy and its place on the admin app removed; `outputs.tf`: `checks_service_token` removed. `checks.sh` production/test: no Access headers; admin must answer an unauthenticated `GET /admin` with a redirect to `https://<team>.cloudflareaccess.com/`; first-register, Origin and same-origin login checks run only for `local` here and in-network on hosts (`host-checks.mjs`). `deploy.yml`: `CF_ACCESS_CHECKS_*` dropped. New `deploy/test/checks-public.test.sh` (fake curl by URL): Access redirect passes; an open admin and a redirect elsewhere fail; no request carries an Access token; in CI and `build-local.sh`. Docs, README and secrets inventory updated; runbook §7 records the real Access answer (open point: a non-browser request may get a different status). `tofu fmt -check`/`validate`, shellcheck, `checks.sh local` and both checks tests pass.


**Founder steps (production).**
- 2026-11-04 — runbook-host-setup §0, Bitwarden: done (reported by the user). Free two-person organisation on bitwarden.eu, created from personal accounts (not a Business/Teams trial).
- 2026-11-04 — GitHub code-owner team named `admins` instead of `founders` (user); `CODEOWNERS` and runbook-first-deploy §1 updated.
- 2026-11-05 — runbook-first-deploy §1, GitHub steps 1–8 (org owners and required 2FA, Actions policy, team `admins`, both public repos with security features, push, build, GHCR package): done (reported by the user). Pushed: `piviki-core` `d9f512d`, `piviki-infra` `e7f3ca0`; the checks-token removal is not pushed (uncommitted). Main ruleset (step 9) pending.
- 2026-11-05 — runbook-first-deploy §1 step 9, `main` ruleset on `piviki-core` (PR, Code Owner review, no force push or deletion): done (reported by the user).
- 2026-11-05 — first `build` run on `main`: green (test, stack, image), reported by the user; image `ghcr.io/piviki-org/piviki-core:d9f512dce998-20261104`; GHCR package public (no `ghcr.env` values needed). The green `stack` job confirms on real Payload and an unseeded database: the login-page check accepts the redirect to create-first-user, and the Compose stack, digest pins and migrations as `piviki_core` work in CI.
- 2026-11-05 — Last step before closing the story (user): set up `AGENTS.md` in both repos with `bmad-project-context`, including the branch convention `<conventional-type>/<story>-<kebab-description>` (e.g. `fix/1.1-remove-access-checks-token`) and Conventional Commits PR titles.
- 2026-11-05 — PRs `fix/1.1-remove-access-checks-token` merged in `piviki-core` and `piviki-infra` (reported by the user). New `main` build tag: pending.
- 2026-11-05 — `typescript` 7.0.2 (PR #1) and `vitest` 5.0.5 (PR #3) merged on `piviki-core` main by the user; `@types/node` 26 to be closed (types must follow the Node 24 runtime). Before closing the story (user): update the spine stack table to TypeScript 7.0.2 via `bmad-architecture`, and add a Dependabot ignore rule for `@types/node` semver-major on a `chore/1.1-…` branch.
- 2026-11-05 — `main` build for `ba75aa2` green (test, stack, image), reported by the user: `ghcr.io/piviki-org/piviki-core:ba75aa21e5be-20261105` (TypeScript 7.0.2, vitest 5.0.5, checks token removed). This is the bootstrap's `--core-commit ba75aa21e5be` and the first deploy's tag.
- 2026-11-05 — runbook-host-setup §1, AWS account: done (reported by the user): free plan, root MFA, root credentials in Bitwarden, eu-north-1, Identity Center users per founder, EBS encryption by default on, SSM Default Host Management off, IMDS defaults, credit budget alerts. Deviation from AD-17 (paid plan before the first data): free plan kept for now; upgrade before the first real catalogue record.
- 2026-11-05 — runbook-host-setup §2–3: security group `piviki-host` (inbound TCP 22 only) and instance `piviki-prod` launched as `t3.small` (reported by the user). Name kept as in the runbook; c7i-flex.large and a separate data volume considered and not used. Post-launch checks, Ubuntu release and public IPv4: pending.
- 2026-11-05 — instance relaunched with an encrypted root volume (the first launch was unencrypted because EBS encryption by default was not on in eu-north-1; that instance was terminated). Public IPv4 auto-assigned and not tracked (it changes on every stop/start; only the founder's admin SSH uses it, nothing in code or config). Remaining post-launch checks (IAM role none, IMDSv2 hop 1, credit Standard, encrypted, termination protection) and Ubuntu release: to confirm.
- 2026-11-05 — host is Ubuntu 26.04.1 LTS with sudo-rs 0.2.13-0ubuntu1.2 (Ubuntu's default `sudo`). The first bootstrap run (as `test`) stopped at `visudo`: `/etc/sudoers.d/piviki-deploy:4:18: syntax error: unknown setting: 'requiretty'`. The file was already in place, so `sudo` stopped working for `ubuntu` (no password, SSM off): root access lost. Recovery: stop the instance, a `#cloud-boothook` in user data deletes the file if `visudo -cf` rejects it, start; clear the user data at a later stop. Fix in `piviki-infra` `bootstrap.sh`: the sudoers file holds only the rule (no `Defaults`; sudo-rs has no `requiretty` and always resets the environment) and is written to a temp file, validated, then moved into place, so a bad rule fails the bootstrap without breaking `sudo`. Re-run the bootstrap only after the fix is on main.
- 2026-11-05 — sudoers fix merged (piviki-infra PR #2); the re-run's checks then showed two false FAILs ("inbound policy is not drop", "unattended upgrades off") on a correct host: `cmd | grep -q` under `pipefail` fails on a match when `cmd` dies of SIGPIPE. Fixed in PR #3 (capture, then match).
- 2026-11-05 — refactor on `refactor/1.1-bootstrap-config-files` (`955ddde`, not pushed): the bootstrap's config files moved from heredocs to `bootstrap/files/` (host paths), installed by one validate-then-rename helper; sshd full config tested with rollback; `--check` reports drift from the repository; admin user recorded in `/etc/piviki/admin-user`. New `host-files` workflow validates the files in `ubuntu:26.04` with the hosts' packages (sudo-rs asserted), each validator proven to reject a known-bad file, plus shellcheck; verified locally in the same image. To do after merge: re-run the full bootstrap on test; make `validate` a required check in the `main` ruleset; the workflow's `ubuntu:26.04` digest is not bumped by Dependabot (manual).
- 2026-11-05 — refactor merged; full bootstrap re-run on the AWS host as `test` (`--core-commit ba75aa21e5be --admin-user ubuntu`): every host check PASS, reported by the user; `unattended-upgrades.service` and `apt-daily-upgrade.timer` both active. Host bootstrapped as test; env files still empty (filled at the first deploy, after the re-plan for test).
- 2026-11-05 — founder confirms host side done: `validate` required in piviki-infra's `main` ruleset; credit Standard, root volume encrypted, termination protection on. The `#cloud-boothook` user data is kept on purpose (it deletes `/etc/sudoers.d/piviki-deploy` at boot only if `visudo -cf` rejects it): a standing recovery path for a sudoers lockout.
- 2026-11-05 — **Where story 1.1 stands (resume point for a new session).** Plan re-planned for test and approved (`ready-for-dev`); resume with `bmad-build story 1`, which goes to implementation. Both repos clean on main: piviki-core `ba75aa21e5bea26fad3b220824da4bbc9dd6b8d7`, piviki-infra `186bdfa80429e0ca2b6081cb23fec7a5c9456c9b`.
  - Done: app, `deploy/` runtime, CI build and deploy workflow (merged, tested locally); Bitwarden; GitHub org, team `admins`, main rulesets (core: Code Owners; infra: `validate` required); AWS account; EC2 test host bootstrapped as `test`, all host checks PASS; spine and tickets updated for test.
  - Next (agent): the Tasks above, on `feat/1.1-…` branches from main in each repo, diff shown before each commit; the user pushes and merges.
  - Then (founder): Cloudflare token and R2 bucket, `tofu/prod` shared apply (no-change import), `tofu/test` apply, GitHub Environment `test`, env files from Bitwarden `test`, `--move-core` to the new core commit, deploy its tag, seed, `test-admin`/`test-api` records, verify line.
  - Before closing: Dependabot ignore rule for `@types/node` semver-major (`chore/1.1-…`); `AGENTS.md` in both repos via `bmad-project-context`.
  - Open with the founders: the AWS free plan's end date and kind of credits (AD-17: record it, upgrade or move test before it).
  - Housekeeping offered, not done (needs the user's yes): delete the merged local branches in piviki-infra (`fix/1.1-checks-grep-pipefail`, `fix/1.1-remove-access-checks-token`, `fix/1.1-sudoers-sudo-rs`, `refactor/1.1-bootstrap-config-files`) and in piviki-core (`fix/1.1-remove-access-checks-token`); the local Docker stack and scratchpad files of the earlier session.

Review 2026-11-06, quick (test re-plan delta, both repos). Verified by the parent session against the code.

| # | Lens | Finding | Verdict | Evidence | Route |
|---|---|---|---|---|---|
| 41 | Q | `host-checks.mjs`: an unparsable `ADMIN_ORIGIN` leaves the host empty, and `hostAgrees('production', '')` is true, so it passes on production | low | empty first label is not test-named; the comment claims it is reported | patch |
| 42 | Q | `deploy/README.md` and runbook §5 say core's refusal fails the deploy at the health wait / stack start; it fails at migrations | low | `deploy.sh:142` runs `core migrate` before `up -d --wait`; `payload.config.ts:12` calls `loadEnv()` at load | patch |
| 43 | Q | `checks.sh` skips an empty host name under `SKIP_PUBLIC` | false | under `SKIP_PUBLIC` checks.sh uses no host names (only the connector count), and `host-checks.mjs` requires all three | — |
| 44 | Q | `scripts/story-1.1/build-local.sh` broken: `dc()` lacks `PIVIKI_ENVIRONMENT`, the in-network checks run lacks it (exit 64), new names test not run | medium | `build-local.sh:41`, `:150-153`; `compose.yml` `${PIVIKI_ENVIRONMENT:?}` | patch |
| 45 | Q | Module `tofu test` resolves unpinned providers (lock ignored, init without `readonly`) | low | `tofu/modules/core-ingress/.gitignore`; `host-files.yml` test step | patch |
| 46 | Q | `coming_soon_records` example shows one apex A record; GitHub Pages apex has four A (and AAAA), inviting a partial import | low | `tofu/prod/terraform.tfvars.example` | patch |
| 47 | Q | AD-17's shared rate limit rule is not in `tofu/prod` | medium | AD-17 Infrastructure as code lists it; the ticket's shared list (zone, coming-soon records, login method) does not | defer |
| 48 | Q | No `AGENTS.md` in either repo | false (scope) | already scheduled before closing the story (Implementation Notes resume point) | — |
| 49 | Q | ACs 1, 3, 4 lack evidence | false | open founder task | — |
