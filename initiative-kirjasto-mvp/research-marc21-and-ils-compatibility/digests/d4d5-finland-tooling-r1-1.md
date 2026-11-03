# Digest: dimensions 4–5, round 1

Budget note: 19 research tool calls, about 14 sources read. Questions 5, 7, 9 and most of 6 not reached. See "Looked for, not found".

## Findings

- claim: Melinda's open (unauthenticated) interfaces are: SRU `https://sru.api.melinda.kansalliskirjasto.fi/bib`, `/aut-subjects` and `/aut-works`; OAI-PMH `https://oai-pmh.api.melinda.kansalliskirjasto.fi/bib` and `/aut-subjects`; Z39.50 `z3950.api.melinda.kansalliskirjasto.fi:210/fin01`. These are all labelled "Open Data". IP-restricted, on request and for Melinda libraries only: SRU `/bibprv` (internal fields), `/aut-names`, `/autprv-works`; OAI-PMH `/aut-names`, `/aut-works`; Z39.50 port 10211 `fin01alma` (Alma integration only). Fennica subset: SRU query `melinda.authenticationcode=finb`, OAI set `fennica`. The page states no record format and no usage limits.
  source: https://www.kiwi.fi/spaces/melinda/pages/166101210/Hakurajapinnat+Search+harvesting+APIs
  publisher: National Library of Finland (Melinda wiki)
  pub_date: 2025-05 (last updated)
  accessed: 2026-11-03
  confidence: high (single primary source; a second source, the search-result snippet of the NLF news item below, agrees that the interfaces are open)
  class: access
  dimension: 4
  question: 2

- claim: The National Library says its copy cataloguing interfaces (Z39.50, SRU) are free to use. They are meant for picking individual records or small sets, not bulk transfer; OAI-PMH is the bulk route. I saw this only in a search-result snippet; I did not open the page.
  source: https://www.kansalliskirjasto.fi/en/news/national-library-finland-makes-its-copy-cataloguing-interfaces-freely-available
  publisher: National Library of Finland
  pub_date: unknown
  accessed: 2026-11-03
  confidence: medium
  class: access
  dimension: 4
  question: 2

- claim: Fennica (the national bibliography) is licensed CC0. Channels: MARC bulk downloads at data.nationallibrary.fi, Z39.50/SRU, OAI-PMH, kansalliskirjasto.finna.fi, Melinda, and Fennica-LD (linked data). The page gives no licence for Melinda as a whole, or for records contributed by other libraries or vendors.
  source: https://www.kiwi.fi/spaces/Datacatalog/pages/86972056/Fennica
  publisher: National Library of Finland (Data catalog wiki)
  pub_date: 2026-07 (last updated)
  accessed: 2026-11-03
  confidence: high for Fennica = CC0; Melinda-wide licence unverified
  class: licence
  dimension: 4
  question: 2

- claim: Joining Melinda is free for libraries on the National Library's side. Costs can still come from ILS vendor add-ons or billable work, plus staff time. Requirements: RDA + MARC 21 (FINMARC must be converted), harmonisation with YSO, prior data cleanup (duplicates, identifiers, mandatory fields, character encoding), an ILS that supports MARC 21 and transfer to an external database, and sharing permission for third-party metadata. Joining is by library or consortium; the page does not say whether non-library organisations can join. Integrations: Aurora first (PIKI was the 2015 pilot), Koha in spring 2021 with Koha-Suomi libraries, Helmet's Sierra via batch import. Melinda interfaces used: SRW/U search, cataloguing write-back, replication, and OAI-PMH harvesting of local records during joining.
  source: https://www.kiwi.fi/display/melinda/Yleisten+kirjastojen+liittyminen+Melindaan
  publisher: National Library of Finland (Melinda wiki)
  pub_date: 2024-07 (last updated)
  accessed: 2026-11-03
  confidence: high (that joining is free and the requirements); low on eligibility of a non-library association (not stated)
  class: access
  dimension: 4
  question: 2

- claim: Koha integration pattern (2020): Koha-Suomi libraries fetch records from Melinda over SRU/Z39.50, and the National Library harvests records back from Koha's OAI-PMH interface. In other words, the ILS has to expose OAI-PMH. Seen only as a search snippet of an attachment.
  source: https://www.kiwi.fi/download/attachments/50168589/Liite%206%20Melinda%20rajapinnat%20Koha-Suomi-TaTi_v1.0.pdf
  publisher: National Library of Finland / Koha-Suomi
  pub_date: 2020-01
  accessed: 2026-11-03
  confidence: medium (snippet only)
  class: practice
  dimension: 4
  question: 2

