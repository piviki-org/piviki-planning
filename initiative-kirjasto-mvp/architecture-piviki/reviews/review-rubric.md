---
type: architecture-review
reviews: _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md
ticket: _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md
altitude: initiative
reviewed: '2026-11-02'
verdict: revise-before-build
---

# Rubric review: Piviki MVP architecture spine

## Verdict

**Revise before build.** The shape is sound: the paradigm, the boundary between core and its clients, the single circulation module and the operational choices all fit a two-person team. No structural rework is needed.

Four defects would still cause wrong behaviour or a failed epic if left in:

- F1: one AD contradicts a requirement.
- F2: one AD makes the epics' own Done-when impossible.
- F3: one AD does not prevent the takeover it names, and the Deferred library works against it.
- F4: the reservation model leaves double-booking open.

All four, and the important findings, can be fixed by editing text in the spine.

## Scorecard

| Criterion | Result | Findings |
|---|---|---|
| Fixes the real divergence points, misses none | Partial | F4, F6, F8, F9, F13 |
| Every Rule is enforceable and prevents its stated divergence | Fail | F3, F4, F5, F6, F7, F9, F11 |
| Nothing Deferred could let two units diverge | Partial | F3, F4, F12 |
| Named tech is verified-current | Pass, with compatibility caveats | F3, F10, Verification table |
| Covers every ticket requirement | Fail on R20; all 25 are mapped | F1, F2, F8 |
| Every dimension decided, deferred or open, especially the operational envelope | Partial; no open-questions section | F10, F13 |
| No over-reach | Minor | Over-reach section |

## Blocking findings

### F1: AD-6 contradicts R20 (policy changes need a code change)

- **Evidence:** AD-6 says policy values are "changed by developers through a versioned migration". R20 says each "can be changed without a code change", and `epic-librarian-catalogue` has "a policy value changes without a code change" as Done-when. A versioned migration is a commit, a CI image build and a deploy. `signupOpen` inherits this, so opening sign-up at launch also becomes a deploy.
- **Fix:** Seed the record through a migration, and make later changes through a developer-only runtime path with no screen. For example, a versioned `policy:set` command in the core image that runs against production, validates values and writes who, when and what to an audit field. This honours both R20 and the Decision that developers make the change with no admin screen.

### F2: `signupOpen` makes production verification impossible (AD-8, AD-17)

- **Evidence:** AD-8 closes sign-up and sign-in "to real patrons" through one boolean, which cannot tell a real patron from a tester. The epics must deploy to production and verify end to end there, including sign-up (`epic-patron-site`), reserve to return (`epic-reservation-loop`), and renewal and reminders (`epic-loan-care`). AD-17 allows only local and production, so production is the only place to do this.
- **Also missing:**
  - How test patrons, loans and emails are removed before launch, or kept out of the success signals.
  - Who can reach `beta.piviki.fi`. The Decision says "non-public address", but a subdomain is public and discoverable through certificate transparency logs. The spine has no access control and no noindex rule.
- **Fix:** Add to AD-8 and AD-17:
  - While `signupOpen` is false, only addresses on a `signupAllowlist` in `tenant-policy` may sign up or sign in.
  - Test records are marked `isTest` or removed by a named pre-launch purge, which is part of `epic-launch`.
  - `beta.piviki.fi` sits behind Cloudflare Access or at least serves `noindex`, and the choice is stated.

### F3: AD-8 does not prevent account takeover, and the Deferred library works against it

- **Pre-hijack gap:** AD-8 links on *Google's* verified email but never requires the local address to be verified, and no AD decides email verification.
  - An attacker can sign up with a victim's address and a known password. When the victim later uses Google, the identity links and both now reach one account, whose reservations and history the attacker can see.
  - Unverified addresses also mean reservation emails can go to a stranger, which is a personal-data issue.
- **`payload-oauth2` 1.0.21 does not implement the Rule by default** (package source read):
  - The authorize endpoint only forwards a `state` value supplied in the query. Nothing generates, stores or checks it, and `state` appears nowhere in `callback-endpoint.js`.
  - No code reads `email_verified`. With `useEmailAsIdentity`, it matches and updates by email unconditionally.
  - `onUserNotFoundBehavior` defaults to `create`, which creates the patron through the Local API. That bypasses AD-5's "Patron creation goes through `/api/v1`".
  - PKCE is available only as an option.
