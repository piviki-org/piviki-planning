# Rubric review: soft-open update (2026-11-05)

Spine: `architecture-piviki.md` (status final, updated 2026-11-05)
Diff: `spine-softopen.diff` (scope, AD-7, AD-8, AD-17 Production start / Environments / Test environment / Cutover, AD-19, Hosts convention, Structural Seed, Open Questions)
Authority: `.memlog.md` entries from "Production start timing (2026-11-05, user)" to the end.

## Verdict

Pass with fixes. Every in-scope memlog decision landed, the AWS-production option is correctly absent, and the closed scans question was removed. No critical findings. Two high findings concern sequencing for the next level down: the spine never says which epic or story owns the real import, or what may wait on the founder-timed production start and the conditional `signupOpen` flip. Sofia's replan needs those answers. Several "launch" and "bulk import" leftovers remain.

## Checklist summary

| Criterion | Result |
|---|---|
| Real divergence points for epics/stories fixed, none missed | Partial: real-import ownership, the floating production-start story, the flag-flip story and the first production patron deploy are still open (H1, H2, M3) |
| Touched AD Rules enforceable and prevent their divergence | Partial: "is not public" can't be enforced (M1); "pilot allowlist" vs "beta allowlist" (M2); trial imports vs "runs once" (M4) |
| Nothing stale or contradictory | Partial: "bulk import" x2, "launch" in AD-8 and AD-17 Prevents, AD-19 row order (L1-L4) |
| Every in-scope memlog decision landed | Yes, with one gap: scans-on-test binds AD-16, but AD-16 is unchanged (M5) |
| Operational envelope complete | Partial: production Google OAuth client lifetime (M6), first production patron/Worker deploy (M3) |

Memlog trace (in scope):

| Memlog entry | Landed at | Status |
|---|---|---|
| Production start timing | AD-17 Production start, Environments ("The soft open follows ...") | Landed. Real-import ownership still open (H1) |
| Patron site before the soft open (beta retired) | AD-17 Environments, Cutover; Hosts convention; Seed diagram | Landed |
| Soft-open sign-up gate | AD-8 last bullet, AD-17 Cutover | Landed. Flip-story sequencing open (H2), naming (M2) |
| Production start owner (hitl founder story; backups, restore drill, mail checked on production; admin+api only) | AD-17 Production start | Landed. Owner of the production checks is implicit (M7) |
| Production host candidates -> dropped for OVH | AD-17 Host / AWS / AD-18 unchanged ("Production never runs on AWS") | Landed correctly (nothing to add) |
| Scans on test | AD-17 Test environment; Open Question removed | Landed in AD-17 only, not AD-16 (M5) |
| Deliverable: spine only | n/a | Respected |

---

## Critical

None.

## High

### H1. No epic or story owns "the real import", and its place in the epic order is undefined

Quotes:
- AD-17 Production start: "run by a founder when the founders choose and always before the real import"
- AD-17 Environments: "The soft open follows `epic-patron-site` and the real import."
- Scope: "epic-librarian-catalogue, epic-initial-load, epic-patron-site, epic-soft-open, ..."

Problem: "real import" is a new term that the spine never ties to an epic. AD-9 still says "The initial load runs once", and AD-7 says "revoked after the initial load". Since production now starts whenever the founders choose, possibly after `epic-patron-site`, the production run of `epic-initial-load` can't be scheduled from the scope order. Sofia can't tell whether `epic-initial-load` closes before `epic-patron-site` with only trial imports done, or stays open until the founder-run production start. The two epics would be sliced differently depending on which reading she picks.

Proposed fix, appended to AD-17 Environments after "The soft open follows `epic-patron-site` and the real import.":

> The real import is the initial load into production: the last story of `epic-initial-load`, which depends only on the production start story and its production checks. Every other `epic-initial-load` story, including the import script and its trial imports, is built and checked on test and waits on nothing in production. `epic-patron-site` waits on neither the production start nor the real import; only `epic-soft-open` waits on both.

Also change AD-9 "The initial load runs once, single-threaded," to:

> The initial load (the real import into production) runs once, single-threaded,

### H2. Nothing says what may wait on the founder-timed production start story or on the conditional flag flip

