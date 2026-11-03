# Reality check: architecture-piviki.md

Checked 2026-11-02. Reviewed `architecture-piviki.md` and `.memlog.md`; no other project file was changed.

## Verdict

**Not ready to bind as written.** Every named technology exists, and every pinned version is current. The stack fits together. But several Rules are false, unenforced or incomplete when run against the real software. Two of them open security holes (F1, F2). Most fixes are one-line changes to a Rule, and none forces a change of technology, except that `payload-oauth2` should stop being the default (F5).

## How this was tested

| Method | Scope |
|---|---|
| `npm view`, GitHub and registry APIs | Versions, peer ranges, engines |
| Reading the published 3.92.1 package source | Payload, `@payloadcms/next`, `db-postgres`, `drizzle`, `storage-s3`; also `payload-oauth2` 1.0.21 |
| Running a throwaway harness | Payload 3.92.1 and `db-postgres` on embedded PostgreSQL 18, with real requests through Payload's `handleEndpoints` |
| Running a throwaway build | Next 16.4.4 with `output: 'export'`, TypeScript 7.0.2 and 5.9.3 |
| Running `wrangler dev` 4.156.1 | Asset routing, `run_worker_first`, `HTMLRewriter` |
| Official docs and web search | Everything else, cited inline |

Limits of these tests:

- The harness ran on Node 24.13.0, not the pinned 24.22.2. It called Payload's handler directly rather than going through Next.
- I added debug prints to `jwt.js` in the scratch copy to diagnose a harness problem. They do not change behaviour.
- `wrangler dev` is local, not production Cloudflare.
- Nothing was run against real R2, the Cloudflare edge, Caddy or OpenTofu (`tofu` is not installed here).

Harness and build are in `/tmp/claude-1000/-home-manhlamt-dev-pikivi/18b2c980-27e4-4394-8c18-46a173fed62a/scratchpad/spike-pay/run.mjs` and `/tmp/claude-1000/-home-manhlamt-dev-pikivi/18b2c980-27e4-4394-8c18-46a173fed62a/scratchpad/spike-next`. They are temporary. "Tested" below means observed in these runs; "Source" means read in the package code, not run.

## Findings

### F1. Blocker: `first-register` lets anyone create an account past all access control (AD-5, AD-7, AD-8)

**Tested.** With `access.create: () => false`, an anonymous `POST /api/patrons/first-register` returned 200 and a logged-in patron while `patrons` was empty. A second call returned 403. The same call on an empty `librarians` returned 200 and a logged-in librarian.

The operation calls `payload.create` with `overrideAccess: true` and is registered on every auth collection with local login (`dist/auth/operations/registerFirstUser.js`, `dist/auth/endpoints/index.js`).

Impact:

- `patrons` is deliberately empty until launch, because `signupOpen` is false. So the sign-up gate can be bypassed for exactly that period.
- Any window between deploying and seeding `librarians` lets a stranger become a librarian.
- AD-5's allow-list does not mention this endpoint, so reading the Rule suggests it is closed.

**Fix.** Add to AD-5 and AD-17:

- Block `/api/*/first-register` at Caddy.
- Reject patron creation in a collection hook unless it comes from `modules/accounts`.
- Create the first librarian by migration or seed before the service is reachable.
- Add a test that the endpoint fails on empty collections.

The Caddy block and hook are my proposals and were not run.

### F2. Blocker: AD-5's deny rule cannot be implemented as written, and its list is incomplete (AD-5)

The core request holds: `/api/v1` custom endpoints do coexist with denied generated routes. **Tested:** the custom endpoint served anonymous callers with 200 and saw `req.user`, while generated `titles` and `loans` returned 403 to patrons. Login and forgot-password worked with read denied.

These parts of the Rule do not hold:

| Case | Tested result | Consequence |
|---|---|---|
| `GET /patrons/me` with `read: false` | 403 | `me` runs with `overrideAccess: false`, so patrons need read access to their own record |
| `GET /patrons` with read limited to self | 200, one document | Generated reads cannot be fully denied; a patron can read their own record through them |
| `POST /patrons/refresh-token` | Works, needs to be allowed | Not on AD-5's list; the default token lasts 7200 s (source), so patrons are logged out after two hours without it |
| `GET /access` | 200 | Also reachable, and not on the list |

Other routes also exist on auth collections: `init`, `unlock`, `verify/:id` (source). GraphQL can be turned off entirely with `graphQL.disable` (source: `config/types.d.ts`), which is simpler than denying it by operation.

