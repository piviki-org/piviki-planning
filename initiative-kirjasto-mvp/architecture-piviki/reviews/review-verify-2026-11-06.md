# Verify review: state buckets per environment (2026-11-06)

- **Spine:** `architecture-piviki.md`, AD-16 last sentence, AD-17 Infrastructure as code, AD-17 Backups (erasure line)
- **Change under review:** `.memlog.md` entry "State buckets per environment (2026-11-06, user)"
- **Lens:** were the committed decisions web-researched or reality-checked, or asserted from training data?
- **Checked:** 2026-11-06, against Cloudflare and OpenTofu documentation fetched that day

## Verdict

**Pass with fixes.** The decision itself holds up: separate per-environment private buckets in the EU jurisdiction, each reached only by its own bucket-scoped token, is supported and is the right way to keep test credentials away from production. Three things need fixing:

1. The stated reason, "an R2 token cannot be limited to part of a bucket", is only half true as of October 2026.
2. Putting state in the same bucket as the backups conflicts with the backup lifecycle and lock rules unless those rules are limited to a prefix.
3. The decision log has no `(version)` or source entry for this change. The claim behind it reads like it came from memory, not from a check.

## Findings

### F1. The reason is imprecise: R2 does support prefix-scoped credentials, but only short-lived ones (medium)

- R2 API tokens (the long-lived S3 access key pairs) can be limited to a set of buckets for Object Read or Object Read & Write. The token page describes no prefix scope. Source: https://developers.cloudflare.com/r2/api/tokens/ (last updated 2026-10-01).
- R2 **temporary credentials** can be limited to specific prefixes or objects inside one bucket. They are derived from a parent R2 token, cannot exceed its permissions, and last at most 604800 s (7 days). Sources: https://developers.cloudflare.com/r2/api/s3/temporary-credentials/ (last updated 2026-10-16); API reference https://developers.cloudflare.com/api/resources/r2/subresources/temporary_credentials/ (fields `prefixes`, `objects`, `ttlSeconds` up to 604800, `parentAccessKeyId`).
- **Effect:** the sentence "an R2 token cannot be limited to part of a bucket" is wrong if a reader takes "token" to mean any R2 credential. The decision still holds. Prefix-scoped temporary credentials expire within 7 days and need a parent token that has the whole-bucket permission, held wherever they are minted. That means a minting step in CI and on hosts, and the parent token reaches every prefix anyway. Separate buckets are simpler and give real isolation.
- **Fix:** reword it along these lines: "since an R2 API token can only be limited to whole buckets; prefix-scoped temporary credentials last at most 7 days and derive from a bucket-wide parent token". Record the sources in the memlog.

### F2. Production state sharing a bucket with the backups conflicts with the 30-day lifecycle rule and the 7-day bucket lock (high)

The change puts production's OpenTofu state in the same private bucket as the dumps and the erasure records. AD-17 Backups says "A lifecycle rule deletes dumps after 30 days, a 7-day bucket lock protects recent ones", but it does not say these rules are limited to a prefix.

