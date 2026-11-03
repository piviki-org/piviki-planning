---
title: 'Technical research: MARC 21 and ILS compatibility'
type: 'technical'
topic: 'MARC21 and ILS compatibility'
decision: 'Which data and interface decisions in the Piviki titles and copies model would make later MARC 21 exchange or migration to an ILS costly, and the cheapest way to avoid each'
source: 'native run'
status: complete
claims_verified: 11
claims_unverified: 11
preset: 'standard'
validation: 'normal'
created: '2026-11-03'
updated: '2026-11-03'
---

# Technical research: MARC 21 and ILS compatibility

**Decision this research serves:** Which data and interface decisions in the Piviki titles and copies model would make later MARC 21 exchange or migration to an ILS costly, and the cheapest way to avoid each

## Executive summary

**Do this:** spend about a day's schema work in `epic-librarian-catalogue` instead of building any MARC code now.
- Give every copy a unique, permanent barcode-style code.
- Store bibliographic data as structured parts: contributors with roles, a split title, publication data, ISBN, a MARC language code, an audience flag.
- Store classification as repeatable (scheme, notation) pairs: YKL plus optional DDC.
- Keep a stable source identifier on any record copied from Melinda.

With that schema, MARC 21 export later is a mapping job with a maintained library (`@natlibfi/marc-record-serializers`), not a re-cataloguing job.

**The three findings that drive this:**
1. **Copy identity is the single costly gap.** Evergreen requires a unique barcode on every copy [5]. Koha matches re-imported items on item number or barcode [3]. Loan migration is keyed on barcodes [16], and migrations without them end in shelf checks [11]. AD-9's `scanId` identifies a scan, not a labelled book. The decision on 3 November 2026 to turn it into a printed, scannable copy barcode closes this gap.
2. **Melinda is the cheapest route to MARC-quality data.** Its open SRU API worked without a login in queries on 3 November 2026, and Melinda holds 828 records with Vietnamese ISBNs (978-604) [49]. Vietnamese children's records there already carry YKL `84.948`, field 008 coded juvenile `vie`, and 041 `vie` [50]. Copying them by ISBN (R12) fills the structured fields for free, provided the schema has somewhere to put them.
3. **Both classification schemes are free to store.** YKL is CC0 [29] with LC source code `ykl` [27]. DDC numbers need no licence; only captions do [30][31]. Vietnamese libraries mostly use DDC 23 (2019 evidence) [32].

**Biggest caveat:** the realistic exchange partner is PIKI, and PIKI's ILS is about to change. Aurora support is ending [40], and the new agreement makes the system a joint procurement [41]. That claim is just past its re-check date (see the staleness map). No vendor is named, so target plain MARC 21 in UTF-8 (ISO 2709/MARCXML), not one system's dialect. Whether an association can contribute records to Finna or Melinda at all is undocumented [46][52].

**Architecture impact (agreed with the user, 3 November 2026):** `scanId` is replaced by `copies.barcode`. Core generates it from one database sequence (prefix plus zero-padded number), it is printed on a label on every book, and librarians scan it for every copy task. The initial load is a one-time operation that creates exactly one copy per title:
1. The scan script classifies each book (computer vision plus an open API or an LLM) and follows the API contract.
2. One import call creates the draft title and its single copy in one transaction and returns the barcode. A re-run skips any book whose title already exists, matching on ISBN first, else normalised title plus author plus volume or series number. Copies carry no bibliographic data, so they are never compared.
3. After librarian review and publishing, labels are printed in scan-batch order with the barcode plus title and author, so each label can be matched to its book. Barcodes of deleted drafts are never reused.

Additional copies are added by hand in the librarian tool, which generates the next barcode the same way. AD-9 needs updating through the architecture skill: the rename, the atomic import call, the title match key, and lookup by barcode in the librarian tool.

## 1. What a MARC 21 record needs from you

**Bottom line:** A MARC record is built from structured parts, not display strings. Each part you don't keep separately now is something someone has to re-key by hand later. Encoding needs only one discipline: store Unicode NFC and compare strings after normalising both sides.

**Version.** MARC 21 Bibliographic is at Update No. 42 (May 2026) [18]. OCLC's validation installed it in July 2026 (medium) [19]. Update 42 "redefines the 041 language code field" (medium) [36], so re-check field 041 before writing the export mapping. The Holdings format is unchanged [36].

**Record header (Leader).** Values a small book catalogue would use [20]:

| Position | Meaning | Value to use |
|---|---|---|
| 06 | type of record | `a` (language material) |
| 07 | bibliographic level | `m` (monograph) |
| 09 | character coding | `a` (Unicode) |
| 17 | encoding level | `7` minimal, `3` abbreviated or `5` partial: honest for volunteer records, and lets an ILS upgrade them later |
| 18 | punctuation | `c` (ISBD punctuation omitted) or blank (non-ISBD) |

The values for positions 17 and 18 are my inference, not a published rule.

**Fields that need structured source data.** The MARC 21 field definitions set out the parts below, but no source found in this research listed which parts a minimal record must have:
- **Names.** Fields 100/700 carry each name with its role, so author, illustrator and translator must be separate people, not one "author" string. To set the first indicator (0 forename, 1 surname, 3 family name), an exporter must know which part of the name is the entry element [34].
  - Vietnamese names put the family name first (Nguyễn Nhật Ánh). Nothing found in this research settles how RDA/LC enter them, so record name parts and the entry element explicitly instead of guessing.