Custom endpoints are also not protected by anything automatically: "[Custom endpoints are not authenticated by default](https://payloadcms.com/docs/rest-api/overview)".

**Fix.** Rewrite the AD-5 bullet to say:

- Patron `read` is limited to the patron's own record, and field access hides everything else.
- `refresh-token` is added to the allowed list.
- GraphQL is disabled.
- Every collection, including defaults, sets `access` explicitly, because the default is `Boolean(user)` (source: `auth/defaultAccess.js`), which any logged-in patron passes.

### F3. High: any logged-in patron can run the job queue (AD-13)

**Tested.** `GET /api/payload-jobs/run` with a patron cookie returned 200 "Success". Anonymous returned 401. Payload's [docs confirm](https://payloadcms.com/docs/jobs-queue/queues) that logged-in users are allowed by default, and the same default covers `handle-schedules` (source).

This contradicts AD-13's "a schedule firing twice" prevention, because patrons can trigger runs at will.

**Fix.** Add to AD-13: `jobs.access.run` returns false for all HTTP callers, since `autoRun` calls the runner directly (`dist/index.js`) and does not need the endpoint.

### F4. High: "scoped key" does not exist natively, and the scan key can publish (AD-9, AD-7)

- **No key scopes.** Payload keys "[are meant to represent a single user](https://payloadcms.com/docs/authentication/api-keys)". The scope has to come from access rules on a dedicated auth collection. **Tested and works:** a `scan-clients` collection with `useAPIKey` and `disableLocalStrategy` authenticated, and was refused `loans` with 403. But that collection is missing from AD-7 and from the Collections convention.
- **The key can publish.** **Tested:** `PATCH /api/titles/{id}` with `{"_status":"published"}` using the key returned 200 and published the title. This breaks AD-9's "A librarian publishes after review" and the "unreviewed scans going public" prevention.

**Fix.**

- Add `scan-clients` to AD-7 and the conventions, with `useAPIKey` and no local login.
- Add to AD-9: a hook rejects `_status: published` unless the caller is a librarian, with a test.

Separately, the key index is an HMAC using the Payload secret (source: `auth/strategies/apiKey.js`), so rotating the secret invalidates keys.

### F5. High: `payload-oauth2` has PKCE but does not meet AD-8 (AD-8, AD-14, AD-5)

Source of 1.0.21, last published 2026-05-12, peer `payload ^3`:

| AD-8 needs | What the plugin does |
|---|---|
| PKCE | Exists, opt-in via `pkceEnabled`. If the verifier cookie is missing it silently sends no verifier. The cookie sets no `Secure` flag |
| State | Only forwards a `state` query value; never generates or checks it on callback. A login CSRF guard must be added through `callbackExtractToken` |
| Link only when email is verified | No `email_verified` check; must be written inside `getUserInfo` |
| Sign-in closed while `signupOpen` is false | Creates the user before the `beforeLogin` hook runs, so a refused login still leaves an account. It also generates a nonce it never checks |
| AD-14 field list | Adds a provider subject field with `read: () => true` by default; AD-14 does not list this field |
| AD-5 allow-list | Adds `/api/patrons/oauth/*`, which is not listed |

It also signs its own JWT without the `authVersion` header that Payload's JWT strategy requires (source: `auth/strategies/jwt.js`), so it relies on its own strategy to accept the token. I did not run this against Google.

**Fix.** Make small core endpoints with `openid-client` 6.8.9 the default, and drop `payload-oauth2` from the Stack and Deferred. AD-8 needs most of this code written anyway. Add a Google subject field to AD-14 either way.

### F6. High: the shared book page needs a Rule; `generateStaticParams` cannot return nothing (AD-15)

**Tested on Next 16.4.4:**

