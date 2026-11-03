# Digest: dimension 3, round 2
## Findings
- claim: Koha "Stage records for import" accepts MARC (ISO 2709) or MARCXML. Record type is Bibliographic or Authority. Encodings offered: UTF-8 (default), MARC8, ISO 5426, ISO 6937, ISO 8859-1, EUC-KR. Settings can be saved as a reusable profile. The manual build header reads "Koha Manual 20261103".
  source: https://koha-community.org/manual/latest/en/html/cataloging.html
  publisher: Koha Community
  pub_date: 2026-11
  accessed: 2026-11-03
  confidence: high
  class: version
  question: 1
- claim: Koha staging can "Look for existing records in catalog" using a matching rule defined in Administration. The ISBN rule matches only exact values unless the AggressiveMatchOnISBN preference is changed. If a match is found the options are "Replace existing record with incoming record", "Add incoming record" or "Ignore incoming record (its items may still be processed)". If there is no match the options are "Add incoming record" or "Ignore incoming record". This page does not say whether 001/035 matching is available.
  source: https://koha-community.org/manual/latest/en/html/cataloging.html
  publisher: Koha Community
  pub_date: 2026-11
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 1
- claim: Koha reads embedded items through "Check for embedded item record data?". The options are "Always add items", "Add items only if matching bib was found", "Add items only if no matching bib was found", "Replace items if a matching bib was found" and "Ignore items". When replacing, "Itemnumbers take precedence over barcodes". The import report counts items added, replaced, and ignored because of a duplicate barcode. Imports run as background jobs and can be reversed with "Undo import into catalog". Implication: stable, unique copy barcodes are the practical key for re-importing or replacing copies.
  source: https://koha-community.org/manual/latest/en/html/cataloging.html
  publisher: Koha Community
  pub_date: 2026-11
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 1
- claim: Koha patron import requires "cardnumber, surname, and all fields you have defined in the BorrowerMandatoryField preference". The extracted text did not name branchcode or categorycode as required. Overwrites match on cardnumber or username; extra matchpoints can be patron attributes marked unique. The `patron_attributes` column is a comma-separated, quoted list of CODE:value pairs, e.g. "INSTID:12345,BASEBALL:Cubs". This is where a legacy patron id would go. Passwords are supplied in plaintext and hashed to bcrypt on import. Dates follow the dateformat preference or ISO. In an overlay, blank columns overwrite existing values.
  source: https://koha-community.org/manual/latest/en/html/tools.html
  publisher: Koha Community
  pub_date: 2026-11
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 1
- claim: Koha's batch checkout-history anonymisation "will replace the patron's borrowernumber from the checkout history" with the AnonymousPatron, for checkouts older than a cutoff date and optionally limited to one library. Per-item counts survive ("how many times a particular item was checked out, but not by whom"). AnonymousPatron must be set first, otherwise the tool errors. Batch deletion skips patrons who have items checked out or a non-zero balance. Implication: an anonymised loan history that keeps the copy and drops the patron link maps directly onto Koha's model.
  source: https://koha-community.org/manual/latest/en/html/tools.html
  publisher: Koha Community
  pub_date: 2026-11
  accessed: 2026-11-03
  confidence: high
  class: practice
  question: 2
- claim: In the Koha `reserves` table, `found` is VARCHAR(1) and NULL while the hold is pending. `priority` (smallint, default 1) is the queue position. `itemnumber` is nullable: "the specific item the hold is placed on or was filled with". There are also `item_level_hold` (bool, default 0), a NOT NULL `biblionumber`, `waitingdate` (date marked waiting) and `expirationdate`. This matches a title-level queue in which a copy is assigned when the hold is marked ready.
  source: https://schema.koha-community.org/main/tables/reserves.html
  publisher: Koha Community (SchemaSpy, main branch)
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 2
- claim: The `found` codes are W = waiting on the hold shelf, T = in transit to the pickup branch, P = processing (returned via SIP self-check while HoldsNeedProcessingSIP is on; a librarian must confirm) and NULL = pending/placed. Bug 37842 reports that GetReserveStatus does not handle T. These come from search snippets of C4/Reserves.pm comments and Bugzilla, not from a page I read.
  source: https://bugs.koha-community.org/bugzilla3/show_bug.cgi?id=37842 ; https://git.koha-community.org/Koha-community/Koha/src/commit/0d663c7e8aae16a1b12c03af20b152b186829abf/C4/Reserves.pm
  publisher: Koha Community
  pub_date: unknown (bug filed 2024-09 approx.)
  accessed: 2026-11-03
  confidence: medium
  class: schema
  question: 2