- claim: Helmet (Helsinki region) runs Sierra (since about 2013), and the City of Helsinki has a procurement under way to renew Helmet's library system. The successor system is not named in what I saw.
  source: https://paatokset.hel.fi/fi/asia/hel-2025-016752 ; https://fi.wikipedia.org/wiki/Helmet-kirjasto
  publisher: City of Helsinki decisions; Wikipedia
  pub_date: 2025 (case number hel-2025-016752)
  accessed: 2026-11-03
  confidence: medium (snippets only; not opened)
  class: ecosystem
  dimension: 4
  question: 1

- claim: PIKI piloted Melinda on Axiell Aurora in 2015. No source found naming PIKI's current ILS in 2025–26, and no PIKI ILS migration news was found. PIKI's public web library has run on Finna (piki.finna.fi) since February 2022, a plain-language version (piki.fi/selko) launched on 2026-10-01, and the PIKI strategy runs 2024–2028.
  source: https://blogs.helsinki.fi/kansalliskirjasto-uutiskirje/piki-kirjastot-aloittivat-melindan-tuotantokayton-ensimmaisena-yleisena-kirjastona/ ; https://piki.finna.fi/Content/piki-info ; https://www.tampere.fi/ajankohtaista/2026/10/01/piki-kirjastot-julkaisivat-selkokielisen-verkkokirjaston
  publisher: National Library newsletter; PIKI; City of Tampere
  pub_date: 2015 (Melinda); 2026-10 (selko)
  accessed: 2026-11-03
  confidence: low for "PIKI still on Aurora" (unverified, about 10 years old); high for the Finna-based OPAC
  class: ecosystem
  dimension: 4
  question: 1

- claim: Koha-Suomi Oy is a non-profit in-house company owned by more than 100 municipalities, founded in 2016 out of the "Kohti Kohaa" project; its managing director is Ari Mäkiranta. Its site does not list customer consortia in text. Historical (2017) figures: about 49 municipalities (about 16%) on Koha versus a majority on Axiell Aurora; Vaski was studying Koha; Lumme, Kyyti, OUTI, Lappi and Vaski were Kohti Kohaa partners (2015). Whether Koha is dominant in 2026 is not established.
  source: https://koha-suomi.fi/ ; https://suomenkirjastoseura.fi/kirjastolehti/avoin-lahdekoodi-yleistyy-kirjastojen-jarjestelmissa/ ; https://suomenkirjastoseura.fi/kirjastolehti/mika-on-kirjastojarjestelmien-tilanne-suomessa-talla-hetkella/
  publisher: Koha-Suomi Oy; Kirjastolehti (Finnish Library Association)
  pub_date: 2026 (site); 2017-06; 2015-06
  accessed: 2026-11-03
  confidence: high (ownership); low (current market share; the figures are stale)
  class: ecosystem
  dimension: 4
  question: 1

- claim: Finna is itself an OAI-PMH data provider at `https://api.finna.fi/OAI/Server`. Its metadata prefixes are oai_dc, oai_vufind_json, marc21 (MARCXML), oai_ead, oai_ead3, oai_forward, oai_lido and oai_qdc. Harvesting rules: resumptionToken batches of about 1,000 records, a non-bot User-Agent (otherwise HTTP 403), a deduplicated default set plus `non_dedup`, and deleted records are not reported. Membership, fees and eligibility are not covered; enquiries go to kansalliskirjasto@helsinki.fi.
  source: https://www.kiwi.fi/display/Finna/OAI-PMH+-haravointirajapinta+Finnan+indeksiin
  publisher: National Library of Finland (Finna wiki)
  pub_date: 2025-04 (last updated)
  accessed: 2026-11-03
  confidence: high
  class: access
  dimension: 4
  question: 3

- claim: YSO labels are in four languages: Finnish, Swedish, English and Northern Sami (no Vietnamese). The licence is CC BY 4.0, not CC0. The Skosmos REST API offers `rest/v1/yso/data` in Turtle, RDF/XML, and MARCXML (lang=fi or lang=sv). The vocabulary was last modified on 2026-11-03, so it is actively maintained.
  source: https://finto.fi/yso/en/
  publisher: National Library of Finland (Finto)
  pub_date: 2026-11 (last modified)
  accessed: 2026-11-03
  confidence: high
  class: licence
  dimension: 4
  question: 4

- claim: `@natlibfi/marc-record` (GitHub NatLibFi/marc-record-js): MIT licence, source written in TypeScript, ships `dist/index.d.ts`. Latest version 11.0.3 (2026-08-27); 105 versions since 2018-08; the 11.0.0 line came out 2026-07/08 (major version bump). It does record-structure validation. Repository: 191 commits, 9 stars, 1 fork, a small community (essentially NLF-maintained). Serialisation lives in a separate package.
  source: https://github.com/NatLibFi/marc-record-js ; https://registry.npmjs.org/@natlibfi%2fmarc-record
  publisher: National Library of Finland (GitHub); npm registry
  pub_date: 2026-08
  accessed: 2026-11-03
  confidence: high
  class: tooling
  dimension: 5
  question: 8

