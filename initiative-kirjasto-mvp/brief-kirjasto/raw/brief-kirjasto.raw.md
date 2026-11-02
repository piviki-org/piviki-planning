---
title: "Product Brief: Kirjasto"
status: draft
created: 2026-10-28
updated: 2026-10-28
---

# Product Brief: Kirjasto

## Executive Summary

Kirjasto is a small integrated library system (ILS) for a new Vietnamese community lending library in Tampere, Finland. The library starts with about 300 Vietnamese books for school-age children and has no premises: families find and reserve books on a website, then collect them from a volunteer's home on weekday evenings.

The MVP has two parts: a catalogue and circulation tool for the librarians, and a patron website in Vietnamese and English where parents browse, reserve, queue for and track their loans. It handles the work a volunteer library can't do by memory: who has which book, who is waiting for it, and when it is due back.

The mission is to keep Vietnamese alive in the next generation by making Vietnamese children's books easy to borrow. Kirjasto is also a portfolio project for its two-person team, a software and DevOps engineer on development and a business master's student on product and operations, built with AI. It is a cornerstone designed to scale to other communities and languages later.

## The Problem

Tampere's Vietnamese community is small but growing fast: 1,365 residents had Vietnamese as their registered mother tongue at the end of 2025, up from 572 in 2021, and about one in five Vietnamese speakers in the region is a child ([Statistics Finland, StatFin 11rm](https://pxdata.stat.fi/PxWeb/api/v1/fi/StatFin/vaerak/11rm.px)). Those figures undercount, because each person registers only one mother tongue, so children in mixed families are often missing.

Vietnamese families in Tampere who want their children reading Vietnamese have few places to borrow Vietnamese books. The whole PIKI network (Tampere and the Pirkanmaa libraries) holds about 180 Vietnamese-language books for all ages, checked in October 2026. Many have no copy available to borrow, and the selection is poorly curated. Kirjasto's 300 children's books alone outnumber the Vietnamese books in every public library in Pirkanmaa combined. The nearest notable collection is a donated bookcase of about 280 books at Oodi, in Helsinki.

A collection of about 300 books has now been acquired, but there is no way for families to find out what's in it or borrow from it. With no premises, nobody can browse a shelf, so the catalogue has to live online. With volunteers lending from home, tracking loans by memory or in a chat thread stops working after the first few dozen loans.

Existing library systems don't fit well. Hosted tools such as TinyCat and Libib are cheap, but show no evidence of a Vietnamese patron interface. Koha's Vietnamese translation has fallen to about 40%, and it is heavyweight for 300 books.

## The Solution

Kirjasto gives the library a public face and a memory.

**For parents, a patron website in Vietnamese and English.** Parents sign up (adults only, confirmed with a tick at registration), browse and search the catalogue, and reserve a book. If it's on loan, they join the queue. The system emails them when the book is ready, reminds them 5, 3 and 1 days before it's due, and weekly once it's overdue. Their account shows what they have, what they're waiting for, and when each book is due.

**For librarians, a catalogue and circulation tool.** Librarians add books quickly, with ISBN lookup where it exists and fast manual entry where it doesn't, since most Vietnamese titles have little online metadata. Donated books are recorded as donated. Reservations move through one simple loop:

> Reserve → Ready for pickup → Picked up (the loan starts) → Returned

Pickup times are arranged in the community group chat, outside the system. Kirjasto flags reservations that have been ready for 7 days without pickup, so a librarian can message the patron before expiring the reservation by hand. It also blocks overdue patrons from reserving until the book comes back, and stops renewals of any book with a queue. Librarians add a patron's phone number at their first pickup.

## What Makes This Different

Kirjasto has no technical moat; it is a fit, not a breakthrough.

- **Vietnamese first.** Families use the library in their own language, in an interface the team controls end to end, rather than a half-translated off-the-shelf catalogue.
- **Built for how this library actually works.** Reserve online, pick up at a volunteer's home, coordinate in a group chat. Off-the-shelf systems assume a building with opening hours.
- **Community owned.** The code and the data belong to the community, not to a subscription service.
- **Made to be reused.** Kirjasto keeps the door open for other diaspora communities to run their own library on it later, in their own language.

## Who This Serves

