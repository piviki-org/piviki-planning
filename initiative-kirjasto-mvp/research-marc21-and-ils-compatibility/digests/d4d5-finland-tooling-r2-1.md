# Digest: dimensions 4–5, round 2

## Findings

### Q1: PIKI ILS, Koha share, Helmet
- claim: A City of Tampere decision by the CIO (tietohallintojohtaja) approved an agreement with Axiell Finland Oy for the Axiell Aurora library system. Annual cost 174 744,50 EUR, contract annexes confidential, case no. TRE:4257/02.07.01/2019. Seen only in a search snippet; the decision date was not visible.
  source: https://tampere.cloudnc.fi/fi-FI/Viranhaltijat/Tietohallintojohtaja/Axiell_Aurora_kirjastojarjestelmaa_koske(98681)
  publisher: City of Tampere (decision register)
  pub_date: unknown (case opened 2019)
  accessed: 2026-11-03
  confidence: medium
  class: ecosystem
  dimension: 4
  question: 1
- claim: A Helsinki decision on the KUVA Suomi.fi integration project lists "Piki-kirjastot/Aurora: Tampereen kaupungin liityntäpalvelin", which says PIKI ran Aurora at that time. This is the second source for Aurora, but it is old (case 2017). Seen only in a snippet.
  source: https://paatokset.hel.fi/fi/asia/hel-2017-007484
  publisher: City of Helsinki
  pub_date: 2017 (approx.)
  accessed: 2026-11-03
  confidence: medium (for "PIKI = Aurora" historically); low (for 2025–26)
  class: ecosystem
  dimension: 4
  question: 1
- claim: Axiell is ending support for Aurora: "Muutos johtuu siitä, että Aurora-järjestelmän tuki päättyy." All libraries using Aurora must move to a new system. The article names Quria (also from Axiell Finland Oy) as the replacement for Leppävirta. Preparations start in late 2026 and the switch is planned for H1 2027. PIKI is not named. Implication: if PIKI is still on Aurora, it faces a system change soon.
  source: https://www.soisalonseutu.fi/paikalliset/9849769
  publisher: Soisalon Seutu (local newspaper)
  pub_date: 2026-10
  accessed: 2026-11-03
  confidence: medium
  class: ecosystem
  dimension: 4
  question: 1
- claim: Tampere's Culture and Leisure Committee approved an updated PIKI cooperation agreement on 2026-05-21. It is expected to take effect on 2026-09-01 and replaces the 2012 agreement. "The library system and related services" are now a binding joint procurement, and Tampere is authorised to prepare and sign joint procurement decisions. The article names no system or vendor and mentions no tender.
  source: https://www.tampere.fi/en/current/2026/05/22/cooperation-between-tampere-and-other-piki-library-consortia
  publisher: City of Tampere
  pub_date: 2026-05
  accessed: 2026-11-03
  confidence: high (for the agreement content)
  class: ecosystem
  dimension: 4
  question: 1
- claim: The Koha-Suomi Oy homepage does not list member municipalities or consortia and gives no count of libraries. It says the owners are municipalities across Finland, the company was founded in 2016 out of the "Kohti Kohaa" project, and it won the COSS Open World Hero 2025 award. There is a map image ("Koha Suomessa", 2024 filename) whose contents could not be read. PIKI is not mentioned.
  source: https://koha-suomi.fi/
  publisher: Koha-Suomi Oy
  pub_date: 2026 (footer)
  accessed: 2026-11-03
  confidence: high (for what the page says); the share of libraries on Koha is not established
  class: ecosystem
  dimension: 4
  question: 1

### Q2: ILS import formats
- claim: Koha imports bibliographic and authority records as "either in MARC or MARCXML format", in two steps: stage into a reservoir, then import. The Stage MARC records tool has fields for "Character encoding" and "Format", plus optional MARC modification templates. Koha's bibliographic export offers "MARC or MARCXML". The manual does not mention MARC-in-JSON.
  source: https://koha-community.org/manual/latest/en/html/cataloging.html ; https://koha-community.org/manual/latest/en/html/tools.html
  publisher: Koha Community (official manual, "latest")
  pub_date: unknown (latest)
  accessed: 2026-11-03
  confidence: high
  class: tooling
  dimension: 4
  question: 2
