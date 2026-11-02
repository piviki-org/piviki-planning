---
title: "Product Brief: Piviki"
status: final
created: 2026-10-28
updated: 2026-10-30
---

# Product Brief: Piviki

## Executive Summary

Piviki (*pikku vietnamilainen kirjasto*, "little Vietnamese library") is a new Vietnamese community lending library in Tampere, Finland, and the small integrated library system that runs it. The library starts with about 300 Vietnamese books for school-age children and has no premises: families find and reserve books on a website, then collect them from a volunteer's home on weekday evenings. The first loan is planned for the end of December 2026.

The minimum viable product (MVP) pairs a catalogue and circulation tool for librarians with a patron website in Vietnamese and English, where parents browse the catalogue, reserve books, and track their loans. It handles what a volunteer library can't do by memory: who has which book, who is waiting for it, and when it is due back.

The mission is to keep Vietnamese alive in the next generation by making Vietnamese children's books easy to borrow. Piviki is also a portfolio project, built with AI, for its two-person team: a software and DevOps engineer on development and a business master's student on product and operations.

## The Problem

The library's 300 children's books outnumber the Vietnamese books in all Pirkanmaa public libraries combined. As of October 2026, the whole PIKI network (Tampere and the Pirkanmaa libraries) holds about 180 Vietnamese-language books for all ages. Many titles have no copy available to borrow, and the selection is poorly curated. The nearest notable collection is a donated bookcase of about 280 books at Oodi, in Helsinki.

Meanwhile, Tampere's Vietnamese community is small but growing fast: 1,365 residents had Vietnamese as their registered mother tongue at the end of 2025, up from 572 in 2021, and about one in five Vietnamese speakers in the region is a child ([Statistics Finland, StatFin 11rm](https://pxdata.stat.fi/PxWeb/api/v1/fi/StatFin/vaerak/11rm.px)).

The library has now acquired about 300 books, but families have no way to find out what's in the collection or to borrow from it. With no premises, nobody can browse a shelf, so the catalogue has to live online. With volunteers lending from home, tracking loans by memory or in a chat thread stops working after the first few dozen loans.

## Who This Serves

- **Parents of school-age children (primary).** Vietnamese, comfortable with technology, living in Tampere. They want their children to read in Vietnamese and keep the language. Accounts are adults only: a parent borrows for the child, and the system holds no data about children.
- **Librarians.** At first the two founders, later other volunteers. They need loans and queues to be clear at a glance.
- **Donors.** Patrons and other community members who give books to the collection.

## The Solution

The system gives the library a public face and a memory.

**For parents, a patron website in Vietnamese and English.** Parents sign up, browse and search the catalogue, and reserve a book. If it's on loan, they join the queue. The system emails them when a book is ready, when it's nearly due, and when it's overdue. Their account shows what they have, what they're waiting for, and when each book is due.

**For librarians, a catalogue and circulation tool.** Librarians add books quickly, with ISBN lookup where a record exists and fast manual entry where it doesn't, since most Vietnamese titles have little online metadata. Donated books are recorded as donated. Reservations move through one simple loop:

> Reserve → Ready for pickup → Picked up (the loan starts) → Returned

Pickup times are arranged in the community group chat, outside the system. The system flags uncollected reservations, blocks overdue patrons from reserving, and stops renewals of books that others are queueing for.

## What Makes This Different

The Piviki system has no technical moat; it is a close fit for this library, not a breakthrough. Hosted tools such as TinyCat and Libib are cheap, but they show no evidence of a Vietnamese patron interface. Koha is heavyweight for 300 books, and its Vietnamese translation is now only about 40% complete.

- **Vietnamese first.** Families use the library in their own language, in an interface the team controls end to end, rather than a half-translated off-the-shelf catalogue.
- **Built for a library with no building.** Off-the-shelf systems assume a building with opening hours.
- **Community owned.** The code and the data belong to the community, not to a subscription service.

## Success Criteria

