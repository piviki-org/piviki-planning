# Adversarial review: Piviki MVP architecture spine

- **Reviewed:** `architecture-piviki.md` (status draft, updated 2026-11-02)
- **Units:** E1 `epic-librarian-catalogue`, E2 `epic-initial-load`, E3 `epic-patron-site`, E4 `epic-reservation-loop`, E5 `epic-loan-care`, E6 `epic-launch`
- **Method:** for each finding, build two units that obey every AD to the letter, then show where they fail to fit together. Anything one later story can settle without another epic noticing was dropped (see the end).

## Verdict

**Not ready to build from. Close the holes first.** The system-level structure is sound: one core, one circulation module, an outbox, derived time states. The holes are in the shared data underneath it. The spine names the entities and rules but leaves open what a queue place is, what ends a loan, what "available" means, and which epic creates which collection. Four of the six epics write or read those shapes, and the epic order forces E3 and E1 to commit to them before E4 exists.

Findings 1 to 4 will cause rework or data errors across epics and must be fixed before E1 or E3 stories are written. Findings 5 to 8 should go in the same edit, because each is a few lines. Findings 9 and 10 are small.

| # | Finding | Units | Severity | Closes with |
|---|---|---|---|---|
| 1 | Queue place and reservation are two owners of one entity | E4, E5, E6 | Critical | AD-3, Conventions, ER |
| 2 | "Available" has no definition or contract, and E3 ships it before E4 | E1, E3, E4, E6 | Critical | AD-3, new AD-19 |
| 3 | Catalogue writes change circulation outcomes outside the module | E1, E2, E4, E6 | Critical | AD-2, AD-9 |
| 4 | A loan has no shared end state, so lost and retained records diverge | E4, E5, E6 | Critical | AD-4, AD-14, Conventions, ER |
| 5 | Title lock cannot serialise patron-wide operations | E4, E5, E6 | High | AD-2, AD-14 |
| 6 | Outbox row shape and deleted-patron behaviour are unspecified | E4, E5, E6 | High | AD-12, AD-14, ER |
| 7 | Title identity and draft scope differ between manual entry and scans | E1, E2, E3, E4 | High | AD-9 |
| 8 | `signupOpen` blocks the epics that must prove sign-in, and the gate version has no owner | E3, E4, E6 | High | AD-8, AD-6 |
| 9 | Donor link and consent: two homes, and no epic owns `patrons` first | E1, E3, E6 | Medium | AD-14, AD-19 |
| 10 | Policy change and first-email ownership conflict with the requirements and tickets | E1, E3, E4, E6 | Medium | AD-6, AD-12, Deferred |

---

## 1. Queue place and reservation are two owners of one entity (Critical)

**The opening in the spine.** Conventions list both `reservations` and `queue-places`, but only reservations have states. AD-3 says "a reservation or queue place ... gets a copy only when marked ready", while the ER diagram gives only reservations a copy link. Queue order is "creation time, then id" with no statement of how that spans two tables. The Deferred table calls the move from return to ready "internal to `modules/circulation`".

**Unit A (E4), compliant.** Joining a queue inserts a `queue-places` row. On return, the head row is deleted and a new `reservations` row is created in `reserved`. Leaving a queue deletes the row.

**Unit B (E5), compliant.** Renewal is refused when `queue-places` has a row for the title, as R6 says.

**Unit C (E6), compliant.** R23 sets queue places to `cancelled` and emails those patrons, with the place as the outbox subject.

**How they collide.**

- E6 has no `cancelled` state on queue places, and E4 has no rows left to refer to once a patron has left or been promoted.
- A different compliant E4 marks the queue place itself ready, which AD-3 literally allows, and keeps it. Then E5 counts it as "others queueing" and refuses renewal while nobody is waiting, and R14's state flow has no record to run on.
- A promoted patron gets a new creation time, so "creation time, then id" no longer reflects first come, first served.

**Exact change: one entity.**

