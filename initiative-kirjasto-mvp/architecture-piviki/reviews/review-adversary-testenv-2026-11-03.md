# Adversarial review: Piviki MVP spine, test-environment amendment, 2026-11-03

- **Target:** `architecture-piviki.md` (updated 2026-11-03)
- **Scope:** today's newest amendment only. After the move to OVH, the AWS EC2 instance is rebuilt as a test environment. Covers AD-17 (Host, AWS, The move step 7, Build and deploy, Environments, Test environment, Infrastructure as code), AD-8 (`patronAccountsAllowed` true on test, founder-only allowlist), AD-12 (test mail from `test.piviki.fi`), AD-18, the Hosts convention, the Structural Seed and the Deferred row "Test environment after the credits". Read against `tickets.toml` and the `epic-librarian-catalogue` stories. Findings already closed by `review-adversary-aws-2026-11-03.md` are not repeated.
- **Method:** for each hole, two units one level down that each follow every AD exactly and still build things that do not fit together. Then the smallest rule that closes it.

## Verdict

**Not ready to build on as written: one Critical and six High holes.** A test environment that runs the production images is a good idea, and the amendment is clear about data: no production dump, bucket or secret ever reaches it. What it does not say is where per-environment values live. The spine still hard-codes production host names inside rules that the shared images and Compose file must enforce: the Caddy path allowlist, the Origin check and the reset links. Four other things are also undefined across the two environments: the erasure ledger, barcodes, outgoing mail and the deploy workflow. The patron site, which is not an image, has no deploy rule at all.

Most holes close with one new AD-17 bullet: "Per-environment configuration". It would list each value that differs, say that it is runtime configuration and never code, and add a deploy check that runs against each environment's own hosts. Findings 2, 3, 4 and 5 then need one sentence each.

---

## 1. Host names and origins are written into rules that the shared images must enforce (High)

**Units:** `epic-librarian-catalogue` story 1 (Caddy, the path allowlist, the `first-register` block and the deploy check) and story 8 (the Caddy Origin check). Against them, the step 7 rebuild of the test environment.

**What the spine says.** AD-5: "`api.piviki.fi` serves only `/api/v1/*`, enforced by a path allowlist in Caddy." AD-7: "Caddy rejects admin requests whose `Origin` is present and is not `https://admin.piviki.fi`." AD-17: the test environment runs "the production images, Compose file and bootstrap" and has its own "origins". The spine never says whether hosts, origins, Payload's `serverURL` and the reset-link base are code or runtime configuration.

**How they diverge:**

- **Builder A** follows AD-5 and AD-7 as written. The Caddyfile has site blocks for `api.piviki.fi` and `admin.piviki.fi`, and the Origin check is the literal string. On test, no site block matches `test-api` or `test-admin`. The step 7 builder adds a catch-all `:80` that proxies everything to core. Now the generated API and `first-register` are reachable on `test-api.piviki.fi` (only Access stands in front), and every admin POST from `test-admin.piviki.fi` is refused, or the check is turned off on test. Either way, test no longer checks the rules production depends on.
- **Builder B** puts the CORS and CSRF lists in `payload.config.ts`. Because the image is shared, it lists `beta.piviki.fi` and `test.piviki.fi` together. Production then accepts credentialed requests from `test.piviki.fi`. That host is same-site with `api.piviki.fi`, so `SameSite=Lax` gives no protection, and it runs a less-guarded build behind a weaker process.
- **Builder C** reads hosts from environment variables. That is compatible with neither of the above, and the story 1 deploy check ("`first-register` returns 404") gets written against a hard-coded host.

**Closing rule (new AD-17 bullet "Per-environment configuration"; amend AD-5 and AD-7 to say "the environment's api host" and "the environment's admin origin"):**

> Every value that differs between environments is runtime configuration in that environment's root-owned file, never in code or the image: the api, admin and patron hosts, the CORS, CSRF and Caddy Origin lists, Payload `serverURL`, Google client and redirect URIs, chat links, the beta allowlist, `patronAccountsAllowed`, the barcode prefix (finding 3), the mail sender and recipient allowlist (finding 4), and the erasure target (finding 2). The Caddyfile takes the hosts as placeholders and returns 404 for any other host. Production's lists never name a test host, and test's never name a production host. The deploy check (path allowlist, `first-register` 404, foreign Origin refused) runs against the target environment's own hosts after every deploy.

---

## 2. Test patron ids written to the production erasure ledger delete real patrons at the next restore (Critical)

