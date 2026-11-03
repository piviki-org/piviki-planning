# Walkthrough: Piviki MVP architecture spine

Review complete 2026-11-03: all 11 blocks accepted; blocks 1, 5, 6, 9, 10, 11 changed during review.

- **Target:** the whole spine, [architecture-piviki.md](../architecture-piviki/architecture-piviki.md) (status final, updated 2026-11-03)
- **Reviewer:** the user
- **Started:** 2026-11-03

## Blocks

- [x] 1. Intent (done, changed during review)
- [x] 2. Broad strokes (done, unchanged)
- [x] 3. Core state and circulation (done, unchanged)
- [x] 4. APIs and patron delivery (done, unchanged)
- [x] 5. Tenancy, identities and sign-up (done, changed during review: AD-7 cookie clause dropped)
- [x] 6. Catalogue (updated today) (done, changed during review: sourceRecordId removed from AD-20)
- [x] 7. Language, email and jobs (done, unchanged)
- [x] 8. Personal data and processors (done, unchanged)
- [x] 9. Operations and collection ownership (done, changed during review)
- [x] 10. Conventions and stack (done, changed during review)
- [x] 11. Periphery (done, changed during review)

## 1. Intent

> Piviki, a Vietnamese community lending library in Tampere with about 300 children's books and no premises, runs on its own small library system: a catalogue and circulation tool for librarians and a Vietnamese and English website where parents browse, reserve and track loans. When this is done, volunteers no longer track loans by memory, families can find and borrow the collection online, and the first real loan has gone through the system, targeted for the end of December 2026. Inputs are the Piviki brief and its addendum.

Correction from the reviewer: the 300 books are the library's initial collection, contributed by the founders, not the whole or final collection; the collection is expected to grow.

The spine's job is to fix only the rules that keep the six epics (librarian catalogue, initial load, patron site, reservation loop, loan care, launch) consistent when built separately, including books added after the initial load.

Source: [initiative ticket Description](../initiative-kirjasto-mvp.md#L16) and the [spine frontmatter](../architecture-piviki/architecture-piviki.md#L1).

## 2. Broad strokes

A modular monolith behind a versioned API, with a static headless frontend: `piviki-core` holds all state and rules, and everything else is a client of it.
Entry points: [Design Paradigm](../architecture-piviki/architecture-piviki.md#L20) (the layers and who owns what), [Structural Seed](../architecture-piviki/architecture-piviki.md#L292) (system and data diagrams), [Invariants & Rules](../architecture-piviki/architecture-piviki.md#L35) (the 20 decisions), [Capability → Architecture Map](../architecture-piviki/architecture-piviki.md#L342) (which requirement lives where).

## 3. Core state and circulation

Core owns all rules; one module changes circulation; one reservation entity and one availability formula; time-based states are derived at read time.
[AD-1](../architecture-piviki/architecture-piviki.md#L50), [AD-2](../architecture-piviki/architecture-piviki.md#L56), [AD-3](../architecture-piviki/architecture-piviki.md#L67), [AD-4](../architecture-piviki/architecture-piviki.md#L79).

## 4. APIs and patron delivery

Two API hosts with separate callers, a static patron site with a small preview Worker, and separate buckets for media and backups.
[AD-5](../architecture-piviki/architecture-piviki.md#L85), [AD-15](../architecture-piviki/architecture-piviki.md#L189), [AD-16](../architecture-piviki/architecture-piviki.md#L198).

## 5. Tenancy, identities and sign-up

Tenancy as data and one policy record; patrons, librarians and the scan client as separate identities in core; the sign-up gate, Google linking and the pre-launch period.
[AD-6](../architecture-piviki/architecture-piviki.md#L98), [AD-7](../architecture-piviki/architecture-piviki.md#L109), [AD-8](../architecture-piviki/architecture-piviki.md#L120).

Changed during review: AD-7 clause on per-hostname cookie names dropped (host-only cookies already separate the hosts). AD-7 was later amended in block 9: the scan-client identity was removed, leaving patrons and librarians.

## 6. Catalogue (updated today)

Barcodes, duplicate matching, the initial import and labels; one Vietnamese text normalisation; one bibliographic shape for every writer, including series.
[AD-9](../architecture-piviki/architecture-piviki.md#L130), [AD-10](../architecture-piviki/architecture-piviki.md#L145), [AD-20](../architecture-piviki/architecture-piviki.md#L240).

Changed during review: AD-20 `sourceRecordId` field dropped for now. AD-9 was later amended in block 9 (scan-clients identity removed).

## 7. Language, email and jobs

Language codes and message files; all library email through one outbox; all scheduled work in one Jobs Queue in the core container.
[AD-11](../architecture-piviki/architecture-piviki.md#L151), [AD-12](../architecture-piviki/architecture-piviki.md#L157), [AD-13](../architecture-piviki/architecture-piviki.md#L168).

## 8. Personal data and processors

What personal data may exist, deletion and retention paths, and which providers may receive patron data.
[AD-14](../architecture-piviki/architecture-piviki.md#L174), [AD-18](../architecture-piviki/architecture-piviki.md#L221).

## 9. Operations and collection ownership

Host, ingress, deploy, hardening, backups and cutover; each collection has exactly one creating epic.
[AD-17](../architecture-piviki/architecture-piviki.md#L204), [AD-19](../architecture-piviki/architecture-piviki.md#L227).

Changed during review: scan-clients identity removed; import runs as a librarian account; titles draft by default unless payload publishes; duplicate refusal for every create with allowDuplicate override; copy guard dropped; restore-sequence rule moved to Deferred. Touches AD-1, AD-5, AD-7, AD-9, AD-19, AD-20 and the Collections convention. AD-2 and AD-5 also edited; AD-9 trimmed after second opinion; the import script keeps a local processed-scans file for re-runs; duplicate refusal also applies when a draft is published.

## 10. Conventions and stack

Shared naming, states, ids and testing conventions, and pinned versions.
[Consistency Conventions](../architecture-piviki/architecture-piviki.md#L259), [Stack](../architecture-piviki/architecture-piviki.md#L274).

Changed during review: piviki-scan repo removed; import script lives in piviki-core; display date format dd-mm-yyyy (Europe/Helsinki) for screens, emails and labels, with APIs and storage unchanged.

## 11. Periphery

What still has to be proven, what is undecided, and what is deliberately left for later.
[Assumptions to Prove](../architecture-piviki/architecture-piviki.md#L363), [Open Questions](../architecture-piviki/architecture-piviki.md#L374), [Deferred](../architecture-piviki/architecture-piviki.md#L384).

Changed during review: accounts-with-no-loans retention decided: 1-year inactivity; AD-6 and AD-14 edited.
