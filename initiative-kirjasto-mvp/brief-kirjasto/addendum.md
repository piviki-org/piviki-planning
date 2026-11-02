# Addendum: Piviki brief

Detail that informs downstream work (PRD, architecture, UX) but doesn't belong in the 1–2 page brief.

- **MVP rules and design notes:** circulation rules and constraints for the architect.
- **Research:** landscape and success-target evidence, dated and sourced.
- **Parked roadmap:** post-MVP ideas the architecture must not rule out.

## MVP rules and design notes

### Circulation rules

- **Pickup and drop-off:** at a librarian's home. Windows are announced weekly in the community group chat, usually weekday evenings 17:00–21:00; weekends depend on the founders' plans.
- **Uncollected reservations:** the system flags a reservation 7 days after it is marked ready for pickup. A librarian contacts the patron on WhatsApp or Zalo, then expires the reservation by hand. No automatic expiry in the MVP.
- **Reminders:** email 5, 3 and 1 days before the due date, then one email a week after the due date until the book is returned.
- **Overdue consequence:** an overdue patron can't reserve until the book is returned. No fines.
- **Queues:** patrons can queue for a book that is on loan. A book with a queue can't be renewed by its current holder.
- **Phone number:** not collected at registration. A librarian adds it at the patron's first pickup, and the privacy notice must cover it.
- **Accounts:** adults only, confirmed with a tick at registration ("I confirm I am 18 or older").
- **Mission line under review:** "Keep Vietnamese alive in the next generation by making Vietnamese children's books easy to borrow."

### Architecture

- **Email:** use a transactional email service (for example, Brevo, Resend, Postmark or Amazon SES) on the library's own domain; free tiers cover the expected volume of about 100 emails a month. Rejected: a personal Gmail account (it mixes personal and library mail and risks restrictions, with a sending limit of about 500 emails a day) and Proton (no public sending API for normal accounts; Mail Bridge needs a paid plan and a local process).
- **Cover-scan cataloguing script:** plain OCR often mangles Vietnamese diacritics; a vision-capable LLM is usually more accurate. Every record gets human review before import. Scanning the back cover or copyright page as well recovers the author, publisher and ISBN. Displaying cover images publicly is common library practice but involves copyrighted images; the risk is low for a non-profit catalogue.
- **Multi-language:** the MVP ships English and Vietnamese on an internationalization (i18n) foundation, so adding languages later is translation work, not a rewrite.
- **Multi-library:** not built in the MVP, but the data model must not rule it out.
- **Donations (MVP):** each donated copy is recorded as donated, with a date. A donation is its own record (copy, optional donor, received date, consent flags), and the donor link can be removed (anonymised) without deleting the copy or its history. No points and no consent step yet; see Design notes: donor record and GDPR.
- **Build vs. adopt:** the collection isn't expected to outgrow the free tiers of existing tools, so scale isn't a reason to build. See Landscape research for the reasons to build.

### Design notes: donor record and GDPR

For when donation perks arrive. The MVP behaviour is under Architecture above.

- **What it is:** a link from a donated copy to the person who donated it, plus the date. The book belongs to the association once donated; only the link to the person is personal data.
- **Legal basis:** consent is recommended (for example, a "Credit this donation to my account" checkbox), with legitimate interest as the alternative. Only patrons with accounts can be credited; walk-in donors are recorded as anonymous.
- **Privacy notice from day one:** say that donations may count toward future member benefits, so enabling perks later isn't a new purpose (purpose limitation).
- **Minimisation:** store donor ID, copy ID and date only. No free-text names or contact details on the donation.
- **Public display:** showing a donor's name publicly (for example, "Donated by …") requires a separate opt-in.
- **Data subject rights:** account deletion anonymises the donor link; a "My donations" list in the patron's account supports access requests.

## Landscape research (2026-10-28)

Web research digest, gathered during discovery. Verify before relying on any single point.

### Build vs. adopt

