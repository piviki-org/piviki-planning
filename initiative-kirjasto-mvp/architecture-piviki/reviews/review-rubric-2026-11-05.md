# Rubric review: architecture spine update 2026-11-05

- Spine: `architecture-piviki.md` (updated 2026-11-05)
- Update under review: the last five `.memlog.md` entries (2026-11-05): AWS EC2 is test from its first bootstrap; production starts empty on OVH VPS-1 before `epic-initial-load`, with no restore-move; AWS paid-plan rule dropped; AD-8 `patronAccountsAllowed` retired; TypeScript 7.0.2.
- Initiative: `initiative-kirjasto-mvp.md`, Requirements R1-R25, Constraints, Done when, Notes
- Method: inspection only, plus text searches of the spine and ticket tree

## Verdict

**Pass, with two medium and several low fixes.** The spine itself is consistent with the update. No text in it still describes a move from AWS to OVH, real data on AWS, AWS as production or pre-launch production, or `patronAccountsAllowed`. The Structural Seed diagram, Deferred rows and AD-21 are clean. The main gaps: retiring `patronAccountsAllowed` leaves the pre-launch patron rules short in two places (F1, F2), and the initiative ticket and three epic tickets still describe the old move plan (F3). F3 is outside the spine but is now its biggest consistency problem.

## Stale-wording sweep

I searched the spine for move, migrate, restore, wipe, AWS, EC2, credits, paid, free plan, pre-launch, first real data, irreplaceable and `patronAccountsAllowed`, and read each hit.

| Location | Text | Result |
|---|---|---|
| AD-8 title | "the pre-launch period" | Fine. It means before `signupOpen`, not pre-launch production |
| AD-17 Host | "Production never runs on AWS"; test "from its first bootstrap" | Correct |
| AD-17 AWS | free plan acceptable; "No real patron's data is ever on AWS" | Correct (see F2, F4) |
| AD-17 Production start | "nothing moves from test" | Correct. The only "move" left says nothing moves |
| AD-17 Ingress | "A scan from outside confirms this before the first real data" | Matches Production start, which runs the scan before the first librarian is seeded |
| AD-17 Backups | "from before the first data that cannot be recreated, the initial load"; restore test "on the production host before the bulk import" | Correct for an empty-start production |
| AD-17 Environments | "production on OVH from the production start" | Correct |
| AD-18 | "AWS, which hosts only the test environment" | Correct |
| AD-21 | "A restore and a rebuild start that tag's checkout"; "Wiping the test host" | Correct. Move and move-freeze wording is gone |
| Structural Seed | `ovh -.no data or secrets to.-> test[Test environment on AWS EC2 ...]` | Correct. No move arrow, and the production subgraph is OVH only |
| Deferred, "Test environment after the credits" | "delete it or move it" | Fine. It means moving test elsewhere, not production (but see F4) |
| Deferred, "OVH data centre" | "before the production start" | Correct |
| Deferred, "Barcode sequence after a restore" | "Before the restore test before the bulk import" | Correct |
| AD-6 policy list, AD-19 table | no `patronAccountsAllowed` | Correct |
| Stack | TypeScript 7.0.2 | Matches the memlog (see F7) |

## Findings

### F1 [Medium] The rules don't say who can hold a production account before launch

AD-8 now limits only test's allowlist: "In the test environment, the beta allowlist holds only addresses the founders control". It says nothing about production's allowlist while `signupOpen` is false. Before this update, `patronAccountsAllowed` was a hard switch on the pre-launch production host. With it gone, any address a founder adds to production's allowlist can create a real patron account on OVH. That can happen before the privacy notice and usage policy are published and before the AD-18 agreements are on file. Today those conditions gate only `signupOpen`.

This conflicts with the initiative Note of 2026-11-01: "sign-up stays closed to real patrons until epic-launch". It also weakens AD-8's own Prevents line ("any path around the adult tick and policy acceptance"). The `epic-launch` reset deletes these accounts, so this is a cleanup measure, not a prevention measure.