- **AD-3, replace the second and third sentences with:** "A queue place is a reservation in state `queued`; there is no separate collection. Every reservation references a title and gets a copy only on the move to `ready`. Queue order is the reservation's creation time, then id, and never changes on promotion."
- **Conventions, Collections:** remove `queue-places`.
- **Conventions, Reservation states:** add `queued` as the first state. Leaving a queue is `cancelled`, never a delete.
- **ER:** remove `QUEUE_PLACES` and its two relations.
- **AD-2:** remove "queue places" as a separate noun, or define it as above.
- **Deferred:** the row on returned books stays. With one table it really is internal.

E5's check then becomes one query, "a `queued` reservation exists for this title". E6's cancellation is the same operation as R24's.

## 2. "Available" has no definition or contract, and E3 ships it before E4 (Critical)

**The opening in the spine.** AD-3 says availability is computed "from copies, active loans and held reservations", and that it is computed by the circulation module. E3 must show availability (R3), but it depends only on E1. Loans and reservations belong to E4. AD-5 then freezes whatever E3 publishes, because `v1` changes only add. Copy states (`available`, `held`, `on_loan`, `lost`) are also stored, so they duplicate what AD-3 derives from loans and reservations.

**Unit A (E3), compliant.** Adds `modules/circulation/availability`, returning `available: boolean`, true when any copy is in state `available`. It publishes that in `/api/v1/public` and `openapi.json`.

**Unit B (E4), compliant.** AD-3 assigns a copy only at ready, so a `reserved` title's copy stays `available`. E4 therefore counts open reservations against copies and needs to tell "on loan, you can queue" from "nothing to lend".

**How they collide.**

- A boolean cannot carry the on-loan case R4 needs, and AD-5 forbids changing it. E4 adds a second field, leaving two availability answers in one response.
- If E4 leaves E3's formula alone, the page says "available" for a title whose only copy is already reserved.
- E6 adds a third case: when all copies are lost, the title is neither available nor on loan, and E3's site has no display for it.
- A title with no copies yet, which E1 allows, has the same gap.

**Exact change.**

- **AD-3, add:** "The stored copy state is the source for availability. Loans and reservations change it in the same transaction. With `free` = copies in state `available` minus reservations in `queued` or `reserved`, a title's public `availability` is `available` if `free` > 0, `on_loan` if `free` <= 0 and at least one copy is not `lost`, and `unavailable` otherwise. Reserving gives `reserved` when `free` > 0, and `queued` when the title is `on_loan`. It is refused with a stable code when the title is `unavailable`."
- **Contract:** the enum `availability` exists from E3's first `openapi.json`. E3 ships the function with the reservation term as zero, and E4 fills it in without changing the API.
- **New AD-19 (collection ownership).** See finding 9 for the full table.

## 3. Catalogue writes change circulation outcomes outside the module (Critical)

**The opening in the spine.** AD-2 protects "reservations, queue places, loans and copy circulation status". AD-5 allows librarians generated writes, and the diagram allows "admin to collections, catalogue only". Creating a copy, deleting a copy or title, and unpublishing a title are all catalogue writes, and none changes a protected field.

**Unit A (E1), compliant.** A librarian can add a donated copy, delete a mistaken copy or title, and unpublish a title in Payload admin. Each is an ordinary collection operation, with no title lock.

**Unit B (E4), compliant.** Queue promotion runs only on return and expiry, as R4 and R24 list. Loans and reservations reference copies and titles.

**How they collide.**

- A new copy on a title with a queue is immediately `available`. Nothing promotes the queue head, so `free` is wrong or the next reserver jumps the queue, which breaks first come, first served.
- Deleting a copy with an active loan either fails on a foreign key or nulls the link, which loses who has which book.
- Unpublishing a title with active loans hides it, because AD-9 exposes "only published titles" on `/api/v1`. E4's account view then cannot name a book the patron holds (R5), and E5's reminder has no title to render.
- E2's re-run touches the same records with the scan key.

**Exact change.**

