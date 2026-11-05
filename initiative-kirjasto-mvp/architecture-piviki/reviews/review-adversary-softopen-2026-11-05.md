# Adversarial review: soft-open update (2026-11-05)

Scope: the spine diff of 2026-11-05 (production start timing, `epic-soft-open`, `beta` retired, the conditional `signupOpen` flip, trial imports on test) and the `.memlog.md` entries from "Production start timing" to the end. The tickets are stale and were read only to understand the units.

Method: for each finding, two units one level down that each obey every AD to the letter but conflict or cause harm under the new sequence, then exact text to close the hole.

**Verdict:** The update is coherent at epic level. The hole is timing. The `signupOpen` flip, and the AD-8 reset with it, can now come long after the soft open, after `epic-reservation-loop` is in production. The reset, the allowlist rule and the patron-site and Google rules were all written for a single launch-day switch.

---

## Critical

### C1. The AD-8 reset after a late flip deletes real circulation state and comes back on restore

**Pair.**
- *Unit A, an `epic-reservation-loop` story, after a browse-only soft open:* a founder on the pilot allowlist reserves a real title. A librarian marks it `ready` and hands it over, so a real `PVK` copy is now `on_loan` to the founder. This is allowed: AD-8 permits allowlisted patrons on production, and AD-2 runs it through the circulation module. A pilot copy may also be marked `lost`, and a pilot patron may be linked as a donor (`donations` exists from `epic-librarian-catalogue`).
- *Unit B, an `epic-soft-open` story, months later when the notice is ready:* it runs "one reset that hard-deletes allowlisted patrons with their reservations, loans and outbox rows and returns affected copies to `available`", then flips the flag.

**Harm.**
1. A copy still in a founder's bag becomes `available`. The first real patron after the flip reserves it, and it is `reserved` with no book on the shelf.
2. "Returns affected copies to `available`" also brings back a pilot copy that is really `lost`.
3. A "hard-delete" is not the AD-14 `modules/accounts` path. It writes no `erasures/{patronId}`, so the next restore (AD-17 requires one yearly) brings the pilot patrons back from any dump taken before the reset.
4. A pilot donation either blocks the delete or leaves a dangling link, depending on how the story implements it.
5. When the reset is written at soft-open time, `reservations`, `loans` and `outbox` may not exist yet. If the flip is deferred, the same script then runs against a schema it was never written for.

**Proposed text (AD-8, replace the reset sentence):**
> Immediately before the flag flips, `epic-soft-open` runs one reset against the schema then running in production. It first refuses to run while any allowlisted patron has an active loan or a `ready` reservation, so every pilot book is physically back and returned through `modules/circulation`. It then deletes every allowlisted patron through the `modules/accounts` deletion path in AD-14, so the circulation withdrawal runs, `donations` and `outbox` links are nulled and `erasures/{patronId}` is written for each. It cancels their open reservations, returns only `held` copies to `available` through queue promotion, and leaves `lost` copies lost. After the reset, the AD-14 canary check runs against a fresh `pg_dump` for every deleted pilot email and id, and the flag flips only if it finds none.

---

## High

### H1. The allowlist may hold real families between the notice and the flip, and the reset deletes them

**Pair.**
- *Unit A:* the privacy notice and the AD-18 agreements are on file, but the soft open waits on `epic-patron-site` or on the founders. AD-8 restricts the allowlist to founder-owned addresses only "until the privacy notice is published and the AD-18 agreements are on file". A pilot story therefore adds a few real friend families to the allowlist, to the letter.
- *Unit B:* the soft-open reset hard-deletes "allowlisted patrons with their reservations, loans": real people's accounts and history, without warning, while real books are in their hands.

The name also drifts. The memlog says "pilot allowlist of founder-owned emails", AD-8 says "the founders' pilot allowlist", which also reads as "the founders' list of pilot families", and AD-6 and AD-17 Per-environment configuration still say "beta allowlist".

**Proposed text (AD-8, replace the allowlist sentence):**
> While `signupOpen` is false, production's pilot allowlist, like test's, holds only addresses the founders control, whatever has been published. Real patrons join only through open sign-up after the flip.

**Proposed text (AD-6, AD-8 and AD-17 Per-environment configuration):** replace "beta allowlist" with "pilot allowlist" throughout.

### H2. The production patron site, its Worker and its routes have no owner or check before the cutover, and the patching rule pulls them forward

