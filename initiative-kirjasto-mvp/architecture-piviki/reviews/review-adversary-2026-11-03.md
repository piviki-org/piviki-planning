# Adversarial review: Piviki MVP spine, 2026-11-03 update

- **Target:** `architecture-piviki.md` (updated 2026-11-03)
- **Scope:** AD-9 (barcode, matchKey, `duplicate_title`, one-copy initial load, labels, barcode lookup), AD-11 (MARC language codes), AD-20 (bibliographic shape), and how they interact with AD-1, AD-2, AD-3, AD-7, AD-10, AD-16 and AD-19.
- **Method:** for each hole, two units one level down that both follow every AD exactly and still build things that do not fit together. Then the smallest rule that closes the hole.

## Verdict

The update fixes the old `scanId` collision and gives the catalogue a shape a MARC export can use. But the new dedupe machinery is under-specified in exactly the places where two builders will choose differently. The `matchKey` formula refers to a value that AD-20 does not store. The duplicate check and the "has no copy" check are check-then-insert with no lock. The label print order cannot stay in step with the physical pile of books. And removing `scanId` left re-runs with no stable key, which also lets a merge bring a duplicate back. Two holes are Critical, two are High, three are Medium, and a few minor ones are listed at the end. Each has a short closing rule.

---

## 1. `matchKey` is not computable the same way twice (Critical)

**Units:** core's `matchKey` hook (epic-initial-load) vs the librarian "possible duplicate" warning in manual entry (epic-librarian-catalogue). A third reader is any check the scan script runs before it creates a record.

**What the spine says.** AD-9: "the ISBN-13 when one exists, otherwise the AD-10-normalised title, first contributor's entry element and volume." AD-20: `entryElement` is one of `surname`, `forename` or `family`. That is a form indicator, like the first indicator of MARC 100, not a name string.

**How they diverge:**

- **Entry element.** Builder A reads the spec literally and concatenates the enum value. Nearly every key then contains `surname`, so different books by different authors with the same title and volume match each other. Builder B parses a surname out of `name`, but nothing says which token. For "Nguyễn Nhật Ánh" the surname is the first token; for "J. K. Rowling" it is the last. B guesses by culture, so the key is not deterministic across writers.
- **Which ISBN.** `isbns` is a list. One builder takes `isbns[0]`; another takes the lowest. Writers order the list differently: the vision model's reading order, Melinda's 020 order, a librarian's typing order. The same book then gets different keys.
- **Set ISBNs.** Series and multi-volume books carry a set ISBN alongside the volume ISBN. If the set ISBN is chosen, volume 2 matches volume 1. The scan client gets `duplicate_title`, and the title it matched already has a copy, so the script skips it. **Volume 2 is silently never catalogued.** The ISBN branch also ignores `volume`, so no fallback catches this.
- **Mixed branches.** A title whose ISBN was read can never match the same book scanned where the ISBN was not read, because the two keys live in different namespaces. That one is acceptable, since merge handles it, but the spine should say it is expected.
- **Empty components.** If the title is unread, or the book has no contributor (folk tales, anthologies), the key degenerates to something like `||1` or `||`. Every such scan after the first gets `duplicate_title`, and all of them are dropped.
- **Volume normalisation.** "Tập 1", "T.1", "1" and "01" are different strings after AD-10, because AD-10 only removes diacritics and lowercases.
- **Tenant.** The spine does not say whether the key is unique per tenant. Under AD-6 it must be.

**Closing rule (tighten AD-9):**

> `matchKey` is computed only by one core function, `catalogue/matchKey`, which the librarian warning also calls, and only for titles whose `title` is non-empty (otherwise `matchKey` is null and never matches). It is `tenant` plus either `isbn:` and the numerically lowest `isbn13` whose `qualifier` is not `set`, followed by `|` and the normalised volume; or `t:` followed by AD-10-normalised `title`, `|`, the AD-10-normalised full `name` of the first contributor with role `aut` (else the first contributor, else empty), `|`, and the normalised volume. Normalised volume is the digits with leading zeros removed when the volume contains a digit, otherwise its AD-10 form. AD-20's `qualifier` vocabulary includes `set`. Keys from the two branches never match each other; the review view's merge handles that case.

---

## 2. Duplicate refusal and "one copy" are check-then-insert races (Critical)

**Units:** two concurrent runs of `piviki-scan`, for example both founders scanning on two machines, or one run with parallel requests. Or a script re-run that overlaps a run that is still going.

**What the spine says.** The `duplicate_title` refusal is a hook that compares against existing titles. A unique index is impossible, because librarians may create duplicates ("warned, not refused"). The copy step is: "creates a copy only when that title was created by a scan client and has none". The script decides "has none" by reading over the generated API and then creates the copy in a second request.

**How they diverge:**

