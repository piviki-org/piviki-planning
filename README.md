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
| `piviki-core` | All state and rules, the librarian tool, jobs, the import script, and the host runtime in `deploy/` (Compose file, Caddy, deploy script and checks) | Payload 3, Next.js, PostgreSQL |
| `piviki-patron` | Static patron site | Next.js static export, Mantine, Cloudflare |
| `piviki-infra` | Shared infrastructure: OpenTofu, host bootstrap, runbooks, secrets inventory | Test on AWS EC2, production on an OVHcloud VPS; Cloudflare Tunnel, Access and R2, OpenTofu |

Code repositories are MIT licensed and live in the `piviki-org` GitHub organisation.

## Workspace layout

This repository (`piviki-planning`) holds the planning artifacts made with the [BMad Method](https://github.com/bmad-code-org/BMAD-METHOD). It is checked out as `_bmad-output/` in the workspace root, next to:

| Path | Contents |
|---|---|
| `_bmad/` | BMad installation and configuration |
| `tulossa/` | The current "coming soon" page at piviki.fi (git: `pivikirjasto/tulossa`) |
| `branding/` | Brand kit, icon and favicon |
| `piviki-core/`, `piviki-infra/` | Code repositories |
| `piviki-patron/` | Code repository for the patron site (not created yet; epic 3) |

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

The work is split into seven epics, built in the order below. Every story is built and checked on the test environment, an AWS EC2 instance at `test.piviki.fi`, `test-admin.piviki.fi` and `test-api.piviki.fi`, which never holds a real patron's data. Production is an OVHcloud VPS that starts empty in the soft-open epic: it gets the real import there, and piviki.fi switches from the coming-soon page to the patron site. Nothing moves from test to production. At the soft open, sign-up opens to anyone only if the privacy notice, usage policy and processor agreements exist; otherwise the site is browse-only, with a pilot of founder-owned accounts.

| # | Epic | Delivers | Needs | Risk |
|---|---|---|---|---|
| 1 | [Librarians keep the catalogue](initiative-kirjasto-mvp/tickets/epic-librarian-catalogue/epic-librarian-catalogue.md) | Platform baseline (scaffold, CI, deploy, OpenTofu, tenant model, i18n), manual entry with images, donated copies, review view, barcodes and labels, backups, hardening, mail | — | medium |
| 2 | [The initial collection is ready to import](initiative-kirjasto-mvp/tickets/epic-initial-load/epic-initial-load.md) | Cover-scan script and a reviewed trial import on test | 1 | high |
| 3 | [Parents find books and sign up](initiative-kirjasto-mvp/tickets/epic-patron-site/epic-patron-site.md) | Bilingual browse and search, sign-up, chat group invite, donor linking | 1 | medium |
| 7 | [Piviki soft opens](initiative-kirjasto-mvp/tickets/epic-soft-open/epic-soft-open.md) | Production starts on OVH, the real import, piviki.fi switches to the patron site, sign-up or browse-only with a pilot | 1, 2, 3 | high |
| 4 | [A book goes from reservation to return](initiative-kirjasto-mvp/tickets/epic-reservation-loop/epic-reservation-loop.md) | Reserve, queue, pickup loop, ready email, overdue block | 3, 7 | high |
| 5 | [Loans are renewed and reminded](initiative-kirjasto-mvp/tickets/epic-loan-care/epic-loan-care.md) | Self-renewal, due and overdue emails, ISBN lookup | 4 | low |
| 6 | [Piviki opens](initiative-kirjasto-mvp/tickets/epic-launch/epic-launch.md) | Retention and inactivity deletion, account deletion, lost copies, success signals, first real loan | 4, 5, 7 | high |

**Never cut:** reservations, the queue, the pickup loop, check-in, the ready-for-pickup email and the overdue block.
**If a milestone slips, cut in this order:** ISBN lookup, then self-renewal, then due-date reminders, then the English interface (multi-language support stays).

**After the MVP (parked):** donation points and perks, a shelf in a partner space, more languages, wider community-hub features. Postal delivery has been rejected.

## Progress

*Last updated: 2026-11-08*

Development has started. Every story of epic 1 is built and checked on the test environment (AWS EC2); production starts empty on an OVHcloud VPS in the soft-open epic.

| Epic, in build order | Status |
|---|---|
| 1. Librarians keep the catalogue | 🚧 In progress: 1 of 15 stories done. Story 1.1, the tracer bullet, is done: a librarian signs in at `test-admin.piviki.fi`, deployed from CI through the test host's Tunnel |
| 2. The initial collection is ready to import | ⏳ Not started |
| 3. Parents find books and sign up | ⏳ Not started |
| 7. Piviki soft opens | ⏳ Not started |
| 4. A book goes from reservation to return | ⏳ Not started |
| 5. Loans are renewed and reminded | ⏳ Not started |
| 6. Piviki opens | ⏳ Not started |

**Next:** story 1.2 (tenant, policy values and patron records), then 1.5 (images upload to public storage) and 1.3 (the whole title form). Story 1.15, added from 1.1's first deploy, separates cloudflared and Postgres from core's deploy before production starts.

## Team

Two people, part-time: a software and DevOps engineer on development, and a business master's student on product and operations. Piviki is also a portfolio project built with AI.