- **Title.** Title proper, subtitle and statement of responsibility are separate subfields (245 $a $b $c).
- **Edition and publication.** Edition (250); place, publisher and year (264).
- **Series** (490).
- **ISBN** (020), with its qualifier ($q), one per format.
- **Language.** Field 041 and positions 35–37 of field 008 use MARC language codes: Vietnamese is `vie` [33], not the BCP 47 `vi` your architecture uses for interface language (AD-11). Keep bibliographic language as its own field.
- **Audience.** Position 22 of field 008 is the target audience (`j` = juvenile).

**Holdings (copies).** Copy data travels in two ways:
- **Standard MARC 21 Holdings (852)** [23]:
  - $a location, $h classification part, $i item part, $j shelving control number
  - $p piece designation: "a bar code number or an accession number"
  - $t copy number
  - first indicator 7 + $2 for the shelving scheme
- **Koha's embedded 952** [3]: the call number lives on the item ($o), not the bib.

A YKL-shelved copy would export roughly as `852 7_ $h 84.2 $i NGU $p <barcode> $2 ykl`. That example is my reading of the field definitions.

**Encoding (Vietnamese).**
- A Unicode record must have position 09 of the record header (Leader) set to `a`. Combining diacritics come *after* the base letter, and the spec orders stacked marks (ế, ặ) explicitly. The specification doesn't require either NFC or NFD [21].
- Koha stores NFC but "all exports (MARC utf8, MARCXML) are performed using NFD normalization" (open bug, March 2026) [22]. A record round-tripped through Koha comes back decomposed, so any matching on title or author must normalise both sides.
- Your AD-10 NFC-on-write plus a normalised search column already covers this. Apply the same normalisation when importing MARC.
- Avoid MARC-8 entirely: converting from it yields decomposed text and mixed forms (medium) [37].

**Finnish practice.**
- The Finnish MARC 21 application is published on the National Library of Finland's site [25][26].
- Subjects are coded `650 #7 ‡a <term> ‡2 yso/fin ‡0 <YSO URI>`. One concept URI carries both the Finnish and the Swedish label [26].
- No Finnish minimal-level definition was found in this research.

*Research stopped after the maximum two rounds, with core questions answered. Still open: Melinda minimum cataloguing level, Vietnamese name entry under RDA, 041 after Update 42.*

## 2. Classification: YKL and DDC

**Bottom line:** Both schemes are free to *store*. Model classification as repeatable (scheme, notation, edition) pairs on the title, and keep the shelf mark on the copy.

**YKL.**
- Published on Finto by the National Library, authored by Helsinki City Library and the YKL working group, licensed CC0 1.0, last modified January 2026 [29].
- MARC carries it in field 084 with `‡2 ykl`, for example, `084 ## ‡a 37.8 ‡2 ykl` [25]. `ykl` is the registered LC source code, formerly `fplcs` (list dated September 2026) [27].
- Finland adds a local first indicator `9` for the YKL fiction subclass [25].
- Libraries put children's fiction under 84.2 (medium, practice evidence) [35].

**DDC.**
- Storing DDC numbers needs no licence. OCLC's WebDewey terms let records "display DDC numbers, but shall not display DDC captions" [30], and the Deutsche Nationalbibliothek states "In principle, no license is required to classify media using DDC numbers" [31].
- So show DDC captions or schedules to patrons only under a licence.
- MARC 082 carries the number. The first indicator distinguishes full (0) and abridged (1) editions; $2 takes the form edition/language/date, for example, `23/eng/20190402` [28]. A Vietnamese-edition code would plausibly be `23/vie`, but that is unconfirmed.
- Vietnam's National Library translated DDC 23 (2013), and a 2019 review reported "most libraries have used DDC 23" (medium-high, 2019) [32]. Vietnamese source records therefore plausibly already carry DDC numbers worth keeping.

**Shelf mark.** In Finnish public libraries the shelf mark is the YKL class plus the *pääsana* (headword). The pääsana is the main-entry author's surname, or else the title, and its first three letters go on the spine label (medium, one library's 2015 guide) [35]. That makes it derived data.

*Research stopped after the maximum two rounds, with coverage.*

## 3. Open ILS data models, migration and copy identity

**Bottom line:** All three open ILSs agree with your model on structure. The holds and the four copy states map cleanly. The costly gap is copy identity: every import, re-import and loan migration path keys physical copies on a unique barcode, and the plan has none.

**Structure.**
- Koha splits a record into the bib (MARC) and its items. Items travel in MARC 952 ($a owning library, $b holding library, $o call number, $p barcode, $y item type, $8 collection, $d date acquired, $e source of acquisition, which the schema describes as "where the item was purchased") [1].
- Evergreen goes bib, then call number (volume), then copy. Every copy must hang off a call number [5].
- FOLIO goes instance, then holdings, then item [8]. Its data import matches items on a barcode carried in a 9xx field, and holdings on HRID or UUID (medium) [10].
- A titles-and-copies model fits all three. Evergreen and FOLIO need a middle "holdings / call number" level that an importer can synthesise from a call number and location on each copy [5][7][8].

