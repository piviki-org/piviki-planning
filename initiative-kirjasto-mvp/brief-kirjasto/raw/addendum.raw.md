# Addendum: Kirjasto brief

Detail that informs downstream work (PRD, architecture, UX) but does not belong in the 1–2 page brief.

## Parked roadmap: community hub

The user sees kirjasto as a cornerstone for a wider hub for Vietnamese people in Tampere, and later perhaps all of Finland. Later activities are expected to add value beyond lending. None are defined yet. Out of scope for the MVP; recorded here so the architecture doesn't close the door on them.

### Design notes for architecture

- **Email:** use a transactional email service (e.g. Brevo, Resend, Postmark, Amazon SES) on the library's own domain, not a personal Gmail or Proton account. A personal Gmail works (API or SMTP app password, about 500 emails a day) but mixes personal and library mail and risks restrictions. Proton has no public sending API for normal accounts; Mail Bridge needs a paid plan and a local process. Free tiers of transactional services cover the expected volume (around 100 emails a month).
- **Cover-scan cataloguing script:** plain OCR often mangles Vietnamese diacritics; a vision-capable LLM is usually more accurate. Every record gets human review before import. Scanning the back cover or copyright page as well recovers the author, publisher and ISBN. Displaying cover images publicly is common library practice but involves copyrighted images; low risk for a non-profit catalogue.
- **Build vs. adopt, honestly:** the collection is not expected to outgrow the free tiers of existing tools, so scale is not a reason to build. The reasons are a Vietnamese-first patron experience, the hub and multi-community ambition, and the portfolio goal.

## Parked roadmap: donation perks

Patrons donate books, and the library catalogues and lends them. After the MVP, each donation earns points toward perks, such as a higher loan limit or more renewals. Not in the MVP; the user explicitly parked it.

### Design note: donor record and GDPR (for architecture)

- **What it is:** a link from a donated copy to the person who donated it, plus the date. The book belongs to the association once donated; only the link to the person is personal data.
- **Data model:** a donation is its own record (copy, optional donor, received date, consent flags). The donor link can be removed (anonymised) without deleting the copy or its history.
- **MVP behaviour:** each donated copy is recorded as donated with a date. The donor field exists but is optional; no points and no consent step yet.
- **When perks arrive:** consent is the recommended legal basis ("credit this donation to my account"), with legitimate interest as the alternative. Only patrons with accounts can be credited; walk-in donors are recorded as anonymous.
- **Privacy notice from day one:** say that donations may count toward future member benefits, so enabling perks later is not a new purpose (purpose limitation).
- **Minimisation:** store donor ID, copy ID and date only. No free-text names or contact details on the donation.
- **Public display:** showing a donor's name publicly (e.g. "Donated by …") requires a separate opt-in.
- **Data subject rights:** account deletion anonymises the donor link; a "My donations" list in the patron's account supports access requests.
- **To verify before launch:** whether a Vietnamese community library's patron list counts as data revealing ethnic origin (GDPR Art. 9). Check the tietosuoja.fi guide for associations. Not legal advice.

## Parked roadmap: partner-space shelf

A borrowed shelf in a partner space (shop, restaurant, community room) as a pickup and browsing point. The user likes the idea, but it needs its own discussion and planning. Postal delivery was considered and rejected as not worth it.

## Parked roadmap: other communities

The user wants to keep kirjasto an ILS so it can later scale up and out to other communities. Both directions are wanted:
- **Multi-library:** other communities run their own library on kirjasto. Not built in the MVP; the data model must not rule it out.
- **More languages:** a late stage. The MVP ships English and Vietnamese UI on an i18n foundation, so adding languages later is translation work, not a rewrite.

## Landscape research (2026-10-28)

Web research digest, gathered during discovery. Verify before relying on any single point.

### Build vs. adopt
- **TinyCat (LibraryThing):** from $3/mo for volunteer-run libraries ($10/mo with paid staff). Circulation, unlimited patrons, self-checkout, Z39.50 import, OPAC; targets collections up to ~20k items. Vietnamese patron UI: no evidence found; check directly.
- **Libib Pro:** $9/mo or $99/yr. Unlimited patrons, public catalogue, patron logins with loan history and holds. Free Basic tier allows 5k items but has no patron management. Vietnamese UI: no evidence found.
- **Koha:** Vietnamese OPAC translation is declining (~59% in 22.11, ~49% in 24.05, ~40% in 25.05). Managed hosting ~AUD 2,150/yr; self-hosting needs real ops work. Overkill at this scale.
- **OpenBiblio:** reported discontinued. **SLiMS:** lightweight, self-hosted, Vietnamese support unverified. **Calibre-server:** browse-only, no circulation.
- **Implication:** hosted tools beat building on cost. The strongest case for building is a Vietnamese-first patron experience, if the hosted tools don't offer one.

### Catalogue metadata for Vietnamese titles
- National Library of Vietnam: no public Z39.50 server found; national bibliography metadata (~30k records since 2009) reportedly public domain.
- Open Library / Google Books: weak coverage of non-English titles; Vietnamese (978-604 ISBNs) not measured in available research.
- Finna API: free, CC0, 13M+ records. Vietnamese coverage likely small, unmeasured.
- Melinda: likely closed to non-member organisations (unverified). WorldCat Metadata API: normally needs a paid OCLC subscription.
- **Implication:** fast manual cataloguing (ISBN scan, cover photo, quick form) matters more than metadata integrations; ISBN lookup is a nice-to-have.

