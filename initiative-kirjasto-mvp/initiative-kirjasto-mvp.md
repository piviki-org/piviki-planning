---
type: initiative
title: Piviki MVP
parent: none
sources:
  - _bmad-output/initiative-kirjasto-mvp/brief-kirjasto/brief-kirjasto.md
  - _bmad-output/initiative-kirjasto-mvp/brief-kirjasto/addendum.md
companions:
  - _bmad-output/initiative-kirjasto-mvp/brief-kirjasto/addendum.md
after: []
risk: high
---

# Piviki MVP

## Description

Piviki, a Vietnamese community lending library in Tampere with about 300 children's books and no premises, runs on its own small library system: a catalogue and circulation tool for librarians and a Vietnamese and English website where parents browse, reserve and track loans. When this is done, volunteers no longer track loans by memory, families can find and borrow the collection online, and the first real loan has gone through the system, targeted for the end of December 2026. Inputs are the Piviki brief and its addendum.

## Outcome

Vietnamese-speaking parents in Tampere can find and borrow Vietnamese children's books for their children, and librarians can see who has which book, who is waiting and when it is due, measured by the brief's Success Criteria six months after the first loan.

## Requirements

- R1 (brief Scope, addendum Multi-language): A patron can use every patron-facing screen in Vietnamese or English, and adding another language needs translation work only.
- R2 (brief Scope, Who This Serves; addendum Accounts, Phone number; Decision 2026-10-31): An adult can sign up after ticking "I confirm I am 18 or older" and accepting the usage policy, and is then asked to join the community chat group; sign-up asks for no phone number and the system holds no data about children.
- R3 (brief Scope, Solution; Decision 2026-10-31): Anyone can browse and search the catalogue without an account, with or without typing Vietnamese diacritics, and sees whether a book is available and its front cover, back cover and, where scanned, its publishing information page.
- R4 (brief Solution, addendum Queues; Decision 2026-10-31): A patron can reserve an available title, or join the first-come, first-served queue for a title on loan, and sees the request on their account; a copy is assigned when the reservation is marked ready, and a returned book or expired reservation passes to the next patron in line.
- R5 (brief Solution): A patron's account shows what they have, what they are waiting for, and when each book is due.
- R6 (brief Scope, addendum Queues): A patron can renew a loan online within the renewal rules in force, and a renewal is refused when others are queueing for the book.
- R7 (brief Scope, addendum Overdue consequence; Decision 2026-10-31): A patron with an overdue book cannot reserve, join a queue or renew any loan until it is returned, and no fine is ever charged.
- R8 (brief Solution, Scope): A patron receives an email when a reserved book is ready for pickup.
- R9 (addendum Reminders): A patron receives an email 5, 3 and 1 days before a loan's due date.
- R10 (addendum Reminders): A patron with an overdue book receives one email a week until it is returned.
- R11 (brief Solution, Scope): A librarian can add and edit a book by fast manual entry, without any online metadata existing for it.
- R12 (brief Scope, Solution): A librarian adding a book can fill its record by ISBN lookup when a record exists.
- R13 (brief Scope, addendum Cover-scan cataloguing script; Decision 2026-10-31): A script turns scanned front covers, back covers and publishing information pages into catalogue records with Vietnamese diacritics intact, and a librarian reviews every record before it is imported in bulk.
- R14 (brief Solution): A librarian can move a reservation through Reserve, Ready for pickup, Picked up (the loan starts) and Returned, and can see current loans and queues at a glance.
- R15 (addendum Uncollected reservations; Decision 2026-10-31): 7 days after a reservation is marked ready for pickup, it is flagged for a librarian to expire by hand, and nothing expires automatically; the librarian tool makes the reservations of patrons with no phone number stand out as needing no chase-up.
- R16 (brief Scope; Decision 2026-10-31): The pickup address never appears on the public site; patrons receive it through the community chat group they are asked to join at sign-up, where a librarian approves each join.
- R17 (brief Scope, addendum Donations; Decision 2026-10-31): A librarian can record a copy as donated with a date and an optional donor, the donor link can be removed without deleting the copy or its history, and no donor name is shown to patrons or the public.
- R18 (addendum Phone number): A librarian can add a patron's phone number at their first pickup.
- R19 (brief Scope, addendum GDPR, Design notes; Decision 2026-10-31): A patron can read a privacy notice that states the retention period, starting at 3 years, and covers the phone number and possible future donation benefits; they can delete their account unless they still hold a book, and deletion cancels their reservations.
- R20 (brief Decided later; Decision 2026-10-31): Loan period, maximum books per patron, renewal rules and limits, retention period, uncollected-reservation window and chat group link are set per tenant, and each can be changed without a code change or blocking launch.
- R21 (brief Milestones): The patron website replaces the coming-soon page at piviki.fi at launch.
- R22 (Decision 2026-10-31): Librarians use their tool in English, and translating it later needs translation work only.
- R23 (Decision 2026-10-31): A librarian can delete a patron's account even while they hold a book; the held copies are marked lost and the patron's reservations are cancelled; when a title's only copy is marked lost, its queue places are cancelled and those patrons are emailed.
- R24 (Decision 2026-10-31): A patron can cancel their own reservation or leave a queue, and the book passes to the next patron in line.
- R25 (Decision 2026-10-31): Every library-owned record belongs to a tenant and Piviki is the only tenant, so a second tenant could be added without changing the data model.