- claim: The Evergreen MARC Batch Import (Vandelay) interface loads "MARC (and MARCXML) records". The upload file can be "MARC (.mrc) or MARCXML (.xml)", and batches of up to about 5 000 records work without problems. Item import uses holdings import profiles ("Default Values for Item Import"). The page does not mention JSON.
  source: https://docs.evergreen-ils.org/docs/latest/cataloging/batch_importing_MARC.html
  publisher: Evergreen ILS documentation
  pub_date: unknown (latest)
  accessed: 2026-11-03
  confidence: high
  class: tooling
  dimension: 4
  question: 2

### Q3: npm MARC libraries
- claim: @natlibfi/marc-record-serializers exports these serializers: Text, Json, AlephSequential, ISO2709, MARCXML, OAI_MARCXML. The README documents a MARCXML reader with an optional namespace prefix, from/to with omitDeclaration and indent options, Aleph Sequential `to`, and a CLI (`npx @natlibfi/marc-record-serializers`). Licence MIT, copyright 2018–2026 University of Helsinki / National Library of Finland.
  source: https://github.com/NatLibFi/marc-record-serializers (README, src/index.js)
  publisher: NatLibFi
  pub_date: 2026-09
  accessed: 2026-11-03
  confidence: high
  class: tooling
  dimension: 5
  question: 3
- claim: The NatLibFi "Json" serializer passes each element of a JSON array straight into `new MarcRecord(...)`. So it uses @natlibfi/marc-record's own JSON shape, not the LoC MARC-in-JSON (MiJ) shape. This is inferred from src/json.js; the MarcRecord object shape itself was not checked this run.
  source: https://raw.githubusercontent.com/NatLibFi/marc-record-serializers/HEAD/src/json.js
  publisher: NatLibFi
  pub_date: 2026
  accessed: 2026-11-03
  confidence: medium
  class: tooling
  dimension: 5
  question: 3
- claim: The NatLibFi ISO2709 code computes field lengths and leader record length in UTF-8 bytes (comment: "fieldLength is the number of UTF-8 bytes in a string"). It writes UTF-8 only, with no MARC-8 handling visible.
  source: https://raw.githubusercontent.com/NatLibFi/marc-record-serializers/HEAD/src/iso2709.js
  publisher: NatLibFi
  pub_date: 2026
  accessed: 2026-11-03
  confidence: medium
  class: tooling
  dimension: 5
  question: 3
- claim: The correct repo is github.com/NatLibFi/marc-record-serializers; the "-js" URL in the brief returns 404. It was last pushed 2026-09-24 and is not archived. Commits (max 100 fetched since 2024-01-01): 2024 = 5, 2025 = 8, 2026 = 14. It has 2 open items, both dependabot PRs from 2026-09. npm latest is 11.0.5 (2026-09-08), with 11.0.4 (2026-08-27) and 11.0.3 (2026-05-06) before it. Dependencies: @natlibfi/marc-record ^11.0.3, @xmldom/xmldom, xml2js, stream-json.
  source: https://api.github.com/repos/NatLibFi/marc-record-serializers ; https://registry.npmjs.org/@natlibfi/marc-record-serializers
  publisher: GitHub API / npm registry
  pub_date: 2026-09
  accessed: 2026-11-03
  confidence: high
  class: version
  dimension: 5
  question: 3
- claim: NatLibFi/marc-record-js (the core record library) was last pushed 2026-10-02 under the MIT licence, with 4 open issues. Commits since 2024: 2024 = 28, 2025 = 14, 2026 = 12.
  source: https://api.github.com/repos/NatLibFi/marc-record-js
  publisher: GitHub API
  pub_date: 2026-10
  accessed: 2026-11-03
  confidence: high
  class: version
  dimension: 5
  question: 3
- claim: marcjs (repo fredericd/marcjs, MIT) can parse Iso2709, Marcxml and MiJ (MARC-in-JSON). It can write Iso2709, Marcxml, MiJ, Text and Json. The ISO2709 parser decodes with `toString('utf8')`, so UTF-8 only. Activity: commits 2025 = 6, 2026 = 1, none in 2024. npm releases: 3.0.1 (2025-02-24), 3.0.2 (2025-03-13), 3.0.3 (2026-09-22). 11 open items, mostly stale dependabot PRs from 2021–2023. Open issue #39 is "Parsers silently accepts broken data" (2022), and open PR #24 is "text encoding option for Iso2709 files" (2018, i.e. no MARC-8).
  source: https://github.com/fredericd/marcjs (README, lib/) ; https://api.github.com/repos/fredericd/marcjs ; https://registry.npmjs.org/marcjs
  publisher: GitHub / npm
  pub_date: 2026-09
  accessed: 2026-11-03
  confidence: high
  class: tooling
  dimension: 5
  question: 3