- `generateStaticParams` returning `[]` fails the build: "returned an empty array... With output: export, at least one route must be generated".
- Placeholder params (`{ lang, id: '_' }`) build, and emit `out/{lang}/book/_.html`.
- `dynamicParams: true` is [unsupported on export](https://nextjs.org/docs/app/guides/static-exports).

So AD-15 is feasible, but only with an unstated placeholder. Without a rule, a build agent will likely try to generate pages from the API, or hit this error.

**Tested in `wrangler dev`:** `run_worker_first: ["/*/book/*"]` matched (`/vi/book/_` got the rewritten tag), and `env.ASSETS.fetch('/vi/book/_')` served the shell for `/vi/book/123` with a preview tag added. Cloudflare documents `*` as [a deep-match glob](https://developers.cloudflare.com/workers/static-assets/binding/#run_worker_first).

Three further problems:

- **Params read `_`.** The prerendered `id` is the placeholder, so client code must read the id from `location.pathname`. Not browser-tested.
- **Client navigation.** `<Link>` to a book requests data files such as `/vi/book/123/__next._tree.txt`, which do not exist. The route's Worker receives these requests. Client behaviour on failure was not browser-tested, so use plain links to book pages or test this in the first story.
- **"Worker runs only on `/{lang}/book/*`" is not quite true.** **Tested:** asset misses such as `/book/x` and `/_next/nothing` also reached the Worker. Set `not_found_handling` so misses stop at the asset layer. I did not run that setting.

**Fix.** Add to AD-15: one placeholder route `/{lang}/book/_`, id read from the URL in the browser, plain links, and `not_found_handling` set.

### F7. Medium: Cloudflare's free plan only weakly covers login and reset (AD-17)

The [free plan](https://developers.cloudflare.com/waf/rate-limiting-rules/) gives 1 rule, Path and Verified Bot fields only, counting by IP, a 10 s period and a 10 s mitigation. So:

- One rule must cover both login collections, forgot-password and reset-password. I could not confirm which operators free rules accept, including an `or` across paths; check this in the dashboard.
- It cannot filter by method.
- Derived, not tested: with a threshold of N, an IP can still make about N attempts every 20 seconds.

Payload's lockout defaults (5 attempts, 10 minutes; source) protect login per account. They do nothing for `forgot-password`, so an attacker can still send many emails to a victim.

**Fix.** Keep the Cloudflare rule as coarse protection. Add to AD-17: a small limiter in core for forgot-password per email and per `CF-Connecting-IP`, using the outbox or a table. Or accept the risk explicitly and record it.

### F8. Medium: `autoRun` works, but starts only on the first request (AD-13)

The memlog's check is right: `autoRun` is for long-running servers, and [not for serverless](https://payloadcms.com/docs/jobs-queue/queues). But crons initialise only when `getPayload` is called with `cron: true`, which happens in `createPayloadRequest` and the admin's `initReq` (source). After a restart with no traffic, nothing runs: no reminders and no retention. If backups are a job, none of those either. Payload's bin script is what the docs recommend for production cron, though `autoRun` remains valid.

**Fix.** Add to AD-13: core calls `getPayload({ config, cron: true })` at boot through Next's `instrumentation.ts` (proposal, not run), and a test confirms jobs run after a restart with no requests.

### F9. Medium: the one Payload cookie is shared across both collections (AD-7)

**Confirmed.** The name is `${cookiePrefix}-token` from global config (source: `auth/cookies.js`), so patrons and librarians overwrite each other's session in one browser. A [maintainer described this](https://www.github.com/payloadcms/payload/discussions/226) as known and complicated. I found no release that changes it.

**Tested:**

- A librarian cookie sent to a `/v1` endpoint gives `req.user.collection === 'librarians'`, so a handler using only `req.user.id` would act on the patron with that number. Both collections use their own numeric ids. [CVE-2026-25574](https://nvd.nist.gov/vuln/detail/CVE-2026-25574) was this kind of bug in `payload-preferences` (fixed in 3.74.0, so 3.92.1 is not affected).
- `librarians/me` with a patron cookie returned `user: null`, so that separation holds.
- An unlisted `Origin` gives no user, which supports AD-7's CSRF rule. A cookie with no `Origin` and no `Sec-Fetch-Site` also gives no user, so server-side tests must send browser-like headers.
- The issued cookie had no `Secure` flag at default config, so AD-7's "Secure" must be set explicitly via `auth.cookies.secure`.

**Fix.** Add to AD-7: one identity per browser, as a stated limit. `/api/v1` accepts only `collection === 'patrons'` through one shared guard. `secure: true` on both collections.

### F10. Medium: drafts and the unique field work, but re-run needs client logic and a publish rule (AD-9)

**Tested.** All of this works on Postgres 18:

| Step | Result |
|---|---|
| Create with `?draft=true` | Creates a draft; required fields are not enforced |
| Second create with the same `scanId` | 400; no automatic upsert (no REST upsert route found) |
| Find `scanId` with `draft=true`, then PATCH | Updates the latest draft |
| Versions table | Has no unique index on `scanId` (source: `disableUnique`), so history does not collide |

Two consequences the Rule does not state:

- **Re-run overwrites a librarian's review.** A PATCH with `draft=true` after publishing creates a new draft from the scan. The public record stayed "Reviewed", but the draft view showed "rerun-after-publish". The next publish would overwrite the librarian's corrections. Fix: the script skips published titles.
- **Main table is stale for drafts.** After a draft PATCH, the main row kept the old name and only the version changed. A query on `name` without `draft=true` found nothing. AD-10's normalised search column will only be current for published data, which is fine for `/api/v1` but must be understood. Keep `/api/v1` reads to `_status: published`, and do any draft search with `draft=true`.

Also, AD-9 gives `media` no key, so a re-run could upload images twice. Not tested.

### F11. Medium: AD-15's "any error serves the page" cannot cover the CPU limit (AD-15)

The memlog (line 38) records from Cloudflare docs that exceeding 10 ms CPU gives Error 1102, and that fail-open applies only to the daily request limit. The AD-15 Rule says "any error", and the memlog's assumption (line 39) was not carried into the document. A Worker cannot catch its own CPU limit. **Unconfirmed in this review:** actual CPU use on Cloudflare; `wrangler dev` does not measure it.

**Fix.** Reword the Rule to "any error it can catch". Add the CPU measurement and fallback to Deferred.

### F12. Medium: R2 with `storage-s3` is documented, with one unresolved risk (AD-16)

**Confirmed.** Payload [documents R2 through `storage-s3`](https://payloadcms.com/docs/upload/storage-adapters) with `region: 'auto'`, `forcePathStyle: true`, a separate public URL, `generateFileURL` and `disablePayloadAccessControl: true`. `@payloadcms/storage-r2` is for Workers only and is beta, so it is correctly not chosen.

**Unconfirmed:**

- The adapter allows `@aws-sdk/client-s3 ^3.614.0` (current 3.1161.0). SDK 3.729+ adds checksums by default, and [reports show R2 failures](https://roboin.io/article/2025/06/05/how-to-fix-cloudflare-r2-internal-error) fixed by `WHEN_REQUIRED`. Those reports are from 2025. [Cloudflare's current compatibility table](https://developers.cloudflare.com/r2/api/s3/api/) lists partial checksum support, and its [SDK example](https://developers.cloudflare.com/r2/examples/aws/aws-sdk-js-v3/) mentions no such setting. I found nothing dated 2026.
- `x-amz-acl` is unsupported on R2 per that table, so leave `acl` unset.

**Fix.** In the first upload story, run a real upload against R2. Set `requestChecksumCalculation` and `responseChecksumValidation` to `WHEN_REQUIRED` if it fails.

### F13. Low: smaller items

- **Caddy (AD-17): the belief is right.** Official docs point to [`caddy-dns` modules](https://caddyserver.com/docs/automatic-https). The `caddy:2.11.7-builder` image exists, and [`caddy-dns/cloudflare`](https://github.com/caddy-dns/cloudflare) was last committed on 2026-03-23 (latest tag v0.2.4). The Stack table does not say Caddy is a custom image, and CI must build it. The token needs `Zone.DNS:Edit` on the zone per [its README](https://raw.githubusercontent.com/caddy-dns/cloudflare/master/README.md), so a compromised VPS could change DNS for `piviki.fi`. A [Cloudflare Origin CA certificate](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/) also meets Full (strict), needing no custom build or token, on all plans. It does need its expiry tracked. Worth reconsidering.
- **TypeScript is missing from the Stack.** `latest` is now 7.0.2. **Tested:** Next 16.4.4 type-checks with it with no flag; its config default sets `useTypeScriptCli: true` (older [docs](https://github.com/vercel/next.js/discussions/95633) describe a flag). I found no evidence on Payload and TS 7. Payload's own [blank template](https://raw.githubusercontent.com/payloadcms/payload/main/templates/blank/package.json) pins TypeScript 6.0.3, so pin that.
- **Node.** 24.22.2 satisfies Payload's `^22.23.4 || >=24.22.1`. Node 24 [entered maintenance on 2026-10-20](https://raw.githubusercontent.com/nodejs/Release/main/README.md), with end of life 2028-04-30. Node 26 became LTS on 2026-10-28 and is allowed by the range but untested here. Keep 24 for now; no change needed.
- **Row locks (AD-2).** **Tested and works:** `SELECT ... FOR UPDATE` in a Payload transaction blocked a second locker, and rollback undid a Local API write passed `{ transactionID }`. But the raw handle comes from `payload.db.sessions[id].db`. `getTransaction` is not exported publicly, and Payload's [transaction docs](https://payloadcms.com/docs/database/transactions) do not cover locks. Put this behind one helper in `modules/circulation`. Every Local API call inside must pass `req`, or it runs outside the transaction and can wait on its own lock.
- **Read-only fields (AD-2).** `admin.readOnly` "[has no effect on the API whatsoever](https://payloadcms.com/docs/fields/overview)"; field `access` is needed too. **Tested:** the scan key's write to a denied field was silently discarded, with a 201 response. [Docs agree](https://payloadcms.com/docs/access-control/fields). So AD-2 needs both settings, and clients get no error. Whether denied field access also shows as read-only in admin is not stated in those docs.
- **OpenTofu (AD-17).** State encryption is real ([docs](https://opentofu.org/docs/language/state/encryption/), since [1.7](https://scalr.com/blog/opentofu-1-7-is-alive)). The PBKDF2 passphrase provider fits a free setup. A lost key means unreadable state, and the Rule names neither a state location nor a key holder. Add both. The Cloudflare provider is on the OpenTofu registry at 5.28.1, matching upstream. I did not run `tofu`, or confirm each needed resource against it.
- **PostgreSQL 18 (AD-17).** In the [official image](https://hub.docker.com/_/postgres) the volume mount moved to `/var/lib/postgresql`. The usual `/var/lib/postgresql/data` mount from older guides is wrong for 18. Note it in the Compose file.
- **Backups (AD-17, Structural Seed).** "Core" making the `pg_dump` means its image must include a Postgres 18 client. This is reasoning, not a test.

## Requested checks at a glance

| Check | Result | See |
|---|---|---|
| Custom REST with generated REST and GraphQL denied | Works, but the Rule's list is wrong and `first-register` bypasses it | F1, F2 |
| API keys scoped to some collections | Only through a dedicated collection and access rules; can publish | F4 |
| Drafts, unique field, update on re-run | Works with client logic; re-run after publish is unsafe | F10 |
| Jobs `autoRun` on a long-running server | Works, lazy start, open run endpoint | F3, F8 |
| Admin field-level read-only | Exists; needs field access as well for the API | F13 |
| Transaction and row lock with Postgres | Works; handle is not a documented export | F13 |
| `storage-s3` with R2 | Documented; checksum behaviour unconfirmed | F12 |
| One cookie across patrons and librarians | Confirmed limit | F9 |
| Caddy DNS challenge needs custom build | Confirmed; alternative exists | F13 |
| Cloudflare free rate limiting for login and reset | Partly; weak, and `or` unconfirmed | F7 |
| Next static export and shared book page | Works only with a placeholder param | F6 |
| TypeScript and Node | Compatible; TypeScript unpinned | F13 |
| OpenTofu state encryption | Confirmed; key and backend not decided | F13 |
| OpenTofu Cloudflare provider | Confirmed on registry; not run | F13 |
| `payload-oauth2` and PKCE | PKCE yes; state and other AD-8 needs no | F5 |

## Version check

All match the Stack table on 2026-11-02:

| Item | Pinned | Latest | Compatibility |
|---|---|---|---|
| `payload` and `@payloadcms/*` | 3.92.1 | 3.92.1 | 4.0.0-beta.6 exists; stay on 3 |
| Next.js | 16.4.4 | 16.4.4 | `@payloadcms/next` needs `>=16.3.3 <17`; Payload's template uses 16.4.2 |
| React | 19.3.1 | 19.3.1 | Payload UI `^19.2.1`; Mantine `^19.2.0` |
| Mantine | 9.7.4 | 9.7.4 | |
| Node.js | 24.22.2 | 24.22.2 (line), 26.13.0 (newest LTS) | See F13 |
| PostgreSQL | 18 | 18.6 | Harness ran on an embedded 18 |
| Caddy | 2.11.7 | 2.11.7 | |
| OpenTofu | 1.13.2 | 1.13.2 | |
| Wrangler | 4.156.1 | 4.156.1 | |
| `payload-oauth2` | 1.0.21 | 1.0.21 | See F5 |
| `openid-client` | 6.8.9 | 6.8.9 | |

Missing from the table: TypeScript and `graphql` (a Payload peer, `^16.8.1`).

Payload 3.92.1 was published today, so a fresh release may still get a quick patch. Pin exact versions, as the table implies.

## Evidence gap in the memlog

The memlog had real evidence for versions, peers, static export limits, jobs, drafts, Workers and hosting. It had none for these Rule dependencies, which were asserted: first-register, access defaults, cookie naming, key scoping, row locks, unique with drafts, R2 config, Caddy's module, Cloudflare's free limits, OpenTofu encryption, and what `payload-oauth2` actually does. The memlog does flag `payload-oauth2` as an assumption to check. This review fills those gaps, except where marked unconfirmed.

Not checked: the data processing agreements in AD-18, OVH details beyond the memlog, and Google's OAuth console behaviour.