Targets six months after the first loan, set conservatively because home pickup takes more effort than a public library visit. They are drafts, to be reviewed with stakeholders. The arithmetic and stretch values are in the addendum.

| Signal | Target | Why this number |
|---|---|---|
| Families registered | 30 | About 15% of the roughly 200 Vietnamese-speaking families with children in Tampere |
| Active families (at least one loan in the last 60 days) | 15 | About half of registered families, a typical active-card ratio |
| Loans per month, by month 6 | 45 | 15 active families borrowing about 3 books a month each, across however many pickups they make |
| Titles borrowed at least once | 35% | About one loan per book per year, the median for small Finnish municipalities |
| Families making two or more pickups | 40% | No benchmark; to check against the first months of data |
| Books added through donations | +30 | A guess; no data found on donation-fed collections |

## Scope

**In the MVP**
- Patron website in Vietnamese and English, built so more languages can be added later
- Adult-only patron accounts, with no data about children
- Catalogue: browse and search for patrons; quick add and edit for librarians, with ISBN lookup as a convenience
- Initial load: a Python script turns scanned cover images into catalogue records, which a librarian reviews and imports in bulk
- Reservations, queues, the pickup loop, manual expiry of uncollected reservations, and check-in
- Online self-renewal by patrons, following whatever renewal rules the stakeholders set
- Automatic email: ready for pickup, due-date reminders, and weekly overdue notices
- Overdue block; no renewal of a book with a queue
- Pickup address shown only to patrons with a confirmed pickup, never on the public site
- Donations recorded on each copy; the donor field exists but is optional
- GDPR basics: privacy notice, minimal data, a stated retention period, and account deletion

**Decided later, with stakeholders.** Placeholders in the system, not blockers:
- Loan period, maximum books per patron, and renewal rules and limits
- Firmer overdue consequences beyond the block, such as a probation period or strikes

**Out of the MVP** (parked in the addendum)
- Donation points and perks
- A shelf in a partner space
- Postal delivery (rejected)
- Running more than one library in one system; the data model must not rule it out
- Languages beyond Vietnamese and English
- Wider community-hub features

## Milestones

Internal targets, not a public promise. The library opens when the software is ready. A coming-soon page is already live at [piviki.fi](https://piviki.fi), with links to the library's Facebook and Instagram accounts; the patron website replaces it at launch.

| Milestone | Target |
|---|---|
| Librarian catalogue system in use; the 300 books imported from cover scans | End of November 2026 |
| Patron website live: sign-up, browse, reserve, queue, and email | Mid-December 2026 |
| First real loan | End of December 2026 |

**If a milestone slips**, cut in this order: ISBN lookup, then online self-renewal, then the due-date reminders (the overdue notice stays), then the English interface (multi-language support stays). Never cut: reservations, the queue, the pickup loop, check-in, the ready-for-pickup email, and the overdue block.

## Risks and Open Questions

- **A tight timeline.** About five weeks to the catalogue and seven to the patron site, for a two-person part-time team. The cut order in Milestones is the fallback.
- **GDPR.** Before launch, check the Data Protection Ombudsman's guide for associations (tietosuoja.fi) to see whether a Vietnamese community library's patron list counts as data revealing ethnic origin, which GDPR protects more strictly.
- **Dependence on two volunteers.** Two people run the library, the software, and the pickups. If either steps back, both the service and the system are at risk.
- **The Multilingual Library.** Finnish libraries can borrow deposit collections from Helsinki's Multilingual Library. Whether Tampere does this for Vietnamese is still to be checked.
- **Under review with stakeholders.** The mission line and the success targets are drafts.

## Vision

If Piviki works in Tampere, it grows in two directions. Within the community, the library becomes the first piece of a wider hub for Vietnamese people in Tampere. Its collection grows through donations and grants, donation perks reward the families who build it, and it may gain a shelf in a partner space. Beyond Tampere, other communities run their own libraries on the same system: first Vietnamese groups elsewhere in Finland, later communities that speak other languages. The aim behind both stays the same: keeping children reading in their family's language and keeping the community connected.