## Constraints

- Never cut: reservations, the queue, the pickup loop, check-in, the ready-for-pickup email and the overdue block (brief Milestones).
- If a milestone slips, cut in this order: ISBN lookup (R12), then self-renewal (R6), then due-date reminders (R9), then the English interface while multi-language support stays (R1); the overdue notice (R10) stays (brief Milestones).
- Milestones are internal targets: catalogue in use with the 300 books imported by end of November 2026, patron website live by mid-December 2026, first real loan by end of December 2026; the library opens when the software is ready (brief Milestones).
- A two-person part-time team builds and runs this, which is why the cut order exists (brief Risks).
- Pickup and drop-off happen at a librarian's home, and pickup times are arranged in the community group chat, outside the system (brief Solution, addendum Pickup and drop-off).
- Tenancy stays minimal: tenant ownership of records and per-tenant policy values only, and it is not in the cut order because adding it later is the cost it avoids; more languages must remain translation work (Decision 2026-10-31, addendum Multi-library, Multi-language).
- The code and the data belong to the community, not to a subscription service (brief What Makes This Different).
- Vietnamese is the first language of the experience, in an interface the team controls end to end (brief What Makes This Different).
- Email goes through a transactional email service on the library's own domain; a personal Gmail account and Proton are rejected (addendum Email).
- Data is minimal, retention periods are stated, each hosting or SaaS provider has a written data processing agreement, and a breach can be notified within 72 hours (addendum GDPR).
- Donor data, when kept, is donor ID, copy ID and date only, with no free-text names or contact details; only patrons with accounts can be credited, and walk-in donors are anonymous; account deletion anonymises the link (addendum Design notes).
- A donor's name is never shown publicly without a separate opt-in (addendum Design notes).

## Done when

- The librarian tool is in use and the 300 books are imported from cover scans after librarian review.
- The patron website is live at piviki.fi in place of the coming-soon page, with sign-up, browse, reserve, queue and email working in production.
- A first real loan has gone through Reserve, Ready for pickup, Picked up and Returned, or is on loan with those steps recorded so far.
- The GDPR question on ethnic-origin data is answered from the Data Protection Ombudsman's guide, and the privacy notice and the usage policy, written by the other founder, are published before launch.
- Every signal in the brief's Success Criteria can be read from the system's records, ready for the six-month review.

## Boundaries

- Out: donation points and perks; a shelf in a partner space; postal delivery (rejected); languages beyond Vietnamese and English; wider community-hub features (brief Scope).
- Out: a second tenant, tenant sign-up or admin screens, per-tenant branding or domains, and tenant switching (Decision 2026-10-31).
- Out: firmer overdue consequences such as probation or strikes, automatic expiry of uncollected reservations, fines, and scheduling pickups in the system (brief Scope, Solution; addendum Circulation rules).
- Touch point: piviki.fi coming-soon page, replaced at launch by the patron website; linked Facebook and Instagram accounts stay outside the system; owner: epic-launch.
- Touch point: community group chat, WhatsApp and Zalo, used by people for the pickup address, pickup windows, join approval and contact; the system only shares the invitation at sign-up; owner: epic-patron-site.
- Touch point: transactional email service on the library's own domain; owner: epic-reservation-loop.
- Tracer path: a librarian imports a book, a parent signs up, joins the chat group and reserves it, the librarian marks it ready, the parent gets the email, the book is picked up, then reminded about and returned.