- claim: Koha's autoBarcode includes an EAN-13 option (added 2012, Bug 6448; validates the check digit) alongside hbyymmincr and OFF. Generation looks up the highest existing barcode in the items table and increments it. A code comment warns that two catalogers can get the same number. Bug 29584 (EAN-13 producing 15 digits) was closed as invalid because autobarcode continues from the highest number already in the database. Bug 31798 (2023) made REST API item creation honour every autoBarcode value. Implication: the series of imported barcodes decides what Koha generates next, so do not import a mix of formats. Current manual wording was not retrieved (the systempreferences URL returned 404).
  source: https://bugs.koha-community.org/bugzilla3/show_bug.cgi?id=29584 ; https://lists.koha-community.org/pipermail/koha-patches/2012-March/016859.html
  publisher: Koha Community
  pub_date: 2012-03 / ~2021-12
  accessed: 2026-11-03
  confidence: medium
  class: pitfall
  question: 1
- claim: Evergreen MARC Batch Import (Vandelay) Holdings Import Profiles name the tag that carries the holdings (example "e.g. 949"). Each mapping takes a subfield code or XPATH. Recommended mappings are Barcode, Call Number, Owning Library and Circulating Library; Circulate and Holdable can also be mapped. The library setting "Vandelay: Generate Default Barcodes" will "Auto-generate default item barcodes when no item barcode is present", with an optional "Vandelay: Default Barcode Prefix". "Merge on Exact Match (901c)" matches records re-loaded from Evergreen on record ID. Match sets combine record attributes or MARC tag/subfield points with AND/OR. Merge profiles have preserve, replace, add and remove specifications.
  source: https://docs.evergreen-ils.org/docs/latest/cataloging/batch_importing_MARC.html
  publisher: Evergreen Project
  pub_date: unknown (latest docs; contains an unfilled "VERSION" placeholder)
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 3
- claim: Older Evergreen mailing-list reports say holdings import runs only with "Auto-Import Non-Colliding Records" and supports only monograph holdings embedded in the bib record (2009–2010). A 2018 report on 3.1.2 says zero items were imported with no error message. I did not check whether this is still true.
  source: https://list.evergreen-ils.org/pipermail/open-ils-general/2010-January/002392.html ; https://list.evergreen-ils.org/pipermail/open-ils-dev/2018-June/010519.html
  publisher: Evergreen mailing lists
  pub_date: 2010-01 / 2018-06
  accessed: 2026-11-03
  confidence: low
  class: pitfall
  question: 3
- claim: FOLIO Data Import match profiles map an incoming MARC Bib 9xx subfield (any 9xx except 999 ff) to Item Barcode. Holdings match on HRID or UUID from a 9xx field. Matching should go top-down (instance, then holdings, then item). Known bug MODDATAIMP-798: a holdings or instance submatch cannot follow a MARC-to-MARC match. MARC Holdings-to-MARC Holdings matching is listed as in development, limited to 001 and 999 ff. Source: search snippets of the FOLIO wiki and docs, which may be dated.
  source: https://folio-org.atlassian.net/wiki/spaces/FOLIOtips/pages/5672633/Data+Import+Functionality+Implementation+Table ; https://docs.folio.org/docs/settings/settings_data_import/settings_data_import/
  publisher: FOLIO Project
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium
  class: schema
  question: 3
- claim: Post-mortem of the University of Missouri System's move from Sierra to FOLIO (decided summer 2021, live June 2022). Early item loads exposed barcode errors, fixed by finding the physical items and correcting the source data before later loads. Location codes had to match exactly; unresolved items were parked in an "error location" and are still being fixed. Some items never migrated: sometimes the bib was missing, sometimes the bib and holdings were created without the item. 035 OCLC numbers without the "(OCoLC)" prefix became generic control numbers and are hard to fix afterwards. The planned iterative review was squeezed out by data volume. Lessons: keep identifiers well formed and prefixed, use exact controlled code lists, and validate items against the shelf before migrating.
  source: https://serials.atla.com/tcb/article/view/3294/4493
  publisher: Atla, Technical Services Quarterly (TCB) vol 31 no 4 (Seth Huber)
  pub_date: 2023-10
  accessed: 2026-11-03
  confidence: high
  class: pitfall
  question: 3