### Q4: live tests
- claim: Melinda SRU query `bath.isbn=978604*` (Vietnamese ISBN prefix) returned numberOfRecords = 828. Sample records have 008 place "vm", language "vie" or eng, and 041 (e.g. `$a vie $h eng`). They carry 084 with `$2 ykl` (e.g. 84.948 for Vietnamese-language fiction; 85.1948 / 85.2948 for children's fiction) and YSO or kaunokki 650s with yso URIs in $0. 020 $q uses the Finnish "pehmeäkantinen".
  source: https://sru.api.melinda.kansalliskirjasto.fi/bib?version=1.1&operation=searchRetrieve&maximumRecords=3&query=bath.isbn%3D978604%2A
  publisher: National Library of Finland (live API)
  pub_date: live
  accessed: 2026-11-03
  confidence: high
  class: live-test
  dimension: 4
  question: 4
- claim: Melinda SRU query `dc.language=vie` returned numberOfRecords = 2191. It matches records with vie anywhere in 041, including German- and English-language Vietnamese textbooks and dictionaries, so it is broader than "books in Vietnamese".
  source: https://sru.api.melinda.kansalliskirjasto.fi/bib?version=1.1&operation=searchRetrieve&maximumRecords=3&query=dc.language%3Dvie
  publisher: National Library of Finland (live API)
  pub_date: live
  accessed: 2026-11-03
  confidence: high
  class: live-test
  dimension: 4
  question: 4
- claim: Melinda SRU query `dc.creator="Nguyễn Nhật Ánh"` returned numberOfRecords = 101. The records have 008 "vm … j … vie" (juvenile), 041 $a vie, and 084 84.948 ykl. A plain keyword query on the same name also returns unrelated hits (e.g. a Norwegian record).
  source: https://sru.api.melinda.kansalliskirjasto.fi/bib?version=1.1&operation=searchRetrieve&maximumRecords=3&query=dc.creator%3D%22Nguy%E1%BB%85n%20Nh%E1%BA%ADt%20%C3%81nh%22
  publisher: National Library of Finland (live API)
  pub_date: live
  accessed: 2026-11-03
  confidence: high
  class: live-test
  dimension: 4
  question: 4
- claim: The Vietnamese text in Melinda records is inconsistent. Diacritics are stored in decomposed form (NFD), with combining marks, e.g. "Nguye&#x302;&#x303;n, Nha&#x323;&#x302;t A&#x301;nh". Some records drop diacritics entirely (e.g. 245 "Cho toi xin mot ve di tuoi tho"), and some drop only part of them ("Nguye&#x302;n, Nha&#x302;t Anh"). Imported titles need Unicode normalisation (NFC/NFD) and diacritic-insensitive matching.
  source: same three Melinda SRU URLs above
  publisher: National Library of Finland (live API)
  pub_date: live
  accessed: 2026-11-03
  confidence: high
  class: live-test
  dimension: 4
  question: 4
- claim: Finna API `filter[]=language:vie` returned resultCount = 4397. The building facet shows "PIKI-kirjastot" = 335 Vietnamese-language records, Helmet = 1612, Fredrika-kirjastot = 1533, Vaski = 402, Varastokirjasto = 589, Tampereen yliopisto = 17. Keyword `lookfor=Nguyễn Nhật Ánh` returned 133 records, of which PIKI holds 6 and Helmet 115. The combined filter `language:vie` + `building:"0/PIKI/"` returned 0 even though the facet shows 335, so treat it as a likely filter-syntax error rather than a real result.
  source: https://api.finna.fi/v1/search?lookfor=&filter%5B%5D=language%3Avie&limit=0&facet%5B%5D=building ; https://api.finna.fi/v1/search?lookfor=Nguy%E1%BB%85n%20Nh%E1%BA%ADt%20%C3%81nh&limit=0&facet%5B%5D=building
  publisher: National Library of Finland / Finna (live API)
  pub_date: live
  accessed: 2026-11-03
  confidence: high (counts); medium (the PIKI-filter anomaly)
  class: live-test
  dimension: 4
  question: 4

### Q5: route into Finna for an association
- claim: Finna membership funded centrally by the Ministry of Education and Culture (OKM) covers OKM-sector agencies, certain archives, university and polytechnic libraries, public libraries, Varastokirjasto, and museums meeting conditions (museum work a statutory task, not run for profit). Organisations outside central funding "voivat jatkossa liittyä" (may in future join) and pay under the current price list. Organisations on the list can also join through the National Library's "Europeana Formula" intermediary service. Process: email finna-posti; it takes about 6 months on average. An association-run community library fits no listed category. The page names no route via a host library.
  source: https://www.kiwi.fi/spaces/Finna/pages/442433934/Ketk%C3%A4+voivat+liitty%C3%A4+mukaan
  publisher: National Library of Finland (Kiwi wiki)
  pub_date: 2025-10
  accessed: 2026-11-03
  confidence: high
  class: access
  dimension: 4
  question: 5

### Q6: harvestable interfaces
- claim: Joining Finna requires your own back-end system with an OAI-PMH harvesting interface. Harvested formats supported: MARC 21, LIDO, EAD3, FORWARD, Dublin Core. Metadata must be published as CC0 open data, and staff are needed for setup and maintenance. Steps: request a test harvest, then scheduled production harvests.
  source: https://www.kiwi.fi/spaces/Finna/pages/442433934/Ketk%C3%A4+voivat+liitty%C3%A4+mukaan ; https://www.kiwi.fi/spaces/Finna/pages/51841675/Finnaan+liittymisen+askelmerkit
  publisher: National Library of Finland (Kiwi wiki)
  pub_date: 2025-10
  accessed: 2026-11-03
  confidence: high (first page fetched); medium (askelmerkit page seen only in a snippet)
  class: access
  dimension: 4
  question: 6
- claim: As a reference pattern, the Kiwi page for Alma publishing to Finna uses a profile with Output format "MARC21 Bibliographic", protocol OAI, set spec "finna", published hourly at bibliographic level. Seen only in a search snippet.
  source: https://www.kiwi.fi/spaces/Finna/pages/134938812/Alman+metadatan+julkaisu+OAI-PMH+lla+Finnaa+varten
  publisher: National Library of Finland (Kiwi wiki)
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium
  class: practice
  dimension: 4
  question: 6

## Leads worth chasing
- Whether PIKI is moving from Aurora to Quria (or to Koha) under the 2026 joint-procurement annex: check tampere.cloudnc.fi decisions for "Quria" and the PIKI system team (järjestelmätiimi). This decides which import formats and Melinda practice PIKI would apply to the association's books.
- The Kiwi page "Kirjastojärjestelmän vaihtaminen" (https://www.kiwi.fi/spaces/Finna/pages/138937033) covers Aurora-to-Koha migration in Finna, and "Finnaan liitetyt taustajärjestelmät" (pageId=51842364) may list the system per organisation. These are the best sources for the Koha share.
- The Finna OAI-PMH harvesting-interface field requirements (https://www.kiwi.fi/display/Finna/OAI-PMH+-haravointirajapinta+Finnan+indeksiin) were not fetched: id format, set structure, deleted-record handling.
- A practical alternative to joining Finna: donate the books to, or deposit them with, PIKI (which already has 335 Vietnamese-language records) and rely on Melinda's 828 records with ISBN prefix 978604 for copy cataloguing.
- Check the MarcRecord JSON shape against MiJ to confirm the Json serializer claim.

## Looked for, not found
- No primary source naming PIKI's ILS in 2025–2026. The two sources found say Aurora but date from 2017–2019, and the 2026 cooperation news names no system.
- No count or share of Finnish public libraries on Koha. The koha-suomi.fi homepage gives no list, and the map image could not be read.
- No outcome of Helmet's Sierra renewal procurement; not searched within budget.
- FOLIO Data Import formats: the docs.folio.org URL tried returned 404, so FOLIO's support for MARC binary, MARCXML or JSON is unverified this run.
- No evidence that Koha or Evergreen accept MARC-in-JSON for import. Their docs list MARC (ISO 2709) and MARCXML only.
- No documented route for an association to contribute records to Melinda through a member library; not searched within budget.