- **AD-2, add:** "Adding a copy, deleting a copy or title, and unpublishing a title are circulation-affecting. Collection hooks route them through `modules/circulation` under the title lock. Adding a copy re-runs queue promotion. Deletion is refused while the record has any reservation, loan or donation, including anonymised ones. Copies leave service through `lost`. Unpublishing is refused while the title has open reservations or active loans."
- **AD-9, add:** "The scan key can create and update, but not delete, and only touches drafts" (see finding 7).
- **AD-5, add:** "A patron's own reservations and loans return the title's display fields regardless of later catalogue changes."

## 4. A loan has no shared end state, so lost and retained records diverge (Critical)

**The opening in the spine.** Reservation states include `picked_up` and `returned`, and loans also carry "the loan's due date and return" (AD-4). That is two records of the same event, and the ER diagram has no link between loan and reservation. Neither has a lost outcome, though `lost` exists for copies. AD-14 counts retention "from a loan's return" and anonymises by naming only loan and donation links.

**Unit A (E5), compliant.** The weekly overdue notice selects loans with a past due date and no return, following AD-4 exactly.

**Unit B (E6), compliant.** R23 sets the copy to `lost`, nulls the loan's patron link as AD-14 says, and leaves the loan otherwise alone, since it was never returned.

**Unit C (E4), compliant.** Keeps terminal reservations (`returned`, `expired`, `cancelled`) with their patron link, as nothing tells it to remove them.

**How they collide.**

- E5's job picks up the lost loan every week with no recipient. Availability's "active loans" counts it indefinitely.
- E6's retention job never reaches that loan, because it has no return date, so the record outlives the period the privacy notice states (R19).
- E6 anonymises loans at the retention date as AD-14 says, while the patron's returned reservations still show which books they borrowed. The reading history the retention rule removes survives one table over.
- If E5 models a renewal as a new loan row, E6's "loans per month" and "families making two or more pickups" are inflated. If E5 updates in place, nothing says so.

**Exact change.**

- **Conventions, Reservation states:** end at `picked_up`, and remove `returned`. Return and loss are recorded only on the loan.
- **Conventions, add a Loans row:** "`reservation`, `copy`, `patron`, `startedAt`, `dueDate`, `renewalCount`, `returnedAt`, `lostAt`. A loan is active while both end fields are null. A renewal updates `dueDate` and `renewalCount` on the same row, and one loan row is one hand-over."
- **AD-4, change to:** "Overdue is computed from an active loan's due date."
- **AD-14, replace the retention bullet with:** "Retention runs as a job. It nulls the patron link on loans counted from `returnedAt` or `lostAt`, on `picked_up` reservations with their loan, and on `expired` or `cancelled` reservations counted from their end time."
- **AD-14, in the deletion bullet:** change "sets loan and donation links to null" to "sets loan, reservation and donation links to null".
- **ER:** add `RESERVATIONS ||--o| LOANS : becomes`.

The six brief signals remain readable from these fields. Distinct pickups can be counted per patron from `startedAt` dates, which is why renewal must not create rows.

## 5. Title lock cannot serialise patron-wide operations (High)

**The opening in the spine.** AD-2 says each operation "locks the title's row". Maximum books, the overdue block and account deletion all span several titles. AD-14 says deletion "cancels through circulation", without saying whether that is one transaction.

**Unit A (E4), compliant.** Pickup locks the title and creates a loan. Reserve locks the title, then checks the limit and overdue state for the patron.

**Unit B (E6), compliant.** Self-deletion (R19) checks that the patron holds no book, calls circulation's cancel once per reservation, each in its own transaction as AD-2 describes, then removes the patron.

**How they collide.**

- Pickup on another title commits after E6's check, so the patron is deleted while holding a book. The loan's link goes null and a copy is `on_loan` with no holder, which R19 forbids.
- A crash partway through leaves a live patron with cancelled reservations.
- Two concurrent reservations for different titles both pass the maximum-books check, since they lock different rows.
- The same applies when E6's deletion or lost marking cascades across several titles, because no lock order is defined.

**Exact change.**

- **AD-2, replace the last sentence with:** "Each operation runs in one database transaction. It locks the patron's row first when a patron is involved, then each affected title's row in ascending id order. Patron-wide checks, including limits, the overdue block and holding a book, are made under the patron lock."
- **AD-14, change the deletion bullet to:** "It calls one `modules/circulation` withdrawal operation, in the same transaction that removes the patron, covering cancellation, lost marking, queue promotion and outbox rows."