Fix: add one sentence to AD-8. Either "Before `signupOpen`, production's beta allowlist also holds only addresses the founders control", or name the condition under which real beta patrons are allowed: notice published and agreements on file. The current Prevents line then holds.

### F2 [Medium] "No real patron's data is ever on AWS" can be broken through `test-api`

`test-api.piviki.fi` is not behind Access, because the Worker reads it. AD-8's allowlist gate applies only "while `signupOpen` is false". The test bullet says the mail adapter refuses unlisted recipients "whatever `signupOpen` says", which implies `signupOpen` may be set true on test, for example to rehearse the cutover. When it is true, anyone can POST a sign-up to `test-api.piviki.fi/api/v1/auth/*`. Their name and email are then stored on AWS. They can never verify, but the data is already there.

The retired flag didn't cover this either, since it was true on test. Still, the update now relies on "No real patron's data is ever on AWS" as the reason the free plan is acceptable, so the rule should be enforceable.

Fix, either:

- "`signupOpen` is always false on test, so only allowlisted founder addresses can sign up there", or
- a test-only refusal of sign-up for any email not on the test recipient allowlist.

### F3 [Medium, outside the spine] The initiative and epic tickets still describe the old plan

The spine is the initiative's companion, so these now contradict it:

- `initiative-kirjasto-mvp.md` Note 2026-11-03: "Production runs on AWS EC2 ... until right after the initial load, then moves to OVH VPS-1 by backup restore; the AWS instance then becomes the test environment". The same note is repeated in `tickets/epic-librarian-catalogue/epic-librarian-catalogue.md`, `tickets/epic-initial-load/epic-initial-load.md` and `tickets/epic-patron-site/epic-patron-site.md`.
- `epic-librarian-catalogue.md` Note: "Stories 10 to 13 are checked on the AWS pre-launch environment; the move story deletes their check records before the final dump."
- `epic-initial-load.md`: "Production has moved to OVH by spine AD-17's seven steps", "Owns the move to OVH and the test environment". Its References point at "AD-17 The move", which no longer exists.
- `epic-patron-site.md`: "Extends by migration ... `patrons` and `patronAccountsAllowed` from epic-librarian-catalogue 1.2".
- `tickets/tickets.toml`: the `epic-initial-load` description says "production then moves to OVH". The `epic-patron-site` dependency needs "the move to OVH, for stories that create a patron". The spine now places production start inside `epic-librarian-catalogue`.
- Story 1.1 is titled "A librarian signs in at admin.piviki.fi in production", but under the new plan it runs on test.
- The initiative Note of 2026-11-01, "Every epic deploys to production at a non-public address", is no longer true for the `epic-librarian-catalogue` stories before production start. Per AD-17, those are built and checked on test only.

Fix: product owner replan (Sofia). Supersede the 2026-11-03 Note, move ownership of production start from `epic-initial-load` to `epic-librarian-catalogue`, remove `patronAccountsAllowed` from the stories, and restate the 2026-11-01 deploy Note. This review changed none of these files.

### F4 [Low] The free-plan end date is not tracked as a trigger

The memlog says the paid-plan rule existed because "free-plan accounts close with their credits". AWS's free plan closes the account when the credits run out or when its fixed period ends, whichever comes first. The Deferred row "Test environment after the credits" triggers only "When the credits run low". The test bullet says "Founders check the AWS bill monthly", but a free-plan account has no bill to check, only credits and an end date.

The cost of missing the date is low, because test holds only data that can be recreated. But production deploys don't depend on test, and patching rules ("test is patched in the same window or stopped") assume someone knows when test will disappear.

Fix: change the Deferred trigger to "when the credits run low or the free plan's end date nears". Reword the test bullet to "check the AWS credits, plan end date and bill monthly".

### F5 [Low] The AD-17 Prevents line doesn't name the risks this update removed

The memlog decision lists what the environment change prevents: "keys and secrets of a temporary host classified as production, real data on a host whose rules allow wiping, and a cross-provider restore-move". AD-17's Prevents line still ends at "production data or secrets reaching the test environment". The Rule does cover those risks ("Production never runs on AWS", "No real patron's data is ever on AWS", fresh production secrets at production start), but the Prevents line doesn't name them.