### GDPR for a Finnish association
- Applies regardless of size. Patrons must be informed at sign-up (privacy notice / tietosuojaseloste is common practice; tietosuoja.fi provides a template).
- A record of processing activities is required "in certain cases" for organisations under 250 staff; regular lending likely counts (inference, not stated by the source).
- Required: stated retention periods, a written data processing agreement with any hosting/SaaS provider, breach notification within 72 hours.
- Likely legal basis: contract (the loan agreement) or legitimate interest.
- Data minimisation (minimal patron fields, deleting loan history) is cheap to design in now.

### Diaspora and local context
- No documented Vietnamese community lending library in Europe running an ILS was found; existing efforts are donated bookcases, kiosks and bookstores.
- Nearby: Tiệm Mọt (online Vietnamese bookstore run from Finland); a ~280-book Vietnamese bookcase donated to Oodi, Helsinki (Oct 2025); Helsinki's Multilingual Library (Vietnamese is not a major collection there).

### Sources
- TinyCat: https://www.capterra.com/p/268527/TinyCat/ · https://www.getapp.co.uk/software/2062129/tinycat
- Libib: https://www.libib.com/pricing · https://support.libib.com/getting-started/get-libib-pro.html
- Koha: https://translate.koha-community.org/projects/koha/25-05/opac-bootstrap/vi · https://prosentient.com.au/koha/pricing · https://kohasupport.com/koha-for-small-libraries/
- National Library of Vietnam: https://ifla.org/?p=105817
- Non-English metadata coverage: https://tagteam.harvard.edu/hub_feeds/119/feed_items/15465118
- Finna API: https://avoindata.fi/data/en/dataset/finna
- GDPR: https://tietosuoja.fi/usein-kysyttya-yhdistystoiminta · https://tietosuoja.fi/-/nain-laadit-tietosuoja-asetuksen-edellyttaman-selosteen-kasittelytoimista
- Context: https://scandasia.com/vietnamese-mother-and-friends-open-online-bookstores-in-scandinavia/ · https://vietnamnews.vn/society/1727794/party-leader-s-spouse-presents-vietnamese-bookcase-to-helsinki-central-library-oodi.html

## Success-target research (2026-10-28)

Basis for the parked Success Criteria in the brief.

### Audience size (Statistics Finland, registered mother tongue, 31 Dec 2025)
- Vietnamese speakers in Tampere: 1,365, growing fast (572 in 2021 → 803 in 2023 → 1,082 in 2024 → 1,365 in 2025, about +26% in the last year). Pirkanmaa: 1,584.
- Children 0–14 in Pirkanmaa: 315, or 20% (51 aged 0–4, 125 aged 5–9, 139 aged 10–14). Adults aged 25–44: 718.
- Estimated families with children 0–14: about 200 (at about 1.6 children per family); 110–130 with a child aged 0–9. This is an estimate, not an official figure.
- Caveats: each person registers only one mother tongue, so children in mixed or Finnish-registered families are missing. Recent growth is mostly people aged 20–34, many likely without children. There were 831 Vietnamese citizens in Tampere in 2024 (a different measure).
- Source: StatFin tables 11rm and 11rl, https://pxdata.stat.fi/PxWeb/api/v1/fi/StatFin/vaerak/11rm.px

### Loans per item per year (Finnish public library statistics 2025)

| Collection | Finland | Tampere |
|---|---|---|
| Children's fiction | 4.2 | 6.8 |
| Children's non-fiction | 3.4 | 6.1 |
| Books in other languages | 2.9 | 4.5 |
| All collections | 2.9 | 5.3 |

- Median for books in small municipalities (under 5,000 residents): 1.0.
- Queensland (Australia) state library standard target: 5.0.
- Little Free Libraries: 12–139 books left or taken per week per box (UBC, 2013).
- Toy and diaspora libraries: no loan counts found.
- Sources: https://tilastot.kirjastot.fi/yearlyreports.php · https://content.plconnect.slq.qld.gov.au/sites/default/files/SLQ-librarycollectionsstandard-201308.pdf · https://news.ubc.ca/2017/11/09/building-community-through-little-free-libraries/

### Share of people who borrow
- Active borrowers (used their card that year) are 33.0% of the population in Finland and 38.6% in Tampere. Each makes about 48–49 loans a year.
- Registered-versus-active ratio: not published in Finland. Pima County, USA: 51–59% of cards active (https://www.library.pima.gov/wp-content/uploads/sites/6/2024/11/2024-11-07-Library-Advisory-Board-Circ-and-Activity-Measures-Systemwide-9-2024.pdf).

### Target arithmetic (conservative / stretch)
- **Families registered, 30 / 60:** 15% / 30% of about 200 families.
- **Active families, 15 / 35:** 50–60% of registered.
- **Loans per month by month 6, 45 / 140:** 15 × 1 pickup × 3 books; 35 × 1 × 4. Stretch matches the Finnish rate of about 4 loans per borrower per month.
- **Loans per item over 6 months, 0.5 / 1.4:**
  - Assumes the average month runs at about 60% of the month-6 rate while the service ramps up.
  - Conservative: about 160 loans over 300 books, about 1.1 a year (the small-municipality median).
  - Stretch: about 500 loans over 350 books, about 2.9 a year (the national rate for books in other languages).
  - Companion measure: 35% / 65% of titles borrowed at least once.
- **Repeat borrowing, 40% / 60%:** no benchmark.
- **Donations, +30 / +100 books:** 30 families × 25% donating × 4 books; 60 × 40% × 4. No data; pure assumption.
- **Weakest inputs:** the family-count estimate and all donation figures.
