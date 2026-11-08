# Verify review: owners move (2026-11-05)

Scope: diff `spine-owner.diff` against `architecture-piviki.md` (AD-17 Production start line 213, AD-19 table lines 242-243) and `.memlog.md` latest entries. Lens: are the committed decisions in the changed lines reality-checked rather than asserted? No web research was done because the diff makes no technology claim.

## Verdict

PASS. No new technology, version, vendor or price appears in the changed lines. The ownership change matches the latest memlog decision ("Owners moved", 2026-11-05) and the rest of the spine. Two minor wording gaps and one memlog bookkeeping gap remain.

## Technology and version check

The changed lines name only things the spine already uses: OVH, Bitwarden `production`, OpenTofu, Cloudflare Tunnel and Access, `first-register`, the `backup` service, `admin.piviki.fi` / `api.piviki.fi` / `piviki.fi`, CORS/CSRF, `noindex`. The removed and added text differ only in who owns the work and what waits on what. No versions, no prices, no new services. The only "verify" fact near this text, the OVH-over-AWS host choice and its cost basis, was checked earlier and is recorded in the memlog (`(version) Cost re-check 2026-11-05`). The diff does not change it.

## Consistency with the spine

- Line 7 scope order (catalogue, initial-load, patron-site, soft-open) fits production start being in `epic-soft-open`.
- Line 141: the real import is "the initial load ... into production" and names no owning epic, so it does not contradict the move.
- Line 218 (backups before the initial load, restore tested on production before the real import), line 220 (Environments: production from the production start; soft open follows `epic-patron-site` and the real import), line 223 (Cutover owned by `epic-soft-open`): all consistent.
- Deferred rows 408 and 411 refer to "the production start" and "the real import" by name, not by owner, so they still hold. Row 413 (vision model, owner `epic-initial-load`) fits that epic still building the import script.
- No leftover text says the production start belongs to `epic-librarian-catalogue` or the real import belongs to `epic-initial-load`.

## Consistency with the memlog

The latest decision entry says: production start and real import are the first two stories of `epic-soft-open`; `epic-initial-load` builds the script and runs trial imports on test; the production start waits only on `epic-librarian-catalogue`'s infrastructure stories, not on `epic-patron-site`. Lines 213 and 242-243 say the same.

## Findings

1. Minor (asserted boundary, undefined term). "`epic-librarian-catalogue`'s infrastructure stories" appears nowhere else in the spine. The spine names only "`epic-librarian-catalogue` story 1" (test environment, line 220) and the "first `epic-librarian-catalogue` story" (two-hostname check, line 385). The production start dependency is therefore stated but not pinned to stories. Suggest naming the stories, or the capabilities: the `piviki-infra` bootstrap, OpenTofu, runbooks, the `backup` service, mail and the `first-register` check.
2. Minor (missing dependency). Line 213 says what the production start waits on, but no line says that the real import, `epic-soft-open`'s second story, waits on `epic-initial-load`'s import script and its trial imports on test. Since the epics are no longer nested, this edge should be written down for Sofia's replan. Wording nit in the same sentence: "and it waits only on" can read as `epic-soft-open` waiting, not the production start. Suggest "and the production start itself waits only on".
3. Minor (memlog bookkeeping). The "Owners moved" entry does not mark as superseded the earlier 2026-11-05 entries "Production start owner" (a story of `epic-librarian-catalogue`) and "Review fixes" (the real import is the last story of `epic-initial-load`). Other entries in the log use "Supersedes ...". Without the marker, the log contains two conflicting live decisions on ownership.