## References

- brief — _bmad-output/initiative-kirjasto-mvp/brief-kirjasto/brief-kirjasto.md, sections The Solution, Scope, Milestones, Success Criteria
- rules and design notes — _bmad-output/initiative-kirjasto-mvp/brief-kirjasto/addendum.md, sections MVP rules and design notes, Landscape research (GDPR), Parked roadmap

## Notes

- Decision: 2026-10-31: The catalogue can be browsed and searched without an account.
- Decision: 2026-10-31: Queues are first come, first served, and an expired reservation or returned book passes to the next patron.
- Decision: 2026-10-31: Search works with or without diacritics.
- Decision: 2026-10-31: Emails go out in the language the patron chose; the librarian tool stays in English on the same i18n foundation.
- Decision: 2026-10-31: The public catalogue shows the front cover, back cover and possibly the publishing information page.
- Decision: 2026-10-31: The address reaches patrons through the chat group they are asked to join at sign-up, replacing the brief's rule that it is shown only to patrons with a confirmed pickup.
- Decision: 2026-10-31: A first-time patron with no phone number on record needs no chase-up: a librarian still clears their reservation by hand when the window ends, and the librarian tool makes those reservations stand out.
- Decision: 2026-10-31: Retention is configurable and starts at 3 years.
- Decision: 2026-10-31: Donor consent is taken at account creation under the usage policy, replacing the addendum's "no consent step yet"; donor names are not shown for now.
- Decision: 2026-10-31: A patron cannot delete their account while holding a book; a librarian can, and marks the books lost; both cancel reservations.
- Decision: 2026-10-31: Patrons can cancel a reservation or leave a queue.
- Decision: 2026-10-31: Reservations and queues are per title, with a copy assigned when marked ready.
- Decision: 2026-10-31: The overdue block also stops joining a queue and renewing other loans.
- Decision: 2026-10-31: The system is multi-tenant in its data and policy only, with Piviki as the sole tenant, replacing the brief's "must not rule it out"; nothing fancier is built now.
- Decision: 2026-10-31: Retention counts from a loan's return, loan records are then anonymised, and deleting an account anonymises that patron's loan history.
- Decision: 2026-10-31: Donor consent is a separate, optional, unticked box at sign-up.
- Decision: 2026-10-31: A patron account belongs to one tenant.
- Decision: 2026-10-31: Developers change per-tenant policy values, with no admin screen in the MVP.
- Decision: 2026-10-31: The back cover and publishing information page are shown publicly in the catalogue, not only scanned for data.
- Decision: 2026-10-31: A librarian approves each chat group join.
- Decision: 2026-10-31: The other founder writes the usage policy, and it blocks launch.
- Decision: 2026-10-31: When a title's only copy is marked lost, its queue places are cancelled and those patrons are emailed.
- Decision: 2026-10-31: The name is Piviki, for the library and the tenant.
- Decision: 2026-11-01: Sliced into six epics, cut by capability delivered together, with the cuttable loan work in its own epic and the platform baseline in epic-librarian-catalogue.
- Decision: 2026-11-01: Every epic deploys to production at a non-public address; the coming-soon page stays and sign-up stays closed to real patrons until epic-launch.
- Decision: 2026-11-01: ISBN lookup (R12) moves from epic-librarian-catalogue to epic-loan-care because it is a nice-to-have; manual entry and the cover-scan import carry the catalogue milestone.
- Decision: 2026-11-01: The catalogue comes first rather than reserve-to-pickup, because reservations need books and accounts and the catalogue is the first milestone.
- Open question: What loan period, maximum books and renewal limits apply at the first real loan, and who confirms them? Stakeholders decide; due dates and reminders need a value by end of December 2026.
- Open question: Does a Vietnamese community library's patron list count as data revealing ethnic origin under GDPR Art. 9? Founders to check tietosuoja.fi before launch; the answer may change the legal basis and the sign-up wording.
- Open question: Which licence makes the code "community owned"? Founders decide before the repository is public.
- Open question: Does Tampere use Helsinki's Multilingual Library deposit collections for Vietnamese? Founders to check; does not block the build.
- Open question: Stakeholders are still reviewing the mission line and the success targets, which are drafts.