**Units:** the step 7 rebuild of the test environment, and the `epic-launch` deletion story. AD-14 requires `modules/accounts` to write `erasures/{patronId}` on every deletion. The canary test also exercises deletion.

**What the spine says.** AD-17 Backups: on deletion, `modules/accounts` writes `erasures/{patronId}` to the private bucket, and a restore re-deletes those ids before the app starts. The test environment has its own "R2 media bucket" and "no backups", and "no production … bucket or secret is ever copied to it". There is one private bucket, and it also holds the OpenTofu state. Ids are generated by Postgres, so they collide between environments.

**How they diverge:**

- **Builder A** runs the same Compose file and core on test. Account deletion has to write an erasure, so A issues a new R2 token for the private bucket. Nothing is copied, so the letter is kept. A founder deletes made-up patron 42 on test, and `erasures/42` lands in the production ledger. The next production restore hard-deletes real patron 42. Nothing shows the error. On the same token, test can also read and overwrite the OpenTofu state object, because R2 tokens are scoped by bucket, not by prefix.
- **Builder B** gives test no private-bucket token. Then deletion fails on test, or B makes the erasure write best-effort. That quietly weakens the guarantee in production as well.
- **Builder C** starts the `backup` container on test with no token. It crash-loops, or someone hands it a token "temporarily".

**Closing rule (AD-17 Test environment and Backups):**

> The test environment holds no token for the private bucket, and does not run the `backup` service (a Compose profile). The erasure target is per-environment configuration. On test it is a test-only bucket or prefix with its own token, never the production private bucket. Deletion fails closed if the configured target cannot be written. Restore re-deletion reads only the production ledger.

---

## 3. Test and production issue the same PVK barcodes, and printed test labels reach the shelf (High)

**Units:** `epic-librarian-catalogue` story 12 (labels; a founder test-prints on a partly used A4 sheet) and story 11 (find by barcode), run on the test environment, against production after the move.

**What the spine says.** AD-9: the barcode is "the prefix `PVK` and a zero-padded number" from one sequence that a migration creates. The test environment's data is "the import script's output". So test's sequence starts at 1, and the same numbers already hang on real books.

**How they diverge:** a founder test-prints labels on test starting at position 5 of a partly used sheet, which story 12 allows. The rest of the sheet is later used in production, or a test sticker ends up on a real book. `PVK000123` from test opens a different, real copy in production, and a loan or return is recorded against the wrong copy. AD-9 fixes the prefix, so a builder cannot give test a different one without breaking the AD. A builder who starts test's sequence at a high offset makes collisions rarer but does not prevent them.

**Closing rule (AD-9):**

> The barcode prefix is per-environment configuration: `PVK` in production only, and `TST` on test and locally. Production's barcode lookup refuses any other prefix with a "not a library label" message. The label view on test prints a visible TEST mark. Test labels are never printed on sheets kept for production.

---

## 4. Test mail reaches real people (High)

**Units:** AD-12's delivery job and the AD-14 reminder, overdue and verification mail, running with `autoRun` on test, against the "made-up patrons" that AD-17 puts there.

**What the spine says.** Test sends "through the same provider from `test.piviki.fi`, with its own credentials". AD-8 limits sign-up to founder-controlled addresses only while `signupOpen` is false. A developer may run `policy:set` on test, and testing the launch flow means flipping `signupOpen` to true there. Made-up patrons need email addresses. Librarians can also create or edit patron records (AD-14 phone, admin).

**How they diverge:**

- **Builder A** seeds made-up patrons such as `minh.nguyen@gmail.com`, or flips `signupOpen` on test, so the allowlist stops applying. Ready, reminder and overdue notices then go to real mailboxes from a domain that looks like the library's.
- **Builder B** points test at a sink, which AD-12 forbids ("through the same provider").
- With most providers, one account's SMTP credentials can send from any domain verified in that account. So "own credentials" does not by itself enforce "never from `piviki.fi`".

**Closing rule (AD-12):**

> On test, `modules/notifications` delivers only to a recipient allowlist in configuration (founder-controlled addresses, or a catch-all on a founder-controlled domain), independent of `signupOpen`. Every other row is marked `dropped`. Made-up patrons use addresses on that domain. Test's credentials belong to a provider sub-account or key that can send only from `test.piviki.fi`.

---

## 5. Two environments on different tags break "works with the previous version" (High)

**Units:** any two migrations in `epic-reservation-loop` or `epic-loan-care` that follow the expand-then-contract pattern, against the on-demand, per-environment deploy workflow.