**Implication:** hosted tools beat building on cost. The reasons to build are a Vietnamese-first patron experience, which no hosted tool shows evidence of offering; the hub and multi-community ambition; and the portfolio goal.

- **TinyCat (LibraryThing):** from $3 a month for volunteer-run libraries ($10 a month with paid staff). Circulation, unlimited patrons, self-checkout, Z39.50 import, and an online public catalogue (OPAC); aimed at collections up to about 20,000 items. Vietnamese patron interface: no evidence found; check directly.
- **Libib Pro:** $9 a month or $99 a year. Unlimited patrons, public catalogue, and patron logins with loan history and holds. The free Basic tier allows 5,000 items but has no patron management. Vietnamese interface: no evidence found.
- **Koha:** Vietnamese OPAC translation coverage is declining (about 59% complete in Koha 22.11, about 49% in 24.05, about 40% in 25.05). Managed hosting costs about AUD 2,150 a year; self-hosting needs real operations work. Overkill at this scale.
- **OpenBiblio:** reported discontinued. **SLiMS:** lightweight, self-hosted, Vietnamese support unverified. **Calibre-server:** browse-only, no circulation.
- Sources: TinyCat https://www.capterra.com/p/268527/TinyCat/ · https://www.getapp.co.uk/software/2062129/tinycat — Libib https://www.libib.com/pricing · https://support.libib.com/getting-started/get-libib-pro.html — Koha https://translate.koha-community.org/projects/koha/25-05/opac-bootstrap/vi · https://prosentient.com.au/koha/pricing · https://kohasupport.com/koha-for-small-libraries/

### Catalogue metadata for Vietnamese titles

**Implication:** fast manual cataloguing (ISBN scan, cover photo, quick form) matters more than metadata integrations; ISBN lookup is a nice-to-have. The cover-scan script addresses this for the initial load.

- National Library of Vietnam: no public Z39.50 server found; its national bibliography metadata (about 30,000 records since 2009) is reportedly in the public domain.
- Open Library and Google Books: weak coverage of non-English titles; coverage of Vietnamese titles (978-604 ISBNs) was not measured in the available research.
- Finna API: free, CC0, more than 13 million records. Vietnamese coverage likely small but unmeasured.
- Melinda: likely closed to non-member organisations (unverified). WorldCat Metadata API: normally needs a paid OCLC subscription.
- Sources: https://ifla.org/?p=105817 · https://tagteam.harvard.edu/hub_feeds/119/feed_items/15465118 · https://avoindata.fi/data/en/dataset/finna

### GDPR for a Finnish association

- **To verify before launch:** whether a Vietnamese community library's patron list counts as data revealing ethnic origin (GDPR Art. 9). Check the Data Protection Ombudsman's guide for associations (tietosuoja.fi). Not legal advice.
- GDPR applies regardless of size. Patrons must be informed at sign-up; a privacy notice (tietosuojaseloste) is common practice, and tietosuoja.fi provides a template.
- A record of processing activities is required "in certain cases" for organisations under 250 staff; regular lending likely triggers this requirement (inference, not stated by the source).
- Required: stated retention periods, a written data processing agreement with any hosting or SaaS provider, and breach notification within 72 hours.
- Likely legal basis: contract (the loan agreement) or legitimate interest.
- Data minimisation (minimal patron fields, deleting loan history) is cheap to design in now.
- Sources: https://tietosuoja.fi/usein-kysyttya-yhdistystoiminta · https://tietosuoja.fi/-/nain-laadit-tietosuoja-asetuksen-edellyttaman-selosteen-kasittelytoimista

### Diaspora and local context

- No documented Vietnamese community lending library in Europe running a library system was found; existing efforts are donated bookcases, kiosks and bookstores.
- Nearby: Tiệm Mọt (an online Vietnamese bookstore run from Finland); a bookcase of about 280 Vietnamese books donated to Oodi, Helsinki (October 2025); Helsinki's Multilingual Library (Vietnamese is not a major collection there).
- Local check by the founders (October 2026): the PIKI network holds about 180 Vietnamese-language books for all ages; many titles have no copy available, and the selection is poorly curated.
- Sources: https://scandasia.com/vietnamese-mother-and-friends-open-online-bookstores-in-scandinavia/ · https://vietnamnews.vn/society/1727794/party-leader-s-spouse-presents-vietnamese-bookcase-to-helsinki-central-library-oodi.html