- **So the Deferred entry is wrong:** "Hidden behind AD-8" assumes either library can meet the Rule. The choice also changes public paths (`/api/patrons/oauth/*`), which are not among the operations AD-5 allows.
- **Fix:**
  - Choose `openid-client` now, behind `/api/v1/auth/google/*`, and drop `payload-oauth2` from the stack. It is small code, the checks are ours, and it needs no AD-5 exception.
  - Require a verified local email before linking. If a password account is unverified when Google links, clear its password and sessions.
  - Decide email verification at sign-up (`auth.verify`), and add verify to AD-5's list.

### F4: AD-3 leaves double-booking and queue-jumping open

- **Evidence:** A copy is assigned only when marked ready, yet availability is computed from "copies, active loans and held reservations". "Held" is undefined.
  - If held means `ready`, a one-copy title with a `reserved` reservation still shows available, and a second patron can reserve it.
  - Nothing says a title with waiting queue places cannot be reserved directly, so a returned book that is not yet assigned can be taken ahead of the queue. That is the state the Deferred return decision creates, and it breaks R4's first-come rule.
- **Related gaps across epics:**
  - `queue-places` has no state list, and nothing says how a queue place becomes a reservation, or whether two collections are needed at all. Renewal refusal (R6), cancellation (R24), lost-copy cancellation (R23) and the account view (R5) all read this.
  - The spine stores `held` and `on_loan` on copies, while AD-3 derives availability from other records. It also keeps `picked_up` and `returned` on reservations alongside `loans`. Those are two truths for one fact.
  - The ER diagram has no link from loan to reservation.
- **Fix:** Add to AD-3:
  - Availability = non-lost copies, minus active loans, minus reservations in `reserved` or `ready`.
  - A title is reservable only when availability > 0 *and* it has no waiting queue place; otherwise the patron joins the queue.
  - Either merge queue places into reservations with a `queued` state (simplest, one ordered list) or define the promotion and the queue-place states.
  - Declare which record holds the truth for "on loan".
  - Move "returned goes straight to ready or waits" out of Deferred, or state the constraint that holds either way: no one can take the book ahead of the queue.

## Important findings

### F5: The AD-2 lock does not cover per-patron limits or multi-title operations

- **Evidence:** The lock is on the title's row. Maximum books and the overdue block are per patron, so two concurrent reservations on *different* titles both pass the limit. Account deletion and lost-copy handling touch many titles, and without a lock order they can deadlock.
- **Fix:** Lock the patron row first when an operation has a patron, then titles in ascending id order.

### F6: AD-2 and AD-14 guard fields, not deletes, so admin and the scan key can bypass the modules

- **Evidence:** AD-2 makes circulation fields read-only, but a librarian in Payload admin, or the scan key on `copies` and `titles`, can still delete or unpublish a title or copy with active loans or queues. AD-14 calls `modules/accounts` the "only path" for deletion, yet nothing denies the delete button on `patrons`.
- **Fix:**
  - Set generated and admin delete access to false on `patrons`, `reservations`, `queue-places` and `loans`.
  - Allow delete on `titles` and `copies` only with no circulation history.
  - Deny delete to the scan key, and guard unpublishing a title with open circulation.

### F7: AD-9 lets unreviewed scans go public, and its "scoped key" has no home

- **Public images:** Scans go into `media`, and AD-16 sends those to a public bucket immediately. Draft status protects the record, not the images, so unreviewed images are reachable by URL. This matters for a mis-scanned page or a photo of something else.
- **Scoped key:** Payload API keys belong to a document in an auth collection (`useAPIKey`) and take that user's access. AD-7 has only `patrons` and `librarians`, so the key would be a librarian with admin access, which contradicts AD-9. `media` is also missing from the collection list.
- **Fix:**
  - Add an auth collection `service-accounts` with `useAPIKey`, no admin access, and access limited to `titles`, `copies` and `media`.
  - Either hold draft images privately until publish, or explicitly accept the risk in AD-9 and narrow "Prevents" to records.
- **Note:** `scanId` upsert is client logic (query then patch), since the REST API has no upsert. The unique index is the real guard, so say so.

### F8: AD-14 is incomplete on retention and contradicts other ADs

- **Anonymisation:** The ticket's Decision is that loan records are *anonymised* after retention. AD-14 says only "retention runs as a job".
- **Uncovered records:**
  - Reservations and queue places after deletion. The rule sets only loan and donation links to null, though the ER diagram allows null on them too.
  - Outbox rows, which may hold a recipient, may be pending at deletion, and can send mail to a deleted patron.
  - Long-inactive accounts.
  - Backups: no dump retention is set, so "deleted" patrons persist indefinitely. The privacy notice must state a period, and restoring a dump can bring deleted patrons back.
- **Contradictions:**
  - "Patrons hold only name, email, language, gate fields and an optional phone number" excludes donor consent (AD-8) and the Google subject id.
  - AD-14 records consent flags on donations while AD-8 takes consent at sign-up. Name the one source.