**Pair.**
- *Unit A, a Dependabot patch to `piviki-patron` after the production start:* AD-17 Patching says "a patch counts only once deployed to every running environment", and production is a running environment. AD-17 Build and deploy lets the workflow name `production`. The story deploys the patron site to production, and its Worker route `piviki.fi/*/book/*`, or a custom domain on `piviki.fi`, goes live next to, or over, the coming-soon page.
- *Unit B, the `epic-soft-open` cutover story:* it assumes no production patron build exists. "Point `piviki.fi` at the patron site" then makes the first-ever production build go public at the moment of the switch. That build has production's `api.piviki.fi`, CSP `connect-src` and Worker read target, and none of it has been checked, with no stated rollback.

**Proposed text (AD-17 Environments, append):**
> Until the soft open, the deploy workflow refuses `production` for `piviki-patron`, and production has no patron static assets, Worker or Worker route. A patron-site patch counts as deployed once it is on test until then. `epic-soft-open` creates production's patron build, Worker and routes. Its first deploy is followed at once by the patron deploy checks against `piviki.fi`. If they fail, OpenTofu restores the coming-soon records, which stay in production's state until the soft open is confirmed.

### H3. Production's Google OAuth client has no owner, and "add `piviki.fi` to Google redirects" names the wrong host

**Pair.**
- *Unit A, an `epic-patron-site` Google story:* it builds and checks Google sign-in only on test, with test's own client (AD-17 Test environment). The redirect URI is on the API host (`/api/v1/auth/google/*`, AD-5 and AD-8), so on production it is `https://api.piviki.fi/api/v1/auth/google/callback`, not on `piviki.fi`.
- *Unit B, the `epic-soft-open` cutover:* "add `piviki.fi` to production's … Google redirects" assumes a production client already exists and that `piviki.fi` is a redirect URI. Neither is true. Also:
  - Nothing says when the production client is created. If the production start creates it, Google deletes it after 6 months unused (AD-17 notes this for test).
  - Publishing the OAuth app beyond test users requires a homepage and privacy-policy URL on an authorised domain. That ties the Google flip to the notice being hosted on `piviki.fi`.

**Proposed text (AD-17 Cutover, replace the first clause):**
> `epic-soft-open` creates production's Google OAuth client, with the exact redirect URI `https://api.piviki.fi/api/v1/auth/google/callback` and `https://piviki.fi` as the only post-sign-in return origin. The client stays in Google's Testing status, with only the founders' pilot addresses as test users, until `signupOpen` flips. It is published, with the privacy notice URL on `piviki.fi`, as part of the flip. Then it adds `piviki.fi` to production's CORS and CSRF origins, …

### H4. A browse-only soft open still shows sign-up, sign-in and Google to the public

**Pair.**
- *Unit A, `epic-patron-site`:* it builds sign-up, sign-in and the Google button as always visible, which is correct for an open library and violates no AD.
- *Unit B, a browse-only `epic-soft-open`:* public parents on `piviki.fi` fill in sign-up and get the AD-7 non-revealing response, then no email ever comes. Or they click Google, and core receives their verified email and `sub` at the callback before the privacy notice exists, which is processing AD-18 says needs Google named first. It is then refused by the allowlist. Reserve buttons from `epic-reservation-loop` (M5) appear on the same pages.

**Proposed text (AD-8, append):**
> `/api/v1/public` exposes whether sign-up is open. While it is not, the patron site shows no sign-up, sign-in, Google or account entry point on any linked page, only a "sign-up opens soon" message. Pilot patrons use an unlinked sign-in path. Core refuses to start the Google flow for anyone without a pilot session or a pilot email hint, and refuses a non-allowlisted sign-up with the stable code `signup_closed` before storing or emailing anything.

---

## Medium

### M1. Deploy and start-up checks that need a patron origin fail on production before the soft open

**Pair.**
- *Unit A, an `epic-patron-site` story:* it adds a deploy check: "a CORS preflight from the patron origin to `/api/v1` is allowed", and core start-up requires a patron origin in the per-environment configuration. Both are allowed by AD-17 ("every deploy runs … checks against the target's own hosts", plus the Per-environment configuration start-up check).
- *Unit B:* production has an empty patron origin list until the soft open (AD-7), so every production core deploy, including a 72-hour critical patch, fails or stops core from starting.

**Proposed text (AD-17 Per-environment configuration, append):**
> Production's patron origin list is empty until the soft open, and core starts with it empty. Deploy checks that need a patron origin are skipped while the target's list is empty, and must pass once it is not.