- **Parents of school-age children (primary).** Vietnamese, comfortable with technology, living in Tampere. They want their children to read and keep their Vietnamese. They browse and reserve on the website and collect books in person. Accounts are adults only: a parent borrows for the child, and the system holds no data about children.
- **Librarians.** At first the two founders, later other volunteers. They catalogue the collection, including donations, mark reservations ready for pickup, check books out and in, and chase late returns. They need loans and queues to be clear at a glance.
- **Donors.** Patrons and other community members who give books. In the MVP a donation is recorded as part of cataloguing. Crediting donors (perks) comes later.

## Success Criteria

Targets for six months after the first loan. They are deliberately conservative: collecting books from a volunteer's home on a weekday evening takes more effort than a visit to a public library. The arithmetic, sources and stretch values are in the addendum.

| Signal | Target | Why this number |
|---|---|---|
| Families registered | 30 | About 15% of the roughly 200 Vietnamese-speaking families with children in Tampere |
| Active families (at least one loan in the last 60 days) | 15 | About half of registered families, a typical active-card ratio |
| Loans per month, by month 6 | 45 | 15 families × 3 books per pickup |
| Titles borrowed at least once | 35% | About one loan per book per year, the median for small Finnish municipalities |
| Families making two or more pickups | 40% | No benchmark; to check against the first months of data |
| Books added through donations | +30 | A guess; no data found on donation-fed collections |

## Scope

**In the MVP**
- Patron website in Vietnamese and English, on an i18n foundation so more languages can be added later
- Adult-only patron accounts, with no data about children
- Catalogue: browse and search for patrons; quick add and edit for librarians, with ISBN lookup as a convenience
- Initial load: a Python script turns scanned cover images into catalogue records, which a librarian reviews and imports in bulk
- Reservations, queues, the pickup loop, manual expiry of uncollected reservations, check-in
- Automatic email: ready for pickup, due-date reminders, weekly overdue notices
- Overdue block; no renewal of a book with a queue
- Donations recorded on each copy; the donor field exists but is optional
- GDPR basics: privacy notice, minimal data, a stated retention period, account deletion

**Decided later, with stakeholders.** Placeholders in the system, not blockers:
- Loan period, maximum books per patron, renewals and their limit, whether patrons can renew online
- Firmer overdue consequences beyond the block, such as a probation period or strikes

**Out of the MVP** (parked in the addendum)
- Donation points and perks
- A shelf in a partner space
- Postal delivery (rejected)
- Running more than one library in one system; the data model must not rule it out
- Languages beyond Vietnamese and English
- Wider community-hub features

## Milestones

Internal targets, not a public promise. The library opens when the software is ready.

| Milestone | Target |
|---|---|
| Librarian catalogue system in use; the 300 books imported from cover scans | End of November 2026 |
| Patron website live: sign-up, browse, reserve, queue, email | Mid-December 2026 |
| First real loan | End of December 2026 |

**If a milestone slips**, cut in this order: ISBN lookup, then online self-renewal, then the 5/3/1-day due reminders (the overdue notice stays), then the English UI (the i18n foundation stays). Never cut: reservations, the queue, the pickup loop, check-in, the ready-for-pickup email and the overdue block.

## Risks and Open Questions

- **A tight timeline.** The library opens when the software is ready, against internal targets of about five weeks for the catalogue and seven for the patron site, for a two-person part-time team. Hold the scope line, and agree in advance what gets cut if a date slips.
- **GDPR.** The library must check whether a Vietnamese community library's patron list counts as data revealing ethnic origin, which GDPR protects more strictly. Check this against the tietosuoja.fi guide for associations before launch.
- **Home address.** Pickup is at a librarian's home. The address must never appear on the public site; it goes only to patrons with a confirmed pickup.
- **Volunteer bus factor.** Two people run the library, the software and the pickups. If either steps back, both the service and the system are at risk.
- **The Multilingual Library.** Finnish libraries can borrow deposit collections from Helsinki's Multilingual Library. Whether Tampere does this for Vietnamese is still to be checked.
- **Mission wording.** "Keep Vietnamese alive in the next generation by making Vietnamese children's books easy to borrow" is a draft, to be reviewed with stakeholders.

## Vision

If Kirjasto works in Tampere, it grows in two directions. Within the community, the library becomes the first piece of a wider hub for Vietnamese people in Tampere, with a collection that grows through donations and grants, donation perks rewarding the families who build it, and perhaps a shelf in a partner space. Beyond it, other communities, first Vietnamese groups elsewhere in Finland and later other languages, run their own libraries on the same system. The aim behind both stays the same: keeping children reading in their family's language and keeping the community connected.