- **Success signals:** Nulling loan links undercounts "families making two or more pickups". Fine at six months, but note it in the Deferred entry, which currently treats this as solved. That entry should also confirm that a loan records a pickup date.
- **Fix:** Add a retention line per record type, including outbox purge on delete and dump expiry (for example 30 days). Add a re-apply-deletions step to the restore procedure, and correct the field list.

### F9: AD-5's auth exception creates a second patron contract

- **Evidence:**
  - Generated auth operations return Payload's error shape, not `{ error: { code, message } }`, so the patron site cannot "translate by code" for login and reset failures. R1 requires every patron screen to be translated.
  - `refresh-token`, verify and unlock are not on the allowed list, so applied literally, sessions die at expiry.
  - Forgot-password email must use the patron's stored language (AD-11), which needs a custom template. AD-5 and AD-11 do not say so.
  - `signupOpen` must be enforced on generated login, which AD-8 implies but no surface owns.
- **Fix:** Put all patron auth behind thin `/api/v1/auth/*` handlers and deny patrons the generated surface with no exception. If that is rejected, list the complete operations and pin their error mapping.

### F10: Operational envelope gaps (AD-13, AD-17)

Not decided, deferred or raised as open:

| Gap | Why it matters | Suggested text |
|---|---|---|
| Backup path conflicts with AD-13 | The Seed draws `core` producing the dump, and AD-13 forces all scheduled work through Payload jobs, so the Node image needs `pg_dump` 18. A silent failure means no backups, and the uptime alert watches availability only | Name who runs it and add an alert when no fresh dump exists |
| Backup timing | Restore is tested only before `epic-launch` closes, but the 300-book catalogue is production data from end of November | Backups run from the first production deploy; test restore before the import |
| Encryption key custody | An encrypted dump with its key only on the VPS is lost with the VPS | Key held off-host by both founders |
| Rollback | Migrations are compatible with the previous version but no rollback is stated | Redeploy the previous image tag; forward-fix schema |
| Registry, CI, deploy trigger, OpenTofu state backend, secret store | Named nowhere. `epic-librarian-catalogue` owns them, so little divergence risk, but four repos must agree | Decide, or defer with a decide-by of that epic |
| Disaster recovery | One VPS with no stated recovery target or rebuild path | One line: rebuild from `piviki-infra` and the last dump, target recovery time |
| Monitoring and logs | AD-14 mentions "error reports" but no tool. AD-18 would forbid most hosted trackers. No log retention stated | Decide: container logs only with N-day rotation |
| Cost | The 15 EUR a month budget is only in the memlog, yet it drove the hosting decision | Record it as a constraint |
| Firewall mechanism | Docker-published ports bypass `ufw`, and Cloudflare's IP ranges change | State the OVH network firewall or the `DOCKER-USER` chain, and how the range list stays current |
| Caddy DNS challenge | Needs a Cloudflare DNS module, absent from the stock image, so CI must build a custom Caddy image. Its token is one more secret | Note it, or use a Cloudflare Origin Certificate, which is simpler and also satisfies Full (strict) |

Also to confirm:

- **Rate limits:** Cloudflare's Free plan allows **one** rule, matching on path, with a 10-second period and 10-second timeout (Cloudflare docs). That is weak for login and reset, and sign-up on `/api/v1` is not covered, so it can be used to send mail to strangers. Confirm the plan and add a core-side limit on sign-up and reset.
- **`autoRun` start:** Payload's docs do not say when crons start. Payload initialises lazily in Next.js, so a restart may leave jobs idle until the first request (unverified). Require eager initialisation through `instrumentation` or a health check, and test it. Reminders and the outbox depend on this.

### F11: AD-15's fail-open promise is not fully achievable

- **Evidence:** "With any error ... it serves the unmodified page." Exceeding the Free plan's 10 ms CPU limit gives Error 1102, and Cloudflare's docs say `passThroughOnException` "does not mitigate failures such as exceeding CPU or memory limits". The memlog records this as an assumption to measure, with a fallback, but the spine dropped it. Book pages are the core browse path.
- **Fix:** Add it to the spine as a stated assumption: measure in the first patron-site story, with the paid plan or dropping previews as the fallback. Also check that `/*/book/*` works in `run_worker_first`. The docs confirm globs and give only trailing-wildcard examples.

### F12: Deferred timing conflicts with the epics

