# Digest: dimension 3, round 1
## Findings
- claim: Koha `items` table maps to MARC21 952 subfields. Confirmed mappings: $a homebranch (owning library, FK branches), $b holdingbranch (current location, FK branches), $o itemcallnumber (sorted via cn_sort), $p barcode, $y itype (FK itemtypes), $8 ccode (collection code, authorized value), $7 notforloan, $1 itemlost, $4 damaged, $0 withdrawn, $d dateaccessioned ("date acquired or added to Koha"), $e booksellerid ("source of acquisition", LONGTEXT, i.e. free text).
  source: https://schema.koha-community.org/main/tables/items.html
  publisher: Koha Community (SchemaSpy output of main branch)
  pub_date: unknown (live, main branch)
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 1, 5
- claim: Koha does not use one enumerated status. It uses several independent flags: itemlost, damaged, withdrawn and notforloan, each an integer authorized value (library-defined list) with a timestamp (itemlost_on, damaged_on, withdrawn_on). On-loan is derived: `items.onloan` holds the due date, or NULL if not checked out. So a single-enum state (available/held/on_loan/lost) maps to Koha in a lossy way. "lost" maps to itemlost=<code>, "on_loan" has to be a real checkout row (it cannot be set as a flag), and "held" is the reserves row's found status, not an item field.
  source: https://schema.koha-community.org/main/tables/items.html
  publisher: Koha Community
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 4
- claim: Koha barcode is VARCHAR(20), nullable, with a unique index (each non-null barcode must be unique). Barcodes over 20 characters will not fit.
  source: https://schema.koha-community.org/main/tables/items.html
  publisher: Koha Community
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 2
- claim: Koha allows items with no barcode. In a 2014 bug discussion, libraries arrived at migration with no barcodes and added them at first checkout. A proposal to make barcodes mandatory (Bug 12348) and a report that the default 952$p is not mandatory (Bug 12256) existed. Whether either was resolved was not verified this run.
  source: https://lists.koha-community.org/archives/list/koha-bugs@lists.koha-community.org/thread/7KUPT7NGVYYJ2IJWOWT6TKMIAYQZ7BQV/
  publisher: Koha bugs mailing list (via search summary, thread not fetched)
  pub_date: 2014-06
  accessed: 2026-11-03
  confidence: low
  class: practice
  question: 2
- claim: Koha holds (`reserves`) are title-anchored. biblionumber is required. itemnumber is nullable and holds either the specifically requested item or the item that filled the hold. `item_level_hold` flag defaults to 0 (title-level). `priority` gives queue position (default 1). `found` is a one-letter status set once the hold is confirmed. `waitingdate` is when the item was put on the pickup shelf, and `expirationdate` is also stored. This matches a per-title queue with a copy assigned at "ready". Values of `found` (W/T/P) were not verified this run.
  source: https://schema.koha-community.org/main/tables/reserves.html
  publisher: Koha Community
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high (structure); low (found codes)
  class: schema
  question: 4