Quotes:
- AD-17 Production start: "Stories before it are built and checked on test, and so are the import script and its trial imports"
- AD-8: "otherwise production opens browse-only, sign-up stays limited to the founders' pilot allowlist, and `epic-soft-open` flips the flag once they are."

Problem: (a) The production start story belongs to `epic-librarian-catalogue` but runs whenever the founders choose. "Stories before it" no longer names a fixed set, and the spine doesn't say whether `epic-librarian-catalogue` can close, or later epics start, before that story runs. (b) The flip story of `epic-soft-open` may wait indefinitely on the privacy notice and agreements. With `epic-soft-open` placed before `epic-reservation-loop`, a strict epic order would block reservation work on legal paperwork. Both are sequencing choices for the epic and story level. The memlog doesn't decide (b), so either the founders decide it or it goes in Open Questions.

Proposed fix (a), replacing "Stories before it are built and checked on test, and so are the import script and its trial imports;" with:

> Every other story is built and checked on test, and so are the import script and its trial imports; no story except the real import and `epic-soft-open` waits on the production start, which may stay open after the rest of `epic-librarian-catalogue` closes;

Proposed fix (b), if the founders agree, appended to AD-8's last bullet:

> The flip is its own story of `epic-soft-open`; it may close after later epics start, and no other epic waits on it, except `epic-launch`'s first real loan.

If they don't agree, add to Open Questions:

> | Does `epic-reservation-loop` wait for `signupOpen` to flip, or only for the browse-only soft open? | Founders | Before Sofia's replan of `epic-soft-open` | AD-8, AD-17 Cutover |

## Medium

### M1. "is not public" can't be enforced and conflicts with `api.piviki.fi` serving public reads

Quote (AD-17 Production start): "Until the soft open, production serves only `admin.piviki.fi` behind Access, `api.piviki.fi` and its deploy SSH host, and is not public."

Problem: `api.piviki.fi` isn't behind Access, and `/api/v1/public` returns every published title once librarians publish after the real import. No check can verify "not public". What actually holds is that no patron-facing origin exists.

Proposed fix, replacing ", and is not public." with:

> ; production's CORS and CSRF lists name no patron site origin, so no browser site can use `api.piviki.fi`, which still answers `/api/v1` with `noindex` and the AD-8 gate. A deploy check confirms the lists hold no origin except `admin.piviki.fi` until `epic-soft-open` adds `piviki.fi`.

### M2. Two names for one allowlist: "pilot allowlist" and "beta allowlist"

Quotes:
- AD-8: "sign-up stays limited to the founders' pilot allowlist"
- AD-6, AD-8, AD-17 Per-environment configuration: "the beta allowlist"

Problem: Now that beta is retired, an implementer could read "pilot allowlist" as a second configuration value. "Browse-only" is also slightly wrong, because allowlisted founders can still sign up and sign in.

Proposed fix, replacing "otherwise production opens browse-only, sign-up stays limited to the founders' pilot allowlist," with:

> otherwise production opens for browsing only, sign-up and sign-in stay limited to the beta allowlist of founder-controlled addresses (the pilot),

Optionally, rename "beta allowlist" to "pilot allowlist" in all four places (AD-6, AD-8 x2, AD-17 Per-environment configuration) so no "beta" remains.

### M3. No story owns the first production patron site and Worker deploy, and nothing stops an early one

Quotes:
- AD-17 Build and deploy: "The patron site deploys through the same workflow, built for that environment"
- AD-17 Cutover: "point `piviki.fi` at the patron site"

Problem: "production has no patron site host" holds only if nothing deploys the production patron build or the `/*/book/*` Worker route before the soft open. The cutover list leaves out the deploy itself, the Worker route on `piviki.fi`, and replacing the coming-soon records in production's OpenTofu state.

Proposed fix, replacing the Cutover bullet with:

> **Cutover:** `epic-soft-open` owns the soft open: make the first production patron site and Worker deploy, which the workflow refuses before then; replace the coming-soon records in production's OpenTofu state so `piviki.fi` points at the patron site; add `piviki.fi` to production's origins and Google redirects; and, per AD-8, run the reset and set `signupOpen`. `epic-launch` owns the first real loan, not the cutover.

### M4. Repeated trial imports on test conflict with "runs once" and the per-environment processed-scans file

