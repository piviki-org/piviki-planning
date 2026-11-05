# Adversarial review: owners moved (2026-11-05)

Scope: the change in the last memlog entry ("Owners moved"): the production start and the real import become the first two stories of `epic-soft-open`; `epic-initial-load` builds the import script and runs trial imports on test. The diff touches spine line 213 (AD-17 Production start) and lines 242-243 (AD-19 table).

Verdict: **Pass with 4 fixes.** No text left in the spine still gives the production start, the real import or the production restore check to `epic-librarian-catalogue` or `epic-initial-load`. But three duties the old owners held implicitly now have no owner: production's import key and service token (and their revocation), reviewing and publishing the real import, and what the real import waits on.

## 1. Sweep for leftover owners and contradictions

| Place | Text | Status |
|---|---|---|
| Frontmatter `scope` | lists all seven epics | OK |
| AD-7 | "The one-time import script uses a librarian account's API key and a Cloudflare Access service token, both revoked after the initial load. The test environment has its own key and service token." | **Gap**: nobody owns creating production's key and token or revoking them; unclear whether test's are revoked too (finding F1) |
| AD-9 | "The initial load, the real import into production, runs once, single-threaded; trial imports on test may run again..." | OK, matches the new split |
| AD-9 | "`epic-librarian-catalogue` owns finding a copy by barcode, the label view, the review view with merging, API keys on librarian accounts, and the Cloudflare Access service token the import script uses." | **Contradiction**: production's Access exists only from the production start ("OpenTofu gives production its own Tunnel and Access"), which is now `epic-soft-open` story 1, so an `epic-librarian-catalogue` story cannot create production's service token (F1) |
| AD-9 | "\"Import\" in R13 means publishing: a librarian reviews every record in the production review view and publishes it." | **Gap**: no epic owns this now; the AD-19 row for `epic-soft-open` says only "runs the real import" (F2) |
| AD-16 | strip/crop, review checklist | OK, no owner named |
| AD-17 Production start | "Only `epic-soft-open`'s later stories wait on the production start, and it waits only on `epic-librarian-catalogue`'s infrastructure stories, not on `epic-patron-site`" | **Ambiguous**: "it" could mean the production start or the epic; "infrastructure stories" is not defined; the real import also needs `epic-initial-load`'s script, which goes unstated (F3) |
| AD-17 Backups | "from before the first data that cannot be recreated, the initial load ... A restore is tested on the production host before the real import, and yearly." | OK with Production start ("checked on production by the production start story before the real import"); the yearly drill has no owner, as before this change (not caused by it) |
| AD-17 Environments | "production on OVH from the production start ... The soft open follows `epic-patron-site` and the real import." | OK |
| AD-17 Cutover | `epic-soft-open` owns | OK |
| AD-19 table | new rows | OK, but see F2 |
| Capability map | R13 → `piviki-core/scripts/import`, review view; no epics named | OK |
| Structural Seed | no epics named | OK |
| Deferred | "OVH data centre ... When ordering, before the production start"; "Barcode sequence after a restore ... Before the restore test before the real import" | **Gap**: the decide-by date now falls inside `epic-soft-open` story 1, but the fix itself (advancing the sequence before core starts) is core code that must already be in the image that story deploys, so it is built before the production start, in some epic that is not named (F4) |
| Assumptions | "Payload runs on two hostnames ... First `epic-librarian-catalogue` story" | OK (proved on test) |