## 6. Outbox row shape and deleted-patron behaviour are unspecified (High)

**The opening in the spine.** AD-12 gives only a unique key of "kind, subject record and date". The ER diagram says only loans trigger outbox rows, yet the ready email's subject is a reservation and R23's is a queue place. The `date` has no meaning or zone. E4 owns the delivery job, and E5 and E6 add kinds.

**Unit A (E4), compliant.** Renders language, subject line and recipient address into the row at enqueue, so that delivery has no dependency on anything else.

**Unit B (E5), compliant.** Enqueues only the kind and loan id and expects the delivery job to resolve the recipient at send time. It uses the job's run date as `date`.

**Unit C (E6), compliant.** Nulls patron links and removes the patron, with "one test per record type that references a patron". The outbox references no patron directly, so it gets no test.

**How they collide.**

- E5's rows break E4's delivery job, and the reverse. There is no single row format.
- After deletion, E4-style rows still hold an email address and go on sending to a deleted person, outside the AD-14 field list. E5-style rows resolve to null and retry forever.
- A reminder queued before return or renewal goes out afterwards.
- Using the run date as `date` allows two sends for one reminder if a job retries across midnight, which is the duplicate AD-12 was written to stop. "One a week" (R10) has no key shape at all.

**Exact change.**

- **AD-12, add:** "A row holds `kind`, `subjectType`, `subjectId`, `patron` and `dedupeDate`. It stores no address, name or rendered text. Delivery resolves recipient, language and content at send time, re-checks that the row still applies, and marks the row `dropped` if the patron is null or the condition has ended. `dedupeDate` is a `Europe/Helsinki` calendar date: the due date for reminders, the due date plus 7n days for overdue notices, and the transaction date for state-change emails."
- **AD-14, deletion bullet:** add outbox rows to the records handled and to the per-record-type tests: pending rows are dropped and the patron link nulled.
- **ER:** replace `LOANS ||--o{ OUTBOX` with `PATRONS |o--o{ OUTBOX : addressed_to`, and describe the subject as polymorphic.

## 7. Title identity and draft scope differ between manual entry and scans (High)

**The opening in the spine.** AD-3 makes the title the unit of queueing and reservation. AD-9 says "records are created as Payload drafts carrying a unique `scanId`", without saying which collection holds `scanId`, whether copies are drafts, or how two physical copies of one book are handled. R13 scans physical books.

**Unit A (E1), compliant.** Manual entry adds a second copy to the existing title. Only `titles` has drafts enabled, since publishing a copy is meaningless to a librarian.

**Unit B (E2), compliant.** One scan gives one title and one copy, each a draft with its own `scanId`, as AD-9 literally says. A re-run updates by `scanId`.

**Unit C (E3), compliant.** Counts copies of published titles for availability.

**How they collide.**

- Two scans of one book create two titles, so E4 runs two separate queues with one book each. A parent can queue for one while the other copy sits free. Nothing in the spine merges them, and the merge gets harder once circulation starts.
- E2 marks copies as drafts while E1 does not version them. Payload rejects or ignores the draft option on one side, and on the other, unpublished draft copies count as lendable.
- A re-run after review overwrites the librarian's corrections, since AD-9 says a re-run updates. On a published title it changes the public record without review.

**Exact change: AD-9, replace the second and third sentences with:**

"Only `titles` has drafts. `copies` are not versioned and are visible only through a published title. `scanId` identifies one scanned physical book and is unique on `copies`. Before upload, the script groups scans into one title by ISBN, or else by normalised title and author, and the review view lets a librarian merge draft titles. A re-run creates what is missing and updates only titles still in draft that a librarian has not edited; it never changes a published or reviewed title."

R13 and E2's Done-when say a librarian reviews "before import", while AD-9 places review before publishing after upload to production. **Record this reading in AD-9**: "import" means publishing. It is the only workable reading with no staging environment, but the wording should match.

## 8. `signupOpen` blocks the epics that must prove sign-in, and the gate version has no owner (High)