Quotes:
- Design Paradigm: "Turns scans into draft catalogue records over the generated API, once"
- AD-9: "keeps one local file of processed scans per environment, and skips those scans on a re-run"
- AD-17 Production start: "the import script and its trial imports"

Problem: A second trial import on test skips every scan, or creates `duplicate_title` skips, unless test's data and the processed-scans file are both reset. The spine doesn't say how a fresh trial starts, so stories may each invent their own reset.

Proposed fix, appended to AD-9's initial-load bullet:

> A fresh trial import on test starts only after test's catalogue is wiped by the bootstrap and test's processed-scans file is deleted together; production's file is never deleted.

### M5. The scans-on-test decision binds AD-16, but AD-16 is unchanged

Quote (AD-16): "The import script strips metadata and crops to the book before upload and before any LLM call ... The review checklist rejects handwritten names and backgrounds."

Problem: The memlog entry "Scans on test" binds AD-16. The restriction "trial imports on test use only that checked set" appears only in AD-17's description of test data, as "the import script's output from the founder-checked scans". AD-16 is where the public-bucket rule lives, and it has no matching line.

Proposed fix, appended to AD-16's Rule:

> On test, the import script runs only on the founders' checked set of the initial collection's scans, already reviewed against this checklist, because test's public bucket has no review before upload.

### M6. Production's Google OAuth client may be deleted before the soft open

Quote (AD-17 Per-environment configuration): "the Google OAuth client, which Google deletes after 6 months unused, so test's is re-created when needed"

Problem: Production now starts whenever the founders choose and may run months before the soft open with no patron site. A production Google client created at production start could sit unused for 6 months and be deleted, and then the soft open would break Google sign-in.

Proposed fix, replacing "so test's is re-created when needed" with:

> so test's is re-created when needed and production's is created by `epic-soft-open`, not at the production start

Also add "create production's Google OAuth client" to the Cutover list.

### M7. Which story checks backups, the restore drill and mail on production is only implied

Quote (AD-17 Production start): "what test cannot show (the `backup` service, its heartbeat, a restore, mail from `piviki.fi`) is checked on production before the real import"

Problem: The memlog puts these checks in the founder-run production start story ("it brings up the chosen host, then backups with the restore drill and mail from piviki.fi are checked on production"). The spine uses passive voice, so the checks could land in `epic-initial-load` or nowhere.

Proposed fix, replacing "is checked on production before the real import" with:

> is checked on production by the production start story, before the real import

## Low

### L1. "bulk import" leftovers

Quotes:
- AD-17 Backups: "A restore is tested on the production host before the bulk import, and yearly."
- Deferred: "Before the restore test before the bulk import"

Fix: replace "bulk import" with "real import" in both places.

### L2. AD-8 Prevents and title still say "launch"

Quotes: "### AD-8: Sign-up gate, Google linking and the pre-launch period" and "and test records reaching launch."

Fix: change the title to "### AD-8: Sign-up gate, Google linking and the period before the soft open" and the Prevents clause to "and test records reaching real patrons at the soft open or the flag flip."

### L3. AD-17 Prevents still says "launch cutover"

Quote: "a launch cutover nobody owns"

Fix: change it to "a soft-open cutover nobody owns".

### L4. AD-19 table row order doesn't match the epic order

Quote: the `epic-soft-open` row sits between `epic-initial-load` and `epic-patron-site`.

Fix: move "| `epic-soft-open` | No collection; it switches `piviki.fi` and the sign-up gate |" below the `epic-patron-site` row.

### L5. The epic order after the soft open is stated only in the scope front matter

Quote (AD-17 Environments): "The soft open follows `epic-patron-site` and the real import."

Fix: append ", and precedes `epic-reservation-loop`" (subject to the H2 decision).

## Checked and clean

- No `beta.piviki.fi` host remains anywhere. The Hosts convention, Environments and the Seed diagram ("Access on admin") are updated.
- AD-7's origin lists and Cutover Google redirects agree that `piviki.fi` is added at the soft open.
- AD-15 and AD-18 need no change. "Production never runs on AWS" correctly reflects that the AWS production option was dropped.
- The Capability map's R21 row (AD-15, AD-17) is still accurate.
- The closed Open Question on scans on test was removed, and no other Open Question or Deferred entry is stale apart from L1.
- Structural Seed: only production is drawn, and test is a dotted edge, which is consistent.
