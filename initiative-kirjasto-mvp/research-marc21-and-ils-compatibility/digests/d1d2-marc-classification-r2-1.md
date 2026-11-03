# Digest: dimensions 1–2, round 2
## Findings
- claim: MARC21 Holdings 852 defines: $a location (MARC org code or name), $b sublocation/collection, $c shelving location, $h classification part, $i item part (Cutter/date/term distinguishing items with same class), $j shelving control number (with ind1=4), $k call number prefix, $m suffix, $p piece designation ("may be an identification number such as a bar code number or an accession number"), $t copy number (not a count), $x nonpublic note, $z public note, $2 source of classification/shelving scheme (only with ind1=7). Ind1 values: # none, 0 LCC, 1 DDC, 2 NLM, 3 SuDoc, 4 shelving control no., 5 title, 6 shelved separately, 7 source in $2, 8 other. → a YKL-shelved copy exports as 852 7_ $h <YKL class> $i <pääsana letters> $p <barcode> $2 ykl.
  source: https://www.loc.gov/marc/holdings/hd852.html
  publisher: Library of Congress
  pub_date: 2021-06
  accessed: 2026-11-03
  confidence: high
  class: standard
  dimension: 1
  question: 1
- claim: MARC21 Holdings format has no changes in Update No. 42 (so 852 page dated 2021-06 is current).
  source: https://loc.gov/marc/marc21_update42_online.html (via search snippet of format change list)
  publisher: Library of Congress
  pub_date: 2026-05
  accessed: 2026-11-03
  confidence: medium
  class: version
  dimension: 1
  question: 1
- claim: Koha embeds item (copy) data in bib field 952: $a home library, $b current library, $c shelving location, $o full call number, $p barcode, $y Koha item type, $8 collection, $2 source of classification/shelving scheme, $t copy number, $h serial enumeration/chronology (UNIMARC equivalent 995). Call number lives at item level (952$o), not bib. On MARC import ("stage records"), "Check for embedded item record data?" options: always add items / only if matching bib found / only if no match / replace items / ignore; itemnumbers take precedence over barcodes when matching for replacement. → a flat export of bib + one 952 per copy with stable barcode is directly loadable into Koha.
  source: https://koha-community.org/manual/latest/en/html/cataloging.html
  publisher: Koha Community
  pub_date: 2026-11 (manual build 20261103)
  accessed: 2026-11-03
  confidence: high
  class: practice
  dimension: 1
  question: 1
