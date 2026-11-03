# Digest: dimensions 1–2, round 1

## Findings

- claim: MARC 21 Bibliographic is at "Update No. 42, May 2026" (the current base to cite). Changes are listed in Appendix G.
  source: https://www.loc.gov/marc/bibliographic/
  publisher: Library of Congress (NDMSO)
  pub_date: 2026-05
  accessed: 2026-11-03
  confidence: high (single primary source; the version claim needs a second source)
  class: version
  dimension: 1
  question: 1

- claim: Leader codes relevant to a small book catalogue: 05 record status (n = new, c = corrected, d = deleted); 06 type of record "a" = "Language material"; 07 bibliographic level "m" = "Monograph/Item"; 09 character coding "#" = MARC-8, "a" = "UCS/Unicode"; 17 encoding level (# full, 3 abbreviated, 4 core, 5 partial/preliminary, 7 minimal, u unknown); 18 descriptive cataloguing form (i = ISBD punctuation included, c = ISBD punctuation omitted, # non-ISBD, u unknown). For a volunteer system that has not inspected items to a cataloguing standard, 17 = 3, 5 or 7 and 18 = c or # are honest values that an ILS can later upgrade (05 = a means "encoding level raised").
  source: https://www.loc.gov/marc/bibliographic/bdleader.html
  publisher: Library of Congress
  pub_date: 2016-11 (page footer; content is stable)
  accessed: 2026-11-03
  confidence: high for the codes; low for which 17/18 values to choose (my inference)
  class: standard
  dimension: 1
  question: 1

- claim: LC's National Level Record requirements give each element a code: M ("must be present in every bibliographic record"), A (mandatory "if it is appropriate ... and if the bibliographic information is available") or O (optional). Minimal level counts only M/A elements. The page is dated 2011-02-09 and based on the 1999 edition through Update 12 (2010), so it is stale. Per-field M/A codes sit on subpages (nlrleaddirect, nlr00x, nlr0xx, nlr3xx ...) that I did not read.
  source: https://www.loc.gov/marc/bibliographic/nlr/nlr.html
  publisher: Library of Congress
  pub_date: 2011-02
  accessed: 2026-11-03
  confidence: high (for the framework); field-level M/A not retrieved
  class: standard
  dimension: 1
  question: 1

- claim: PCC/BIBCO "core" level (Leader/17 = 4) was retired in 2010 and replaced by the BIBCO Standard Record (encoding level blank). "Core level" is therefore a historical label, not a current target. The useful concept is a minimum set of data elements.
  source: https://loc.gov/aba/pcc/bibco/coreintro.html (via search snippet; page not fetched)
  publisher: Library of Congress / PCC
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium (snippet only)
  class: standard
  dimension: 1
  question: 1

- claim: The National Library of Finland edits the Finnish and Swedish versions of MARC 21. The Finnish format is at http://marc21.kansalliskirjasto.fi/ and the English wiki at https://www.kiwi.fi/display/Marc21/In+English. It offers conversion tooling (USEMARCON). Joining is free. The service page says nothing about RDA, minimum levels, YKL/084 or YSO.
  source: https://www.kansalliskirjasto.fi/en/services/marc-21
  publisher: National Library of Finland
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high
  class: practice
  dimension: 1
  question: 2

- claim: The current Finnish guidance for RDA description in MARC 21 is kept separately from the Doria "Kuvailuohjeet" collection, which holds Fennica authority-entry guides. No Finnish "minimitaso" (minimal-level) definition was found.
  source: https://doria.fi/handle/10024/94458 (search snippet)
  publisher: National Library of Finland (Doria)
  pub_date: unknown
  accessed: 2026-11-03
  confidence: low
  class: practice
  dimension: 1
  question: 2

- claim: In the MARC Unicode encoding, Leader/09 must be "a". Combining diacritics go "_following_ the base letter they modify", the reverse of MARC-8 order. With several marks above a letter, the one nearest the base comes first, and a mark below goes before one above. This applies to Vietnamese letters that stack a vowel modifier with a tone mark (e.g. ế, ặ). The LC page names no normalization form (NFC/NFD). It warns that MARC-8 to Unicode conversion has "complexities" (field 066 and the $6 script code are dropped).
  source: https://www.loc.gov/marc/specifications/speccharucs.html
  publisher: Library of Congress
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high
  class: standard
  dimension: 1
  question: 3

- claim: Koha stores bibliographic data as NFC but exports as NFD: "bibliographic data stored in Koha is normalized as NFC" and "all exports (MARC utf8, MARCXML) are performed using NFD normalization". Bug 42157 (2026-03-24, status NEW, enhancement) asks for a configurable MARCExportUnicodeNormalization setting. Practical consequence: a home-grown system that compares or deduplicates Vietnamese strings must normalize (store NFC, compare after normalizing) or round-trips through Koha will produce byte-different strings.
  source: https://lists.koha-community.org/archives/list/koha-bugs@lists.koha-community.org/thread/5RHEQ5R3FLCPQXP5NXFWOSCRAHWEAYQF/
  publisher: Koha community bug tracker
  pub_date: 2026-03
  accessed: 2026-11-03
  confidence: high (primary bug text)
  class: pitfall
  dimension: 1
  question: 3

- claim: Supporting history (search snippets, not fetched). MARC::Charset output from MARC-8 conversion is decomposed (NFD/NFKD), per a 2008 koha-devel thread. "a lot of Koha databases have mixed Unicode normalization forms" (2009 IRC). A MarcEdit author argues NFC is the long-term direction and says NFKD breaks rendering for some languages. Avoid MARC-8 entirely for Vietnamese.
  source: https://lists.koha-community.org/pipermail/koha-devel/2008-April/030175.html ; https://blog.reeset.net/archives/2790
  publisher: Koha devel list; Terry Reese (MarcEdit) blog
  pub_date: 2008-04; unknown
  accessed: 2026-11-03
  confidence: medium (snippets)
  class: pitfall
  dimension: 1
  question: 3

- claim: YKL ("Yleisten kirjastojen luokitusjärjestelmä") is published on Finto by Kansalliskirjasto, authored by Helsingin kaupunginkirjasto and the YKL working group. It is licensed CC0 1.0. Last modified 2026-01-13. Its labels are in Finnish and Swedish, and it describes itself as a classification for multilingual library collections.
  source: https://finto.fi/ykl/fi/
  publisher: National Library of Finland (Finto)
  pub_date: 2026-01
  accessed: 2026-11-03
  confidence: high
  class: licence
  dimension: 2
  question: 6

- claim: Verkko-YKL moved from ykl.kirjastot.fi to Finto.fi in early January 2019, together with Kaunokki/Bella and HKLJ. ykl.kirjastot.fi is now only a redirect notice.
  source: https://ykl.kirjastot.fi/
  publisher: Kirjastot.fi
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high
  class: practice
  dimension: 2
  question: 6

- claim: In practice YKL 84.2 is narrative fiction ("KAUNOKIRJALLISUUS 84 KERTOMAKIRJALLISUUS"), and Finnish libraries apply it to children's fiction too. A catalogue record for Lindgren's *Melukylän lapset* shows "84.2 ykl", which supports "ykl" as the 084 $2 source code. I found no official YKL statement on children's material (I believe it is handled by a shelving location or collection code rather than a separate class, but this is unverified).
  source: https://www.hyvinkaa.fi/globalassets/kulttuuri-ja-vapaa-aika/kirjasto/pdf-tiedostot/uutuusluettelo-ln-04.pdf ; https://literarybibliography.eu/en/search/record/f000307440.html (search snippets)
  publisher: Hyvinkää city library; literarybibliography.eu
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium (practice evidence, not the standard)
  class: practice
  dimension: 2
  question: 6, 8

- claim: MARC 084 (Other Classification Number) carries non-LC/non-DDC schemes. $2 holds the source code, taken from LC's "Classification Scheme Source Codes" list (example: 084 ##$a...$2laclaw). I could not confirm "ykl" on the LC list in this run (the fetch aborted).
  source: https://loc.gov/marc/bibliographic/bd084.html (search snippet)
  publisher: Library of Congress
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium
  class: standard
  dimension: 2
  question: 6, 8

- claim: DDC licensing. OCLC's WebDewey Schedule 8 (US) lets bibliographic records "display DDC numbers" but they "shall not display DDC captions". Use is for internal, noncommercial creation of records. On termination, DDC numbers already added need not be removed. DDC, Dewey and WebDewey are OCLC trademarks.
  source: https://www.oclc.org/content/dam/legal/schedules/en_us/Schedule-08-WebDewey-EN-US.pdf (search snippet)
  publisher: OCLC
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium (snippet of a primary legal text)
  class: licence
  dimension: 2
  question: 7

- claim: Deutsche Nationalbibliothek: "In principle, no license is required to classify media using DDC numbers". What OCLC protects by copyright is "everything that goes beyond individual DDC numbers and class headings". This is the second source agreeing that storing DDC numbers in 082 needs no licence, while shipping DDC schedules or captions in the UI does.
  source: https://www.dnb.de/EN/Professionell/DDC-Deutsch/_content/ddcDeutschDDCkostenfrei_akk.html
  publisher: Deutsche Nationalbibliothek
  pub_date: unknown
  accessed: 2026-11-03
  confidence: high (two sources agree)
  class: licence
  dimension: 2
  question: 7

- claim: Vietnam uses DDC 23. The NLV published a Vietnamese DDC 23 translation (4 volumes, launched 2013-11-29). The NLV's 2014–2019 review conference (Da Nang, 1–2 Nov 2019) reported "most libraries have used DDC 23", with special and multidisciplinary libraries lagging. A ministry decision applied MARC21, AACR2 and DDC nationwide (search snippet). The NLV also published an abridged DDC 14. So Vietnamese source records plausibly already carry 082 DDC 23 numbers, which can be imported as-is.
  source: https://nlv.gov.vn/ef/summary-conference-on-application-of-dewey-decimal-classification-vietnamese-edition-23-in-viet-nams-libraries-2014-2019.html ; https://nlv.gov.vn/ef/launching-ceremony-and-inauguration-of-the-training-workshop-dewey-decimal-classification-vietnamese-edition-23.html
  publisher: National Library of Vietnam
  pub_date: 2019-11
  accessed: 2026-11-03
  confidence: medium-high (primary but 2019; no current 2025–26 statement found)
  class: practice
  dimension: 2
  question: 7

## Leads worth chasing
- The marc21.kansalliskirjasto.fi structure: the /bib/0XX.htm guess returned 404. Find the real path for 084/082/650/041 and any Finnish local fields (e.g. 9XX), and check whether YSO is coded as 650 $2 yso/fin.
- The LC Classification Scheme Source Codes list (https://www.loc.gov/standards/sourcelist/classification.html; the fetch aborted). Confirm "ykl". The DDC 082 edition code is a $2 number like "23", or "23/fin" for translations; also check the abridged edition codes.
- The LC NLR subpages (nlrleaddirect.html, nlr00x.html, nlr0xx.html, nlr3xx.html) for field-level M/A codes at minimal level, including 008/22 target audience (j = juvenile) and 008/35-37 language. Not retrieved.
- Koha manual, items section: the 404 at /manual/latest/en/html/cataloguing.html. Try the cataloguing "item records" or "MARC bibliographic frameworks" pages for 952 subfields (o call number, p barcode, y itype, 1 lost, 7 notforloan, a/b home/holding branch) and the itemcallnumber preference.
- Finnish public-library minimum-level guidance, perhaps from Kirjastot.fi or a Finna/Melinda consortium (Melinda's cataloguing levels), plus the RDA in Finland toolkit pages.
- Holdings exchange: MARC 21 Holdings format 852 ($h/$i call number parts) versus embedded item fields (949/999 used by others). Not searched this round.
- The Unicode/LC page states mark order but no normalization form. Check whether the MARC 21 spec, Part 4, says anything about NFC/NFD. Search the Koha bug tracker for "Vietnamese".

## Looked for, not found
- A Finnish-specific minimal-level ("minimitaso") definition for MARC records: none found.
- An explicit "$2 ykl" in an authoritative standards page: practice examples only.
- Vietnamese-specific normalization pitfalls in Koha: none found. Only general NFC/NFD issues.
- Current (2025–26) NLV statement on its DDC edition.
- Koha 952 subfield documentation: the fetch failed (404), so item data requirements are unevidenced this round.
- Q5 (data-model pitfalls, e.g. a single author string or year stored as text): no sources consulted this round. Only inferences, so none are recorded as findings.
- Q8 (where the call number lives, bib versus item): not evidenced beyond the YKL practice snippets.
