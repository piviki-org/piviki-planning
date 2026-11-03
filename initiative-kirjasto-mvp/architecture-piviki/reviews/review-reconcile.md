# Reconcile review: Piviki MVP architecture spine

Reviewed `architecture-piviki.md` against `initiative-kirjasto-mvp.md`, the addendum's MVP rules and Design notes, and `.memlog.md`. The Success Criteria table in `brief-kirjasto.md` was read only to check Done-when item 5.

## Verdict

**Needs fixes before adoption; no structural rework.** The paradigm and all 18 decisions hold up, and every R-id is bound and mapped. One requirement is contradicted (R20), one memlog assumption is dropped and its decision overclaims (AD-15), one Done-when item has no home (the GDPR question), and the rest are gaps in AD-3, AD-5, AD-8, AD-9 and AD-14.

Severity: **High** means it contradicts an input or would make epics diverge. **Medium** means an input is weakened or a decision is incomplete. **Low** means wording or a record to tidy.

## High

### H1. R20: AD-6 makes a policy change a code change

- **Input:** R20 says each value "can be changed without a code change or blocking launch". The Decision says developers change the values, with no admin screen.
- **Spine:** AD-6 says values are "changed by developers through a versioned migration". A migration is committed code, shipped through CI and deployed under AD-17. The memlog records no decision for this.
- **Fix:** Have developers change values with a small audited command against the running core, with no build or deploy, and use migrations only for the initial seed. If the founders prefer migrations, record that as a decision and have Sofia amend R20 to "without changing module code". Either way, this needs a founder's call.

### H2. Memlog: the Worker CPU assumption is missing, and AD-15 promises more than it can deliver

- **Input:** The memlog assumes the Worker stays under 10 ms CPU on the free plan, to be measured in the first patron-site story, with the paid plan or dropping previews as the fallback. It also records that exceeding the limit gives Error 1102, and that fail-open applies only to the daily request limit.
- **Spine:** The assumption and fallback appear nowhere. AD-15 says "any error or a timeout of 1 second" serves the unmodified page, and claims to prevent "a preview failure breaking the page". Worker code cannot catch its own CPU kill, so that failure breaks the page.
- **Fix:** Add a Deferred row: "Worker CPU under the free limit; measure in the first patron-site story; fallback is to drop previews, or the paid plan if the budget allows". Limit AD-15's claim to failures the Worker can catch, and name the CPU limit as the remaining risk. Also state how the daily limit is set to fail open.

### H3. R4: availability does not clearly count reservations with no copy yet

- **Input:** R4 lets a patron reserve an available title, with a copy assigned when marked ready.
- **Spine:** AD-3 computes availability from "copies, active loans and held reservations". "Held" is undefined here. The only `held` in the spine is a copy state, and it applies from ready. A reservation in state `reserved` has no copy, so a one-copy title could read as available and accept a second reservation.
- **Fix:** Define it in AD-3: available copies are those not `lost`, not on an active loan, and not matched by an open reservation in `reserved` or `ready`. Say that a title with none available takes queue places. R4 says "on loan", so also confirm the same applies when copies are only spoken for.

### H4. R19, R23 and the retention Decision: AD-14 omits the rules it exists to fix

AD-14 claims to prevent "deletion or retention missing a record", but:

- **Retention outcome is missing.** The Decision says "loan records are then anonymised". AD-14 says only "Retention runs as a job counted from a loan's return", so a story could delete the rows and lose the success signals.
- **The two deletion paths are not distinguished.** A patron is refused while holding a book (R19). A librarian can proceed, and held copies become lost (R23). AD-14 describes one path with neither rule.
- **Linked records are incomplete.** AD-14 nulls only loan and donation links. The ER diagram also has nullable patron links on reservations and queue places, and outbox rows can name a patron or their email.
- **Backups have no retention.** The Constraint says "retention periods are stated". Nightly dumps under AD-17 keep deleted patrons for an unstated time, and logs have no period either.

**Fix:** Add to AD-14: "at the end of retention the loan's patron link is nulled and the loan kept". State both deletion paths in one sentence. Make the rule "every reference to a patron, including reservations, queue places and outbox", with sent or pending outbox rows purged or anonymised. Set a backup and log retention period for the privacy notice to state.

### H5. Done-when 4: the GDPR question and the launch documents have no home