- **Two titles.** Run A and run B both create the same book. Each hook reads "no match" before the other commits, so both titles are created.
- **Two copies.** Both runs get `duplicate_title` for title T, both read "T has no copy", and both create one. T now has two copies and two barcodes, which breaks "exactly one copy per title" and puts two labels into the print run for one physical book.
- **AD-2 does not cover this.** It serialises circulation operations under title locks, but the "has none" decision is made in the client, outside any transaction. AD-2's lock does nothing for it.

**Closing rule (add to AD-9):**

> A `scan-clients` title create takes a transaction-scoped advisory lock on `matchKey` before its duplicate check. A `scan-clients` copy create is refused with the stable code `copy_exists` when the title already has a copy, checked under the title's row lock (the AD-2 order). The script never decides "has none" itself; it attempts the create and treats `copy_exists` as done.

---

## 3. Re-runs have no stable key, and merges bring duplicates back (High)

**Units:** the epic-initial-load re-run path vs the review view's merge (also epic-initial-load) and later librarian edits.

**What the spine says.** Before the update, `scanId` was the script's idempotency key. The barcode replaced it, but the barcode is assigned by core after the create, so the script cannot know it in advance. The only correlation left between "this photo set" and "that record" is `matchKey`.

**How they diverge:**

- **Different extraction.** The vision model is deferred and may change, or a prompt may be tuned. Run 2 then reads a title slightly differently, so for every book without an ISBN the `matchKey` differs, and run 2 creates a second draft and a second copy. "A re-run creates only what is missing" holds only if extraction is bit-for-bit stable.
- **Merge brings the loser back.** The librarian merges draft T2 into T1: the copies move and T2 is deleted. A re-run then finds no title with T2's key, creates T2 again with a fresh copy and barcode, and the merge is undone.
- **Updates go to the wrong title.** The script cannot tell which record to update. Re-run updates land on whatever title the key happens to match now.

**Closing rule (add to AD-9 and AD-20):**

> `copies.scanRef` is an optional, write-once identifier supplied by `scan-clients` (the photo set id), unique per tenant, never shown to patrons. The script first looks up a copy by `scanRef`. If one exists, it updates that copy's title (subject to the eligibility hook) and creates nothing. Merge moves copies with their `scanRef`, so a re-run finds the moved copy and does not recreate the loser.

This also makes most of hole 2 disappear for the single-machine case, and it needs no new endpoint. It is only a filtered read over the generated API.

---

## 4. Label order cannot stay in step with the physical pile (High)

**Units:** the label print view (epic-librarian-catalogue or epic-initial-load; the spine does not say which) vs the person sticking labels onto the stack of scanned books. Circulation screens downstream then trust those labels.

**What the spine says.** Labels print "only for copies of published titles, ordered by copy creation", and the label shows barcode, title and first contributor. The memlog says the creation order is meant to be the scan-batch order of the physical pile.

**How they diverge:**

- **Gaps in the run.** Review is partial and out of order. If copies 41–60 are in the pile but 47 is still a draft, or was merged away, the print run for that range has 19 labels for 20 books. Everything after 46 is now off by one in pile order. The title on the label is the only guard, and Vietnamese titles that look alike (series volumes) make wrong sticking likely. A wrong label permanently binds a physical book to the wrong title, so every later loan, reservation and loss is recorded against the wrong record.
- **Books that never got a copy.** A book whose scan got `duplicate_title` against a librarian-created title gets no copy, because the script creates copies only for scan-client titles. It therefore has no label and no record, and nobody is told. The spine's "further copies are added by librarians" assumes the librarian knows.
- **No printed state.** Nothing records whether a label was printed. One builder prints "all published since last time" and invents a `labelPrintedAt` field. Another prints a barcode range. AD-19 gives no owner for that field.

**Closing rule (tighten AD-9):**

> The label view prints a contiguous range of copy creation order and emits a placeholder slot, showing the barcode and "not printed: draft / removed", for every copy in the range that is not printable, so slot order matches pile order. `copies.labelledAt` is set by the view, and the view also lists unlabelled printable copies. The scan script writes a local report of every scan that ended in `duplicate_title` against a non-scan-client title, so a librarian adds those copies by hand.

The spine should also name the epic that owns the label view (AD-19 style), since both catalogue epics can claim it.

---

## 5. Provenance flags have no shape or owner, and merge has no mutation path (Medium)

**Units:** epic-librarian-catalogue, which creates `titles` and `copies` and ships first, vs epic-initial-load, whose hooks need "created by a scan client" and "edited by a librarian".

**How they diverge:**

