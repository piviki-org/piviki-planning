# Handoff to Sofia (bmad-agent-po): replan for environments and hosts

Written 2026-11-05 at the end of a build session on `epic-librarian-catalogue` story 1.1. The
founders approved the change below; the architecture spine is already updated and final. This
file is a work request for the tickets only. Read the sources it names; do not rely on its
summaries where a source exists.

## The change (approved by the founders, 2026-11-05)

- **The AWS EC2 instance is the test environment from its first bootstrap**: `test-admin.piviki.fi`,
  `test-api.piviki.fi`, `ssh-test.piviki.fi`, GitHub Environment `test`, OpenTofu `tofu/test`,
  Bitwarden collection `test`.
- **Production is one OVH VPS-1 at the production names**, brought up by a story of
  `epic-librarian-catalogue` placed before `epic-initial-load`, and it **starts empty**.
- **There is no move by backup restore.** Nothing written before the initial load is
  irreplaceable (the epic's own decision of 2026-11-03), so nothing moves from test.
- Stories before the production start are built and checked on **test**. What test cannot show
  (the `backup` service, its heartbeat, a restore, mail from `piviki.fi`) is checked on
  production before the bulk import.
- AD-8's `patronAccountsAllowed` is **retired**; the beta allowlist now gates patron creation on
  every path, and production's allowlist holds only founder-controlled addresses until the
  privacy notice and AD-18 agreements exist. `signupOpen` is never true on test.
- The AWS account stays on the **free plan** (AWS holds only recreatable test data); the founders
  record its end date and upgrade or move test before it runs out.

Why: cleaner separation. The AWS host's keys and secrets are test from the start, and no real
data is ever on a host whose rules allow it to be wiped or closed.

## Sources (authoritative)

- Spine: `_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md`,
  status final, updated 2026-11-05. Rules now in force: **AD-5** (seeding per environment),
  **AD-8** (sign-up gate without `patronAccountsAllowed`), **AD-17** (Host, AWS, Production start,
  Backups, Environments, Test environment, Per-environment configuration, Infrastructure as code),
  **AD-18**, **AD-21**; Hosts convention; Stack (TypeScript 7.0.2).
- Spine decision log: `architecture-piviki/.memlog.md`, every entry dated 2026-11-05.
- Spine reviews with ticket-line lists: `architecture-piviki/reviews/review-rubric-2026-11-05.md`,
  `review-adversary-2026-11-05.md`, `review-verify-2026-11-05.md`.

## Ticket text that is now stale (found 2026-11-05; re-check before editing)

Initiative `initiative-kirjasto-mvp.md`:
- l.118 Decision 2026-11-01 "Every epic deploys to production at a non-public address …":
  epics before the production start deploy to test.
- l.126 Decision 2026-11-03 "Production runs on AWS EC2 … then moves to OVH … by backup
  restore": superseded.

`tickets/tickets.toml` (epics):
- l.12 `epic-initial-load` description "production then moves to OVH and the AWS instance …".
- l.22 `after` needs "the move to OVH, for stories that create a patron".

`tickets/epic-librarian-catalogue/epic-librarian-catalogue.md`:
- l.50 Done-when "The platform runs in production …": still true, now on OVH from the
  production start.
- l.79 Decision "every story tests itself and is checked in production": now on test until the
  production start.
- l.81 Decision 2026-11-03 production on AWS then move: superseded.
- l.85 Decision "Stories 10 to 13 are checked on the AWS pre-launch environment; the move story
  deletes their check records": superseded (they are checked on test; check records on
  production are deleted before the import, spine AD-17).
- Owns platform baseline: add the production-start story.

`tickets/epic-librarian-catalogue/tickets/tickets.toml` (stories):
- Story 1 (l.6–11): title "… at admin.piviki.fi in production" → the test environment at
  `test-admin.piviki.fi`; description "paid-plan AWS account" (free plan now), production names
  and `tofu` per environment; verify line ("naming production", `admin.piviki.fi`,
  `api.piviki.fi`); `hitl` comment "paid-plan AWS account".
- Story 2 (l.19–20): description builds `patronAccountsAllowed` "false on the AWS host until the
  move" — retired; verify "runs policy:set in the production core container" and "creating a
  patron is refused while patronAccountsAllowed is false".
- Verify lines that say "on production": l.31 (story 3, also `PVK` → `TST` on test), l.53,
  l.88, l.99 (mail from an `@piviki.fi` address — production only, see spine AD-17 Production
  start; the test sender is `testmail.piviki.fi`), l.110, l.121 (`PVK`), l.143, l.154.
- New story: **production start on OVH**, before the stories `epic-initial-load` pins to (6, 7,
  8). Spine AD-17 "Production start" gives its content.

`tickets/epic-initial-load/epic-initial-load.md`:
- l.17, l.28, l.29, l.35, l.41 ("AD-17 The move"), l.45: the move to OVH and the AWS rebuild
  as test are gone; the epic no longer owns the move or the test environment.

`tickets/epic-patron-site/epic-patron-site.md`:
- l.37 "`patrons` and `patronAccountsAllowed` from epic-librarian-catalogue 1.2": the flag is
  retired.
- l.47 Decision 2026-11-03 production on AWS then move: superseded.

## For Sofia to decide

- Where the production-start story sits in `epic-librarian-catalogue`, its verify line and
  `hitl`; whether the email story (l.98–99) comes after it.
- How each story before it is verified on test (host names, `TST` prefix, test mail sender).
- Whether `epic-initial-load`'s dependency on epic 2 ("the move to OVH …") becomes a dependency
  on the production-start story.

## Not Sofia's

- **Story 1.1's plan file**
  (`tickets/epic-librarian-catalogue/tickets/story-a-librarian-signs-in-at-admin-piviki-fi-in-production-plan.md`)
  is re-planned through `bmad-build` after the ticket update; its Implementation Notes record the
  founder steps done so far (Bitwarden, GitHub, AWS account, the `t3.small` instance in
  `eu-north-1`, public IPv4 `13.53.218.99`, not yet bootstrapped).
- The spine: already updated; if a ticket change seems to need a spine change, raise it rather
  than edit the spine.

## Open items carried over

- Spine Open Question (2026-11-05): may real, unreviewed scans imported on test land in test's
  public media bucket, or does test's import use made-up scans or a private bucket? Founders,
  before the first import run on test.
- The AWS free plan's end date and the kind of credits: the founders are checking Billing.