- **Input:** The Art. 9 question must be answered before launch, and the Notes say the answer "may change the legal basis and the sign-up wording". The usage policy "blocks launch". The privacy notice must be published and must state 3 years, the phone number and possible future donation benefits (R19, addendum).
- **Spine:** None of this appears in Deferred, and nothing ties it to `signupOpen`. The map gives R19 to `modules/accounts`, with no location for the notice or policy. The 3-year starting value is absent. `policyVersion` assumes a policy exists but does not say where it is published.
- **Fix:** Add a Deferred row for the Art. 9 answer, by "before `epic-launch` closes", noting that it can change the AD-8 gate wording. Add to AD-8 or AD-17 that `signupOpen` stays false until the notice and policy are published. Name where the two documents live. Note that retention is seeded at 3 years.

### H6. R2, AD-5 and AD-8: the gate has paths around it

- **AD-5** says "Patron creation goes through `/api/v1`" and allows only login, logout, me, forgot-password and reset-password outside it. Google first sign-in creates a patron through the auth flow (`payload-oauth2` or small core endpoints), which is neither. As written, AD-5 forbids what AD-8 requires.
- **AD-8** says sign-in stays closed while `signupOpen` is false, but login is an allowed generated operation, so no stated rule enforces this at that surface.
- **`signupOpen` also closes the team out.** The Decision says sign-up is closed "to real patrons", and the tracer path needs a parent to sign up in production before launch. A single tenant-wide flag leaves no way for testers to do this.
- **Memlog is weakened.** It says a first-time Google user "can do nothing until" the gate is passed. AD-8 blocks only `/api/v1` writes.

**Fix:** Add the Google callback to AD-5's exception list and say that any patron creation sets no gate fields. Make `signupOpen` enforced in `modules/accounts` on every creation and login path. Add an allowlist of test emails or similar. Align the wording with the memlog, or note that own-account reads are deliberately allowed.

## Medium

### M1. R13 and the addendum: scan writes can change reviewed records

- AD-9 makes a re-run "update rather than duplicate" and says records are "created" as drafts. Nothing stops a re-run overwriting a librarian's corrections or changing a published record, and nothing stops the key setting `_status` to published. This is the failure AD-9 claims to prevent.
- R13 and the addendum say review comes before import. The spine stores drafts first and treats publishing as the import. That is reasonable and matches the memlog, but it should be said, because Done-when 1 reads "imported … after librarian review".
- **Fix:** The key can only write drafts and cannot publish. A re-run skips a record once published or edited by a librarian. Add one line: "import means publish; the draft stage is the review".

### M2. AD-9 and AD-16: unreviewed images are already public

AD-9 scopes the scan key to `media`, and AD-16 has a single public bucket for catalogue images. Images for unpublished drafts are therefore publicly reachable by URL before review, while AD-9 claims to prevent "unreviewed scans going public" and limits exposure only for `/api/v1` and availability.
**Fix:** Either hold draft images privately until publish, or state explicitly that unguessable URLs are accepted for draft images.

### M3. Decision on donor consent: AD-14 excludes a field AD-8 requires

AD-14 says patrons hold "only name, email, language, gate fields and an optional phone number". AD-8 adds a donor consent field and the Google link needs an identity. AD-14 also puts "consent flags" on donations, so the source of consent is unclear, given consent is taken at account creation.
**Fix:** Extend AD-14's list with donor consent and the Google subject, and state that the patron's field is the source of consent.

### M4. Memlog constraint: the budget and month-to-month term are missing

The memlog's "Budget about 15 EUR a month; operations must stay simple and cheap" is absent, as is the decision's "month to month at 4.49 EUR" (the 12-month term auto-renews). This constraint drove the hosting and static-export choices and limits later ones, such as the Worker paid plan, the email provider and new processors.
**Fix:** Add both to AD-17, or a short constraint line in Design Paradigm.

### M5. Open question: first loan policy values are not tracked

The Notes need loan period, maximum books and renewal limits by end of December 2026, with someone confirming them. AD-6 holds the fields, but no values or decide-by exist, and AD-4, R6 and R9 depend on them.
**Fix:** Add a Deferred row, decided by the stakeholders, by first due date.

### M6. Email Constraint: Deferred loosens the rejected options