- **Provenance not stamped from the start.** The eligibility hook needs per-title provenance. If epic-librarian-catalogue does not stamp it from its first version, the titles librarians created or edited before epic-initial-load cannot be told apart from scan drafts when initial-load adds the field by migration. One builder backfills "librarian" on everything; another leaves it null and treats null as eligible. The second lets the script overwrite a librarian's work, which AD-9 says must never happen.
- **Merge as a plain admin edit.** One builder implements merge as admin edits to `copies.title`. Another routes it through `modules/circulation`. AD-2 routes adding and deleting a copy through the module but says nothing about moving one. A plain `copy.title` write on a published target with `queued` reservations skips queue promotion. Deleting the emptied loser is refused if any copy on it has a donation, unless the copies are moved first.

**Closing rule (AD-19 note plus AD-2):**

> `titles` and `copies` carry `createdByKind` (`librarian` or `scan-client`) and `titles.librarianEditedAt`, stamped by core hooks from epic-librarian-catalogue's first migration. A copy's `title` changes only through a `modules/circulation` move operation, which locks both titles in id order and re-runs promotion on the target. Merge is that move followed by deleting the empty draft.

---

## 6. Barcode "never reused" does not survive a restore (Medium)

**Units:** the AD-17 restore procedure vs AD-9's sequence. Physical labels outlive the database.

**How they diverge.** Labels printed after the last nightly dump carry barcodes the restored sequence will hand out again. That breaks "never reused" and gives two books the same label. The restore steps re-delete erasures but do nothing to the sequence. Separately, a `pg_dump` taken without sequence state (data-only, or a selective table copy) restarts the sequence at 1.

**Closing rule (add to AD-17 restore):**

> Before the app starts after a restore, the copy barcode sequence is set to max(existing number) plus a fixed gap (for example 1000), and the restore test asserts this.

---

## 7. Language codes: ordering, validation list and patron filtering (Medium)

**Units:** `piviki-scan` (the LLM says "Vietnamese and English") vs ISBN lookup (Melinda gives 008/35-37 plus 041) vs the patron site (a filter by book language, with BCP 47 in its URLs).

**How they diverge:**

- **Order.** The scan writes `['eng','vie']` in alphabetical order. The lookup writes the main language first. A MARC export needs a primary language for 008, so the two records export differently.
- **Code list.** "MARC language codes" does not say whether the codes are validated against a list or how obsolete codes are handled. One builder hard-codes `vie` and `eng`. Another accepts any three letters.
- **Patron filter.** The patron site may filter by `vi`, the patron's BCP 47 language, against stored `vie` codes. AD-1 forbids rules on the patron site, but the mapping has to live somewhere.

**Closing rule (AD-11 / AD-20):**

> `languages[0]` is the main language of the text. Core validates codes against the MARC code list bundled in core. `/api/v1` filters and returns MARC codes only, and the patron site shows their names from message files keyed by MARC code.

---

## Minor (Low)

- **Initial copy state.** AD-2 denies copy state on the generated API, so a scan-client copy create cannot set it. State that core sets `available` on every create, and refuses `state` from any client.
- **Where ISBN lookup lives.** The deferred lookup function is "in core", but the scan script cannot reach it. If the script later does its own Melinda lookup, there are two lookup implementations that map 100/700 differently. State that the only lookup is the core function, and that `piviki-scan` does no external lookup unless core exposes that function to `scan-clients`.
- **"Runs once" vs re-runs.** "The initial load runs once" contradicts "a re-run therefore creates only…". Say "one campaign, re-runnable until review starts".
- **`matchKey` on update.** AD-9 refuses duplicates only on create. A scan-client update that changes the key into a collision passes. Apply the same refusal on scan-client updates.

## Attacked and dropped

- **Barcode collisions between script and manual entry.** Closed by the single sequence and the core hook.
- **The scan client publishing or deleting.** Closed by the AD-9 hook and AD-7.
- **AD-10 and Melinda NFD text.** Closed: normalisation happens on write.
- **Lock order with copy creates.** Copy create locks only the title, which is consistent with AD-2.

## Combined change list

| # | Severity | Change |
|---|---|---|
| 1 | Critical | Precise `matchKey` formula in one core function: lowest non-set ISBN plus volume, else title, first `aut` full name and normalised volume; null when the title is empty; scoped per tenant; add `set` to the ISBN qualifier vocabulary |
| 2 | Critical | Advisory lock on `matchKey` for scan-client title creates; `copy_exists` refusal checked under the title row lock |
| 3 | High | `copies.scanRef`, write-once and unique per tenant, as the re-run key; merge keeps it |
| 4 | High | Label ranges with placeholder slots and `labelledAt`; script report of skipped books; name the label view's owning epic |
| 5 | Medium | `createdByKind` and `librarianEditedAt` stamped from epic-librarian-catalogue's first migration; copy move only through circulation |
| 6 | Medium | Advance the barcode sequence on restore |
| 7 | Medium | `languages[0]` is the main language; validated MARC code list; API speaks MARC codes |