Outside the spine (not in scope, flagged for Sofia's replan): `tickets/epic-initial-load/epic-initial-load.md` still says "Every book in the initial collection is in the production catalogue with its images", "Production has moved to OVH by spine AD-17's seven steps", and "Uses from epic-librarian-catalogue: ... the tested restore (1.7)". That text is stale twice over (the AWS-to-OVH move was dropped earlier), and the memlog says Sofia applies the replan.

## 2. Two units that each obey every AD yet clash

### Clash A: the service token and key for the real import

- Unit 1, an `epic-librarian-catalogue` story "Import credentials": following AD-9 ("owns ... API keys on librarian accounts, and the Cloudflare Access service token the import script uses"), it creates a service token in OpenTofu and an API key on a librarian, on test (AD-17: "every other story ... is built and checked on test"). Done.
- Unit 2, `epic-soft-open` story 1 "Production start": following AD-17, it creates production's Tunnel and Access and seeds the first librarian. Nothing in its rule mentions an import key or a service token, so it creates neither. Done.
- `epic-soft-open` story 2 "Real import" then has no production key or token. It creates them itself (no AD gives it that duty), or someone adds a production token to `epic-librarian-catalogue`'s scope before production's Access exists. After the run, AD-7's "revoked after the initial load" belongs to nobody, so the token can outlive the import with write access to production's generated API.

### Clash B: epic-initial-load "done" vs import means publishing

- Unit 1, `epic-initial-load`'s last story "Trial import": it runs the script on test, and a librarian reviews and publishes in test's review view. It obeys AD-9 (trial imports on test) and AD-17 (built and checked on test). The epic counts as done.
- Unit 2, `epic-soft-open` story 2 "Real import": following the AD-19 row ("runs the real import with a librarian account") and AD-9 ("The import script only creates records; it never updates, publishes or deletes"), it ends when the script exits with every scan processed and every record still a draft.
- AD-9's "a librarian reviews every record in the production review view and publishes it" now sits between the two, in neither unit. The cutover ("The soft open follows `epic-patron-site` and the real import") can then open `piviki.fi` on a catalogue of drafts, so `/api/v1` shows no titles. R13 is met in neither epic.

## 3. Fixes (exact text)

### F1: owner of production's import credentials and their revocation (AD-7, AD-9)

AD-7, replace:
> The one-time import script uses a librarian account's API key and a Cloudflare Access service token, both revoked after the initial load. The test environment has its own key and service token.

with:
> The one-time import script uses a librarian account's API key and a Cloudflare Access service token. Production's pair is created by `epic-soft-open`'s real import story, after the production start, and revoked by that story once its run is complete and before the story closes. The test environment has its own pair, kept for trial imports.

AD-9, replace:
> `epic-librarian-catalogue` owns finding a copy by barcode, the label view, the review view with merging, API keys on librarian accounts, and the Cloudflare Access service token the import script uses.

with:
> `epic-librarian-catalogue` owns finding a copy by barcode, the label view, the review view with merging, API keys on librarian accounts, and the Cloudflare Access service-token setup the import script uses, with test's pair; production's pair belongs to the real import (AD-7).

### F2: owner of review and publishing (AD-9, AD-19)

AD-9, replace:
> "Import" in R13 means publishing: a librarian reviews every record in the production review view and publishes it.

with:
> "Import" in R13 means publishing: a librarian reviews every record in the production review view and publishes it, within `epic-soft-open`'s real import story, which ends only when no draft from the run is left. `epic-initial-load` is done when a full trial import on test has been reviewed and published in test's review view.

AD-19 table, replace:
> | `epic-soft-open` | No collection; it starts production, runs the real import with a librarian account, and switches `piviki.fi` and the sign-up gate |

with:
> | `epic-soft-open` | No collection; it starts production, runs the real import with a librarian account and publishes it after review, and switches `piviki.fi` and the sign-up gate |

### F3: what the production start and the real import wait on (AD-17 Production start)

Replace:
> Only `epic-soft-open`'s later stories wait on the production start, and it waits only on `epic-librarian-catalogue`'s infrastructure stories, not on `epic-patron-site`;

with:
> Only `epic-soft-open`'s later stories wait on the production start. The production start waits only on the `epic-librarian-catalogue` stories that build the bootstrap, the Tunnel and Access configuration, the deploy workflow and its checks, the `backup` service and restore runbook, and the mail adapter, not on `epic-patron-site`; the real import also waits on `epic-initial-load` and on `epic-librarian-catalogue`'s review view and API keys;

### F4: barcode-sequence fix has no builder before the production start (Deferred)

Replace the row:
> | Barcode sequence after a restore | `pg_dump` restores the sequence, but barcodes issued after the last dump could be reissued; candidate fix is advancing the sequence by a margin before core starts | Before the restore test before the real import |

with:
> | Barcode sequence after a restore | `pg_dump` restores the sequence, but barcodes issued after the last dump could be reissued; candidate fix is advancing the sequence by a margin before core starts | In `epic-librarian-catalogue`'s `backup` and restore runbook story, so the image the production start deploys has it |

## 4. Named-tech facts

The diff touches no product, version, price or vendor fact: the Stack table and the OVH, AWS, Bitwarden and OpenTofu claims are unchanged. Nothing to re-verify.