- claim: `@natlibfi/marc-record-serializers`: MIT, latest 11.0.5 (2026-09-08), 125 versions since 2018-09, releases monthly through 2026, 1,729 downloads in the month to 2026-11-01. No `types` field in the package manifest, so TypeScript types are not declared. Its dependencies include @xmldom/xmldom, xml2js and stream-json, which points to MARCXML and JSON support. The formats themselves (ISO 2709, Aleph sequential, etc.) are not verified this run.
  source: https://registry.npmjs.org/@natlibfi%2fmarc-record-serializers ; https://api.npmjs.org/downloads/point/last-month/@natlibfi/marc-record-serializers
  publisher: npm registry
  pub_date: 2026-09
  accessed: 2026-11-03
  confidence: high (activity, licence); low (format list)
  class: tooling
  dimension: 5
  question: 8

- claim: `marcjs`: MIT, latest 3.0.3 (2026-09-22). Release gaps: 2.0.x in 2021-01, 3.0.1–3.0.2 in 2025-02/03, then 3.0.3 in 2026-09. Sporadic but alive; created 2013. No types declared; dependencies are `he` and `nopt`.
  source: https://registry.npmjs.org/marcjs
  publisher: npm registry
  pub_date: 2026-09
  accessed: 2026-11-03
  confidence: high
  class: tooling
  dimension: 5
  question: 8

- claim: Vietnam has a national MARC 21 standard, TCVN 7539:2005 (MARC 21 for bibliographic data). A Vietnamese MARC 21 translation was published in 2004, and a December 2004 workshop agreed on using a "MARC 21 rút gọn" (concise) format. A 2020 journal article covers MARC21 use at the National Library of Vietnam (not read). No evidence found of a public Z39.50 or SRU server at NLV.
  source: https://thuvienphapluat.vn/TCVN/Linh-vuc-khac/TCVN-7539-2005-Thong-tin-va-tu-lieu-Kho-mau-MARC-21-cho-du-lieu-thu-muc-907220.aspx ; https://search.huc.edu.vn/Record/hucDS-HUC-4065
  publisher: Thư viện Pháp luật (standards register); Hanoi University of Culture catalogue
  pub_date: 2005; 2020
  accessed: 2026-11-03
  confidence: medium (MARC21 is the standard); low (any machine interface)
  class: practice
  dimension: 4
  question: 6

## Leads worth chasing
- Current Finnish ILS map: the "Koha Suomessa" map image on koha-suomi.fi, the kirjastot.fi statistics or the kirjastot.fi directory field "kirjastojärjestelmä", and the outcome of the Helmet procurement (paatokset.hel.fi hel-2025-016752) to see whether the successor is Koha or something else.
- PIKI's current ILS: PIKI annual report or Tampere city decisions (try "PIKI Aurora", "PIKI Quria", "Pirkanmaan kirjastot kirjastojärjestelmän kilpailutus").
- The overall Melinda data licence (records outside Fennica, vendor records such as BTJ/Kirjavälitys): check data.nationallibrary.fi and the Melinda "Tietojen käyttö" terms.
- Finna joining terms for non-library organisations ("Finna-palvelusopimus", "Finnan asiakkaaksi"); small associations may need to join via a host organisation such as PIKI or a museum.
- YKL in Finto (finto.fi/ykl): licence and languages. Also whether YKL is published in English.
- Multilingual Library (Monikielinen kirjasto, Helsinki) and its Vietnamese holdings: search Finna for language=vie.
- `@natlibfi/marc-record-serializers` README for the exact format list (ISO2709, MARCXML, MARC-in-JSON, AlephSequential); also the NatLibFi `melinda-*` repos for SRU client code (`@natlibfi/sru-client`).
- How Koha, Evergreen and FOLIO import (.mrc, MARCXML) and their OAI-PMH/SRU support: not researched this round.
- ISBN coverage for prefix 978-604 in Finna, Melinda SRU, Open Library and Google Books: not tested. A live SRU query `bath.isbn=978604*` against Melinda would give direct evidence.

## Looked for, not found
- A 2025–26 source naming PIKI's ILS. Not found; the only evidence is the 2015 Aurora pilot.
- Any 2025–26 count or share of Finnish municipalities on Koha. Not found; the 2017 figure is about 16%.
- A Melinda-wide data licence statement. Only Fennica = CC0 was found.
- Whether a non-library association can join Melinda or Finna. Not stated on the pages read.
- A public Z39.50 or SRU endpoint at the National Library of Vietnam.
- Q5 (Nordic minority-language precedent), Q7 (ILS import formats), Q9 (ISBN lookup coverage) and the ISBN-prevalence part of Q6. Not researched because the budget ran out.