**Copy identity.**

| System | Barcode rule | Fallback for missing barcodes |
|---|---|---|
| Evergreen | `barcode TEXT NOT NULL`, unique among non-deleted copies [5] | Batch import can generate barcodes with a prefix; an `@@` barcode is rewritten to `@@` plus the copy id [7] |
| Koha | `VARCHAR(20)`, nullable, unique when present [1] | None found |
| FOLIO | Optional in the schema but treated as unique in practice [8][17] | A recognised clean-up cost: libraries add suffixes to make barcodes unique, or move duplicates into a "former identifier" field [17] |

- Koha's staged import can replace existing items on re-import, and it reports items it skipped as duplicate barcodes [3].
- The Missouri Sierra-to-FOLIO post-mortem found barcode errors on its first item loads, fixed only by finding the books on the shelf. Some items never migrated [11].
- No source quantified the cost of retro-barcoding. That cost is inferred, not evidenced.

**Barcode practice.** Libraries mostly use Codabar or Code 39. A common layout is 14 digits: a type digit (3 = item), a library code, a sequence number and a check digit. There is no universal standard, so match the target ILS before printing labels (medium) [14]. Koha's auto-barcode feature generates the next number from the highest barcode already stored, so imported barcodes in mixed formats confuse it (medium) [15].

**Status.** Each system's statuses can represent your four states:
- **Koha** keeps lost, damaged, withdrawn and not-for-loan as separate flags. Lost, damaged and withdrawn each record the date they were set. "On loan" is derived from the due date on the item, and "held" lives on the hold, not the item [1].
- **Evergreen** has one status column. Available, Checked out, On holds shelf and Lost all exist [6].
- **FOLIO** has a fixed list of 21 statuses that includes Available, Checked out, Awaiting pickup and Declared lost [9].

The mapping is lossless as long as "lost" carries a date and "on loan" can be derived from the active loan.

**Holds and queues.** Koha's `reserves` row matches your design closely [2]:
- the title is required, and the copy is null until the hold is filled
- holds are title-level by default
- `priority` is the queue position
- there are separate dates for waiting since and expiry
- `found` is null while pending and `W` when waiting on the shelf (codes medium confidence)

A per-title queue with the copy assigned at `ready` is the native Koha semantics.

**Loan history and anonymisation.** Koha anonymises old checkouts by swapping the patron for a designated `AnonymousPatron`, which keeps per-copy loan counts [4]. Your plan to anonymise after the retention period, keeping the copy and dropping the patron, maps directly onto this.

**What a migration needs from every copy.** A vendor guide (medium) lists [12]:
- barcode
- library or branch
- shelving location
- item type
- call number
- copy number
- price
- circulation status
- lost/damaged/withdrawn status

Its named pitfalls are exports without barcodes, branches or item types; free-text values that must be mapped to codes; and one-row-per-copy exports that turn into duplicate bibs. The Missouri post-mortem adds that location codes had to match exactly, and that identifiers missing their source prefix (035 without "(OCoLC)") are hard to repair later [11].

**Patrons, loans and holds.** Koha's patron import needs a `cardnumber` and a `surname`, and carries legacy ids as `CODE:value` patron attributes [4]. Moving active loans has no built-in Koha importer. Practitioners write scripts that look up item barcode → item and cardnumber → patron, or replay an offline-circulation file keyed by barcode. That evidence is a 2011 thread and is low confidence; current tooling was not checked [16].

**Donors.** MARC 541 (Source of Acquisition) has a privacy indicator (0 = private, 1 = not private), with $a source, $c method (for example, gift), $d date and $e accession number [13]. Koha's 952$e is a text field its schema describes as "where the item was purchased", so a donor fits it only loosely [1]. Whether Koha hides it from the public catalogue (OPAC) by default was not checked.

*Research stopped after the maximum two rounds, with coverage. Still open: Koha OPAC visibility of 952$e, current loan-import tooling, Finnish barcode conventions.*

## 4. The Finnish ecosystem

**Bottom line:** Finland gives you free, open, CC0-friendly *inputs*: Melinda records over SRU, YKL, and Fennica. Its *outputs* (Finna, Melinda membership) are designed for municipal libraries running a real ILS. The realistic exchange partner is a library consortium such as PIKI, not the national services directly.

**Which ILS PIKI runs.**
- PIKI ran Axiell Aurora: a 2019 City of Tampere agreement (about €175k/year) and a 2017 Helsinki decision both name it (medium) [38][39].
- Axiell is ending Aurora support, and Aurora libraries are moving, for example, to Axiell Quria in 2027 (medium, October 2026) [40].
- PIKI's updated cooperation agreement (effective around September 2026) makes the library system a binding joint procurement signed by Tampere, with no vendor named [41].
- **PIKI is therefore likely to change systems soon, to an ILS not yet known.** Don't build against any one vendor's format; target plain MARC 21.
- Koha-Suomi Oy is owned by more than 100 municipalities [42], but no current figure for the share of Finnish libraries on Koha was found.
- Helsinki's Helmet runs Sierra and is procuring a replacement (medium) [48].