**What the spine says.** "Each must work with the previous version until that deploy is confirmed. Rollback is redeploying the previous tag." Deploys are manual, per environment and per tag. Nothing says whether production must take every tag, whether test comes first, or what "previous" means.

**How they diverge:**

- **Builder A** reads "previous" as the previous commit. Tag N+1 stops reading a column, and N+2 drops it. Test took N+1 and then N+2, and is fine. Production goes straight from N to N+2. During the rollout, and on rollback to N, core N runs against a schema without the column.
- **Builder B** reads "previous" as "the tag running in this environment". B writes migrations that can't be checked in CI, because CI doesn't know what production runs.
- **Builder C** treats a deploy to test as a gate before production. Before the move there is no test, and Deferred says test may be deleted, so C's gate either blocks production or gets skipped.

**Closing rule (AD-17 Build and deploy):**

> "Previous version" means the tag running in the target environment. The deploy workflow records each environment's running tag, and refuses a production deploy that skips a tag containing an expand migration whose contract follows. A destructive or contract migration ships only in a tag after the expanding tag has been confirmed in production. Test is never a gate for production. Test may be wiped and rebuilt at any time instead of migrating backwards.

---

## 6. The patron site and Worker have no per-environment build or deploy rule (High)

**Units:** an `epic-patron-site` story deploying the static site and Worker, and the AD-5 rule that the two halves are versioned apart.

**What the spine says.** AD-15: static export, a Worker on `/*/book/*`, and a CSP in headers. AD-17 Build and deploy covers images that hosts pull. The patron site is not an image, and no rule covers its deploy. AD-5 protects an older patron site against a newer core, not the reverse.

**How they diverge:**

- **API origin.** A static export bakes in `NEXT_PUBLIC_API`. Builder A builds once per environment. Builder B builds once and picks the API host from `window.location`, so the test host map ships in public code. The CSP `connect-src` is hard-coded either way. Builder A's test site blocks `test-api`. Builder B's production CSP allows `test-api`.
- **Worker.** One Worker with a zone route `*piviki.fi/*/book/*` also catches `test.piviki.fi`, and fetches previews from the production API (or the reverse). A second Worker per environment needs its own route and API variable, which no rule asks for.
- **Version skew.** Patron deploys run on merge, through Wrangler, while core deploys are manual. A patron site generated from test core's newer `openapi.json` goes to production while production core is older. New calls return 404, and AD-5's additive rule does not cover this direction.

**Closing rule (AD-15 and AD-17 Build and deploy):**

> The patron site and Worker are built per environment from that environment's configuration (API origin, CSP, Worker route bound to that environment's patron host only). They deploy through the same manual workflow, naming an environment and a commit. A patron deploy names the core tag whose `openapi.json` it was generated from, and is refused if the target environment runs an older core tag.

---

## 7. Step 7 can put a second connector on the production Tunnel (High)

**Units:** step 7 (rebuild the EC2 instance as test "with its own Tunnel"), against step 6's check that the production Tunnel "lists exactly one connector".

**What the spine says.** Step 4 issues new R2 tokens, a new deploy key and new GHCR credentials for OVH, and revokes the AWS ones. It does not rotate the Tunnel token. Step 6 checks for one connector and then ends the freeze. Step 7 happens after that check, on the same machine, with the same bootstrap.

**How they diverge:** the step 7 builder runs the bootstrap, which reads the tunnel token from the secrets file. The production token is still valid, either pasted from the password manager or left behind in a place the wipe missed (shell history, `docker inspect`, journald). The test host then joins the production Tunnel as a second connector. Cloudflare spreads production traffic, including patron sign-ups, across OVH and AWS. That is real patron data on AWS and against a test database, which breaks AD-17 and AD-18. Step 6's check has already passed, so nothing catches it. A careful builder creates a new Tunnel, but nothing makes this the only possible result.

**Closing rule (AD-17 The move):**

> Step 4 also rotates the production Tunnel token. After step 7, the production Tunnel is re-checked for exactly one connector, and test's Tunnel ingress names only test hosts. The production Tunnel connector count is part of the deploy check.

---

## 8. Restore drills after the move have an obvious wrong place to run (Medium)

**Units:** the yearly restore test (AD-17 Backups) and the "Recovery" runbook, against the test environment.

**How they diverge:** the test instance runs "the production images, Compose file and bootstrap". It is the cheapest place to run the yearly restore drill, so Builder A runs it there. A production dump holds patrons, so real patron data is now on AWS, which AD-17 and AD-18 forbid. The rule "no production dump is ever copied to it" is in the Test environment bullet, but the restore runbook says nothing about where a drill runs. Builder B runs it on a founder's machine.

