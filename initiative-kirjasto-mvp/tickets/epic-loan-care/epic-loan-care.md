---
type: epic
title: "Loans are renewed and reminded"
parent: initiative-kirjasto-mvp
covers: [R6, R9, R10, R12]
sources: []
companions: []
after: []
refined: false
risk: low
---

# Loans are renewed and reminded

## Description

Patrons renew online within the rules, and get due-date reminders and weekly overdue notices. Librarians can also fill a book record by ISBN lookup, a nice-to-have placed here because this epic holds the cuttable scope.

## Outcome

For parents, loans are easier to keep on time: they renew online and are reminded before and after the due date.

## Done when

- A patron renews within the rules, and is refused when others queue for the book or they have an overdue book.
- Reminder emails arrive 5, 3 and 1 days before the due date.
- One overdue email goes out each week until the book is returned.
- A librarian fills a book record by ISBN lookup when a record exists, and manual entry still works when none does.
- Deployed to production and verified end to end with the epics it builds on.

## Boundaries

- Holds most of the cuttable scope: ISBN lookup (R12), self-renewal (R6) and reminders (R9) may be cut per the initiative's cut order; the overdue notice (R10) is never cut and then moves to its own story here.
- Not here: the overdue block itself (epic-reservation-loop).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
