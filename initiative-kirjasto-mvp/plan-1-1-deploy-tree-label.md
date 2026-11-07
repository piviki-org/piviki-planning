---
title: 'Deploy compares the image''s deploy/ fingerprint, not the tag''s commit'
type: 'bugfix'
ticket: ''
created: '2026-11-06'
status: 'built'
baseline_revision: 'aa64421fd3e77bac91fef42a9ccba29e36001e76' # piviki-core; piviki-infra 3ac32aef13a97b8f4c453d01e398b6d849cc76cc
route: 'oneshot'
route_source: 'auto'
risk: 'medium'
review: 'quick'
review_source: 'pinned'
lenses_ran: ['quick']
review_loop_iteration: 0
context: []
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** Fix on story 1.1. `deploy.sh`'s `require_matching_checkout` looks the tag's commit up in the host's checkout. The host never fetches (AD-21), so every tag built after the last `--move-core` is refused, even when only the app changed. That contradicts the "Later deploys" rule in `deploy/README.md` and `runbook-first-deploy.md`.

**Approach:** the user chose option A on 2026-11-06.
- **CI stamps the fingerprint.** Each image gets the label `fi.piviki.deploy-tree`: the git tree id of `deploy/` at the commit it was built from (`git rev-parse HEAD:deploy`).
- **The host compares it.** After pulling and before anything starts, `deploy.sh` compares that label with `HEAD:deploy` in the checkout, instead of looking up the commit.
- **Refusals.** It refuses a missing label, a mismatch, or a dirty checkout; each refusal names `--move-core`.
- **AD-21's guarantees stay:**
  - the checkout is clean;
  - `deploy/` is identical;
  - the host never fetches;
  - the image job pushes only from `refs/heads/main`;
  - the deploy workflow already refuses commits that aren't on `main`.
- **Tests and docs.** Update `deploy.test.sh`, the `deploy.sh` header, `deploy/README.md`, and the comments in `deploy.yml`/`build.yml`. Also update the "Later deploys"/"Rollback" wording in piviki-infra's runbook where needed.

</frozen-after-approval>

## Implementation Notes

**Oneshot:** about 100 lines across deploy.sh, its test, build.yml and three docs, in one well-mapped area.

- **The first image has no label.** `aa64421…`'s image was built before this change, so it would be refused. The first test deploy uses the merge commit of this fix. No tag has been deployed yet, so no rollback target is lost.

- Files: core `deploy/deploy.sh` (`require_clean_checkout` before the pull, `require_matching_deploy_tree` after it), `deploy/test/deploy.test.sh`, `.github/workflows/build.yml` (image job main-only, `fi.piviki.deploy-tree` label, build fails without a tree id), `.github/workflows/deploy.yml` comments, `deploy/README.md`; infra `docs/runbook-first-deploy.md` (§5, Later deploys, Rollback), `README.md` sequence diagram.
- The Approach's "each refusal names `--move-core`" overstates it: only the mismatch can be fixed by moving; the missing/malformed label says to deploy a newer tag, the dirty checkout to restore it.

- 2026-11-06, after review, the user's decision: the fingerprint covers only the files in `deploy/` a host reads. It is no longer the tree id of the whole folder. `deploy/fingerprint.sh` (new) hashes `git ls-tree -r` of `deploy/`, leaving out `README.md`, `compose.local.yml`, `local-env/`, `test/` and `checks.sh`. The runner runs `checks.sh` from the tag's own commit, and the host never runs it. The list is an exclude list, so any new file counts by default. CI and `deploy.sh` both pipe the listing into the same script. The label is renamed `fi.piviki.host-runtime` (64 hex). The test now proves a README-only change keeps the fingerprint and a `compose.yml` change alters it.

- Committed 2026-11-06: piviki-core `1e11a5a206fd1060db12707c82015d6be3cd0aa8` on `fix/1.1-deploy-tree-label`; piviki-infra `9debc526afdc02d826108c060463b6d88ecad4b2` on `feat/1.1-test-environment`. Not pushed.

## Review Triage Log

Quick pass: 8 findings; 6 patched, 1 false, 1 deferred.
- patch (low): mismatch message asked for a full commit id; `--move-core` accepts the tag's 12 hex. Now `--move-core <commit>`, test asserts it.
- patch (low): a malformed label was reported as missing; now its own message, tested before anything starts.
- patch (low): stale `deploy.yml` step comment.
- patch (low): infra README sequence diagram showed the old check.
- patch (medium): `echo "…$(git rev-parse …)"` would push an image with an empty label; the step now fails without a 40-hex tree id.
- patch (low): runbook Rollback didn't say pre-label tags are always refused.
- false: "each refusal names --move-core". The plan wording overstated it, and naming it on a missing label or a dirty checkout would mislead (see Implementation Notes).
- defer (maybe-false, would be medium): a workflow on another branch with `packages: write` can still push to GHCR with any label. This isn't new, because the old check also trusted the image's content for a tag naming a main commit. Root control is unchanged. Entry in deferred-work.md.

## Verification

**Commands:**
- `deploy/test/deploy.test.sh` -- expected: all PASS
- `deploy/test/{checks-public,checks-connector,host-checks-names}.test.sh` -- expected: all PASS
- `bash -n deploy/deploy.sh` -- expected: no output
