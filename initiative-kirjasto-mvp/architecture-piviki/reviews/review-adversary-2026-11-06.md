# Adversary review: per-environment state buckets (2026-11-06)

- **Spine:** `architecture-piviki.md` (updated 2026-11-06)
- **Change under review:** `.memlog.md` entry "State buckets per environment (2026-11-06, user)": AD-17 Infrastructure as code (state per environment in its own private R2 bucket, EU, own R2 token; "no credential used for test can reach production's state or backups"), AD-16 last sentence (backups go to production's private bucket), AD-17 Backups ("its environment's private bucket" for erasure records). The Test environment bullet already said test has its own private bucket and no token for production's.
- **Lens:** adversary. Build two units one level down that each follow every AD to the letter and still don't fit together.
- **Also read (to check the units are realistic, not to review them):** `piviki-infra/tofu/{prod,test}/versions.tf`, `backend.hcl.example`, `docs/runbook-first-deploy.md` §2.1, `docs/secrets-inventory.md`. Holes H3 and H2 already show up in that code.

## Verdict

**Needs fixes before final.** The change closes the hole it was aimed at: test's state token can no longer reach production's state. It does this by putting the state in the same bucket as production's backups and erasure records. That creates new conflicts inside one environment's bucket that the spine doesn't settle. The bucket rules written for backups break OpenTofu state (H1). Every host token that can write the bucket can also overwrite or delete state (H2). The text also doesn't say who owns the bucket and its tokens (H4), or which bucket a restore reads erasures from (H5). The new sentence "no credential used for test" holds only because the shared Cloudflare API token has no R2 permission. That token is used for test runs and can still change production's routing (H3).

## Holes

### H1 (high): backup lifecycle and lock rules hit OpenTofu state and erasure records in the same bucket

- **Unit A, the backup story (`epic-librarian-catalogue`):** follows AD-17 Backups exactly. "A lifecycle rule deletes dumps after 30 days, a 7-day bucket lock protects recent ones." Rules are set per bucket in R2, and the spine names no prefix, so the story sets both rules on all of `piviki-prod-private-eu`.
- **Unit B, the OpenTofu config (`piviki-infra/tofu/prod`):** follows AD-17 Infrastructure as code exactly. It keeps state at `tofu/prod/terraform.tfstate` in the same bucket, with `use_lockfile = true` (already in `versions.tf`).
- **What breaks:**
  - The 7-day lock blocks overwriting the state object, so a second apply within 7 days fails. The cutover's rollback ("re-applying the coming-soon record") is exactly that kind of apply.
  - `use_lockfile` writes `.tflock` and then deletes it. The lock refuses the delete, so the state stays locked.
  - Production's state holds only shared resources and rarely changes. If no apply runs for 30 days, the lifecycle rule deletes it.
  - The lifecycle rule also deletes `erasures/{patronId}` after 30 days. Copies of a dump can live longer than that: OVH's included backup, or a restore on a founder's machine (allowed by AD-17). Restoring one brings a deleted patron back with no erasure record left to re-delete.
- **Smallest tightening (AD-17 Backups):** "The lifecycle rule and the bucket lock apply only to the `dumps/` prefix. Erasure records are never expired and are kept while any copy of a dump older than them can exist." With H2's fix (state in its own bucket), the state part goes away completely.

### H2 (high): host tokens for the private bucket can reach state, and "only that environment's own R2 token" contradicts the backup container's own token

- **Unit A, the OpenTofu runbook:** follows Infrastructure as code: the bucket is "reached only by that environment's own R2 token". It creates exactly one token per bucket, `r2-tofu-state-token-prod`, and notes it is "never on a host" (runbook §2.1 already says "One R2 API token per bucket").
- **Unit B, the backup story plus the accounts/erasure module:** follows Backups. The `backup` container has "its own ... R2 token". Core must write `erasures/{patronId}` to the same bucket, so core needs a write token too. Either the stories make a second and third token for the bucket, which breaks A's "only ... own R2 token", or they reuse A's token on the host, which breaks the runbook and "each service gets only its own secrets".
- **What breaks:** the memlog itself gives the reason: an R2 token can't be limited to part of a bucket. So core's erasure token (on an internet-facing container) and the backup token can each overwrite or delete production's encrypted state. A compromised core can also delete erasure records, and dumps once their 7-day lock has passed. The change applies this reasoning between environments but not inside one.
- **Smallest tightening (AD-17 Infrastructure as code):** replace "each in that environment's own private bucket ... and reached only by that environment's own R2 token" with: "each in that environment's own private **state** bucket in R2's EU jurisdiction, reached only by that environment's founder-held state token, never on a host or in GitHub. Backups and erasure records go to a separate private **data** bucket per environment. Its tokens are the `backup` container's and core's, each limited to that one bucket." AD-16 then reads "production's private data bucket", and Backups "its environment's private data bucket".

### H3 (high): one Cloudflare API token runs OpenTofu for both states

- **Unit A, the OpenTofu runbook and secrets inventory:** `cloudflare-tofu-api-token`, filed in Bitwarden `production` and "used for both states". Its permissions are account-wide Tunnel Edit, Access Apps/Policies/Service Tokens/Organizations Edit, and DNS Edit on `piviki.fi`. This follows the letter. It is not an R2 token, so it reaches neither state nor backups, and test's state still holds only test-named resources.
- **Unit B, the Test environment and the AD-17 Prevents line:** "production data or secrets reaching the test environment", "two environments owning one Cloudflare resource", "no credential used for test can reach production's ...".
- **What breaks:**
  - Every `tofu -chdir=tofu/test apply` runs with a production secret, and that secret can repoint `api.piviki.fi`, delete production's Tunnel, or rewrite production's Access policies.
  - A later story that wants OpenTofu to manage the test media bucket adds "Workers R2 Storage: Edit" to this one token. That still follows the letter as written ("R2 token" is read as "R2 API token"). The token can then delete production's lock and lifecycle rules, then production's backups, and through R2's S3 credentials derived from the token, read or write production's state.
  - Cloudflare can't scope Tunnel, Access or DNS-record edits to one environment, so some reach between environments stays unavoidable in one account and one zone.
- **Smallest tightening (AD-17 Infrastructure as code), replacing the last clause:** "Each state is applied with that environment's own Cloudflare API token, which has no R2, API Tokens or Account Settings edit permission. No token, key or passphrase stored for test (Bitwarden `test`, the test host, GitHub Environment `test`) can reach production's state, backups or erasure records. Cloudflare cannot limit Tunnel, Access or DNS edits to one environment, so test's Cloudflare token is founder-held only, and its reach into production's routing is accepted." Also, R2 buckets, their rules and their tokens are never OpenTofu resources (see H4).

### H4 (medium): two owners for the private buckets, their rules and their tokens, and production's bucket exists before the production start

- **Unit A, the runbook (`epic-librarian-catalogue` story 1):** creates both private buckets and both state tokens by hand in the dashboard (runbook §2.1). Production's bucket and token have to exist then, because production's state holds the zone and the One-time PIN login method that test depends on.
- **Unit B, the production start story (`epic-soft-open`):** "production secrets are created new in Bitwarden `production`, and OpenTofu gives production its own Tunnel and Access". Read literally, that includes production's private bucket and its token: a story could create them new, or bring them under OpenTofu.
- **Unit C, the backup story:** sets lifecycle and lock rules. Unit A's runbook or an OpenTofu `cloudflare_r2_bucket_lifecycle` resource could also own these. With two owners, a later `tofu apply` silently removes the backup story's rules, or the reverse.
- **What breaks:** two owners of one bucket configuration. Also, either the production start "recreates" a bucket that already holds production's state, or it rotates the state token and leaves a stale copy behind.
- **Smallest tightening (AD-17 Infrastructure as code):** "The private buckets, their lifecycle and lock rules, and every R2 token are created by hand by a founder through the `piviki-infra` runbook, never by OpenTofu. Production's state bucket and state token are created by `epic-librarian-catalogue` story 1, because production's state holds the shared resources. The production start creates production's data bucket and its host tokens."

### H5 (medium): a restore has no defined place to read erasures from, a production deletion may succeed without its record, and the re-delete bypasses `modules/accounts`

- **Unit A, the accounts/erasure module:** writes `erasures/{patronId}` to "its environment's private bucket", meaning the erasure target in the host's runtime config. Only the Test bullet says "a deletion whose erasure record cannot be written fails". In production the module may write after commit, best effort, and still follow the letter.
- **Unit B, the restore runbook:** "a restore re-deletes those ids before the app starts". The phrase "its environment" sits in the write clause, not the restore clause. A restore on a founder's machine (allowed) has no environment name or erasure target, so it reads nothing, or whatever bucket the machine's config names. Its re-delete runs "before the app starts", which means plain SQL. That conflicts with AD-14 ("`modules/accounts` is the only path for account deletion") and skips the circulation withdrawal (cancellations, lost marking, queue promotion, nulling links).
- **What breaks:** restored dumps bring back patrons whose deletion record was never written or was never looked up. Or the re-delete leaves loans, reservations and outbox rows pointing at deleted patrons. And if the module writes the record before commit and the transaction then rolls back, a later restore deletes a live patron.
- **Smallest tightening (AD-17 Backups):** "In every environment `modules/accounts` writes the erasure record before the deletion commits, and a deletion whose record cannot be written fails. A restore reads the erasure records from the data bucket of the environment the dump came from, with a founder-held read token, refuses to continue if it cannot read them, and re-deletes each id still present through `modules/accounts`, run as a one-off core command with jobs and HTTP off, before the app serves traffic. An id with no patron is skipped." Then move the Test bullet's "fails" clause here.

### H6 (medium): the per-environment start check doesn't cover buckets or the erasure target

- **Unit A, the env-file manifest / host secrets:** per Per-environment configuration, "R2 buckets and tokens, the erasure target" are runtime configuration.
- **Unit B, core's start check:** refuses to start only when "the barcode prefix, the mail recipient allowlist, the `backup` service or the host names disagree" with the environment name.
- **What breaks:** a production env file filled with test's erasure target and test's token (both named `piviki-<env>-...`, so it's easy to copy the wrong one) starts cleanly. Production's erasure records then go to test's bucket, where no production restore looks (and a production patron id ends up on AWS-reachable configuration). The opposite case, test filled with production's values, is blocked only because the token is missing, which is a runtime failure, not a start check.
- **Smallest tightening (AD-17 Per-environment configuration):** add "the media bucket, the erasure target and the backup bucket" to the list core and the deploy checks compare against the environment name. Bucket names carry the environment name, and a deploy check writes and reads back a probe object in the erasure target.

### H7 (low): the Structural Seed still shows one shared private bucket

- The seed node `backup[R2 private, EU: backups, erasures, state]` sits under one Cloudflare box with no environment. Read on its own, as seed authors will, it describes the single shared bucket this change replaces.
- **Fix:** label it "R2 private, EU, per environment: production backups, erasures, state" (or two nodes after H2), and let the test node keep "own ... buckets".

## Units checked and found consistent

- AD-8 pilot cleanup writes erasure records through normal deletion. That fits Backups once H5's ordering is fixed.
- Production start's check records are deleted before the import and their erasure records land in production's bucket. No conflict, as long as H1's lifecycle exclusion keeps them.
- Test's wipe (AD-21) and the AWS account closing leave test's R2 bucket on Cloudflare. Production depends on nothing in it, so this stays consistent.
- AD-18: R2 in the EU jurisdiction is Cloudflare, which is already a named processor. No change.
