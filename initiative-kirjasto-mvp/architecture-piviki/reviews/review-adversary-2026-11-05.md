# Adversarial review: environments update (2026-11-05)

Spine: `architecture-piviki.md` (status final, updated 2026-11-05). Scope: the five 2026-11-05 decisions in `.memlog.md`: AWS is test from its first bootstrap, production is an empty OVH VPS-1 started by an `epic-librarian-catalogue` story before `epic-initial-load`, the paid-plan rule is dropped, `patronAccountsAllowed` is retired, and stories before the production start are built and checked on test. Method: build two units one level down that each follow every AD to the letter but do not fit together, or one unit that follows every AD and still causes harm. I reviewed by reading only. I checked `piviki-infra/tofu/prod` and `docs/secrets-inventory.md` only to confirm the shapes that units are already copying.

**Verdict:** No traces of the move are left in the spine, and the test/production split reads cleanly. Making test the *first* environment opens three holes the old order hid:
- the zone-wide Cloudflare resources that two OpenTofu states share;
- backups and the restore test, which cannot be "built and checked on test" because test runs no backup;
- pre-launch patron data on production, now that `patronAccountsAllowed` is gone.

None of these needs a new AD. Each closes with one or two sentences.

## Findings

### F1 [high] Two OpenTofu states can both own the `piviki.fi` zone and its zone-wide resources

**Pair.**
- Unit A is story 1 (now on test). It follows AD-17 ("OpenTofu manages Cloudflare resources for both environments, including DNS, with one encrypted state per environment") and builds `tofu/test`. The natural way to do that is to copy `tofu/prod`, which today does three zone-wide things: it imports `cloudflare_zone.piviki`, creates the Access one-time-PIN identity provider when no id is given, and (from story 8) creates the one Cloudflare rate-limit rule. Test is now the first environment, so `tofu/test` is the state that imports the zone and creates the IdP.
- Unit B is the production start. It follows the same sentence and runs `tofu/prod`, which imports the same zone again and either creates a second OTP IdP or adopts the first one by id. It also creates "one Cloudflare rule" for login, reset and sign-up (AD-17 Hardening). The free plan allows only one such rule per zone, so either the apply fails or the rule ends up owned by whichever state wrote it last.