Fix: add "or real data on a host that may be closed or wiped" to the Prevents line, so a later edit that loosens the AWS rules shows what it reopens.

AD-8's Prevents line ("any path around the adult tick and policy acceptance, account takeover through Google, and test records reaching launch") still matches its Rule after the retirement, apart from the F1 gap. Every clause has a Rule bullet: the gate, the Google linking, and the allowlist with reset.

### F6 [Low] The spine doesn't say who brings up test

Production start has an owner: "a story of `epic-librarian-catalogue`, before `epic-initial-load`". The test environment is now said to exist "from its first bootstrap", and stories before production start are "built and checked on test". But no AD names who creates it. The old owner, `epic-initial-load`'s move story, no longer applies. The memlog implies story 1.1. Two epics could each assume the other bootstraps test, or that it already exists.

Fix: one clause in AD-17 Test environment, for example "brought up by `epic-librarian-catalogue` story 1, before any other story". This mirrors how the Bitwarden setup is assigned.

### F7 [Low] TypeScript 7.0.2 is verified for `piviki-core` only

The memlog verifies TypeScript 7.0.2 on `piviki-core` (typecheck, tests, `next build`, Compose). The Stack table is shared by both repositories, but `piviki-patron` (Next.js export, Mantine, `openapi.json` type generation) has no recorded check. TypeScript 7 is the native compiler, and tools that use the older compiler API, such as some lint and type-generation tools, can break.

Fix: either note "verified on core; patron adopts at its first story", or add a row to Assumptions to Prove: "patron's toolchain, including OpenAPI type generation, runs on TypeScript 7 | first `epic-patron-site` story | pin patron to 6.x".

### F8 [Low] Some AD-17 wording is loose

- Production start requires "the hardening checks" to pass, but no list by that name exists. AD-21 lists deploy checks, and AD-17 Hardening lists properties, not checks. A unit can't tell which checks block production start. Suggest "the AD-17 Hardening and AD-21 deploy checks".
- The Deferred row "EC2 instance size and region", decide by "When creating the instance": if story 1.1 has already bootstrapped test, the instance exists and this row is past its decide-by date. Record the chosen size and region, or move the row out of Deferred.

## Checklist walk

| Check | Result |
|---|---|
| Fixes the real divergence points for the level below, misses none | Mostly. Production start, test-first build, fresh production secrets and the empty start are fixed. Who bootstraps test is missing (F6) |
| Every AD Rule enforceable and prevents its divergence; AD-8 and AD-17 Prevents match their Rules | AD-8 matches, but its pre-launch production gap is open (F1). AD-17 Rule is sound; Prevents could name the new risks (F5). "No real patron's data on AWS" needs a mechanism (F2) |
| Nothing under Deferred lets two units diverge | Holds. All Deferred rows are single-owner or no-effect. Trigger wording issues in F4 and F8 |
| Covers every hosting, environment, data location, backup and GDPR processor requirement | R19 deletion and retention: AD-14 and AD-17 erasure records. Constraint "written data processing agreement for each provider": AD-18 before `signupOpen`; AWS, GitHub and Bitwarden hold no patron data. Constraint "breach notified within 72 hours": Breach readiness. Note 2026-11-01 "closed to real patrons until launch": weakened (F1). Data location: OVH EU data centre, R2 EU private bucket, Bitwarden EU, test in an EU AWS region |
| Deployment and environments, infra and provider strategy, operations decided, deferred or open | Decided: environments, hosts, deploy, IaC, secrets, patching, backups, recovery, breach. Deferred: OVH data centre, EC2 size, test after credits. Open: patron deploy home |
| No leftover move, AWS-production, real-data-on-AWS or `patronAccountsAllowed` text | Clean in the spine. Stale in the tickets (F3) |
| Structural Seed and Deferred/Open Questions wording | Clean |

## Not checked

- Whether AWS's current free-plan terms (period length, what closure deletes) are as stated in F4. This review did no web lookup.
- Whether the downstream tickets are otherwise consistent. They were only searched for stale terms.
