# Adversarial review: Piviki MVP spine, repo-split amendment, 2026-11-04

- **Target:** `architecture-piviki.md` (updated 2026-11-04)
- **Scope:** today's amendment only: the Design Paradigm rows "Host runtime | `piviki-core/deploy`" and "Shared infrastructure | `piviki-infra`", the new last sentence of AD-17 "Build and deploy", AD-17 "Recovery" and "Infrastructure as code" (repo split, Bitwarden), AD-18 (Bitwarden), and the "Repositories" convention. Read against what is on disk today: `piviki-infra/{bootstrap,caddy,compose,deploy,docs}` and `piviki-core/.github/workflows/deploy.yml`. Findings closed by the 2026-11-03 reviews are not repeated.
- **Method:** for each hole, two units one level down that each follow every AD to the letter and still build things that do not fit together, then the smallest rule that closes it, worded for the spine.

## Verdict

**Not ready to build on as written: one Critical, four High and three Medium holes.** Moving the host runtime into `piviki-core` is the right direction, and the reason given (the Compose file decides every container's privileges, so a deploy must not be able to replace it) is sound. The amendment creates a second version axis on every host, though, and never ties it to the first. A deploy names an image tag. The Compose file, Caddy config and deploy script come from whatever commit a founder last pulled. Nothing says those two must match, how the bootstrap in one repo finds files in the other, which commit the move, a restore or the test rebuild uses, or where the patron site's deploy now lives. Today's files also contradict the new split: `piviki-infra` still holds `caddy/`, `compose/` and `deploy/`, and the core deploy workflow checks out `piviki-infra` to run `deploy/checks.sh`.

Most findings close with one rule: **the host runs only the `deploy/` tree of the commit its image tag was built from** (finding 1). Findings 4 and 5 then need one sentence each.

---

## 1. The host checkout drifts from the deployed image tag (Critical)

**Units:** any `piviki-core` story that changes both code and `deploy/` (for example, the `epic-reservation-loop` story that adds the `backup` service, a new env var, or a new `/api/v1/public` path in the Caddy allowlist), against the deploy workflow and the rollback rule.

**What the spine says.** "A deploy only names the image tag and never fetches or replaces them." "Rollback is redeploying the previous tag." AD-17 Host: "The same x86_64 images and Compose file run on both." Migrations must work with the tag running in the target. No rule ties the checkout's commit to the tag's commit.

**How they diverge:**

- **Builder A** (a core story) adds `MAIL_RECIPIENT_ALLOWLIST` to `compose.yml` and a new path to the Caddy allowlist in commit X, and the code at X reads them. Tag X is built. A founder deploys tag X without pulling. The deploy script obeys AD-17 and does not fetch. Core starts without the variable: on test the mail adapter either fails open or every send fails, and the new endpoint returns 404 behind Caddy. The deploy checks pass, because they check what already existed.
- **Builder B** (rollback) pulls the checkout to X+5, which hardens core with a new `tmpfs`, then rolls back to tag X−1, per AD-17. The old image now runs under a Compose file it was never tested with. Or the founder also checks out X−1 by hand, and the hardening is silently gone. Both obey the spine.
- **Builder C** (the two environments) pulls test's checkout to main and leaves production's alone. "The same Compose file runs on both" is now false, and test no longer checks what production runs.
- AD-17 Patching ("a patch counts only once deployed to every running environment") cannot be proved for `caddy`, `cloudflared`, `postgres` or `backup`: their versions live in the checkout, not in the recorded tag.

**Closing rule (replace the last sentence of AD-17 "Build and deploy"):**

> The deploy script, Compose file, Caddy config and deploy checks on a host come from a root-owned checkout of `piviki-core` at a fixed path, which a founder pulls and the bootstrap installs. A deploy never fetches or replaces them. The deploy script refuses a tag unless the checkout's `deploy/` tree is identical to `deploy/` at the commit the tag was built from (`git diff --quiet <tag commit> HEAD -- deploy/`), so a founder pulls the checkout to that commit before deploying, including before a rollback. Every image the Compose file runs other than core is pinned by digest in `deploy/`, so changing any of them is a core commit and a new tag. The workflow records, per environment, the core tag and so the `deploy/` tree it ran with.

---

## 2. The bootstrap needs files from two repositories, and the host files have two owners (High)

**Units:** the `piviki-infra` bootstrap story (owner of "host bootstrap" and the "secrets inventory"), against the `piviki-core` story that owns `deploy/`.

**What the spine says.** `piviki-infra` holds "the host bootstrap, a secrets inventory by name". `piviki-core` "holds its own host runtime under `deploy/`". "Each service gets only its own secrets from a root-owned file." The checkout is one "the bootstrap installs". The spine does not say where the checkout lives, which ref the bootstrap clones, whether the bootstrap copies the deploy script or points at it, or who decides which env files and keys exist.

**How they diverge:**

- **Builder A** (infra) does what `bootstrap.sh` does today: it creates the `/etc/piviki/*.env` templates (`core.env`, `caddy.env`, `cloudflared.env`, ...) and copies `deploy.sh` to `/usr/local/sbin/piviki-deploy`, which the `deploy` user's sudoers entry names. The copy is made from the checkout at bootstrap time. A founder later pulls core for finding 1. The sudoers-whitelisted copy is still the old script until someone re-runs the bootstrap.
- **Builder B** (core) adds a `backup` service whose Compose entry reads `/etc/piviki/backup.env`. The bootstrap in another repo does not create it. `docker compose up` fails on the missing env file, or a founder creates it by hand, which breaks "set up only by the `piviki-infra` bootstrap".
- The secrets inventory (infra) lists host keys by file and key, while the files that read them are in core. Two owners of one list, in two repos, with no rule for which one changes first.

**Closing rule (AD-17 "Infrastructure as code", after "`piviki-core` holds its own host runtime under `deploy/`"):**

> `piviki-core/deploy` is the one source for everything on a host that serves core: the Compose file, Caddy config, deploy script, deploy checks and a manifest of each service's env file and key names. The `piviki-infra` bootstrap does the operating-system work and clones `piviki-core` to `/opt/piviki/core` at a commit a founder names. It creates the env files named in that manifest, empty, and points the sudoers entry and helpers at the checkout's files, never at copies. The secrets inventory lists host secrets by referring to the manifest, and itself lists only secrets that are not on a host: OpenTofu, GitHub Environments and Cloudflare. A change to which env files or keys exist is a `piviki-core` commit. The bootstrap needs no change for it.

---

## 3. The patron site's deploy has no home (High)

**Units:** the `epic-patron-site` deploy story in `piviki-patron`, against the core deploy workflow in `piviki-core/.github/workflows/deploy.yml` and the OpenTofu Cloudflare configuration in `piviki-infra`.

**What the spine says.** "The patron site deploys through the same workflow, built for that environment, and is refused if the target runs an older core tag than the one its types came from." GitHub Environments hold "per-environment deploy secrets". OpenTofu "manages Cloudflare resources for both environments". The new rows give the host runtime to core and the shared infrastructure to infra. The patron site is in neither, and is not on a host.

**How they diverge:**

- **Builder A** reads "the same workflow" literally and adds a patron job to `piviki-core`'s `deploy.yml`, which checks out `piviki-patron` and runs Wrangler. Core's GitHub Environments now hold a Cloudflare Workers token, and core's repo deploys another repo's code at a ref the core workflow chooses.
- **Builder B** puts a separate `deploy.yml` in `piviki-patron` with its own GitHub Environments, because Environments are per repository. It cannot easily see "the tag running in each environment", which core's workflow records as deployments in `piviki-core`. It skips the older-core-tag check, or hard-codes it.
- **Builder C** (infra) manages the Worker script, routes and custom domain in OpenTofu, because they are "Cloudflare resources". Builder B's `wrangler deploy` then overwrites the script and routes, and the next `tofu apply` reverts them. That gives two owners for one Worker.
- Nothing names the core tag that `openapi.json` came from, so neither A nor B has a value to compare.

**Closing rule (replace the patron sentence in AD-17 "Build and deploy"):**

> The patron site deploys from a manual workflow in `piviki-patron` that names one environment and one patron commit, with its own GitHub Environments holding only that environment's Cloudflare deploy token. Its generated types record the core tag of the `openapi.json` they came from. The workflow reads the target's running core tag from `piviki-core`'s deployment records and refuses if that tag is older. Wrangler owns the Worker script and the static assets. OpenTofu owns DNS, custom domains, routes and Access, never the script. The patron site has no host files.

---

## 4. The move and the test rebuild do not name the checkout (High)

**Units:** the move story in `epic-initial-load` (AD-17 steps 1 to 7), and the restore runbook (AD-17 "Recovery").

**What the spine says.** Step 1: "Deploys are frozen." Step 3: "On OVH, bootstrapped and started with the exact image tag AWS last ran." Step 7: the EC2 instance's "database, volumes, logs and secrets are wiped ... and the bootstrap rebuilds it as the test environment". Recovery: "rebuild from `piviki-infra`, `piviki-core` and the latest dump". A pull of the checkout is not a deploy ("a deploy only names the image tag").

**How they diverge:**

- **Move builder** follows step 3. The OVH bootstrap clones `piviki-core` main, which is newer than AWS's checkout, and starts the exact tag AWS ran under a different Compose and Caddy config. The freeze in step 1 does not stop this, or a founder pulling AWS's checkout mid-move, because neither is a deploy.
- **Test-rebuild builder** follows step 7. The wipe list does not include `/opt/piviki/core` or the deploy history under `/var/lib/piviki`. The rebuilt test host keeps production's last recorded tag as its "previous" for rollback. Its checkout is still at whatever production last ran, rather than the commit the founder names for test.
- **Restore builder** follows Recovery. The dump does not record which core tag wrote it, so "`piviki-core`" means main. A restore onto a newer tag runs migrations against the dump, and that breaks AD-17's migration rule ("each must work with the tag running in the target environment").

**Closing rule (AD-17 "The move" step 1 and step 3, step 7, "Backups", "Recovery"):**

> Step 1: the freeze covers pulling any host's `piviki-core` checkout as well as deploys. Step 3: the OVH checkout is at the commit of the tag AWS last ran. Step 7: the wipe also removes the checkout and the deploy state, and the bootstrap clones afresh at a commit a founder names for test. Backups: each dump's name or a sidecar records the core tag running when it was taken. Recovery: rebuild from `piviki-infra` at main, `piviki-core` checked out at the dump's recorded tag, secrets from Bitwarden, and the latest dump. The app starts on that tag before any newer deploy.

---

## 5. Deploy checks still have two owners in two repos (High)

**Units:** the existing `piviki-core/.github/workflows/deploy.yml` (it checks out `piviki-org/piviki-infra` at `vars.INFRA_REF || 'main'` and runs `piviki-infra/deploy/checks.sh`), against the new Design Paradigm row "Host runtime | `piviki-core/deploy` | ... deploy script and its checks".

**What the spine says.** Core owns "deploy script and its checks". Infra owns "runbooks". "Every deploy runs the `first-register`, path allowlist and one-connector checks against the target's own hosts." The spine does not say where the off-host checks live, or that they come from the same commit as the tag.

**How they diverge:** **builder A** moves `checks.sh` and `host-checks.mjs` into `piviki-core/deploy`, per the new row. **Builder B** treats the one-connector check, which calls the Cloudflare API with a Tunnel id from OpenTofu, as "shared infrastructure" and leaves it in infra. The workflow now runs off-host checks from infra main against a host whose allowlist came from a core commit. A new path in core's Caddy config is either untested or fails a check written against the old list. Until a story moves the files, `piviki-infra` still holds `caddy/`, `compose/` and `deploy/`. A builder told that core owns them, and one who sees them in infra, edit different copies.

**Closing rule (AD-17 "Build and deploy", and a migration note for `epic-librarian-catalogue` story 1 or whichever story owns the move of the files):**

> Every deploy check, on and off the host, lives in `piviki-core/deploy` and runs from the commit of the tag being deployed. The workflow never checks out `piviki-infra`. Values the checks need from OpenTofu, such as the Tunnel id, are GitHub Environment variables. One story moves `caddy/`, `compose/` and `deploy/` from `piviki-infra` to `piviki-core/deploy`, and deletes them from `piviki-infra` in the same change. After it, `piviki-infra` contains no file that a host's containers read.

---

## 6. "Bitwarden holds every secret" contradicts the offline backup key (Medium)

**Units:** the Bitwarden setup in `epic-librarian-catalogue` story 1, against the backups story (AD-17 "Backups").

**What the spine says.** Bitwarden "holds every secret, and the host files and GitHub Environment secrets are copies of it". Backups: "Both founders hold the private key offline". Each founder "keeps their recovery code offline".

**How they diverge:** **builder A** stores the `age` private key in Bitwarden, because Bitwarden holds every secret. A Bitwarden compromise then exposes every backup, which is what the offline key exists to prevent. **Builder B** keeps it offline only, which breaks "holds every secret". The same applies to the OpenTofu state encryption passphrase, which neither rule mentions. AD-17 step 4 also issues new tokens without saying whether Bitwarden or the host file is updated first. A builder who writes the host file first can leave Bitwarden holding a revoked token, and Recovery then restores from that.

**Closing rule (AD-17 "Infrastructure as code"):**

> It holds every secret except the `age` private key and the Bitwarden recovery codes, which exist only offline with each founder. That includes the OpenTofu state passphrase. A secret is issued or rotated into Bitwarden first, then copied to the host file or GitHub Environment. A rotation is complete only when the old value is revoked and no copy differs from Bitwarden.

---

## 7. The root-owned checkout trusts whatever is on `main` (Medium)

**Units:** the bootstrap or host story that installs the checkout, against the GitHub organisation settings in `piviki-infra`.

**What the spine says.** Repositories are public, in `piviki-org`. The checkout is root-owned "because the Compose file decides every container's privileges". A founder "pulls" it. The spine does not say what the pull is checked against.

**How they diverge:** **builder A** pulls `main`. Any account with write access, a compromised founder token or a merged Dependabot PR that touches `deploy/` then sets container privileges at the next pull. That bypasses the separation the sentence was written to protect. **Builder B** adds branch protection, and A's host still pulls unprotected refs. Both obey the spine.

**Closing rule (AD-17 "Build and deploy"):**

> Changes under `piviki-core/deploy/` need review by the other founder through CODEOWNERS and branch protection on `main`. A founder pulls the host checkout only to a commit on protected `main`, and the deploy script refuses a checkout with local changes.

---

## 8. Recovery and the paradigm rows miss the Bitwarden and patron pieces (Medium)

**What the spine says.** Recovery lists `piviki-infra`, `piviki-core` and the latest dump. It leaves out the secrets source (Bitwarden), the `age` key and `piviki-patron`. The Capability map gives R21 (replacing the coming-soon page) to "`piviki-infra`, Cloudflare". After finding 3, that is the patron deploy plus OpenTofu DNS. The Design Paradigm row for `piviki-patron` does not say it owns its own deploy.

**How they diverge:** a recovery builder rebuilds the host and leaves the patron site unrebuilt, or the patron site is rebuilt from infra, which has no patron code.

**Closing rule:** Recovery: "rebuild from `piviki-infra`, `piviki-core` at the dump's recorded tag, `piviki-patron` at its last deployed commit, secrets from Bitwarden, the offline `age` key and the latest dump; the target is one working day." Patron site row: "Static screens, the preview Worker and their deploy workflow; no rules." R21: "`piviki-patron` deploy, `piviki-infra` OpenTofu DNS".

---

## Summary of proposed wording changes

| # | Severity | Where | Change |
|---|---|---|---|
| 1 | Critical | AD-17 Build and deploy | The deploy script refuses unless the checkout's `deploy/` equals `deploy/` at the tag's commit. Non-core images are pinned by digest in `deploy/`. Record tag and `deploy/` per environment |
| 2 | High | AD-17 Infrastructure as code | `deploy/` holds an env-file manifest. The bootstrap clones `/opt/piviki/core` at a named commit, creates the env files from the manifest and points at the checkout, never at copies. The inventory refers to the manifest |
| 3 | High | AD-17 Build and deploy | The patron deploy workflow lives in `piviki-patron` and reads core's deployment records. Types carry their core tag. Wrangler owns the script, OpenTofu the routes and DNS |
| 4 | High | AD-17 The move, Backups, Recovery | The freeze covers pulls. OVH's checkout is at AWS's tag commit. The step 7 wipe removes the checkout and deploy state. Dumps record the tag. Restore checks out that tag |
| 5 | High | AD-17 Build and deploy | All deploy checks in `piviki-core/deploy`, run from the tag's commit. One story moves the files out of infra and deletes them there |
| 6 | Medium | AD-17 Infrastructure as code | Bitwarden holds everything except the `age` key and recovery codes. Rotate into Bitwarden first |
| 7 | Medium | AD-17 Build and deploy | CODEOWNERS and branch protection on `deploy/`. Pull only from protected `main`. Refuse a dirty checkout |
| 8 | Medium | Recovery, paradigm row, R21 | Add `piviki-patron`, Bitwarden and the `age` key to Recovery. The patron row owns its deploy |