**Melinda (national union catalogue).**
- Melinda's Open Data SRU and OAI-PMH endpoints are open (wiki updated May 2025). Some other instances are IP-restricted to Melinda libraries [43]. Live queries on 3 November 2026 worked without a login [49]. SRU and Z39.50 are free and intended for single records or small sets (medium) [44].
- Joining Melinda is free from the National Library's side, but requires RDA plus MARC 21, matching subjects to YSO, and an ILS that can transfer records out [46]. Eligibility for a non-library association isn't stated.
- In the Koha-Suomi pattern, the National Library harvests records back from the ILS's OAI-PMH interface (medium, 2020) [47].
- Fennica, the national bibliography, is CC0 [45]. No licence statement covering Melinda as a whole was found.

**Live coverage test (3 November 2026).**
- Melinda has **828 records with ISBN prefix 978-604** (Vietnam) [49] (re-run and confirmed). The prefix covers every Vietnamese publisher, so only some of these are children's books. One sample was conference proceedings.
- A query for one popular Vietnamese children's author returned 101 records, carrying [50]:
  - field 008 coded juvenile and `vie`
  - field 041 `$a vie`
  - YKL `084 ‡a 84.948 ‡2 ykl`, the YKL class for Vietnamese-language fiction

  Children's fiction also appears under 85.1948 / 85.2948.
- In the sampled records, Vietnamese text is stored **decomposed (NFD)**, and some records have lost diacritics partly or wholly [50].
- Finna shows 4,397 Vietnamese-language records nationally, **335 held by PIKI-kirjastot** [51].
- So copy cataloguing from Melinda by ISBN is realistic for part of the collection. Every imported record must be normalised to NFC and matched in a way that ignores diacritics.

**Finna (national discovery portal).**
- Funding from the Ministry of Education and Culture covers public, university and polytechnic libraries, certain archives and qualifying museums. Others "may in future join" at list price. No category fits an association [52].
- Joining needs your own back-end system with an OAI-PMH interface serving MARC 21, metadata published CC0, and about six months [52].
- Finna itself serves MARCXML over OAI-PMH at `api.finna.fi/OAI/Server` [53].

**Finto vocabularies.** YSO (General Finnish Ontology) is CC BY 4.0, not CC0. Its labels are in Finnish, Swedish, English and Northern Sami, with no Vietnamese. It's available through the Skosmos REST API, including as MARCXML [54].

**Vietnam.** Vietnam has a national MARC 21 standard, TCVN 7539:2005 (medium) [59]. No public Z39.50/SRU service at the National Library of Vietnam was found.

*Research stopped after the maximum two rounds. Still open: PIKI's next ILS and its timing, Koha share, Helmet outcome, Melinda licence as a whole, whether an association can join Melinda through a member.*

## 5. Interfaces and tooling

**Bottom line:** You don't need to build MARC export now. A well-structured schema can be serialised later with a maintained TypeScript library from the National Library of Finland.

**What ILSs import.** Koha takes ISO 2709 (.mrc) or MARCXML, UTF-8 by default [3]. Evergreen takes .mrc or MARCXML, in batches of up to about 5,000 records [7]. Neither documents MARC-in-JSON. FOLIO's import formats are unverified in this research (its documentation page returned 404). So **ISO 2709 or MARCXML in UTF-8** is the target.

**Node and TypeScript libraries.**

| Library | Maintainer | Licence | Latest | Formats | Activity and caveats |
|---|---|---|---|---|---|
| `@natlibfi/marc-record` | National Library of Finland | MIT | 11.0.3, August 2026 | Record model | Written in TypeScript and ships types; 54 commits since 2024 [57] |
| `@natlibfi/marc-record-serializers` | National Library of Finland | MIT | 11.0.5, September 2026 | Text, own JSON shape, Aleph Sequential, ISO 2709 (UTF-8, byte-length correct), MARCXML, OAI MARCXML | Monthly releases through 2026; no declared types [56][57] |
| `marcjs` | Individual maintainer | MIT | 3.0.3, September 2026 | Reads and writes ISO 2709, MARCXML and LoC MARC-in-JSON; UTF-8 only | Sporadic; open issue says its parsers "silently accept broken data" [58] |

For reading Melinda records and writing exports, `@natlibfi/*` is the stronger choice. Its maintainer runs Melinda.

**Interfaces to expose later.** To be harvestable, by Finna or by a consortium's ILS, the system would serve OAI-PMH with a `marc21` (MARCXML) metadata format and stable record identifiers [52][53]. Nothing here needs to exist in the MVP. What must exist from day one is a **stable, never-reused identifier per title and per copy**, because harvests and re-imports key on it [3][53].

*Research stopped after the maximum two rounds, with coverage. Still open: FOLIO import formats.*

## Cross-dimension insights