## Success-target research (2026-10-28)

Basis for the Success Criteria in the brief, which adopts the conservative values.

### Targets

| Metric | Conservative | Stretch | Basis |
|---|---|---|---|
| Families registered | 30 | 60 | 15% / 30% of about 200 families |
| Active families | 15 | 35 | 50–60% of registered |
| Loans per month, by month 6 | 45 | 140 | 15 active families × about 3 books a month; 35 × about 4. Measured in books per family per month, not pickups: families may collect in one visit or several. Stretch matches the Finnish rate of about 4 loans per borrower per month. |
| Loans per item over 6 months | 0.5 | 1.4 | About 160 loans over 300 books (about 1.1 loans per item a year, the small-municipality median); about 500 loans over 350 books (about 2.9 loans per item a year, the national rate for books in other languages) |
| Titles borrowed at least once | 35% | 65% | Companion measure to loans per item |
| Families making two or more pickups | 40% | 60% | No benchmark |
| Books added through donations | +30 | +100 | 30 families × 25% donating × 4 books; 60 × 40% × 4. No data; pure assumption |

- Loans per item assumes the average month runs at about 60% of the month-6 rate while the service ramps up.
- **Weakest inputs:** the family-count estimate and all donation figures.

### Audience size (Statistics Finland, registered mother tongue, 31 December 2025)

- Vietnamese speakers in Tampere: 1,365, growing fast (572 in 2021 → 803 in 2023 → 1,082 in 2024 → 1,365 in 2025, about +26% in the last year). Pirkanmaa: 1,584.
- Children aged 0–14 in Pirkanmaa: 315, or 20% of Vietnamese speakers there (51 aged 0–4, 125 aged 5–9, 139 aged 10–14). Adults aged 25–44: 718.
- Estimated families with children aged 0–14: about 200 (at about 1.6 children per family); 110–130 with a child aged 0–9. This is an estimate, not an official figure.
- Caveats: each person can register only one mother tongue, so the figures undercount; children in mixed or Finnish-registered families are often missing. Recent growth is mostly people aged 20–34, many likely without children. There were 831 Vietnamese citizens in Tampere in 2024 (a different measure).
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

- Active borrowers (people who used their card that year) are 33.0% of the population in Finland and 38.6% in Tampere. Each active borrower makes about 48–49 loans a year.
- Registered-versus-active ratio: not published in Finland. Pima County, USA: 51–59% of cards active (https://www.library.pima.gov/wp-content/uploads/sites/6/2024/11/2024-11-07-Library-Advisory-Board-Circ-and-Activity-Measures-Systemwide-9-2024.pdf).

## Parked roadmap

### Community hub

The founders see Piviki as a cornerstone for a wider hub for Vietnamese people in Tampere, and later perhaps all of Finland. Later activities would add value beyond lending; none are defined yet. This section is about what the hub offers; reuse of the system by other communities is under Other communities.

### Donation perks

Patrons donate books, and the library catalogues and lends them. After the MVP, each donation would earn points toward perks, such as a higher loan limit or more renewals. The founders explicitly parked this; the data design is in Design notes: donor record and GDPR.

### Partner-space shelf

A borrowed shelf in a partner space (a shop, restaurant or community room) as a pickup and browsing point. The founders like the idea, but it needs its own discussion and planning. Postal delivery was considered and rejected as not worth it.

### Other communities

The founders want to keep Piviki an integrated library system (ILS) so it can later scale up and out:
- **Multi-library:** other communities could run their own library on Piviki.
- **More languages:** a late stage.

The MVP constraints for both are under Architecture.