- claim: Evergreen `asset.copy` requires `barcode TEXT NOT NULL`, with a unique index among non-deleted copies (`copy_barcode_key ... WHERE deleted = FALSE`). Each copy must hang off a call_number (NOT NULL), so the hierarchy is bib -> call number (volume) -> copy. A copy also needs circ_lib, location, status, loan_duration and fine_level. Evergreen has a placeholder mechanism: a barcode starting '@@' is rewritten by trigger to '@@'||id. This is a first-class pattern for copies with no physical barcode.
  source: https://raw.githubusercontent.com/evergreen-library-system/Evergreen/main/Open-ILS/src/sql/Pg/040.schema.asset.sql
  publisher: Evergreen project (source code, main branch)
  pub_date: unknown (main branch, fetched 2026-11-03)
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 1, 2
- claim: Evergreen copy status is a single FK to config.copy_status. Seed values: 0 Available, 1 Checked out, 2 Bindery, 3 Lost, 4 Missing, 5 In process, 6 In transit, 7 Reshelving, 8 On holds shelf, 9 On order, 10 ILL, 11 Cataloging, 12 Reserves, 13 Discard/Weed, 14 Damaged, 15 On reservation shelf, 16 Long Overdue, 17 Lost and Paid, 18 Canceled Transit. The custom states map cleanly to these: available->0, held(ready)->8, on_loan->1, lost->3.
  source: https://raw.githubusercontent.com/evergreen-library-system/Evergreen/main/Open-ILS/src/sql/Pg/950.data.seed-values.sql
  publisher: Evergreen project (source code)
  pub_date: unknown (main branch)
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 4
- claim: FOLIO item schema requires only materialTypeId, permanentLoanTypeId, holdingsRecordId and status. Barcode is optional at schema level, described as a "Unique inventory control number ... used largely for circulation". There are separate fields: accessionNumber ("also called inventar number"), hrid (system-assigned sequential human-readable ID), itemIdentifier and copyNumber. The hierarchy is instance -> holdings record (required parent) -> item.
  source: https://github.com/folio-org/mod-inventory-storage/blob/master/ramls/schemas/item-storage/item.json
  publisher: FOLIO project (source code)
  pub_date: unknown (master; latest mod-inventory-storage release v30.0.9, 2026-10-28)
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 1, 2
- claim: FOLIO item status is a fixed enum: Aged to lost, Available, Awaiting pickup, Awaiting delivery, Checked out, Claimed returned, Declared lost, In process, In process (non-requestable), In transit, Intellectual item, Long missing, Lost and paid, Missing, On order, Paged, Restricted, Order closed, Unavailable, Unknown, Withdrawn. Mapping: available->Available, held(ready)->Awaiting pickup, on_loan->Checked out, lost->Declared lost (or Aged to lost or Missing; the source must distinguish them). Secondary sources say many statuses are set only by system actions, not by hand.
  source: https://github.com/folio-org/mod-inventory-storage/blob/master/ramls/schemas/item-storage/itemStatus.json ; https://folio-at-cu.readthedocs.io/en/latest/settings/item-status.html (search snippet only)
  publisher: FOLIO project; Cornell (secondary)
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high (enum); medium (manual-setting restriction)
  class: schema
  question: 4
- claim: FOLIO practice treats item barcodes as unique. A Jira ticket describes adding suffixes to make barcodes unique without relabelling. Libraries leave the barcode blank at order time and add it later. In consortia, duplicate barcodes across institutions are a known problem, and the advice is barcodes longer than 8 digits with an institution prefix.
  source: https://folio-org.atlassian.net/browse/UXPROD-3384 (search snippet); https://info.library.okstate.edu/foliocataloging/duplicate_barcodes
  publisher: FOLIO Jira; Oklahoma State Univ. Library
  pub_date: 2026-09 (OSU page update)
  accessed: 2026-11-03
  confidence: medium
  class: practice
  question: 2
- claim: Koha migration vendor guidance lists these per-item fields: barcode, branch, shelving location, item type, call number, copy number, price, circulation status, and lost/damaged/withdrawn status. Pitfalls named: source exports lacking barcodes, branches or item types; free-text values ("Main Library") needing mapping to Koha codes ("MAIN"); lost/withdrawn/damaged/reference statuses needing mapping; one-row-per-copy exports turning into duplicate bib records per copy. The advice is to stage a multi-copy sample and verify item counts.
  source: https://kohasupport.com/knowledge-base/bibliographic-vs-item-records-koha-migration/
  publisher: kohasupport.com (vendor; marketing register)
  pub_date: 2026-05
  accessed: 2026-11-03
  confidence: medium
  class: pitfall
  question: 3
- claim: Koha has no general built-in importer for active checkouts. Practitioners write scripts that resolve patron cardnumber -> borrowernumber and item barcode -> itemnumber, or generate an offline-circulation .koc file (barcode-keyed) and upload it. So active-loan migration in practice keys on item barcode plus patron card number.
  source: https://lists.koha-community.org/pipermail/koha-devel/2011-August/036048.html (search summary; fetch of reply 404'd)
  publisher: koha-devel mailing list
  pub_date: 2011-08
  accessed: 2026-11-03
  confidence: low (old, not fetched; current tooling not checked)
  class: practice
  question: 3