- **Email provider:** "First epic that sends email" is in practice `epic-librarian-catalogue` (librarian password reset) or `epic-patron-site` (patron reset and sign-up), not `epic-reservation-loop`, which the initiative names as owner. Set the decide-by to the platform baseline, and tell Sofia the touch point moves.
- **Compliance prerequisite:** AD-18 requires processor agreements "before launch", but real scans and test emails run earlier. No patron data is at stake until the first real sign-up, so this is acceptable. Tie it to `signupOpen`.

### F13: No Open Questions section

The ticket's open items that touch architecture have no home:

- **Art. 9** may change the legal basis and sign-up wording, which affects the AD-8 gate fields and AD-14.
- **Loan values** need a first value, and the `tenant-policy` seed depends on them.
- **The licence** is decided here (MIT) but is still open on the ticket, which says the founders decide. The memlog says Sofia should record it. Confirm and close it there.
- **R21 cutover** has no owned steps: adding `piviki.fi` to CORS and CSRF origins and Google redirects, flipping `signupOpen`, retiring `beta`, and the coming-soon page's hosting. The `tulossa` repo has no deploy configuration, so its hosting cannot be read from it.

**Fix:** Add a short section with owner and decide-by for each, plus a cutover line in AD-17.

## Minor

- **ER diagram:** `LOANS ||--o{ OUTBOX` is wrong. Ready and lost-copy emails have reservations and queue places as subject, and AD-12's key is generic.
- **AD-12:** It says circulation writes outbox rows "with the state change", but reminders have no state change. State that the reminder job writes them through notifications, with the same unique key. Delivery is at-least-once, so a crash after SMTP send can duplicate, and "prevents duplicates" holds only for retries before the send. Accepting that is fine; say so.
- **AD-6:** "Every query filters by it" has no enforcement. A shared base access function, plus a test on the generated surface and modules, would make it checkable. Keep it light given the single tenant.
- **R2 and chat link:** R20 has one "chat group link", but the touch point names WhatsApp and Zalo. Clarify one or two.
- **AD-8:** `policyVersion` has no source for the current version and no rule for re-acceptance when the policy changes.

## Over-reach

These are not shared invariants. They are safe to keep as implementation notes but should not bind as Rules:

| Item | Why | Suggest |
|---|---|---|
| AD-15 `HTMLRewriter` attribute setters | One Worker, one owner. The real invariant is no rules, secrets or personal data, and fail-open | Drop the mechanism |
| AD-16 "resized for web on upload" | One upload path; no other unit depends on it | Move to a story |
| AD-17 "Automatic security updates are on" and similar host hardening | Single owner, no divergence | Keep as an operations checklist. It is within the altitude's operational dimension, so low priority |
| Agent rules row in Conventions | Workspace hygiene, not architecture | Move to project context |

The rest earned its place. AD-10 looks detailed, but scans, admin entry and search all pass through it. AD-18 is a real constraint from the ticket.

## Verification of named tech

Checked 2026-11-02 against the npm registry, GitHub releases, nodejs.org and endoflife.date.

| Name | Spine | Found | Result |
|---|---|---|---|
| `payload` and the four `@payloadcms` packages | 3.92.1 | 3.92.1 latest; engines `^22.23.4 \|\| >=24.22.1` | Current |
| Node.js | 24.22.2 | v24.22.2 newest 24.x | Current |
| Next.js | 16.4.4 | Latest; `@payloadcms/next` peer `>=16.3.3 <17` | Current, compatible |
| React | 19.3.1 | Latest; Mantine peer `^19.2.0` | Current, compatible |
| `@mantine/core` | 9.7.4 | Latest | Current |
| Wrangler | 4.156.1 | Latest | Current |
| Caddy | 2.11.7 | v2.11.7 | Current; see F10 on the DNS module |
| OpenTofu | 1.13.2 | v1.13.2 | Current |
| PostgreSQL | 18 | 18.6, supported to 2030 | Current |
| `openid-client` | 6.8.9 | Latest | Current |
| `payload-oauth2` | 1.0.21 | Latest, published May 2026, peer `payload ^3` | Current, but does not meet AD-8 by default; see F3 |

Not rechecked here: OVHcloud VPS-1 plan and price (memlog cites the OVH catalogue). No build was run, so compatibility rests on declared peer ranges.

## Requirements trace

All of R1 to R25 appear in `binds` and the Capability Map. Coverage gaps by substance:

- **R20:** contradicted (F1).
- **R4:** queue rule not enforced by the model (F4).
- **R19:** retention behaviour incomplete (F8).
- **R13:** reviewed before public holds for records, not images (F7).
- **R1:** login and reset error text (F9).
- **R21:** cutover unowned (F13).

The ticket's constraints on the cut order, tenancy, email, GDPR and ownership are otherwise honoured. The cuttable R6, R9 and R12 can be dropped without touching any AD.
