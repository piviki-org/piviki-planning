# Review log: walkthrough-architecture-piviki

Target: ../architecture-piviki/architecture-piviki.md (whole spine)

<!--
Append-only record of outcomes/decisions.
Main purpose: persisted context for resuming or analyzing
the review process. You're writing this log for consumption
by another LLM session.

Record decisions and their reasons, user constraints, unresolved
findings, and references needed to recover evidence. Link to large
results rather than copying them here.
This is NOT a transcript or internal deliberation.
The review narrative (the human-facing file) owns review blocks
and their statuses.

Log entry:
Use format below. Omit empty fields. In Result, give each finding's
disposition: fixed, accepted as-is, deferred, or open. When a
disposition changes, append a new entry; do not edit earlier ones.
Record the actual session ID supplied by the environment and an ISO
8601 timestamp with local timezone for each entry. If the session ID
is unavailable, write "unavailable" rather than inventing one.

Token budget: up to 150 tokens per entry.
The smaller the better, but do not sacrifice precision.
-->

## 1 — Setup — narrative created

Session: unavailable · Timestamp: 2026-11-03T11:10:50+02:00

- Action: created narrative with 11 blocks
- Result: block 1 current

## 2 — Block 1 Intent — reviewer correction

Session: unavailable · Timestamp: 2026-11-03T11:13:45+02:00

- Action: reviewer said not to present 300 books as the whole collection; it is the initial collection from the founders.
- Result: narrative block 1 annotated; spine has no '300' mention (no change); constraint for all later blocks: describe 300 as the initial collection only.
- Open: initiative ticket Description, epic-initial-load title/description, tickets.toml epic 2 title/description and milestone lines say "the 300 books"; ticket wording change belongs to Sofia (bmad-agent-po).

## 3 — Block 1 Intent — accepted

Session: unavailable · Timestamp: 2026-11-03T11:16:43+02:00

- Result: user accepted; AD-9 stays unchanged (import script runs once for the initial collection). Direction recorded: next phase will extend the librarian tool to read book covers and import data from other sources or an LLM; later additions go through the librarian tool, not the script.
- Open: none for the spine; note this future feature is not in the initiative ticket.

## 4 — Block 2 Broad strokes — accepted

Session: unavailable · Timestamp: 2026-11-03T11:19:08+02:00

- Result: user accepted, no changes. User constraint: commit only after the review is done (_bmad-output has 3 untracked folders: architecture-piviki, research-marc21-and-ils-compatibility, walkthrough-architecture-piviki).

## 5 — Block 3 Core state and circulation — accepted

Session: unavailable · Timestamp: 2026-11-03T11:21:29+02:00

- Result: user accepted AD-1..AD-4 with no changes and no moves taken.

## 6 — Block 4 APIs and delivery — second opinion on AD-5

Session: unavailable · Timestamp: 2026-11-03T11:25:55+02:00

- Action: fresh subagent reviewed why two API surfaces.
- Result: verdict keep. Reasons: Payload exposes all collections, auth endpoints, GraphQL and jobs by default; Access on admin host hides them; patron writes need hand-written /api/v1 anyway (AD-2 transactions, AD-8 gate, stable codes); openapi contract shields patron site from Payload upgrades. Cost mainly the hand-written v1 auth (1–2 weeks). Suggested: (1) v1 auth as thin wrappers over Payload Local API (already an Assumption to Prove); (2) drop AD-7 per-hostname cookie names, since host-only cookies already separate hosts.
- Open: user to decide on (2).

## 7 — Block 4 — AD-7 cookie-name question deferred to block 5

Session: unavailable · Timestamp: 2026-11-03T11:29:13+02:00

- Result: user asked to mark the rule with details and decide in block 5; spine unchanged. User asked what the v1 auth assumption was; answered from Assumptions to Prove row (L369).
- Evidence: ../architecture-piviki/architecture-piviki.md#L117
- Open: AD-7 per-hostname cookie names: keep or drop, decide in block 5.

## 8 — Block 4 APIs and delivery — accepted

Session: unavailable · Timestamp: 2026-11-03T11:30:52+02:00

- Result: user accepted AD-5, AD-15, AD-16 unchanged after second opinion; AD-7 cookie-name question moves to block 5.

## 9 — Block 5 — AD-7 edited

Session: unavailable · Timestamp: 2026-11-03T11:34:33+02:00