**The opening in the spine.** AD-8 says sign-up and sign-in "stay closed to real patrons" while `signupOpen` is false, without defining "real" or the enforcement point. AD-17 has only local and production. Every epic must be "deployed to production and verified end to end". AD-5 leaves Payload's own login endpoint on `patrons` reachable. `policyVersion` is on the patron, but nothing says where the current version lives, and E6 owns the policy texts.

**Unit A (E3), compliant.** Checks `signupOpen` in the `/api/v1` sign-up and the Google callback.

**Unit B (E4), compliant.** Reads the flag as a full closure and adds a Payload login hook, so sign-in is closed.

**Unit C (E6), compliant.** Opens sign-up by flipping the flag.

**How they collide.**

- With Unit A, password login on the generated endpoint is open to any account. With Unit B, E3's and E4's own production verification cannot sign in.
- Each epic invents its own test-account path: seed migration, environment list or manual database row.
- Test loans made during E4 and E5 stay in production. AD-14 anonymises rather than deletes, so even deleting the test patrons leaves those loans counted in "loans per month" and "titles borrowed", and E6's "first real loan" is not actually first.
- E3 stores `policyVersion` from a site constant, or from the client, before E6 publishes a policy. Beta accounts have "accepted" a text that never existed, and E6 has no rule for a later version.

**Exact change: AD-8, replace the last sentence with:**

"While `signupOpen` is false, `modules/accounts` allows sign-up and sign-in only for emails on a server-configured beta allowlist. It enforces this on `/api/v1` sign-up, the Google callback and a `beforeLogin` hook on `patrons`. Before the flag flips, `epic-launch` runs one pre-launch reset that hard-deletes allowlisted patrons with their reservations, loans and outbox rows, and returns affected copies to `available`. `currentPolicyVersion` lives in `tenant-policy`; core copies it to the patron when the gate completes and never accepts it from a client. A patron whose `policyVersion` differs from the current one gets `gate_required` until they accept again. Developers bump it only for a material change."

**AD-6:** add `signupOpen` and `currentPolicyVersion` to the policy field list.

## 9. Donor link and consent: two homes, and no epic owns `patrons` first (Medium)

**The opening in the spine.** R17 sits in E1, and E1's Done-when includes "an optional donor". The donor is a patron, but AD-7's `patrons` arrives with E3. AD-8 makes donor consent a "separate, optional, unticked field" with no location. AD-14 puts "consent flags" on donations, which contradicts the initiative's constraint "donor ID, copy ID and date only". AD-14 also says patrons hold "only name, email, language, gate fields and an optional phone number", which excludes `tenant` (required by AD-6 and shown in the ER), the Google identity and the consent field if it is on the patron.

**Unit A (E1), compliant.** Creates a minimal `patrons` collection to link to, or leaves donor as an unlinked optional value set later. It stores consent flags on the donation, entered by the librarian.

**Unit B (E3), compliant.** Creates `patrons` as the AD-7 auth collection and stores sign-up consent on it.

**How they collide.** Two migrations claim one collection slug, and consent has two values that can disagree. A librarian can credit a patron who never ticked the box, and the tick E3 collects changes nothing. E6's deletion handles whichever it knows about.

**Exact change.**

- **AD-14, patron list, replace with:** "Patrons hold only `tenant`, name, email, language, gate fields, `donorConsentAt`, Google subject id and an optional phone number that only librarians can read or set."
- **AD-14, donations, replace with:** "Donations store copy, optional donor and date, with no consent flags or free-text identity. A donor can be linked only when `donorConsentAt` is set."
- **New AD-19: Collection ownership.** "Each collection is first created by one epic, and later epics extend it only by migration: E1 `tenant-policy`, `librarians`, `titles`, `copies`, `media`, `donations`, and `patrons` with its full AD-14 fields and no public route. E3 adds patron auth routes, the gate and the availability function. E4 `reservations`, `loans`, `outbox`."

If `outbox` is needed earlier, see finding 10; auth mail bypasses it.

## 10. Policy change and first-email ownership conflict with the requirements and tickets (Medium)