Deferred says "Any provider with SMTP and a data processing agreement fits". The Constraint requires a transactional service and rejects personal Gmail and Proton.
**Fix:** "Any transactional email service with SMTP, the library's domain and a data processing agreement; personal Gmail and Proton are rejected."

## Low

| # | Item | Issue | Fix |
|---|---|---|---|
| L1 | R15, Boundaries | "Nothing expires automatically" is implied by AD-4, never stated, yet AD-13 allows jobs to call modules | Add to AD-4: "Only a librarian moves a reservation to `expired`" |
| L2 | R7, Boundaries | "No fine is ever charged" and no firmer consequences are unstated | One clause in AD-2 or AD-4: the block is the only consequence, with no fee fields |
| L3 | R6 | "Refused when others are queueing" appears only as "renewal checks" | Name it in AD-2 if it is meant as an invariant |
| L4 | Decision: one tenant per patron | Only implied by the ER diagram | Add to AD-6, and decide email uniqueness per tenant now, since R25 forbids a later model change |
| L5 | Constraint: DPA for "each hosting or SaaS provider" | AD-18 covers patron data only, so the scan LLM provider and GitHub have no stated position | Say that they hold no patron data and whether a DPA is needed |
| L6 | Constraint: 72-hour breach notice | Deferred names a contact but not the 72 hours | Add "72 hours" to the Deferred row |
| L7 | Constraint: cut order | Not recorded; only ISBN lookup is called cuttable. Nothing contradicts it | Add one line that R12, R6, R9 and the English interface must be removable without touching never-cut work |
| L8 | Decision: "non-public address" | `beta.piviki.fi` is reachable by anyone and nothing stops indexing | Add `noindex` or access control to AD-17 |
| L9 | R3 | AD-16 says "cover and information-page images"; the back cover is only implied | Say "front cover, back cover and publishing information page" |
| L10 | Frontmatter `scope` | Says "six epics" but lists five system parts, which are not the epics | Name the six: `epic-librarian-catalogue`, `epic-patron-site`, `epic-reservation-loop`, `epic-loan-care`, `epic-initial-load`, `epic-launch` (confirm against the epic files) |
| L11 | AD-8 `Binds` | Lists R7, but AD-8 has nothing to do with the overdue block | Remove R7 |
| L12 | ER diagram | Only `LOANS` triggers `OUTBOX`, yet R8 comes from reservations and R23 from queue places | Add both, or make the subject generic |
| L13 | Collections convention | Omits `media`, which AD-9 scopes the key to; queue places have no state list, though R23 cancels them | Add `media` and the queue place states |
| L14 | AD-4 vs AD-12 | "Jobs send emails" conflicts with "Only `modules/notifications` sends" | "Jobs trigger emails through `modules/notifications`" |
| L15 | Done-when 5 | Signals are readable, with two limits: deletion nulls the patron link, which understates repeat families, and "pickups" must be inferred by grouping loans by patron and date, since a visit is not recorded | Note both in the Deferred row, and correct its claim that records exist "through AD-14", which is the rule that removes links |

## Decision record

- **Spine decisions with no memlog entry.** All 18 decisions are marked `[ADOPTED]` while the spine's status is `draft`. The memlog ends at the nameserver check and has no entry for the transaction and locking rule in AD-2, or for AD-4, AD-6's migration rule, AD-10, AD-12, AD-13, most of AD-14, `beta.piviki.fi` or the `pivikirjasto` organisation. These may have been accepted elsewhere, but the memlog cannot confirm it. Log them or show them to the user at sign-off.
- **MIT licence.** It landed in AD-18, but the initiative still lists the licence as an open question. Sofia still has to record it, as the memlog notes.
- **Binding drift.** The memlog binds the patron-site decision to R16, and AD-15 binds it to R21 instead. R16 is covered by AD-14, so this needs no change.

## Checked and landed

- **Requirements with no findings:** R1, R5, R8, R9, R10, R11, R12, R14, R16, R17, R18, R21, R22, R24, R25.
- **Constraints:** minimal tenancy, pickup at a librarian's home and outside the system, community ownership, Vietnamese first, donor minimisation and no public donor name.
- **Memlog decisions:** core owns the rules, the single circulation module, Mantine on the patron site only, one repository per project, Payload auth for both identities with Cognito and Auth0 rejected, verified-email Google linking, one SMTP path, OVHcloud on Docker Compose, and building in CI.