**Closing rule (AD-17 Backups):**

> After the move, a restore drill runs only on OVHcloud or a founder's own machine, never on the test environment or any AWS host, and the restored data is deleted after the drill.

---

## 9. The import script and its processed-scans file have no target environment (Medium)

**Units:** `epic-initial-load`'s import script (AD-9), run against production for the initial load and later against test, whose data AD-17 says is "the import script's output".

**What the spine says.** The script calls `admin.piviki.fi` with a librarian API key and an Access service token, "both revoked after the initial load". It "keeps a local file of processed scans and skips them on a re-run". The import runs "once".

**How they diverge:**

- After the initial load, the processed file lists every scan. A founder points the script at test, and it skips all of them, so test gets nothing. The founder deletes the file. Later, a production re-run to finish a crashed batch re-creates every title, and the duplicate check refuses each one with `duplicate_title`. The run looks healthy, but the scans it should have imported are not created.
- AD-7 revokes "the" API key and service token after the initial load. One builder reads that as all environments and can no longer load test. Another keeps a token alive on production "for test loads".
- A script whose base URL defaults to `admin.piviki.fi` sends a test run to production.

**Closing rule (AD-9, AD-7):**

> The import script takes the target environment as a required argument with no default, and keeps one processed-scans file per environment. The AD-7 revocation applies to the production key and token. Test has its own librarian API key and service token, issued by `epic-librarian-catalogue` story 6's mechanism.

---

## 10. One OpenTofu state for both environments couples them (Medium)

**Units:** the step 7 builder adding test's Tunnel, DNS, Access apps and media bucket, and the later "delete it when the credits run low" (Deferred).

**What the spine says.** "OpenTofu manages Cloudflare resources for both environments, … with encrypted state in the private bucket." It does not say one state or two.

**How they diverge:** Builder A adds test's resources to the existing root module. Every test change then plans against production, and a `tofu destroy` to retire test destroys production's DNS, Tunnel and buckets. The test DNS records are also created before test's first librarian is seeded, which goes against AD-5's order. Builder B uses a second state, which A's tooling and runbooks don't expect.

**Closing rule (AD-17 Infrastructure as code):**

> Each environment has its own OpenTofu state and root configuration, from one shared module. Test's state never contains a production resource, and destroying it touches nothing in production. Test's DNS records follow AD-5's order (librarian seeded first).

---

## 11. Access on all test hosts conflicts with the Worker and with browser calls (Medium)

**Units:** an `epic-patron-site` story verifying search and previews on test, against AD-17 ("all behind Cloudflare Access") and AD-15 ("the Worker … holds no rules, secrets").

**How they diverge:** with Access on `test-api`, the Worker's public read gets a redirect to the Access login. The Worker serves the unchanged page, so previews and the 10 ms assumption can never be tested on test. Browser `fetch` calls from `test.piviki.fi` to `test-api` fail CORS on Access's redirect unless the CORS preflight passes through Access and the user already has an Access cookie for `test-api`. Builder A gives the Worker a service token, which breaks AD-15. Builder B adds a bypass on `test-api`, which breaks the "all behind Access" rule. In production, `api.piviki.fi` is not behind Access, so test does not match production on this point either.

**Closing rule (AD-17 Test environment):**

> On test, Access protects `test.piviki.fi` and `test-admin.piviki.fi`. `test-api.piviki.fi` matches production: it is not behind Access, its sign-up and sign-in are limited by `patronAccountsAllowed`, the founder-only allowlist and the mail allowlist, and its public reads serve only test catalogue data.

---

## 12. Deploy targets named after hosts flip meaning at the move (Medium)

**Units:** story 1's CI workflow (built before the move, when there is only one host), and the move.

**How they diverge:** the workflow says only that "a manual workflow run names one environment". Builder A names the GitHub environment `aws`, with that host's SSH secret, because there is only one environment. After step 7, `aws` is test. Builder B adds `ovh` at the move, and a founder who knows "aws is where production runs" deploys an untested tag to what is now test, or the reverse: an old runbook deploys to `aws` thinking it is production. The deploy check and the heartbeat move in the same way.

**Closing rule (AD-17 Build and deploy):**

> Workflow environments are named `production` and `test`, never after a host. The move changes only `production`'s host and deploy credentials. `test` is created at step 7 with its own deploy user and key. Production deploys need both founders' approval, or one founder's explicit confirmation.

---