- claim: Migration projects capture "current circulation state (checkouts, holds and fines)" with helper scripts separate from the bib/item load (Millennium->Koha, Code4Lib 2010). Vendor checklists say to verify migrated holds' pickup branches and suspended holds.
  source: https://code4lib.org/conference/2010/walls/ ; https://help.bywatersolutions.com/portal/en/kb/articles/data-testing-in-koha (search snippets)
  publisher: Code4Lib; ByWater Solutions
  pub_date: 2010; unknown
  accessed: 2026-11-03
  confidence: low
  class: practice
  question: 3
- claim: Current versions seen: Koha tags up to v26.05.05 (26.05 stable series). FOLIO mod-inventory-storage v30.0.9 was released 2026-10-28. The Evergreen GitHub mirror returned no tags, so the release was not verified.
  source: https://api.github.com/repos/Koha-Community/Koha/tags ; https://api.github.com/repos/folio-org/mod-inventory-storage/releases/latest
  publisher: GitHub mirrors
  pub_date: 2026-10
  accessed: 2026-11-03
  confidence: medium (single source each; Koha tag list may not be sorted by date)
  class: version
  question: 1

## Synthesis for the decision (inferences from findings above)
- Copy identity: all three ILSs key circulation (and, in practice, loan migration) on a human-scannable unique item barcode. Evergreen requires one, Koha and FOLIO make it optional but unique. An internal DB id alone is the costliest gap. The cheapest fix is a unique, immutable, human-readable copy identifier (≤20 chars to fit Koha) from day one, even if no labels are printed yet. Evergreen's '@@id' placeholder shows that "barcode = prefix + internal id" is an accepted fallback.
- Holds: per-title queue with copy assigned at "ready" matches Koha (title-level hold, itemnumber filled on trap, priority, waitingdate) and Evergreen/FOLIO status "On holds shelf"/"Awaiting pickup". Store queue position and ready/expiry dates explicitly.
- Status: keep lost/damaged/withdrawn as separate facts (Koha has them as orthogonal flags), with the date each was set. Derive on_loan from an active loan row rather than storing it only as an item state.
- Item needs: owning location/branch, item type, call number (even simple), date acquired, source of acquisition (Koha 952$e is free text). Without these, migration needs values made up for every item.

## Leads worth chasing
- Koha manual: patron import (CSV, cardnumber as key), Stage MARC records for import (item matching on barcode), bulkmarcimport.pl, and current status of Bug 12348/12256 (barcode mandatory?). Also `itemBarcodeFallbackSearch` syspref.
- Koha `reserves.found` codes (W waiting, T transit, P processing) from the reserves table definition in kohastructure.sql or the manual.
- Koha `old_issues` / `statistics` tables for circulation history, and whether migrations carry history at all.
- Evergreen migration tooling (Equinox migration-tools repo) and the FOLIO Data Import / MARC holdings import, plus a FOLIO post-mortem (e.g. Cornell, Chalmers).
- MARC 541 (Immediate Source of Acquisition) vs Koha 952$e for donor provenance; privacy handling (541 first indicator: private/non-private).
- Koha-Suomi / Finnish Koha consortium migration notes (barcode conventions, Finnish public-library practice).
- Retro-barcoding cost figures; label symbology (Code 39 / Codabar / Code128) practice for small libraries.

## Looked for, not found
- Official docs.folio.org inventory page (404 at guessed URL). FOLIO barcode DB-level unique index not confirmed from source (grep of schema.json returned nothing).
- Primary text of the Koha 2011 checkout-import thread (404).
- Koha MARC21 default framework YAML (path changed; fetch failed), so the 952 subfield list comes from the schema page only.
- Any quantitative post-mortem on retro-barcoding cost, or encoding (Vietnamese diacritics/UTF-8 normalization) migration failures. Not searched within budget.