- claim: Michigan State's FOLIO migration put duplicate or second barcodes in a repeatable "Former identifier" field, which can be searched only by query and is slow. This is from a search snippet; I did not read the page.
  source: https://lib.msu.edu/FolioAtMSUL
  publisher: Michigan State University Libraries
  pub_date: unknown
  accessed: 2026-11-03
  confidence: low
  class: pitfall
  question: 3
- claim: MARC 541 (Source of Acquisition) is a repeatable bibliographic field. First indicator: # = no information, 0 = private, 1 = not private (indicator values added 2004). Privacy means information the institution does not want displayed publicly. Subfields: $a source, $b address, $c method (gift/purchase/deposit), $d date (preferably yyyymmdd), $e accession number, $f owner, $h purchase price, $n extent / $o type of unit, $3 materials specified, $5 institution. Implication: export donor data only as 541 with first indicator 0, or leave it out of the export entirely.
  source: https://www.loc.gov/marc/bibliographic/bd541.html
  publisher: Library of Congress
  pub_date: unknown (current LC page)
  accessed: 2026-11-03
  confidence: high
  class: schema
  question: 4
- claim: Library item barcodes are usually Codabar or Code 39. A common 14-digit layout is a type digit (2 = patron, 3 = item), a 4-digit library code, a sequence number and a mod-10 check digit. SirsiDynix Symphony requires 14 characters at 9.4 characters per inch. Common label sizes run from 0.5×1.5 in to 1×2 in. There is no universal structure, so check the target ILS before ordering labels. Sources are vendor docs and help pages (SirsiDynix/IU, Follett, The Library Store), seen only as search snippets.
  source: https://sirsi.sitehost.iu.edu/Helps/Symphony/Workflows/English/Content/Topics/09-ConfigPolicies/FAQs/FAQ_Barcode_Specs.htm ; https://www.thelibrarystore.com/barcode-basics ; https://destinyhelp230prod.follettsoftware.com/Content/c_barcode_symbology.htm
  publisher: SirsiDynix (IU-hosted help), Follett, The Library Store
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium
  class: practice
  question: 5

## Leads worth chasing
- Koha record matching rules: does the default install ship 001/035 (control-number) and ISBN rules? Check https://koha-community.org/manual/latest/en/html/administration.html#record-matching-rules
- Koha systempreferences in the current manual (autoBarcode exact option labels, itemBarcodeFallbackSearch, itemBarcodeInputFilter, AnonymousPatron, OPACPrivacy). The page path has changed; find it from the manual index.
- Koha offline circulation (.koc) upload and `misc/migration_tools/bulkmarcimport.pl` options (-match, -insert/-update, -item-action). The tools page I read does not describe .koc.
- Koha 952$e (source of acquisition) and whether it is hidden in the OPAC by default via framework visibility.
- FOLIO folio_migration_tools (FOLIO-FSE) for loans and requests keyed on item barcode and patron barcode.
- Koha-Suomi and Finnish public library barcode conventions (koha-suomi.fi docs, Kirjastot.fi).

## Looked for, not found
- Current Koha manual text for autoBarcode and itemBarcodeFallbackSearch (cataloguingpreferences.html and systempreferences.html both returned 404 under /latest/).
- Koha .koc offline circulation file format and keys, and documented import paths for active checkouts or holds.
- Whether Koha anonymisation also covers old_reserves (the tools page covers checkout history only).
- Any first-hand account of a small library migrating without barcodes and what it cost; per-label costs; label placement guidance.
- Finnish "nidetunnus" barcode conventions. The only hit was a 2015 Jyväskylä Voyager-to-Koha report noting the nidetunnus stored in 773$9, which is not relevant here.
- MARC 561 (ownership history) and how ILSs keep donor data out of the OPAC; not fetched because the call budget ran out.