### M2. Production checks after the real import, or after the flip, are unruled

AD-17 rules check records only "before the real import". After that:
- An `epic-librarian-catalogue` or `epic-loan-care` story may check labels, merging or donations on production with a check title or copy. A check donation makes the copy undeletable (AD-2), so it stays in the catalogue with a real `PVK` barcode. The same applies before the import: a check donation blocks the "deleted before the import" rule.
- After the flip, an `epic-reservation-loop` or `epic-loan-care` check by a founder takes a real queue place, ordered by creation time, ahead of real families.

**Proposed text (AD-17 Production start, append):**
> Check records on production never include donations. After the real import, production checks create no titles, copies or donations. After the flip, no check on production creates a reservation or loan. Circulation is checked on test, and mail from `piviki.fi` is checked once, at the production start.

### M3. "Production … is not public" is not literally true

`api.piviki.fi` is reachable from the internet before the soft open (AD-5). It serves public reads of every published title, plus the auth endpoints, and the public R2 bucket holds the covers. A story that relies on "not public", for example skipping the rate-limit rule or `noindex`, is wrong.

**Proposed text (AD-17 Production start, replace "and is not public"):**
> and has no patron site and no link from `piviki.fi`. `api.piviki.fi` is reachable and gated by the pilot allowlist, the rate limits and the hardening checks apply from the production start, and it sends `X-Robots-Tag: noindex`.

### M4. The flip conditions are spread across four places, and `epic-launch` does not depend on the flip

What must hold before `signupOpen` flips is now stated piecemeal:
- AD-8: the privacy notice, the usage policy and the AD-18 agreements.
- AD-17 Breach readiness.
- The Open Questions due "Before `signupOpen`".
- The Google publishing requirement (H3).

A flip story that reads only AD-8, which the update names as the condition, flips without the breach runbook. `epic-launch` "owns the first real loan" but has no stated dependency on the flip or on `epic-reservation-loop` being in production.

**Proposed text (AD-17 Cutover, append):**
> `signupOpen` flips only when one checklist owned by `epic-soft-open` is complete: the privacy notice and usage policy published on `piviki.fi`, the AD-18 agreements on file, the AD-17 breach-readiness items, every Open Question due before `signupOpen` closed, the Google client published (H3) and the AD-8 reset with its canary passed. `epic-launch` starts only after the flip and after `epic-reservation-loop` is in production. The first real loan is the first loan created after the flip.

### M5. After the soft open, half-built capabilities ship to a public site

**Pair.**
- *Unit A, an `epic-reservation-loop` patron story:* it deploys "Reserve" to production. This is allowed: the core tag check passes because the endpoint exists.
- *Unit B, the librarian ready and hand-over story:* it is not deployed yet. With the flag already flipped, real families queue for books nobody can hand over.

**Proposed text (AD-17 Build and deploy, append):**
> After the soft open, a patron-facing action is shown on production only when every core and librarian part it needs is deployed there. Until then, the build for `production` hides it.

---

## Low

### L1. Trial imports on test against AD-9's "runs once"
AD-9 says "the initial load runs once" and that the key and service token are "revoked after the initial load". Trial imports on test run many times. Review and merge work done on test is lost, because nothing moves from test.
**Proposed text (AD-9):** "The initial load runs once on production. Trial imports run on test as often as needed, each after test's catalogue is wiped. Review on test is practice only, and labels are never printed from test for real books."

### L2. Wording left over from "launch"
- AD-8 Prevents ("test records reaching launch") and AD-17 Prevents ("a launch cutover nobody owns") should say the soft open, or the flip.
- R21 ("replaces the coming-soon page … at launch") now maps to the soft open. Add `epic-soft-open` to the R21 row in the Capability Map.
- The AD-19 table lists `epic-soft-open` before `epic-patron-site`, which reverses the build order. Move it below `epic-patron-site`.

### L3. Pilot loans and the success signal
Success is measured "six months after the first loan". Pilot loans before the flip must not count.
**Proposed text (Deferred, Success-signal reports row):** "The first loan is the first created after the `signupOpen` flip. Pilot loans are deleted by the AD-8 reset and never counted."

### L4. Production's Google client could be created too early
If anyone creates production's client at the production start, it can lapse after 6 months unused before the flip. H3's text closes this by having `epic-soft-open` create it.