**10a. Spine against requirement rather than a pure unit pair.** AD-6 says values are changed "through a versioned migration", and AD-17 means a migration is a commit, a CI build and a deploy. R20 and E1's Done-when say a value "changes without a code change". A compliant E1 cannot pass its own Done-when. Each builder will resolve this differently, and E6 needs a real answer because the loan values are still an open question up to first loan.

**Exact change, AD-6:** replace "changed by developers through a versioned migration" with "seeded by a migration, then changed by a developer running a `policy:set` command in the core container, which validates the value and keeps history through Payload versions, with no commit or deploy". If the founders accept a migration as "no code change", amend R20 instead. Either way, one of them must change.

**10b. First-email ownership.** The Deferred table says the email provider is decided by the "first epic that sends email". E4's ticket owns "email sending". AD-5 and AD-12 make E1 (librarian forgot-password) and E3 (patron forgot-password) send mail through the shared adapter first. A compliant E3 picks a provider, and then a compliant E4, which owns it by ticket, picks another or reconfigures it.

**Exact change.**

- **Deferred, Email provider row:** change "Decide by" to `epic-librarian-catalogue`.
- **AD-12, add:** "E1 sets up the SMTP adapter and sending domain, and E4 adds the outbox and delivery job."
- **E4's ticket boundary** needs a matching edit by the ticket owner. This review changes no ticket.

---

## Combined change list

| Target | Change | Findings |
|---|---|---|
| AD-2 | Patron-then-title lock order; catalogue operations that affect circulation are routed or refused | 3, 5 |
| AD-3 | Queue place is a `queued` reservation; availability formula and enum | 1, 2 |
| AD-4 | Overdue computed from an active loan | 4 |
| AD-5 | Patron's own records keep title display fields | 3 |
| AD-6 | `policy:set` command; add `signupOpen`, `currentPolicyVersion` | 8, 10a |
| AD-8 | Beta allowlist, enforcement points, pre-launch reset, policy version source | 8 |
| AD-9 | Drafts on titles only; `scanId` on copies; grouping and merge; re-run limits; meaning of "import" | 3, 7 |
| AD-12 | Outbox row shape, send-time rendering, `dedupeDate`; E1 owns adapter | 6, 10b |
| AD-14 | Patron field list; donation fields; one withdrawal transaction; retention covers reservations; outbox on deletion | 4, 5, 6, 9 |
| New AD-19 | Collection ownership by epic | 2, 9 |
| Conventions | Drop `queue-places`; reservation states `queued` to `picked_up`, plus `expired`, `cancelled`; Loans row | 1, 4 |
| ER | Drop `QUEUE_PLACES`; add reservation-to-loan; outbox addressed to patron | 1, 4, 6 |
| Deferred | Email provider decided in E1 | 10b |

## Attacked and dropped

These were tried and left out, because one later story can settle them without another epic noticing, or because the ADs already hold.

- **Phone number write path (`modules/circulation` in the map, accounts implied by AD-14):** only E4 writes it, and deletion removes the whole patron either way.
- **Payload auth responses outside `openapi.json` and outside the `{ "error" }` shape:** only E3 consumes them. A thin wrapper in that epic settles it.
- **`payload-oauth2` creating patrons outside `/api/v1`:** AD-8's gate still holds and the choice is in one E3 story, though that story should note AD-5's wording.
- **Backup schedule (AD-13 "all scheduled work" against a host-level `pg_dump`):** owned wholly by E1 and its infrastructure, with E6 only testing a restore.
- **Static export against dynamic book URLs:** AD-15's shared page plus Worker settles it within E3.
- **Three image fields on a title (front, back, information page):** E2 depends on E1 for the record format, and both read R3. Settled at E1's schema story.
- **Cookie, CORS and origin change at the `piviki.fi` switch:** one E6 configuration story. Cookies stay on `api.piviki.fi`, and both origins are same-site.
- **Tenant reference target and tenant filtering:** set once by E1, and later epics inherit it.
- **Quick return-to-ready choice:** internal once finding 1 is applied, as the Deferred table already claims.