- **ISBN lookup is not just a nice-to-have.** R12 sits in `epic-loan-care` as cuttable. Combining dimensions 1 and 4 shows it is also the cheapest way to get records an ILS will accept: Melinda supplies the YKL class, language, audience and structured name and title fields [50]. But this only pays if `titles` can hold those parts. The schema decision is urgent; the lookup feature can wait.
- **One name decision serves two needs.** The Finnish shelf mark uses the first letters of the main-entry name [35]. To set the first indicator of MARC field 100, an exporter must know whether the entry element is a surname, forename or family name [34]. Vietnamese names are family-name first, so recording each contributor's entry element serves both the spine label and the MARC export.
- **Decomposed Unicode arrives from both directions.** Koha exports NFD [22], and Melinda records are NFD, with some diacritics lost [50]. AD-10's NFC-on-write must therefore cover every way data comes in (scan script, Melinda import, future MARC import), not only librarian edits. Matching on title or author must ignore diacritics.

## Recommendations

Ranked by how costly the decision is to reverse later. Unless noted, each feeds the `epic-librarian-catalogue` stories, which create `titles` and `copies`.

| # | Costly decision to avoid | Cheapest fix now | Confidence |
|---|---|---|---|
| 1 | Copies identified only by the database id or a scan id | **`copies.barcode`** (replaces `scanId`): generated by core from one sequence, unique, permanent, never reused, at most 20 characters, plain ASCII digits with a prefix (for example, `PVK000123`). Printed on a label on every book and scanned for every copy task. Import re-runs match on the title (ISBN, else title plus author plus volume), never on copies, so **update AD-9**. Also keep title ids permanent and never reused [1][3][5][53] | High (schemas, two systems agree) |
| 2 | One free-text author string; title as one string | `contributors` as an ordered list of {name as written, entry element (surname / forename / family-name-first), role code from the MARC relator list: `aut`, `ill`, `trl`, …}. Title split into title, subtitle and statement of responsibility [34] | Medium (field definitions; no minimal-level source) |
| 3 | No edition, publication or physical-description fields | Optional `edition`, `publisher`, `place`, `year` (integer), `pages`, `series` + number; repeatable `isbns` with a qualifier (paperback, hardback) [49] | Medium |
| 4 | Using BCP 47 `vi` as the bibliographic language | Separate `language` field (MARC/ISO 639-2/B: `vie`), repeatable for bilingual books, plus `audience = juvenile` on titles. Keep BCP 47 for interface language (AD-11) [33] | High for the code; re-check 041 after Update 42 [36] |
| 5 | One classification column, or no scheme recorded | Repeatable `classifications` of {scheme: `ykl` / `ddc`, notation, edition such as `23`} on titles. A `callNumber` (shelf mark) on the copy, computed from YKL plus the first three letters of the entry element and editable [25][27][28][35] | High |
| 6 | Copy-catalogued records losing where they came from | Store `sourceRecordId` with its source prefix (for example, Melinda's id with its control-number source), never a bare number [11][49] | Medium (one post-mortem) |
| 7 | Copies without a location or item type | `location` and `itemType` codes on each copy, even if every copy is `PIVIKI` / `BOOK` today. An importer needs exact codes [11][12] | Medium |
| 8 | Status that loses facts | Keep `lost` with a `lostAt` date. Derive "on loan" from the active loan. Leave room for `damaged` / `withdrawn` as dated flags rather than new enum values. The plan is already close [1][6][9] | High |
| 9 | Text not normalised on import | Extend AD-10's NFC normalisation to every way data comes in, and match on the normalised column [22][50] | High |
| 10 | Donor exposed in an export | When exporting, use MARC 541 with first indicator `0` (private), or leave the donor out. The current design is fine [13] | High |
| 11 | Subjects as free text | If subjects are added later, store {vocabulary, concept URI, label, language} (YSO is CC BY 4.0, so attribution is required) [26][54] | High |
| — | Building MARC export or OAI-PMH now | Don't. Use `@natlibfi/marc-record` + `-serializers` (MIT, TypeScript) when a partner asks, and target ISO 2709 or MARCXML in UTF-8, not one vendor's dialect. Feeds a later epic [56][57] | Medium |

**Labels:** print `copies.barcode` as Code 39 or Codabar, the symbologies most library scanners read, after checking what the receiving ILS expects [14]. Print after review and publishing, in scan-batch order, with the title and author on each label.

**For `epic-loan-care` (R12):** use Melinda SRU (`sru.api.melinda.kansalliskirjasto.fi/bib`, query `bath.isbn=`) as the first ISBN lookup source. It is free, open and already includes YKL [43][49].

## Open questions

- **Which ILS PIKI is moving to, and when?** Ask PIKI or the Tampere city library directly; that's quicker than more desk research.
- **Can an association contribute records to Finna or Melinda, or deposit books through PIKI?** Ask finna-posti and PIKI. The answer decides whether exporting to Finland ever happens.
- **What licence covers Melinda records as a whole?** Only Fennica is confirmed CC0 [45]. Settle this before showing copied records publicly; ask the National Library.
- **How do RDA and LC enter Vietnamese names?** This affects recommendation 2's default. Check the LC-PCC policy statements or VIAF.
- **What did Update 42 change in field 041, and is there a code for the Vietnamese DDC edition (`23/vie`)?** Re-read loc.gov bd041 and bd082 when building the export.
- **What share of the 828 Melinda records with 978-604 ISBNs are children's books?** This sets realistic copy-cataloguing coverage. Run a short query with a juvenile filter, or test the actual 300 ISBNs.
- **Remaining smaller gaps:** Finnish barcode conventions, the current share of Finnish libraries on Koha, the Helmet procurement outcome, FOLIO import formats, whether Koha shows 952$e in the public catalogue, current Koha loan-import tooling, and the Finnish minimal cataloguing level.

## Source appendix

Numbers 24 and 55 were not assigned.

| # | Supports | Publisher | Pub date | Accessed | Confidence |
|---|---|---|---|---|---|
| [1] | Koha items/952 fields, unique barcode, separate status flags | [Koha community schema](https://schema.koha-community.org/main/tables/items.html) | live | 2026-11-03 | high |
| [2] | Koha holds title-level, priority, found codes | [Koha community schema](https://schema.koha-community.org/main/tables/reserves.html) | live | 2026-11-03 | high |
| [3] | Koha staged import, formats, embedded items, matching; 952 call number on item | [Koha Manual](https://koha-community.org/manual/latest/en/html/cataloging.html) | 2026-11 | 2026-11-03 | high |
| [4] | Koha patron import, anonymisation | [Koha Manual, tools](https://koha-community.org/manual/latest/en/html/tools.html) | 2026-11 | 2026-11-03 | high |
| [5] | Evergreen copy barcode NOT NULL unique, bib→call number→copy | [Evergreen source](https://raw.githubusercontent.com/evergreen-library-system/Evergreen/main/Open-ILS/src/sql/Pg/040.schema.asset.sql) | live | 2026-11-03 | high |
| [6] | Evergreen copy statuses | [Evergreen source](https://raw.githubusercontent.com/evergreen-library-system/Evergreen/main/Open-ILS/src/sql/Pg/950.data.seed-values.sql) | live | 2026-11-03 | high |
| [7] | Evergreen Vandelay import, holdings profiles, generated barcodes, formats | [Evergreen docs](https://docs.evergreen-ils.org/docs/latest/cataloging/batch_importing_MARC.html) | unknown | 2026-11-03 | high |
| [8] | FOLIO item schema, barcode optional, accession number/HRID | [FOLIO mod-inventory-storage](https://github.com/folio-org/mod-inventory-storage/blob/master/ramls/schemas/item-storage/item.json) | 2026-10 | 2026-11-03 | high |
| [9] | FOLIO item status enum | [FOLIO mod-inventory-storage](https://github.com/folio-org/mod-inventory-storage/blob/master/ramls/schemas/item-storage/itemStatus.json) | 2026-10 | 2026-11-03 | high |
| [10] | FOLIO data import matching | [FOLIO wiki](https://folio-org.atlassian.net/wiki/spaces/FOLIOtips/pages/5672633/Data+Import+Functionality+Implementation+Table) | unknown | 2026-11-03 | medium |
| [11] | Sierra→FOLIO post-mortem: barcodes, locations, 035 prefix | [Atla Theology Cataloging Bulletin](https://serials.atla.com/tcb/article/view/3294/4493) | 2023-10 | 2026-11-03 | high |
| [12] | Migration item fields and pitfalls | [kohasupport.com](https://kohasupport.com/knowledge-base/bibliographic-vs-item-records-koha-migration/) | 2026-05 | 2026-11-03 | medium |
| [13] | MARC 541 privacy indicator | [Library of Congress](https://www.loc.gov/marc/bibliographic/bd541.html) | live | 2026-11-03 | high |
| [14] | Library barcode symbologies and layout | [SirsiDynix help (IU)](https://sirsi.sitehost.iu.edu/Helps/Symphony/Workflows/English/Content/Topics/09-ConfigPolicies/FAQs/FAQ_Barcode_Specs.htm) | unknown | 2026-11-03 | medium |
| [15] | Koha autoBarcode increments highest barcode | [Koha Bugzilla](https://bugs.koha-community.org/bugzilla3/show_bug.cgi?id=29584) | 2021-12 | 2026-11-03 | medium |
| [16] | Loan migration keyed on barcode and cardnumber | [koha-devel list](https://lists.koha-community.org/pipermail/koha-devel/2011-August/036048.html) | 2011-08 | 2026-11-03 | low |
| [17] | FOLIO duplicate-barcode clean-up | [Oklahoma State University Library](https://info.library.okstate.edu/foliocataloging/duplicate_barcodes) | 2026-09 | 2026-11-03 | medium |
| [18] | MARC 21 Bibliographic Update 42 | [Library of Congress](https://www.loc.gov/marc/bibliographic/) | 2026-05 | 2026-11-03 | high |
| [19] | OCLC validation installed Update 42 | [DCRM-L list](https://listserver.lib.byu.edu/pipermail/dcrm-l/2026-July/007538.html) | 2026-07 | 2026-11-03 | medium |
| [20] | Leader codes | [Library of Congress](https://www.loc.gov/marc/bibliographic/bdleader.html) | 2016-11 | 2026-11-03 | high |
| [21] | MARC Unicode: Leader/09, combining-mark order | [Library of Congress](https://www.loc.gov/marc/specifications/speccharucs.html) | unknown | 2026-11-03 | high |
| [22] | Koha stores NFC, exports NFD | [Koha bugs list, bug 42157](https://lists.koha-community.org/archives/list/koha-bugs@lists.koha-community.org/thread/5RHEQ5R3FLCPQXP5NXFWOSCRAHWEAYQF/) | 2026-03 | 2026-11-03 | high |
| [23] | Holdings 852 subfields | [Library of Congress](https://www.loc.gov/marc/holdings/hd852.html) | 2021-06 | 2026-11-03 | high |
| [25] | Finnish 084 YKL practice, indicator 9 | [National Library of Finland](https://marc21.kansalliskirjasto.fi/bib/05X-08X.htm) | 2026-05 | 2026-11-03 | high |
| [26] | Finnish 6XX YSO with URI | [National Library of Finland](https://marc21.kansalliskirjasto.fi/bib/6XX.htm) | 2025-09 | 2026-11-03 | high |
| [27] | `ykl` LC classification source code | [Library of Congress](https://www.loc.gov/standards/sourcelist/classification.html) | 2026-09 | 2026-11-03 | high |
| [28] | 082 indicators and $2 edition form | [Library of Congress](https://www.loc.gov/marc/bibliographic/bd082.html) | 2024-06 | 2026-11-03 | high |
| [29] | YKL on Finto, CC0 | [Finto / National Library of Finland](https://finto.fi/ykl/fi/) | 2026-01 | 2026-11-03 | high |
| [30] | DDC numbers displayable without captions | [OCLC WebDewey Schedule 8](https://www.oclc.org/content/dam/legal/schedules/en_us/Schedule-08-WebDewey-EN-US.pdf) | unknown | 2026-11-03 | medium |
| [31] | No licence needed for DDC numbers | [Deutsche Nationalbibliothek](https://www.dnb.de/EN/Professionell/DDC-Deutsch/_content/ddcDeutschDDCkostenfrei_akk.html) | unknown | 2026-11-03 | high |
| [32] | Vietnamese DDC 23, most libraries use it | [National Library of Vietnam](https://nlv.gov.vn/ef/summary-conference-on-application-of-dewey-decimal-classification-vietnamese-edition-23-in-viet-nams-libraries-2014-2019.html) | 2019-11 | 2026-11-03 | medium |
| [33] | MARC code `vie` | [Library of Congress](https://www.loc.gov/marc/languages/language_code.html) | 2026-01 | 2026-11-03 | high |
| [34] | X00 name indicators | [Library of Congress](https://www.loc.gov/marc/bibliographic/bdx00.html) | unknown | 2026-11-03 | medium |
| [35] | Finnish shelf mark (pääsana) practice; 84.2 children's fiction | [Hyvinkää city library](https://www.hyvinkaa.fi/globalassets/kulttuuri-ja-vapaa-aika/kirjasto/pdf-tiedostot/hyllytys--ja-aakkostusohjeet.pdf) | 2015-03 | 2026-11-03 | medium |
| [36] | Update 42 changes (041; Holdings unchanged) | [Library of Congress](https://loc.gov/marc/marc21_update42_online.html) | 2026-05 | 2026-11-03 | medium |
| [37] | MARC-8 conversion yields decomposed text | [koha-devel list](https://lists.koha-community.org/pipermail/koha-devel/2008-April/030175.html) | 2008-04 | 2026-11-03 | medium |
| [38] | Tampere Aurora agreement | [City of Tampere](https://tampere.cloudnc.fi/fi-FI/Viranhaltijat/Tietohallintojohtaja/Axiell_Aurora_kirjastojarjestelmaa_koske(98681)) | 2019 | 2026-11-03 | medium |
| [39] | Piki-kirjastot/Aurora | [City of Helsinki](https://paatokset.hel.fi/fi/asia/hel-2017-007484) | 2017 | 2026-11-03 | medium |
| [40] | Aurora support ending | [Soisalon Seutu](https://www.soisalonseutu.fi/paikalliset/9849769) | 2026-10 | 2026-11-03 | medium |
| [41] | New PIKI agreement, joint system procurement | [City of Tampere](https://www.tampere.fi/en/current/2026/05/22/cooperation-between-tampere-and-other-piki-library-consortia) | 2026-05 | 2026-11-03 | high |
| [42] | Koha-Suomi Oy ownership | [Koha-Suomi Oy](https://koha-suomi.fi/) | 2026 | 2026-11-03 | high |
| [43] | Melinda open SRU/OAI-PMH/Z39.50 | [Kiwi / National Library of Finland](https://www.kiwi.fi/spaces/melinda/pages/166101210/Hakurajapinnat+Search+harvesting+APIs) | 2025-05 | 2026-11-03 | high |
| [44] | Copy-cataloguing interfaces free | [National Library of Finland](https://www.kansalliskirjasto.fi/en/news/national-library-finland-makes-its-copy-cataloguing-interfaces-freely-available) | unknown | 2026-11-03 | medium |
| [45] | Fennica CC0 | [Kiwi data catalogue](https://www.kiwi.fi/spaces/Datacatalog/pages/86972056/Fennica) | 2026-07 | 2026-11-03 | high |
| [46] | Joining Melinda: free, requirements | [Kiwi / Melinda](https://www.kiwi.fi/display/melinda/Yleisten+kirjastojen+liittyminen+Melindaan) | 2024-07 | 2026-11-03 | high |
| [47] | Koha-Suomi ↔ Melinda via OAI-PMH | [Kiwi attachment](https://www.kiwi.fi/download/attachments/50168589/Liite%206%20Melinda%20rajapinnat%20Koha-Suomi-TaTi_v1.0.pdf) | 2020-01 | 2026-11-03 | medium |
| [48] | Helmet Sierra and procurement | [City of Helsinki](https://paatokset.hel.fi/fi/asia/hel-2025-016752) | 2025 | 2026-11-03 | medium |
| [49] | Live: 828 Melinda records with 978-604 | [Melinda SRU](https://sru.api.melinda.kansalliskirjasto.fi/bib?version=1.1&operation=searchRetrieve&maximumRecords=3&query=bath.isbn%3D978604%2A) | live | 2026-11-03 | high |
| [50] | Live: Vietnamese children's records with ykl, vie; NFD | [Melinda SRU](https://sru.api.melinda.kansalliskirjasto.fi/bib?version=1.1&operation=searchRetrieve&maximumRecords=3&query=dc.creator%3D%22Nguy%E1%BB%85n%20Nh%E1%BA%ADt%20%C3%81nh%22) | live | 2026-11-03 | high |
| [51] | Live: Finna vie records, PIKI 335 | [Finna API](https://api.finna.fi/v1/search?lookfor=&filter%5B%5D=language%3Avie&limit=0&facet%5B%5D=building) | live | 2026-11-03 | high |
| [52] | Finna membership terms and requirements | [Kiwi / Finna](https://www.kiwi.fi/spaces/Finna/pages/442433934/Ketk%C3%A4+voivat+liitty%C3%A4+mukaan) | 2025-10 | 2026-11-03 | high |
| [53] | Finna OAI-PMH, marc21 prefix | [Kiwi / Finna](https://www.kiwi.fi/display/Finna/OAI-PMH+-haravointirajapinta+Finnan+indeksiin) | 2025-04 | 2026-11-03 | high |
| [54] | YSO CC BY 4.0, languages, REST API | [Finto](https://finto.fi/yso/en/) | 2026-11 | 2026-11-03 | high |
| [56] | NatLibFi serializers formats, activity | [GitHub NatLibFi](https://github.com/NatLibFi/marc-record-serializers) | 2026-09 | 2026-11-03 | high |
| [57] | @natlibfi/marc-record typed, MIT, releases | [npm registry](https://registry.npmjs.org/@natlibfi%2fmarc-record) | 2026-08 | 2026-11-03 | high |
| [58] | marcjs formats, UTF-8 only, activity | [GitHub fredericd/marcjs](https://github.com/fredericd/marcjs) | 2026-09 | 2026-11-03 | high |
| [59] | Vietnam MARC 21 standard TCVN 7539:2005 | [Thư viện Pháp luật](https://thuvienphapluat.vn/TCVN/Linh-vuc-khac/TCVN-7539-2005-Thong-tin-va-tu-lieu-Kho-mau-MARC-21-cho-du-lieu-thu-muc-907220.aspx) | 2005 | 2026-11-03 | medium |

## Staleness map

Computed by `recon_kit.py staleness`. Freshness windows in months: version 1, tooling/ecosystem/access/live test 6, standard/licence 12, pattern/practice 24.

| Claim | Class | Pub | Re-check by | Stale now |
|---|---|---|---|---|
| [22] Koha NFC/NFD export bug | version | 2026-03 | 2026-04-01 | yes |
| [18][36] MARC Update 42; 041 change | version | 2026-05 | 2026-06-01 | yes |
| [3][5] Koha import / Evergreen barcode rules | version | 2026-11 | 2026-12-01 | no |
| [41] PIKI joint system procurement | ecosystem | 2026-05 | 2026-11-01 | yes |
| [38] PIKI on Aurora | ecosystem | 2019 | 2019-07-01 | yes |
| [40] Aurora end of support | ecosystem | 2026-10 | 2027-04-01 | no |
| [43] Melinda interfaces open | access | 2025-05 | 2025-11-01 | yes by date; live test on 2026-11-03 shows they still work |
| [52] Finna membership terms | access | 2025-10 | 2026-04-01 | yes |
| [56] NatLibFi serializers | tooling | 2026-09 | 2027-03-01 | no |
| [49][50][51] Live Melinda/Finna counts | live-test | 2026-11 | 2027-05-01 | no |
| [27][33] `ykl`, `vie` codes | standard | 2026 | 2027-01 / 2027-09 | no |
| [29][30] YKL CC0; DDC numbers free | licence | 2026 | 2027-01 / 2027-11 | no |
| [32] Vietnam uses DDC 23 | practice | 2019-11 | 2021-11-01 | yes |
| [11] FOLIO post-mortem | pattern | 2023-10 | 2025-10-01 | yes |
| [2] Koha title-level holds | pattern | 2026-11 | 2028-11-01 | no |

Nine claims are already past their window. The claim with the earliest re-check date (2019-07-01), "PIKI is on Aurora", is the one to refresh first: ask PIKI which system replaces Aurora. The Koha NFD export bug and the 041 change should be re-read when MARC export is actually built.