- **Bucket lock:** a rule without a prefix covers every object in the bucket, and it blocks **overwrites** and deletes. A blocked request fails with HTTP 409 `ObjectLockedByBucketPolicy`. Rules can be limited to a prefix. Source: https://developers.cloudflare.com/r2/buckets/bucket-locks/ (last updated 2026-10-28). With a bucket-wide lock, every `tofu apply` that rewrites the state would fail for 7 days after each write. Deleting the `.tflock` file at the end of each run would fail too, which leaves the state locked.
- **Lifecycle:** rules can be limited to a prefix. A rule without a filter applies to the whole bucket. Source: https://developers.cloudflare.com/r2/buckets/object-lifecycles/ (last updated 2026-10-26). A bucket-wide 30-day delete rule would remove the state file whenever 30 days pass without a rewrite, and it would remove erasure records, which must outlive every restorable dump.
- R2 has no object versioning (`PutBucketVersioning` is not implemented: https://developers.cloudflare.com/r2/api/s3/api/). Deleted or overwritten state cannot be recovered from the bucket.
- **Fix:** AD-17 Backups should say that the lifecycle and lock rules apply only to the dump prefix, for example `dumps/`. Name the prefixes for state (for example `tofu/`) and `erasures/`, and add a check that state and erasures are outside both rules. Also decide how state is recovered, since R2 has no versioning (for example, the encrypted state is copied after each apply). Test's bucket has no dumps, so this applies only to production.

### F3. In production, other tokens on the same bucket can reach the state (medium)

R2 object permissions are bucket-wide (F1). In production the `backup` container's token and core's erasure-writing token are Object Read & Write tokens on the same private bucket as the state. Either one could read the encrypted state, or overwrite or delete it. In the other direction, the OpenTofu token can delete dumps outside the lock window. The change prevents test reaching production, as stated. It does not keep a compromised core or backup container on the production host away from production's state.

- **Fix:** accept this and say so in the spine, or give state its own bucket per environment, for example `piviki-prod-tofu-eu`. A separate bucket costs nothing extra and also removes F2's interaction with the lifecycle and lock rules.

### F4. The jurisdiction claim is confirmed, with one detail to pin down (low)

- EU-jurisdiction buckets exist. The jurisdiction is chosen at creation and cannot be changed afterwards. Jurisdictional buckets can be reached only through `https://<ACCOUNT_ID>.eu.r2.cloudflarestorage.com`. Source: https://developers.cloudflare.com/r2/reference/data-location/.
- An R2 token is **not** per jurisdiction. The token page's policy example grants one token buckets in both `default` and `eu` (`com.cloudflare.edge.r2.bucket.<ACCOUNT>_eu_<BUCKET>`). What differs per jurisdiction is the endpoint, not the credential. The spine does not claim tokens are per jurisdiction, so nothing in it is wrong. The secrets inventory should still say that each token is limited to its environment's named bucket, not to "all buckets".
- **Fix:** none in the spine. The `piviki-infra` seed should use the `.eu.` endpoint (see F5). Bucket locks on a jurisdictional bucket through the API need the `cf-r2-jurisdiction: eu` header (bucket-locks page).

### F5. The OpenTofu s3 backend works with R2, but Cloudflare's example uses the default endpoint (low; not yet checked against R2)

- Cloudflare documents R2 as a Terraform s3 backend with `region = "auto"`, `endpoints = { s3 = "https://<ACCOUNT_ID>.r2.cloudflarestorage.com" }`, `use_path_style = true` and the skip flags (credentials validation, metadata API check, region validation, requesting account ID, s3 checksum). Source: https://developers.cloudflare.com/terraform/advanced-topics/remote-backend/ (last updated 2026-04-21). For the EU buckets, the endpoint has to be the `.eu.` host. The page does not show this.
- The OpenTofu 1.13 s3 backend has all these options, and its native lock (`use_lockfile = true`) relies on conditional writes with `If-None-Match`. Source: https://opentofu.org/docs/language/settings/backends/s3/. R2 implements `If-None-Match` on PutObject atomically: at most one of several concurrent writes succeeds, and the others get 412. Source: https://developers.cloudflare.com/r2/api/s3/api/ (last updated 2026-10-14). So native locking should work, but neither vendor documents the combination. One bucket per environment fits the backend's one-bucket-per-backend-block model, with a separate `tofu/prod` and `tofu/test` root.
- "Encrypted state" refers to OpenTofu's client-side state encryption, not to anything the s3 backend page covers. The spine does not say which. That is fine for a spine, but the seed should name it.
- **Fix:** the `piviki-infra` seed, or the first IaC story, runs `tofu init` and `apply` against the `.eu.` endpoint with `use_lockfile = true`. It confirms a second run waits on the lock and that the lock file is removed afterwards (this also checks F2). Record the result as a `(version)` memlog entry.

### F6. No sources are recorded for this change (process)

Earlier changes record what was checked as `(version)` entries with URLs, for example Bitwarden on 2026-11-04 and costs on 2026-11-05. The 2026-11-06 entry gives "an R2 token can only be scoped to a whole bucket" as its reason with no source, and F1 shows that claim is incomplete. Add a `(version)` entry citing the R2 tokens, temporary-credentials, data-location, bucket-locks and object-lifecycles pages, with today's date.

## What could go out of date

- R2 temporary credentials are recent (page updated 2026-10-16). If Cloudflare later adds prefix conditions to long-lived token policies, F1's reasoning changes, though separate buckets would still be the simpler choice.
- The 7-day maximum for temporary credentials appears only in the API reference.
- The R2 S3 compatibility table (conditional writes, no versioning) and the Cloudflare Terraform backend example (written for the v4 provider) change often. Check them again at the first IaC story.
- OpenTofu native S3 locking needs version 1.10 or later. The Stack table pins 1.13.2, which is fine.

## Sources

- https://developers.cloudflare.com/r2/api/tokens/
- https://developers.cloudflare.com/r2/api/s3/temporary-credentials/
- https://developers.cloudflare.com/api/resources/r2/subresources/temporary_credentials/
- https://developers.cloudflare.com/r2/reference/data-location/
- https://developers.cloudflare.com/r2/buckets/bucket-locks/
- https://developers.cloudflare.com/r2/buckets/object-lifecycles/
- https://developers.cloudflare.com/r2/api/s3/api/
- https://developers.cloudflare.com/terraform/advanced-topics/remote-backend/
- https://opentofu.org/docs/language/settings/backends/s3/