## 13. "Own Google redirect URIs" invites sharing the production OAuth client (Medium)

**How they diverge:** AD-17 says test has its own "Google redirect URIs". Builder A reads that literally and adds test's URIs to the production OAuth client. Test then needs that client's secret, which breaks "no production secret is ever copied to it". It also means production's client accepts a flow ending on a test host. Builder B creates a second client.

**Closing rule (AD-17 Test environment):**

> Test has its own Google OAuth client. The production client lists only production redirect URIs.

---

## 14. Patching never reaches a host, and rebuilt images reuse a tag (High)

**Units:** story 8's monthly image rebuild, and the 72-hour rule, against on-demand, per-environment deploys and step 3's "exact image tag AWS last ran".

**What the spine says.** Images are "tagged by commit" and "rebuilt monthly". Deploying is manual. "A critical internet-reachable flaw is patched within 72 hours, or core is taken offline." Nothing says who deploys a rebuild, or to which environment.

**How they diverge:**

- **Builder A** rebuilds the same commit and pushes the same tag again. The tag is now mutable. "Redeploy the previous tag" and step 3's "exact image tag" stop naming a fixed image, and production and test can run different images under one tag.
- **Builder B** gives rebuilds new tags. Nothing deploys them, so production stays on the unpatched image until someone remembers.
- Test sits behind the Tunnel and is internet-reachable. Nobody owns patching it, so it stays on an old tag, on the same zone and as the same founders' accounts.
- Dependabot PRs merge, and nothing ships them.

**Closing rule (AD-17 Build and deploy and Patching):**

> Image tags are immutable: the commit plus the build date. A monthly rebuild or a 72-hour patch counts as done only when it is deployed to every running environment. Test is patched within the same window or stopped. The deploy workflow shows each environment's tag age.

---

## 15. Minor

- **The test environment could run on the production host (Low).** Deferred allows "move it" when the credits run low. The cheapest move is a second Compose project on the OVH VPS, which goes against "production depends on nothing in it" (shared CPU, disk, kernel and Docker daemon). Rule: test never runs on the production host.
- **The credits run out silently (Low).** The account is on a paid plan, so test simply starts billing the card. AWS Budgets would be a service other than EC2, which "No other AWS service is used" forbids. Rule: a founder owns a calendar check of the credit balance, or the AWS rule allows a billing alert.
- **One free rate-limit rule per zone (Low).** Test shares the `piviki.fi` zone. A builder who adds a second rule for test fails `tofu apply` on the free plan. A builder who widens the one rule makes test and production share a counter per address. Rule: the one rule covers test hosts, or test relies only on the limiter in core. Update "Assumptions to Prove".
- **Structural Seed (Low).** The node "Access on admin and beta" does not show test's Access. The test node's arrow "no data from" points from test to production, but data must not flow from production to test, so the arrow reads backwards. The Hosts row leaves out `beta.piviki.fi`, which is the production patron host until launch.

---

## Attacked and dropped

- **Cookie collisions between `admin` and `test-admin`.** Cookies are host-only (AD-7), and Access cookies are per host. Nothing is shared.
- **The AD-8 launch reset touching test.** The reset is an `epic-launch` step in production. Test is wiped or rebuilt freely and needs no reset. Test is a good place to rehearse the reset, but nothing conflicts.
- **False heartbeat alarms from test.** The heartbeat watches only the current host, and test runs no backup (once finding 2 is closed).
- **Chat links pointing at the real group.** AD-17 already gives test its own chat link values.
- **Mixing media buckets.** Test has its own media bucket. Uploads go through core's configured adapter, including the import script's, so no writer picks a bucket itself.
- **Tenant-policy drift between environments.** Both are seeded by the same migration. `policy:set` audits each one locally. Drift on test is intended, and production values are never read from test.
- **The pre-move period without a test environment.** No story before the move needs test. `epic-librarian-catalogue` verifies in production by design, and patron-creating checks come after the move. The only real effect is the naming in finding 12.
- **Certificates for test hosts.** One-level names already keep Cloudflare's free certificate.
- **Founders' real addresses on test against AD-18.** Founders are the controller, testing with their own consent. They are not patrons whose data the privacy notice covers. AD-18's "no real patron's data" still holds if findings 4, 7 and 8 are closed.
- **SameSite between `test.piviki.fi` and `api.piviki.fi` on its own.** This only matters if production's origin lists accept a test host. Finding 1 forbids that, and exact origin lists then block it.
- **Dependabot per environment.** It works per repository, not per environment. Its only gap is shipping updates, which finding 14 covers.
