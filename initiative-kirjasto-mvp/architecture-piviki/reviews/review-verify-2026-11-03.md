# Verification review: 2026-11-03 update (AD-9, AD-11, AD-20, Deferred)

Reviewer lens: is every committed decision backed by web research or a live check, not asserted from training data?
Date checked: 2026-11-03. Scope: AD-9 (barcode sequence, matchKey, `duplicate_title`), AD-11 (MARC language codes), AD-20 (relators, ISBN-13, `ykl`/`ddc`, DDC numbers without captions, `sourceRecordId` prefix), Deferred rows (Melinda SRU, ISO 2709/MARCXML, `@natlibfi/marc-record-serializers`).

## Verdict

The update holds up. Every external fact checked is correct for the current versions. The Melinda prefix `(FI-MELINDA)` was the only claim with no source in the research document, and a live Melinda record now confirms it. Three things need tightening before an implementer can build from the spine without guessing: the HTTP status and response shape of `duplicate_title`, where the barcode sequence is created, and the shape of the `sourceRecordId` example.

## What was checked and how

| # | Claim | Check | Result |
|---|---|---|---|
| V1 | Payload 3.92.1 is current | npm registry `payload` dist-tags | Confirmed: `latest` is 3.92.1 (4.0 is beta/canary) |
| V2 | PostgreSQL 18.6 | postgresql.org/versions.json | Confirmed: latest minor of 18 is 18.6 |
| V3 | A collection hook can read a Postgres sequence in the request's transaction | Payload source at tag v3.92.1: `packages/drizzle/src/utilities/getTransaction.ts`, `packages/drizzle/src/postgres/execute.ts`; Payload transactions docs | Confirmed, with caveats (F2). The request's transaction is `req.transactionID`. Its Drizzle handle is `payload.db.sessions[await req.transactionID]?.db`, falling back to `payload.db.drizzle`, which is what Payload's internal `getTransaction(adapter, req)` does. `payload.db.execute({ db, sql })` or `db.execute(sql\`select nextval('…')\`)` runs raw SQL on it. The `sql` tag comes from `@payloadcms/db-postgres/drizzle`. The docs cover passing `req` and Drizzle access, but not raw SQL inside a transaction. That part rests on reading the source. |
| V4 | A hook can throw an error that gives a REST client a stable code | Payload source v3.92.1: `errors/APIError.ts`, `utilities/formatErrors.ts`, `utilities/routeError.ts` | Confirmed, with caveats (F1). `new APIError(message, status, data, isPublic)`. The REST response is `{ errors: [{ name: 'APIError', message, data }] }` with HTTP `status`. A status of 500 makes the error non-public, and the body becomes "Something went wrong." with `data` dropped. A collection `afterError` hook can reshape the response. |
| V5 | MARC relator codes `aut`, `ill`, `trl` | loc.gov/marc/relators/relaterm.html | Confirmed: author [aut], illustrator [ill], translator [trl] |
| V6 | MARC language codes `vie`, `eng` (AD-11) | loc.gov/marc/languages/language_code.html | Confirmed |
| V7 | `ykl` and `ddc` are LC classification source codes | loc.gov/standards/sourcelist/classification.html | Confirmed: both are listed (`ykl` links to finto.fi/ykl). Research [27] also notes `ykl` replaced `fplcs`. |
| V8 | Melinda record id prefix `(FI-MELINDA)` | Live Melinda SRU query (`sru.api.melinda.kansalliskirjasto.fi/bib`, `dc.title=kalevala`) | Confirmed: `001 000023874`, `003 FI-MELINDA`, `035 $a (FI-MELINDA)000023874`. The ids are 9-digit and zero-padded (F3). The same records also carry `(FI-ASTERI-N)` and `(FI-ASTERI-W)` in authority links, so the prefix varies by source. |
| V9 | Melinda open SRU needs no login | Same live query | Confirmed: answered without credentials. A `bath.isbn=978604…*` truncation query returned 0 records. Research §1 used exact-ISBN queries, so check exact-match behaviour when R12 is built. |
| V10 | DDC numbers storable without a licence; captions not | Research [30][31] (OCLC WebDewey terms, DNB) | Sourced in research. Not re-fetched in this review. |
| V11 | ISO 2709 / MARCXML in UTF-8 as the export target | Research [3][7] (Koha and Evergreen manuals) | Sourced in research. Not re-fetched. |
| V12 | `@natlibfi/marc-record-serializers` | npm registry | Confirmed: 11.0.5 (2026-09-08), MIT. Named in the research, not in the spine (F5). |

## Findings

### F1 (Medium): `duplicate_title` has no HTTP status or wire shape