- Action: removed the sentence "Cookie names are per hostname, so a librarian's cookie is never sent to api.piviki.fi." from AD-7 (../architecture-piviki/architecture-piviki.md#L117).
- Result: fixed per user decision; architecture memlog updated.

## 10 — Block 5 Tenancy, identities, sign-up — accepted

Session: unavailable · Timestamp: 2026-11-03T11:35:49+02:00

- Result: user accepted AD-6, AD-7 (as amended), AD-8.

## 11 — Block 6 — AD-20 edited

Session: unavailable · Timestamp: 2026-11-03T12:46:32+02:00

- Action: removed the `sourceRecordId` row from the AD-20 field table (../architecture-piviki/architecture-piviki.md#L240).
- Result: fixed per user decision ("drop source record id for now"); architecture memlog updated.
- Open: re-add by migration when a copy-cataloguing source such as Melinda ISBN lookup is built (research recommendation 6).

## 12 — Block 6 — Melinda excluded

Session: unavailable · Timestamp: 2026-11-03T12:48:04+02:00

- Action: Deferred row "ISBN lookup source" (../architecture-piviki/architecture-piviki.md#L390) changed from "candidate in research §4" to "Melinda is not used as a record source".
- Result: per user, records will not be imported from Melinda specifically; supersedes the Open note in entry 11 — sourceRecordId would return only if some other copy-cataloguing source is adopted.

## 13 — Block 6 — Melinda exclusion reverted

Session: unavailable · Timestamp: 2026-11-03T12:48:35+02:00

- Action: restored Deferred row "ISBN lookup source" (../architecture-piviki/architecture-piviki.md#L390) to "candidate in research §4".
- Result: user said "revert that"; entry 12's decision is withdrawn; entry 11's Open note stands again.
- Open: re-add sourceRecordId by migration when a copy-cataloguing source such as Melinda ISBN lookup is built (as entry 11).

## 14 — Block 6 Catalogue — accepted

Session: unavailable · Timestamp: 2026-11-03T12:48:59+02:00

- Result: user accepted AD-9, AD-10, AD-20 (sourceRecordId removed; ISBN-lookup Deferred row unchanged after revert).

## 15 — Block 7 Language, email, jobs — accepted

Session: unavailable · Timestamp: 2026-11-03T12:50:54+02:00

- Result: user accepted AD-11, AD-12, AD-13 unchanged.

## 16 — Block 8 Personal data and processors — accepted

Session: unavailable · Timestamp: 2026-11-03T12:55:41+02:00

- Result: user accepted AD-14, AD-18 unchanged; the next-phase note (an LLM called from core for cover reading must also hold no patron data under AD-18) was raised but not taken up.

## 17 — Block 9 — scan-clients removal requested

Session: unavailable · Timestamp: 2026-11-03T12:59:17+02:00

- Result: user decided there will be no scan-client identity; the one-time import runs with librarian rights. Affects AD-5 (admin host callers; Access service token), AD-7 (identities), AD-9 (draft-only, duplicate_title refusal, copy_exists guard keyed on scan-clients), AD-19 (epic-initial-load creates scan-clients).
- Open: user to choose A (dedicated librarian account + ?import=1 flag keeps import rules in core), B (script enforces rules), or C (drop import rules, rely on review).

## 18 — Block 9 — import identity: option A rejected

Session: unavailable · Timestamp: 2026-11-03T13:02:13+02:00

- Result: user rejected the import flag. Decision: every title is created as draft unless the API payload explicitly sets it published (applies to all writers).
- Open: proposed to user — duplicate_title refused for every create unless payload sets an override (e.g. allowDuplicate: true); drop the copy_exists guard; remove scan-clients from AD-5, AD-7, AD-19; script runs as a librarian account plus an Access service token. Awaiting confirmation.

## 19 — Block 9 — spine edited (scan-clients removed)

Session: unavailable · Timestamp: 2026-11-03T13:06:26+02:00

- Action: edited AD-1, AD-5, AD-7 (title now "Patrons and librarians are separate identities in core"), AD-9, AD-19 (epic-initial-load creates no collection), AD-20 Prevents, Collections convention.
- Result: fixed per user — no import flag; draft default for all writers; duplicate_title for all creates unless allowDuplicate: true (for books without ISBN; librarian tool offers it, script never sends it); copy_exists guard, createdBy and librarianEditedAt dropped; script only creates; librarian API key + Access service token revoked after load. Lint clean (4 low false positives).

## 20 — Block 9 — restore sequence rule parked

Session: unavailable · Timestamp: 2026-11-03T13:12:56+02:00

- Action: removed AD-9 clause "a restore sets the sequence past the highest stored barcode"; added Deferred row "Barcode sequence after a restore" (candidate: advance sequence by a margin before core starts; decide before relying on restores in production).
- Result: user parked it; earlier claim corrected — pg_dump restores sequence values, the real risk is barcodes issued after the last dump being reissued. No AD-17 change.
- Open: Deferred row "Barcode sequence after a restore", decide before relying on restores in production.

## 21 — Block 9 — second opinion on AD-9

Session: unavailable · Timestamp: 2026-11-03T13:15:05+02:00

- Action: fresh subagent reviewed AD-9 as amended.
- Result: sound with small fixes. Proposed clear fixes: define matchKey "otherwise" (new title has no non-set ISBN) and use one catalogue-wide lock; AD-2 allow copy create on generated API with core-set state, add merge to hook-routed list; script order media → title → copy, orphan media flagged; AD-19 assigns review view, merge, librarian useAPIKey and Access service token to epic-librarian-catalogue; fix structure diagram arrow, AD-5/AD-7 wording, note generated-API error shape differs from /api/v1.
- Open for user: A re-run safety (script-local processed-scans file vs drop re-run from Prevents); B duplicate check also on publish; C trim implementation detail (keep key recipe).

## 22 — Block 9 — AD-9 second-opinion fixes applied

Session: unavailable · Timestamp: 2026-11-03T13:22:09+02:00

- Action: edited the Invariants diagram (script arrow now through hooks), AD-2 (copy create allowed with core-set state; merging routed through circulation), AD-5 (script uses a librarian API key), AD-9 (matchKey text key when the new title has no non-set ISBN; catalogue-wide serialisation; 409 data.code duplicate_title; images → title → copy with orphan images flagged; epic-librarian-catalogue owns lookup, label view, review/merge, librarian API keys, Access service token; generated-API error shape noted). Option C applied: trimmed sequence name, hook timing, APIError constructor, advisory-lock mechanism, label ordering/fields; key recipe kept.
- Result: fixed.
- Open: A re-run safety, B duplicate check on publish.

## 23 — Block 9 — re-run safety (A) applied

Session: unavailable · Timestamp: 2026-11-03T13:25:50+02:00

- Action: AD-9 now says the import script keeps a local file of processed scans and skips them on a re-run; core stores nothing about scans.
- Result: fixed per user ("a. yes").
- Open: B — user asked why check duplicates on publish; explained (review edits such as correcting a misread title or adding an ISBN can create a duplicate that the create-time check never sees); awaiting decision.

## 24 — Block 9 — duplicate check on publish (B) applied; block accepted

Session: unavailable · Timestamp: 2026-11-03T13:31:13+02:00

- Action: AD-9 duplicate refusal now covers a title create or a publish of a draft.
- Result: user accepted ("b, understood. good to go"); block 9 done, changed during review (scan-clients removal, AD-2/AD-5/AD-9 edits, restore-sequence rule deferred, re-run file, publish check).

## 25 — Block 10 — piviki-scan repo removed

Session: unavailable · Timestamp: 2026-11-03T13:35:11+02:00

- Action: import script moved to piviki-core/scripts/import, run on a founder's machine over the generated API (user chose A over running on the server via Local API). Edited Design Paradigm row, Repositories convention, both diagrams, AD-16, AD-18, Capability map R13 row, Deferred vision-model row.
- Result: fixed; lint clean.

## 26 — Block 10 Conventions and stack — accepted

Session: unavailable · Timestamp: 2026-11-03T13:35:53+02:00

- Action: Ids-and-dates convention now says dates shown to people (screens, emails, labels) are dd-mm-yyyy in Europe/Helsinki; APIs and storage keep ISO 8601 and YYYY-MM-DD.
- Result: user accepted ("ok, done"); block 10 done, changed during review (piviki-scan removed, display date format).

## 27 — Block 11 — inactivity limit decided

Session: unavailable · Timestamp: 2026-11-03T13:44:02+02:00

- Action: added to AD-14 an inactivity rule (accounts with no sign-in, reservation or loan event for the policy inactivity window, starting at 1 year, deleted via modules/accounts after a 30-day warning email; accounts holding a book or open reservation exempt); added "inactivity window" to AD-6's tenant-policy list; narrowed the Open Questions row to proof of acceptance after deletion and backups with re-deletion.
- Result: fixed per user ("inactivity limit of 1 year").
- Open: the 30-day warning email was the lead's addition, pending user confirmation.

## 28 — Block 11 — inactivity warning email removed

Session: unavailable · Timestamp: 2026-11-03T13:45:57+02:00

- Action: AD-14 inactivity rule now says "without a warning email".
- Result: fixed per user; closes entry 27's pending item.

## 29 — Wrap-up — review complete

Session: unavailable · Timestamp: 2026-11-03T13:47:18+02:00

- Result: user accepted block 11 and asked to wrap up; spine re-finalized (architecture memlog event logged; lint clean).
- Open: commit of _bmad-output (awaiting user go-ahead); ticket wording about "the 300 books" for Sofia; ask Sofia to refresh the spine as a companion and slice epics.