- claim: Finnish public-library shelf label practice (one library's instruction): the shelf mark's "pääsana" is either author or title; its first three characters are printed on the small spine label; if pääsana is an author name, shelve alphabetically by surname, otherwise by title. Evidence is a search-engine snippet of the PDF (PDF body not machine-readable via fetch).
  source: https://www.hyvinkaa.fi/globalassets/kulttuuri-ja-vapaa-aika/kirjasto/pdf-tiedostot/hyllytys--ja-aakkostusohjeet.pdf
  publisher: Hyvinkään kaupunginkirjasto
  pub_date: 2015-03
  accessed: 2026-11-03
  confidence: medium
  class: practice
  dimension: 1
  question: 1
- claim: Pitfall implied: the 3-letter pääsana depends on the main entry (surname for author, else title). For Vietnamese names (family name first) a model that cannot tell which name part is the entry element cannot derive the pääsana or 100 ind1 correctly. Store name parts/entry element explicitly. (Inference from the shelving rule + MARC 100 indicators below.)
  source: https://www.hyvinkaa.fi/globalassets/kulttuuri-ja-vapaa-aika/kirjasto/pdf-tiedostot/hyllytys--ja-aakkostusohjeet.pdf ; https://www.loc.gov/marc/bibliographic/bdx00.html
  publisher: Hyvinkää library; Library of Congress
  pub_date: 2015-03; unknown
  accessed: 2026-11-03
  confidence: low
  class: pitfall
  dimension: 2
  question: 1, 2
- claim: LC Classification Scheme Source Codes lists "ykl" = "Yleisten kirjastojen luokitusjärjestelmä = Finnish public libraries classification system [former code: fplcs]"; "ddc" = Dewey decimal classification and relative index (OCLC). Page dated 2026-09-17.
  source: https://www.loc.gov/standards/sourcelist/classification.html
  publisher: Library of Congress
  pub_date: 2026-09
  accessed: 2026-11-03
  confidence: high
  class: standard
  dimension: 2
  question: 4
- claim: 082 DDC: ind1 0 full / 1 abridged / 7 other edition specified in $2. $2 = edition information as edition/language/date: print = edition no./MARC language code/year; electronic (WebDewey) = edition/language/yyyymmdd date assigned. Examples: "$2 23/eng/20190402", "$2 22/ger", "$2 5/nor/20071204". No Vietnamese example; a Vietnamese-translation DDC 23 number would by pattern be "$2 23/vie" (+ optional /date) — inference, not attested.
  source: https://www.loc.gov/marc/bibliographic/bd082.html
  publisher: Library of Congress
  pub_date: 2024-08 (page label June 2024)
  accessed: 2026-11-03
  confidence: high (pattern) / low ("23/vie" specifically)
  class: standard
  dimension: 2
  question: 4
- claim: National Library of Finland MARC21 format docs: 084 "Muu luokitus" carries YKL as "084 ## ‡a 37.8 ‡2 ykl"; ind1 = 9 is a Finnish-specific indicator for YKL fiction supplementary class (e.g. "084 9# ‡a Historia ‡2 ykl"); 084 should not be used for schemes without an LC ‡2 source code. 082 ‡2 dates per page use format VVVVKKPP. Page updated 4.5.2026. Real path: /bib/05X-08X.htm.
  source: https://marc21.kansalliskirjasto.fi/bib/05X-08X.htm
  publisher: Kansalliskirjasto (National Library of Finland)
  pub_date: 2026-05
  accessed: 2026-11-03
  confidence: high
  class: standard
  dimension: 2
  question: 5
- claim: Finnish 6XX practice: ind2 = 7 with ‡2 source and ‡0 URI. Examples: "650 #7 ‡a kirjastot ‡2 yso/fin ‡0 http://www.yso.fi/onto/yso/p2787"; "651 #7 ‡a Ranska ‡2 yso/fin ‡0 http://www.yso.fi/onto/yso/p104968" (same URI with ‡2 yso/swe for Swedish label Frankrike); genre "655 #7 ‡a oppikirjat ‡2 slm/fin ‡0 http://urn.fi/URN:NBN:fi:au:slm:s633". → store subject/genre as concept URI + label + language, not free text; one URI serves all language labels.
  source: https://marc21.kansalliskirjasto.fi/bib/6XX.htm
  publisher: Kansalliskirjasto
  pub_date: 2025-09
  accessed: 2026-11-03
  confidence: high
  class: standard
  dimension: 2
  question: 5
- claim: MARC language codes correspond to ISO 639-2/B ("bibliographic" codes = MARC codes, based on English names; T codes aligned with 639-1); exceptions include Serbian/Croatian moved to T codes (srp, hrv) and MARC group codes; LC is maintenance agency for both. "vie" for Vietnamese NOT directly confirmed this run (LC code page fetch failed) — unverified belief.
  source: https://www.loc.gov/standards/iso639-2/develop.html ; https://www.loc.gov/marc/tn080701.html (search snippets)
  publisher: Library of Congress
  pub_date: unknown / 2008-07
  accessed: 2026-11-03
  confidence: medium
  class: standard
  dimension: 2
  question: 3
- claim: Update No. 42 (May 2026) "redefines the 041 language code field and adds some new codes" (bib format) — so 041 semantics changed recently; re-check bd041 before fixing the export mapping.
  source: https://loc.gov/marc/marc21_update42_online.html (search snippet)
  publisher: Library of Congress
  pub_date: 2026-05
  accessed: 2026-11-03
  confidence: medium
  class: version
  dimension: 2
  question: 3, 6
- claim: Second source for Update 42: OCLC's WorldCat Validation release listed "MARC 21 Bibliographic Update No. 42 (Announced May 2026)" for installation 15 July 2026; a later message confirmed installation that day. LC announced Update 42 on 2026-05-27 with 60-day lead time.
  source: https://listserver.lib.byu.edu/pipermail/dcrm-l/2026-July/007538.html ; https://listserver.lib.byu.edu/pipermail/dcrm-l/2026-July/007558.html
  publisher: OCLC (posted to DCRM-L list)
  pub_date: 2026-07
  accessed: 2026-11-03
  confidence: medium (search snippets, not fetched)
  class: version
  dimension: 2
  question: 6
- claim: MARC X00 personal name ind1: 0 forename, 1 surname, 3 family name; current personal names constructed per RDA (OCLC). No retrieved source states Vietnamese-specific RDA/LC practice; a 2011 MARBI paper notes outstanding questions on non-Western given/surname forms.
  source: https://www.loc.gov/marc/bibliographic/bdx00.html ; https://oclc.org/bibformats/en/1xx/100.html ; https://loc.gov/marc/marbi/2011/2011-06.html (search snippets)
  publisher: Library of Congress; OCLC
  pub_date: unknown; 2011
  accessed: 2026-11-03
  confidence: medium
  class: standard
  dimension: 2
  question: 2

## Leads worth chasing
- LC-PCC PS / RDA Toolkit instructions for Vietnamese names (and LC NAR for "Nguyễn, Nhật Ánh" — id.loc.gov returned 403 to fetch; try id.loc.gov/authorities/names/label/ or VIAF).
- loc.gov/marc/bibliographic/bd041.html post-Update 42: what "redefines 041" means for $a/$h/$k and $2.
- LC "Subject Heading and Term Source Codes" page (loc.gov/standards/sourcelist/subject.html) to confirm yso/fin, yso/eng, slm/fin entries — Finnish docs use them, LC listing not fetched.
- Hyvinkää shelving PDF full text (and Kirjastot.fi / Finna "hyllypaikka" docs) for children's codes and YKL 84.2 picture books.
- Melinda / Kansalliskirjasto "kuvailutaso" minimum-level guidance (kiwi.fi Melinda pages).
- loc.gov/marc/relators/ for aut/ill/trl codes for 100/700 $4.

## Looked for, not found
- Explicit LC page text confirming "vie" (LC ISO 639-2 page fetch returned no Vietnamese; MARC languages intro PDF unreadable).
- Any attested "082 $2 23/vie" example or rule for coding a translated DDC edition beyond the edition/language/date pattern.
- Finnish children's collection location codes ("lasten") — no source retrieved.
- Melinda minimum cataloguing level — not searched (budget).
- 949/999 vendor holdings fields — not researched (budget).
- 245/250/264/300/490/020$q/008/22 /520 pitfall sources — not retrieved this round (budget); remain unverified beliefs.