AD-9 says the scan client gets "the stable code `duplicate_title` and the existing title's id". Payload has no `code` field. The code has to go in `data`. If the hook throws with the default status, or with 500, `routeError` hides the error and the client sees only "Something went wrong.". `piviki-scan` (AD-9's re-run logic) depends on matching this exactly, and it lives in a different repository.

**Fix.** Add one line to AD-9: the hook throws `APIError('duplicate_title', 409, { code: 'duplicate_title', titleId })`. The generated REST API returns HTTP 409 with `errors[0].data.code === 'duplicate_title'` and `errors[0].data.titleId`. `piviki-scan` matches on `data.code`, never on `message`. Name it in the Consistency Conventions error table if one exists.

### F2 (Medium): Where the barcode sequence comes from and which hook assigns it

Payload's schema generator does not create Postgres sequences, and dev `push` won't create one either. If nobody writes it down, the first `nextval` fails in a fresh environment. Payload runs `beforeValidate`, then field validation, then `beforeChange` (this ordering is from training knowledge, not re-checked here). So a `required` barcode field set in `beforeChange` fails validation first. "Refuses any value from a client" also differs from field-level `access.create: false`, which silently strips the value instead of refusing it. Postgres sequences are non-transactional, so a rolled-back create leaves a gap. That is consistent with "never reused", but readers may expect consecutive numbers.

**Fix.** In AD-9, state that a core migration (`payload migrate:create`) creates the sequence, for example `copy_barcode_seq`. Say the hook runs on `operation === 'create'`, in `beforeValidate`, or with the field not `required`, and throws a 400 `APIError` when a client sends a value. State that barcode numbers may have gaps. Optionally name the transaction handle (`payload.db.sessions[await req.transactionID]?.db ?? payload.db.drizzle`), so the read uses the request's connection and no fresh pool connection is taken mid-request.

### F3 (Low): The `sourceRecordId` example doesn't match real Melinda ids

Real Melinda ids are 9-digit and zero-padded: `(FI-MELINDA)000023874`. The example `(FI-MELINDA)123` invites implementers to strip zeros. The id would then no longer match the record's 035 or 001/003 on a later re-copy or export.

**Fix.** Change the example to `(FI-MELINDA)000023874`. Add "stored exactly as the source's 035 `$a`, or 003 and 001 combined". Optionally note that the prefix is the source's MARC organisation code, so other sources keep their own prefix, for example `(OCoLC)`.

### F4 (Low): matchKey refusal is check-then-insert

AD-9 doesn't say what stops two concurrent scan creates with the same `matchKey` from both passing the hook. A unique index isn't possible, because librarian duplicates are allowed. This is outside the verification lens but affects the re-run guarantee.

**Fix.** Either state that the initial-load script is single-threaded, or have the hook take `pg_advisory_xact_lock(hashtext(matchKey))` in the request's transaction before checking.

### F5 (Info): The serializer library and Melinda's SRU endpoint aren't named in the spine

The Deferred rows say "ISO 2709 or MARCXML" and "Melinda's open SRU". The library and the endpoint appear only in the research document. That is acceptable for deferred items, and both are verified (V9, V12).

**Fix (optional).** Add "see research §4 and §6" to those two rows, so whoever picks them up finds `@natlibfi/marc-record-serializers` 11.x and `sru.api.melinda.kansalliskirjasto.fi/bib` without re-researching.

## Unconfirmed

- The Payload hook order (`beforeValidate`, then validate, then `beforeChange`) is stated from training knowledge and was not re-fetched in this review.
- DDC caption licensing and the ILS import formats were not re-fetched here. They rest on research sources [30][31][3][7], which were checked on 2026-11-03.
- Melinda's behaviour for exact-ISBN SRU queries on 978-604 ISBNs was not re-run here. The truncated query returned 0 records.

## Sources

- Payload source at v3.92.1: https://github.com/payloadcms/payload/tree/v3.92.1 (`packages/drizzle/src/utilities/getTransaction.ts`, `packages/drizzle/src/postgres/execute.ts`, `packages/payload/src/errors/APIError.ts`, `packages/payload/src/utilities/formatErrors.ts`, `packages/payload/src/utilities/routeError.ts`)
- Payload docs: https://payloadcms.com/docs/database/transactions, https://payloadcms.com/docs/database/postgres
- npm registry: https://registry.npmjs.org/payload, https://registry.npmjs.org/@natlibfi/marc-record-serializers
- PostgreSQL versions: https://www.postgresql.org/versions.json
- MARC relators: https://www.loc.gov/marc/relators/relaterm.html
- MARC languages: https://www.loc.gov/marc/languages/language_code.html
- Classification source codes: https://www.loc.gov/standards/sourcelist/classification.html
- Melinda SRU (live): https://sru.api.melinda.kansalliskirjasto.fi/bib?operation=searchRetrieve&version=1.1&query=dc.title%3Dkalevala&maximumRecords=3
