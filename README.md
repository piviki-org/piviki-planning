# Piviki

**Piviki** (*pikku vietnamilainen kirjasto*, "little Vietnamese library") is a Vietnamese community lending library in Tampere, Finland, and the small library system that runs it.

The library starts with an initial collection of about 300 Vietnamese children's books and has no premises. Families find and reserve books on a website in Vietnamese or English, then pick them up from a volunteer's home. The system keeps track of what volunteers can't hold in their heads past the first few dozen loans: who has which book, who is waiting for it, and when it is due back.

**Mission:** keep Vietnamese alive in the next generation by making Vietnamese children's books easy to borrow.

## What the MVP does

- **Patron website (Vietnamese and English):** anyone can browse and search the catalogue, with or without diacritics. Adults can sign up, reserve a book or join its queue, renew a loan and see their loans, and they get emails when a book is ready, nearly due or overdue.
- **Librarian tool (English):** add books by manual entry or ISBN lookup, record donated copies, and move a reservation through *Reserve → Ready for pickup → Picked up → Returned*, with loans and queues visible at a glance.
- **Cover-scan import:** a one-time script turns scanned covers into catalogue records, and a librarian reviews every record before the bulk import.
- **No fines.** A patron with an overdue book can't reserve, queue or renew until they return it.

## Architecture at a glance

A modular monolith behind a versioned API, with a static headless frontend. Details are in the [architecture spine](initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md).

| Repository | Role | Stack |
|---|---|---|
| `piviki-core` | All state and rules, the librarian tool, jobs, the import script | Payload 3, Next.js, PostgreSQL |
| `piviki-patron` | Static patron site | Next.js static export, Mantine, Cloudflare |
| `piviki-infra` | Infrastructure, Compose file, runbooks | AWS EC2 until the move, then OVHcloud VPS; AWS test environment after; Cloudflare Tunnel/R2, OpenTofu |

Code repositories are MIT licensed and live in the `piviki-org` GitHub organisation.

## Workspace layout

This repository (`piviki-planning`) holds the planning artifacts made with the [BMad Method](https://github.com/bmad-code-org/BMAD-METHOD). It is checked out as `_bmad-output/` in the workspace root, next to:

| Path | Contents |
|---|---|
| `_bmad/` | BMad installation and configuration |
| `tulossa/` | The current "coming soon" page at piviki.fi (git: `pivikirjasto/tulossa`) |
| `branding/` | Brand kit, icon and favicon |
| `piviki-core/`, `piviki-patron/`, `piviki-infra/` | Code repositories (not created yet) |

Key planning documents:

- [Product brief](initiative-kirjasto-mvp/brief-kirjasto/brief-kirjasto.md) and [addendum](initiative-kirjasto-mvp/brief-kirjasto/addendum.md)
- [Initiative: Piviki MVP](initiative-kirjasto-mvp/initiative-kirjasto-mvp.md): requirements R1–R25, constraints, decisions, open questions
- [MARC 21 and ILS compatibility research](initiative-kirjasto-mvp/research-marc21-and-ils-compatibility/research-marc21-and-ils-compatibility.md)
- [Architecture spine](initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md) and its [walkthrough](initiative-kirjasto-mvp/walkthrough-architecture-piviki/walkthrough-architecture-piviki.md)
- [Ticket tree](initiative-kirjasto-mvp/tickets/tickets.toml)

## Roadmap

The milestones are internal targets. The library opens when the software is ready.

| Milestone | Target |
|---|---|
| Catalogue in use, initial collection (about 300 books) imported | End of November 2026 |
| Patron website live at piviki.fi | Mid-December 2026 |
| First real loan | End of December 2026 |

The work is split into six epics, built in this order. Every epic deploys to production at a non-public address (`beta.piviki.fi`), and sign-up stays closed to real patrons until the launch epic. Production runs on AWS EC2 until right after the initial load, then moves to an OVHcloud VPS, and the AWS instance becomes a test environment.

| # | Epic | Delivers | Needs | Risk |
|---|---|---|---|---|
| 1 | [Librarians keep the catalogue](initiative-kirjasto-mvp/tickets/epic-librarian-catalogue/epic-librarian-catalogue.md) | Platform baseline (scaffold, CI, deploy, tenant model, i18n), manual entry, donated copies | — | medium |
| 2 | [The initial collection is imported](initiative-kirjasto-mvp/tickets/epic-initial-load/epic-initial-load.md) | Cover-scan script, review, bulk import | 1 | high |
| 3 | [Parents find books and sign up](initiative-kirjasto-mvp/tickets/epic-patron-site/epic-patron-site.md) | Bilingual browse and search, sign-up, chat group invite, donor linking | 1, 2 (the move) | medium |
| 4 | [A book goes from reservation to return](initiative-kirjasto-mvp/tickets/epic-reservation-loop/epic-reservation-loop.md) | Reserve, queue, pickup loop, ready email, overdue block | 3 | high |
| 5 | [Loans are renewed and reminded](initiative-kirjasto-mvp/tickets/epic-loan-care/epic-loan-care.md) | Self-renewal, due and overdue emails, ISBN lookup | 4 | low |
| 6 | [Piviki opens](initiative-kirjasto-mvp/tickets/epic-launch/epic-launch.md) | Privacy notice, usage policy, account deletion, piviki.fi switch, first loan | 4, 5 | high |

**Never cut:** reservations, the queue, the pickup loop, check-in, the ready-for-pickup email and the overdue block.
**If a milestone slips, cut in this order:** ISBN lookup, then self-renewal, then due-date reminders, then the English interface (multi-language support stays).

**After the MVP (parked):** donation points and perks, a shelf in a partner space, more languages, wider community-hub features. Postal delivery has been rejected.

## Progress

*Last updated: 2026-11-03*

Planning and architecture are done. Development has not started.

| Epic | Status |
|---|---|
| 1. Librarians keep the catalogue | ⏳ Up next |
| 2. The initial collection is imported | ⏳ Not started |
| 3. Parents find books and sign up | ⏳ Not started |
| 4. A book goes from reservation to return | ⏳ Not started |
| 5. Loans are renewed and reminded | ⏳ Not started |
| 6. Piviki opens | ⏳ Not started |

**Next:** build epic 1, the platform and the librarian catalogue tool, so the initial collection can be imported by the end of November.

## Team

Two people, part-time: a software and DevOps engineer on development, and a business master's student on product and operations. Piviki is also a portfolio project built with AI.
