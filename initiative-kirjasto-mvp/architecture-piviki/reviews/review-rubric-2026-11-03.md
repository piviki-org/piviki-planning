---
type: architecture-review
reviews: _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md
baseline: architecture-piviki.before.md (2026-11-02)
scope: '2026-11-03 update: AD-9, AD-11, AD-20, Copy identity convention, capability rows R11/R12, R13, R14, two Deferred rows'
altitude: initiative
reviewed: '2026-11-03'
verdict: revise-before-build
---

# Rubric review: 2026-11-03 update to the Piviki MVP spine

## Verdict

**Revise before build (small edits).** The update closes the gap the MARC 21 research found: copies now have a permanent identity and every writer stores the same bibliographic shape. AD-11's language-code split is clean. Two defects in AD-9 would still cause wrong behaviour in `epic-initial-load`:

- F1: the title match key is built from a field that AD-20 defines as an enum, so different books collide. Under the script rule, the second book is then dropped without notice.
- F2: the one-copy and draft-only limits on the scan client are things the script chooses to do. Core does not enforce them, and the "half-finished imports" listed under Prevents are only recoverable, not prevented.

Text edits to AD-9 and AD-20 fix both. No structural rework is needed.

## Scorecard

| Criterion | Result | Findings |
|---|---|---|
| Fixes the real divergence points, misses none | Partial | F4, F7, F9 |
| Every Rule is enforceable and prevents its stated divergence | Fail | F1, F2, F3, F5 |
| Nothing under Deferred lets two units diverge | Pass, with a note | F10 |
| Covers every requirement it binds | Pass, with gaps in the map | F4, F8 |
| Consistent with AD-1, AD-2, AD-7, AD-10, AD-16, AD-19 | Partial | F2 (AD-2), F3 (AD-10), F9 (AD-19) |
| Decisions not rationale; invariants not full shape | Mostly | F11 |

## Findings

### F1 (high): `matchKey` uses the entry element, which AD-20 defines as an enum

AD-9 builds the fallback key from "first contributor's entry element". AD-20 defines `entryElement` as one of `surname`, `forename` or `family`, which names a kind of name part, not the text of a name. Taken literally, the key is normalised title + `family` + volume. Two different books with the same title, such as a common title like "Truyện cổ tích" by different authors, get the same key. The second create is refused with `duplicate_title`. The script then sees that the existing title already has a copy and creates nothing. The second book ends up with no record, no copy and no label, and nobody is told.

The same gap affects AD-20's own stated aim. To build MARC 100 `$a` in inverted form (`Smith, John`) and to derive the Finnish shelf-mark headword, an exporter needs the entry element as text. With only the name as written plus the kind of part, it has to parse multi-word names, which is exactly the guessing the research warned against for Vietnamese names.

**Fix:** In AD-20, give each contributor the text of the entry element. For example, `{ name, entryName, entryElement, role }`, where `entryName` is the part entered first, such as `Nguyễn`. Alternatively, store the name in inverted form. In AD-9, key on the AD-10 search-normalised `entryName` of the first contributor, or on the full `name` if F1 is fixed some other way.

### F2 (high): scan-client copy limits rest on the script, not on core

- "After `duplicate_title` it creates a copy only when that title was created by a scan client and has none" describes what the script does. Under AD-1, core owns the rules, and AD-9 itself gives `scan-clients` blanket create on `copies`. A bug in the script or a second run with different input can therefore give a title two copies, or add a copy to a title that has been published or edited by a librarian.
- Adding a copy to a published title goes through AD-2's queue promotion. That is the "script touching circulation" which AD-9 claims to prevent.
- The Prevents line names "half-finished imports". The Rule makes two calls, title then copy, so a failure between them still leaves a title with no copy. The rule only makes that recoverable on a re-run. The research recommended one atomic import call.

**Fix:** Add a core hook rule: a `scan-clients` copy create is refused, with a stable code, unless the title is a draft, was created by a scan client, has not been edited by a librarian and has no copy. Then either change the Prevents wording to "half-finished imports that a re-run cannot complete", or make the title-plus-copy create one transaction in core, for example through a single import endpoint or a title-create hook that creates the copy.

### F3 (medium): `matchKey` is ambiguous on ISBNs and on which normalisation applies

- `isbns` is a list (AD-20), but the key is "the ISBN-13". A title stored with its hardback ISBN does not match a scan of the paperback, and the spine does not say which ISBN is chosen when there are several.
- "AD-10-normalised" can mean either of AD-10's forms: NFC on write, or the lowercased, diacritic-free search column. Different choices give different duplicate behaviour. The research asks that matching ignore diacritics (Melinda and Koha deliver NFD).
- Because librarians are only warned, several titles can share a key, so "the existing title's id" in the error is not always a single id.