**Harm.** The Deferred row "Test environment after the credits: delete it or move it" invites a `tofu destroy` of `tofu/test`. That destroy deletes the imported zone (and with it the coming-soon page, which the epic's "Must not change" protects), the OTP IdP that production's Access uses, and possibly the rate-limit rule. Each state follows AD-17 to the letter. AD-17 never says which state owns a resource that both environments share.

**Close in AD-17, Infrastructure as code.** Add after "one encrypted state per environment in the private bucket":

> Resources the environments share — the zone and its settings, the coming-soon records, the Access identity provider, the one rate-limit rule (whose expression names both environments' hosts), and the private bucket holding state — live only in the production state; the test state holds only test-named records, applications, Tunnel, tokens and buckets, and destroying it touches nothing else.

The production state does not exist until the production start. So also say which state story 1 creates first. Either story 1 creates `tofu/prod` holding only the shared resources, or "the production state is created first, holding only the shared resources, before the test state".

### F2 [high] The backups and the restore test have no environment they can be built and checked in

**Pair.**
- Unit A is the backup story (today story 7). It follows "Stories before it [the production start] are built and checked on test". It also follows "[test] runs no `backup` service and holds no token for the production private bucket". So on test it can build the container but cannot run it, watch a nightly dump, or trigger the heartbeat.
- Unit B is the production start story. It follows its bullet: "The outside port scan, the `first-register` deploy check and the hardening checks pass there". Nothing in that bullet starts backups, checks the heartbeat or sends mail. The epic can therefore reach `epic-initial-load` with backups never having run anywhere. The only backup gate left is "A restore is tested on the production host before the bulk import", and nothing sequences that against the production start.

**Second harm: the restore test can pass while proving nothing.** Production starts empty, so a dump taken right after the production start holds one librarian and no titles, copies, barcodes, media references or erasure records. Restoring it "on the production host" follows AD-17. It does not exercise:
- re-deleting erasures,
- the barcode sequence (Deferred: "Barcode sequence after a restore"),
- the core tag recorded in the dump (AD-21).

**Third harm: records the checks leave behind.** The old move step 1 deleted check records before the final dump. With that step gone, the spine has no rule for records that checks create *on production* between the production start and the bulk import, such as a title made to give the restore something to restore, or a test-printed label. It also has no rule for whether their PVK barcodes may be consumed.

**Close in AD-17.**
- Production start: replace "Stories before it are built and checked on test." with:

  > Stories before it are built and checked on test, except what test does not run: the `backup` service, its heartbeat, the restore and mail from `piviki.fi` are checked on production after the production start and before the bulk import.

- Backups: after "A restore is tested on the production host before the bulk import", add:

  > , from a dump holding at least one title, copy, media reference and erasure record; check records created on production are removed before the import, and the barcodes they used are not reissued.

  This also gives the Deferred barcode-sequence row something to test against.

### F3 [high] Retiring `patronAccountsAllowed` removes the only guard on patron records reaching production before the privacy notice and agreements

**Unit that follows every AD and causes harm.** The production start runs before `epic-initial-load`, months before `epic-patron-site`. On production `signupOpen` is false, so AD-8 lets in "emails on the beta allowlist". AD-8 limits the allowlist's contents only "In the test environment". A founder adds a few real parents as beta testers to production's allowlist. Allowlisted sign-ups are then legal. Their data reaches OVH, Cloudflare and the mail provider, while AD-18 requires agreements on file only "before `signupOpen`" and AD-8 requires the notice only before the flag flips.

**Second path.** AD-8's gate applies to "sign-up and sign-in ... on every path including Google". A librarian creating a `patrons` record in Payload admin or the generated API is neither of those. AD-14 denies admin and generated *deletes* on `patrons`, not creates. The retired flag was the only thing that refused "every patron create, including through admin and the generated API" (old story 2). So the AD-19 row "`patrons` ... no public route", created by this epic, still lets a librarian create a real patron on production. A donation story could do exactly that, to record a donor or a phone number.

**Close in AD-8.**
- Change "In the test environment, the beta allowlist holds only addresses the founders control" to:

  > In the test environment, and in production until the privacy notice and usage policy are published and the AD-18 agreements are on file, the beta allowlist holds only addresses the founders control.

- Add:

  > Patron records are created only by `modules/accounts`; admin and generated creates on `patrons` are denied.

  (Or put the second sentence in AD-14 beside the delete rule.)

Optionally, in AD-18, change "before `signupOpen`" to "before the first real patron's data reaches it".

### F4 [medium] A wrong runtime value at the production start permanently stamps the real collection as test

**Unit that follows every AD and causes harm.** AD-17 makes the barcode prefix, mail sender, test recipient allowlist, backup on/off, hosts and erasure target separate runtime values. The production start fills the production env files from the manifest (AD-21 lists key names, not values). A founder copies test's file and leaves `TST` in it. Core follows AD-9 and assigns `TST000001` onward to all ~300 imported copies. "A barcode never changes", so the error is permanent. The label story follows "test labels carry a TEST mark" and prints TEST on every real label. Production's own lookup accepts the prefix, so no check fails.

Each value is independent, so other combinations also pass every AD: test with the recipient allowlist off, or production with backups off. Nothing ties the values to a single name for the environment. `piviki-infra` already writes `/etc/piviki/environment`, but the spine does not mention it.

**Close in AD-17, Per-environment configuration.** Add:

> Each host has one environment name, `production` or `test`, set by the bootstrap. Core refuses to start, and every deploy check fails, when a value derived from it disagrees: barcode prefix `PVK`/`TST`, the test recipient allowlist on exactly in test, the `backup` service only in production, and the host names. The TEST mark on labels follows the prefix.

### F5 [medium] Host names and the "before any DNS record" rule assume production is the first environment

- **AD-5 against the production start.** AD-5 says "The first librarian is seeded over SSH before any public DNS record exists." Test's records, and the coming-soon records, already exist when production starts, so the production start cannot follow AD-5 literally. It follows its own wording instead ("before the `admin` and `api` records exist"). A deploy check written to AD-5's text would block the production start. **Close in AD-5:** "before that environment's `admin` and `api` records exist."
- **SSH host names.** CI reaches each host "over SSH through the host's own Tunnel", but the Hosts convention names no SSH host. `tofu/prod` uses `ssh-prod.piviki.fi`. The memlog says `ssh-test`. One unit writes `test-ssh` to match `test-admin`/`test-api`, another writes `ssh-test` to match `ssh-prod`. Since "neither environment's lists name the other's hosts", a mismatch between the workflow's Environment secret and the Tunnel ingress shows up only on the first deploy. **Close in the Hosts convention:** add "`ssh-prod.piviki.fi` and `ssh-test.piviki.fi` for CI deploys through each host's Tunnel".

### F6 [medium] Test's "import script's output" puts real, unreviewed scans in public places

**Unit that follows every AD and causes harm.** AD-17 says test's data is "the import script's output". AD-9 gives the script an environment argument. `epic-initial-load` follows both and runs the real scans against test, again and again. Each run uploads cropped book images to test's public media bucket (unguessable, but fetchable by URL, as AD-9 accepts *for production review*). Some of those images show children's handwritten names, which AD-16 relies on the production review checklist to reject. On test nobody reviews and nobody cleans up. AD-14 ("no data about children") and AD-16 assume that review always happens. The test environment's "can be stopped at any time" makes it even less likely anyone cleans the bucket.

**Close in AD-17, Test environment.** After "Its data is the import script's output", add:

> , run only on scans whose images a founder has checked against AD-16's checklist, or on a fixed sample set; test's media bucket is emptied when a trial import is discarded.

### F7 [medium, to verify] The free plan has its own end date, separate from the credits

AD-17 drops the paid-plan rule on the grounds that "the founders keep watching the credits and the AWS bill". As I understand the AWS free plan introduced in 2025, the account closes automatically when its credits run out *or* after about 6 months from creation, whichever comes first, unless it is upgraded. Some services and instance types may also be restricted. I could not check this here; I fetched nothing. If it holds, test disappears around May 2027 even with credits left, which could fall in the middle of `epic-patron-site` or `epic-launch` while "stories ... are checked on test". **Close in the Deferred row "Test environment after the credits":** "...when the credits run low or the free plan's end date nears, whichever is first; upgrade the plan or re-home test before then." A founder should confirm the plan's terms on the AWS free-plan page first.

### F8 [low] Which private bucket holds test's state, and what the lock covers

- AD-17 IaC: "one encrypted state per environment in the private bucket". Test environment: "its own private test bucket". A unit that keeps test's state in test's private bucket cannot have `tofu/test` create that same bucket (chicken and egg). A unit that keeps it in the production private bucket means anyone running `tofu/test` holds a production-bucket credential. That is allowed on a founder's machine, but the spine should say so. **Close:** "both states live in the production private bucket, written only from founders' machines."
- AD-17 Backups: "a 7-day bucket lock protects recent ones". Read literally as a lock on the whole bucket, it blocks OpenTofu state overwrites for 7 days. Story 7 already scopes it, but the spine does not. **Close:** "a 7-day lock on the dump prefix".

### F9 [low] Migration and checking rules assume production exists

- "a migration that removes or renames ships only after the tag that stopped using the old shape is confirmed in production". Before the production start no production exists. One unit holds every contract migration until then; another treats the rule as trivially met. Both readings follow the rule. **Close:** "Before the production start, migrations need only work with test's running tag; production starts by applying them all in order on an empty database."
- The app-lane stories (10 to 13 today) may finish after the production start. The spine says only where stories *before* it are checked. **Close:** "Stories after it are checked on production, without leaving check records (see F2)."

## Ticket lines that will need changing (spine findings aside)

- `epic-librarian-catalogue.md` Done-when "The platform runs in production...": the order changes, and production is a separate story.
- `epic-librarian-catalogue.md` Notes, decisions of 2026-11-03: "Production runs on AWS EC2 ... then moves to OVH VPS-1 by backup restore", "Stories 10 to 13 are checked on the AWS pre-launch environment; the move story deletes their check records", "Story 1 is the tracer bullet ... in production", and "Story 7 also removes the test records of stories 3 to 6".
- `tickets.toml` story 1: title "in production"; description "on a paid-plan AWS account" and production host names; verify "naming production", "admin.piviki.fi", "api.piviki.fi"; `hitl` comment "paid-plan AWS account".
- Story 2: the `patronAccountsAllowed` clause in the description and the verify; add the admin/generated create denial (F3).
- Stories 3, 5, 6, 8, 9, 10, 11, 13: each verify says "On production" / "production review view" / "PVK"; these become test (`TST`), apart from what F2 moves to production.
- Story 7: "the test records from stories 3 to 6 are removed" no longer applies; the restore drill needs a non-empty dump and must come after the production start (F2).
- Story 9: verify "from an @piviki.fi address ... on admin.piviki.fi" cannot be checked on test, which sends from `testmail.piviki.fi`.
- A new production-start story is needed, placed before `epic-initial-load`'s pins on 6, 7 and 8. `epic-initial-load` should pin to it as well.
- `piviki-infra/tofu`: today only `prod/` exists, and it imports the zone and creates the OTP IdP. `test/` must not copy those (F1).