**Fix:** Say that a create matches when any of its ISBN-13s is in an existing title's `isbns`; otherwise it falls back to the search-normalised key. Name AD-10's search normalisation explicitly. When several titles match, return the oldest id.

### F4 (medium): AD-20 does not say which fields are required

The table fixes shape but not whether a field must be present. R11 requires fast manual entry with no online metadata. If one epic's validation requires `contributors` with roles, `languages` and `audience`, manual entry slows down and scan drafts may fail validation. Those are the writers AD-20 exists to align.

**Fix:** Add one line, for example: "Only `title` is required. `languages` defaults to `[vie]` and `audience` to `juvenile`. Every other field is optional, but where it is present it must have the shape above."

### F5 (medium): the review merge is unspecified now that every draft has a copy

The rules imply that duplicates which slip past `matchKey`, such as one scan with an ISBN and one without, become two draft titles, each with its own copy and its own physical book. AD-9 says the review view merges draft titles but not what happens to their copies. If the merge deletes the losing title with its copy, a real book loses its record and barcode.

**Fix:** "Merging moves every copy to the kept title, and its barcode, unchanged, and then deletes the empty draft."

### F6 (low): skipped scans are silent

After `duplicate_title`, when the existing title already has a copy or was entered by a librarian, the script creates nothing. This is the right outcome if the scan was a repeat. It is wrong if the scan was a second physical copy, and "further copies are added by librarians" only works if librarians know about that book.

**Fix:** Have the script list every scan it skipped along with the matched title id, so that a librarian can add a copy or discard the scan during review.

### F7 (low): barcode format leaves a scanner and label divergence open

- The pad width is not fixed. Typed lookup (`PVK123` against `PVK000123`) and Koha-style next-number generation after a migration both depend on it.
- The symbology is not fixed. The `PVK` prefix rules out Codabar, the other common library symbology. Label printing and the scanners bought for it have to agree.

**Fix:** Fix the width, for example `PVK` plus 9 digits. Either state the symbology (Code 128 or Code 39) or add a Deferred row decided before the first label print. Also state that lookup accepts an unpadded or lowercase barcode as typed.

### F8 (low): capability map misses two copy-acting requirements

AD-9 now says every librarian screen that acts on a copy finds it by barcode. The R17 donations row and the R19/R23 lost-marking row act on copies but do not list AD-9.

**Fix:** Add AD-9 to the "Governed by" column of the R17 row and the R19/R23 row.

### F9 (low): the shared barcode lookup has no owner

Barcode lookup is used by catalogue screens, `epic-reservation-loop` (ready, pickup, return), donations and lost marking. AD-19 assigns collections, not shared admin functions, so two epics may each build a lookup.

**Fix:** State that `epic-librarian-catalogue` delivers the barcode assignment hook, the sequence and one barcode lookup in `src/admin` that later epics reuse. Say who builds label printing, probably `epic-initial-load`.

### F10 (low, Deferred): the MARC export row claims more than AD-20 holds

"AD-20 holds the structure an export needs" is not quite true. Holdings export also needs a call number or shelf mark and a location per copy, which AD-20 leaves to the code, and 008 needs a publication year. Deferring these is safe, because only one future unit consumes them. However, the research's other day-one requirement is not stated anywhere: title ids are permanent and never reused.

**Fix:** Reword the row to "AD-20 and `copies.barcode` hold the identities and structure that are costly to add later." Add "title ids are never reused" to AD-9, or to the Ids and dates convention.

### F11 (low): rationale and over-specification

- The Deferred "Research favours Melinda's open SRU" row is rationale, not a decision. Either remove it or turn it into a decision, such as "Melinda SRU unless the lookup story finds it unusable".
- The AD-9 label details (ordering by copy creation, the three printed fields) concern one unit only. The invariant is that each label can be matched to its book and that only published titles' copies get labels. Ordering by creation also breaks after the re-runs F2 allows. Consider moving those details to the story.
- The Copy identity convention says "the database id never appears on a label or screen". Payload admin edit URLs and default columns show ids, so this cannot be enforced as written. **Fix:** "Librarian screens show and search copies by barcode (`useAsTitle: barcode`); the id is never printed or used as the copy's identifier."

## What is sound

- Barcode assignment in a core hook from one sequence, whoever creates the copy, and refusing client values, fits AD-1 and AD-7 and fully prevents divergent numbering.
- The AD-11 and AD-20 split between BCP 47 interface codes and MARC bibliographic codes is consistent and removes a real trap.
- Restricting classification to notations, with no DDC captions, captures a licensing invariant.
- AD-20 as a fixed subset with "other fields optional, added by migration" stays within AD-19 and does not freeze the full data shape.
- AD-10's NFC-on-write already covers Melinda and future MARC input, because it applies to all writes.
