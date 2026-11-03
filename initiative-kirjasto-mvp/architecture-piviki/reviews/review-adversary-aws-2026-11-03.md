# Adversarial review: Piviki MVP spine, AWS-before-the-move amendment, 2026-11-03

- **Target:** `architecture-piviki.md` (updated 2026-11-03)
- **Scope:** today's amendment. Production runs on one x86_64 EC2 instance in an EU AWS region until a move to an OVHcloud VPS-1, which `epic-initial-load` owns, right after the initial load is published and before `epic-patron-site` deploys sign-up. Covers AD-17 (Host, AWS before the move, The move, Ingress, Backups, Recovery, Cutover, Infrastructure as code), AD-18, the Structural Seed, Deferred and Open Questions, read against `tickets.toml` (epic order) and the `epic-librarian-catalogue` stories.
- **Method:** for each hole, two units one level down that each follow every AD exactly and still build things that do not fit together. Then the smallest rule that closes it.

## Verdict

**Not ready to build on as written: one Critical and three High holes.** Moving the host is the right call, and the "AWS before the move" rule is tight about which AWS services are allowed. What it leaves loose is the boundary of the move itself. "Deploys sign-up" can be read three ways. The epic order lets `epic-patron-site` run alongside `epic-initial-load` and deploy to the AWS host. The move has no ordered steps, so the old host's `cloudflared`, `backup` and CI deploy target can still be live after the final dump. Ingress is enforced by an AWS security group that has no counterpart on OVH. And the owning epic's ticket does not mention the move at all. Each hole has a short closing rule below. Most can be closed by one new "The move" sequence in AD-17 and one guard in `modules/accounts`.

---

## 1. "Before `epic-patron-site` deploys sign-up" does not stop a patron account being created on AWS (Critical)

**Units:** an `epic-patron-site` sign-up story (auth endpoints, the gate, Google), and `epic-initial-load`'s move. A third: the `epic-patron-site` donor-linking story, whose verify step needs a consenting patron in production.

**What the spine says.** AD-17: "no patron account is ever created on AWS", and the move happens "before `epic-patron-site` deploys sign-up". AD-8: while `signupOpen` is false, sign-up and sign-in are allowed for allowlisted emails. `tickets.toml`: `epic-patron-site` comes `after` `epic-librarian-catalogue` only, so it can run in parallel with `epic-initial-load`. Every epic has "Deployed to production and verified end to end". CI deploys every merge to the one production host.

**How they diverge:**

- **Builder A** reads "deploys sign-up" as "merges the sign-up code". CI deploys it to the AWS host, which is production until the move. The story's verify step has a founder on the beta allowlist sign up on `beta.piviki.fi`, and AD-8 allows that. A patron row now exists on AWS. That breaks AD-17, and AD-18, because AWS is not named in the privacy notice.
- **Builder B** reads it as "a real patron signs up", which means `signupOpen`. Allowlisted test accounts are then fine before the move, so B does the same thing as A on purpose.
- **Builder C** reads it literally and keeps all sign-up code off `main` until the move. Then `epic-patron-site` stalls behind the review of all 300 books, which is the end-of-November milestone, against a mid-December patron-site target. Nothing in the epic order says so.
- AD-6 puts the beta allowlist in server configuration. On AWS, that is patron-to-be email addresses on the AWS disk, even if nobody signs up.
- Nothing in core enforces the rule. It depends on two epics running in parallel getting their timing right.

**Closing rule (tighten AD-17 "AWS before the move", add to AD-8):**

> "Deploys sign-up" means "any path that can create a `patrons` row runs in production". `modules/accounts` refuses every patron create (sign-up, Google, admin, generated API) unless the server configuration value `patronAccountsAllowed` is true. The value is false and the beta allowlist is empty on the AWS host. The move runbook sets them on OVH only. Sign-up code may deploy to AWS behind this guard. In `tickets.toml`, every `epic-patron-site` story whose verify step creates a patron, including donor linking, is `after` the `epic-initial-load` move story. A test confirms that every create path is refused while the value is false.

---

## 2. The move has no ordered steps, so two hosts can both be live (High)

**Units:** the move runbook (`epic-initial-load`) and three things that keep running on the AWS host until "the EC2 instance and its disks are deleted": `cloudflared`, the `backup` container, and the CI deploy job, which `epic-patron-site` merges keep triggering in parallel.

**What the spine says.** "Writes stop, a final dump is restored on OVH …, the Tunnel is pointed at the new host, and the EC2 instance and its disks are deleted." It does not say how writes stop, which processes stop, what "pointed" means, or what happens to CI.

**How they diverge:**

- **Tunnel.** Builder A reuses the same tunnel and installs its token on OVH. If AWS `cloudflared` is still up, Cloudflare spreads requests across both connectors. Half the admin traffic then writes to the stale AWS database after the dump: split brain, with lost writes and barcodes issued on the wrong host. Builder B creates a new tunnel and switches the DNS CNAMEs in OpenTofu. Builder A's runbook has no OpenTofu step, and B's does. The two runbooks and the secrets inventory differ.
- **Backups.** The AWS `backup` container keeps running nightly. If dump names are date-based, the AWS dump and the OVH dump for the same night collide. Either the 7-day bucket lock refuses the second write, so the OVH dump fails, or the stale AWS dump wins. Either way the heartbeat sees "a dump arrived" and stays quiet.
- **Deploys.** A merge to `piviki-core` between the final dump and the switch deploys to AWS and runs a migration there that OVH never gets. Or the deploy target is switched first, and OVH starts a newer tag than the one that wrote the dump. AD-17's rule that migrations must work with the previous version does not cover a restore that skips a version.
- **Final dump.** One builder triggers the `backup` container on demand, which proves the R2 and `age` path that "the move is run as a restore" implies. Another pipes `pg_dump` over SSH. Story 7 gives the container no run-now command.

**Closing rule (replace AD-17 "The move" with an ordered sequence):**

> The move runs in this order: (1) freeze the deploy workflow; (2) stop every service on AWS except `postgres` and `backup`, which stops writes and removes the AWS Tunnel connector; (3) the `backup` container takes the final dump through its run-now command; (4) OVH restores it with the restore runbook, starting the exact image tag that AWS last ran; (5) OVH `cloudflared` starts with the same tunnel, managed in OpenTofu, so there is no DNS change; (6) the deploy target secrets point at OVH and the freeze ends; (7) AWS is stopped completely, and then the instance and disks are deleted. Dump object names include the host name, and the heartbeat checks only the current host's name.

---

## 3. Ingress is enforced by an AWS security group that does not exist on OVH (High)

**Units:** story 1's host bootstrap and outside scan, done on AWS, and the move story's OVH host, built with the same bootstrap.

**What the spine says.** "The host opens no inbound 80 or 443, only SSH by key", and "a scan from outside confirms this before the first real data". The first real data is the initial load, which is on AWS. The move re-runs nothing.

**How they diverge:**

- **Builder A (story 1)** enforces ingress with the security group, which AD-17 explicitly allows, and writes no host firewall. The scan passes.
- **Builder B (the move)** runs the same `piviki-infra` bootstrap on an OVH VPS. OVH VPS ports are open by default. Any published Compose port (Caddy on `:80` for a local test, say) is now reachable. A later `ufw` would not help either, because Docker's iptables rules go around `ufw`. Nothing re-runs the scan, because "before the first real data" already happened on AWS.
- **OS choice.** On EC2, Amazon Linux 2023 is the default, with `dnf-automatic` and the `ec2-user` conventions. On OVH it would be Debian or Ubuntu, with `unattended-upgrades`. Story 8's "automatic OS security updates" and the deploy user end up as two different bootstraps, and the OVH one is untested.
- Story 8's other host-level checks are in the same position: the SSH settings, the Docker log rotation and the per-provider MFA (AWS, then OVH).

**Closing rule (tighten AD-17 Ingress and Hardening):**

> The Compose file publishes no ports at all. The host bootstrap enforces default-deny inbound except SSH on the host itself, independent of any provider firewall, which is only a second layer. Both hosts run the same OS image, for example Debian 13, set by the bootstrap. The bootstrap is the only way host-level settings are made, and it is idempotent. Before the Tunnel moves, the move re-runs the outside scan, the `first-register` deploy check and story 8's host-level checks on OVH. "Before the first real data" becomes "before each host serves production".

---

## 4. The owning epic does not know it owns the move, and story 1 still targets a VPS (High)

**Units:** `epic-initial-load` (ticket) and `epic-librarian-catalogue` story 1.

**What the spine says.** `epic-initial-load` owns the move. That epic's ticket says "Owns the script and the review-and-import path". Its Done-when has no move, no OVH order and no restore. Story 1 says "run on the VPS", has the HITL step "order the VPS", and verifies with "an outside scan of the VPS shows only SSH".

**How they diverge:** a builder of story 1 orders the OVH VPS, as the ticket says, and production starts on OVH. Another follows the spine and starts on EC2. The planner of `epic-initial-load` slices only the script and the review, and nobody plans the move. Then `epic-patron-site` is blocked by finding 1's rule with no story that unblocks it.

**Closing rule (spine and tickets):**

> AD-17 names the move as a story in `epic-initial-load`, after the last publish of the initial load, with the verify step "OVH serves admin.piviki.fi from the restored dump, barcodes continue above the margin, the outside scan of OVH shows only SSH, and the EC2 instance and disks are gone". Update `epic-initial-load`'s Boundaries and Done-when to match, and change story 1 in `epic-librarian-catalogue` from "VPS" to "the EC2 instance".

---

## 5. OVH's included backup bypasses erasure re-deletion and the 30-day limit (Medium)

**Units:** `modules/accounts` deletion with `erasures/{patronId}` and the restore runbook, against an operator restoring from OVH's included VPS backup or snapshot during a Recovery.

**What the spine says.** "OVH's included backup is listed in the data inventory". Nothing else in the spine defines the data inventory, says whether the OVH backup stays on, or says how long it keeps data.

**How they diverge:** after the move, the OVH backup holds patron data unencrypted, on OVH's own retention schedule. Under the one-working-day Recovery target, one founder rolls back the VPS from the OVH snapshot, which is fastest. That brings back deleted patrons without running the erasure re-deletion, which is only in the R2 restore runbook. Another founder follows the runbook. The privacy notice states one retention period, and the system now has two.

**Closing rule (AD-17 Backups and Recovery):**

> Either turn OVH's included backup off, or allow a restore from it only through the restore runbook's erasure re-deletion before core starts, and state its retention in the privacy notice. The data inventory is a named file in `piviki-infra`.

---

## 6. Secrets across the move: copied or rotated, and readable through IMDS on AWS (Medium)

**Units:** story 1's secrets file and bootstrap on AWS, and the move story.

**How they diverge:**

- **Copy or rotate.** One builder copies the root-owned secrets file to OVH: Postgres passwords, the Tunnel token, the R2 tokens for media and backup, SMTP, the GHCR pull credential and the deploy key. Another rotates them all. The secrets inventory and the breach runbook then describe different worlds. The AWS `backup` R2 token, if it is not revoked, can still write to the private bucket after the instance is deleted, for example from a leftover snapshot.
- **IMDS.** IMDSv2 is required, but the hop limit is not set. Amazon Linux 2023 AMIs default it to 2, so containers can reach the metadata service. With no IAM role there are no AWS credentials to steal. But a bootstrap that passes secrets in EC2 user-data, which is a natural choice on AWS, makes them readable from core through an SSRF.

**Closing rule (AD-17 AWS before the move, The move):**

> No secret goes in EC2 user-data, and the IMDS hop limit is 1, or IMDS is turned off. The move issues new R2 tokens and a new deploy key for OVH, and revokes the AWS ones before the instance is deleted. Every other secret is copied, and the secrets inventory records the choice for each one. GHCR packages are public, so the host holds no pull credential.

---

## 7. Recovery after the move could rebuild on AWS (Medium)

**Units:** the Recovery runbook and AD-18.

**How they diverge:** "The same x86_64 images and Compose file run on both", and the credits are still there, so a founder who has an OVH outage rebuilds "from `piviki-infra` and the latest dump" on a fresh EC2 instance in an hour. That dump now holds patrons, and AD-18 allows AWS only before the move. Another founder orders a new VPS.

**Closing rule (AD-17 Recovery):**

> After the move, Recovery rebuilds only on OVHcloud, or on a provider that the privacy notice already names. AWS is never a recovery target once patron data exists.

---

## 8. Host-local state does not survive the move (Medium)

**Units:** story 8's "about 90-day ids-only log of librarian logins and generated writes" and the ids-only logs with about 30-day rotation, against the move, which carries only Postgres.

**How they diverge:** one builder writes the 90-day audit log to a host volume or a log file. The move leaves it on the AWS disk, which is then deleted, so the 90-day promise breaks at the move. Another builder writes it to a Postgres table, where it survives. The same split applies to Payload media cache files and anything else a builder puts on a volume.

**Closing rule (AD-17 Host):**

> Anything that must outlive a host is stored in Postgres or R2. Host volumes other than Postgres data are disposable. The move and restores carry nothing else.

---

## 9. Minor

- **Stale Deferred row (Low).** "Barcode sequence after a restore" is still deferred with "decide by before the move". But AD-17 "The move" and story 7 already decide it, by advancing the sequence by a margin. A reader of Deferred may treat it as open. Remove the row, and state the margin, for example 1000, in AD-9 or the runbook, so that story 7 and the move use the same number.
- **SMTP port on AWS (Low).** EC2 blocks outbound port 25 by default. A story 9 builder who picks a port-25 relay passes locally and fails on AWS. Pin the submission port to 587 or 465 in AD-12.
- **Credits expiry vs a slipping load (Low).** If the initial load slips, the AWS host stays, and the credits may expire or the move may block `epic-patron-site` (finding 1). State that the move may happen any time after story 7's restore drill, and that "right after the initial load is published" is the latest point, not the only one.

---

## Attacked and dropped

- **Barcode reuse across the move.** Writes stop before the final dump, so the restored sequence is exact. The margin only matters for disaster restores. Covered by the stale Deferred row above.
- **Erasure re-deletion during the move.** If finding 1 is closed, AWS holds no patron and no `erasures/` objects exist yet. Re-deletion is a no-op and does no harm.
- **Cutover (`epic-launch`) against the move (`epic-initial-load`).** Different steps at different times, in sequence: the move comes before sign-up, and the cutover after the beta. Nothing overlaps. Each has one owner.
- **Cloudflare Access and the import service token.** Not tied to a host, and revoked after the initial load, before the move.
- **Google redirect URIs, CORS and the `api`/`admin` hostnames.** Not tied to a host. They do not change at the move.
- **R2 media and backup buckets.** Unchanged at the move. Dumps from the AWS period age out under the lifecycle rule and hold no patron data.
- **DNS in OpenTofu.** Tunnel CNAMEs point at the tunnel id, not the host IP. Once finding 2 fixes "same tunnel", the move needs no DNS change.
- **AWS region and founder librarian data on AWS.** EU region, founders' own accounts, and AD-18 already lists AWS as holding no patron data.
- **Public visitor IP addresses in AWS logs.** `beta` is behind Access and `piviki.fi` is the coming-soon page, so only scanners reach `api.piviki.fi`. The logs are deleted with the disk.
- **"Only EC2, disk and security group" against a public IPv4 or Elastic IP.** Both are EC2 features, and the Open Question already covers their cost.
- **The Structural Seed label.** The host subgraph names both providers correctly. The mermaid node id `ovh` is cosmetic.
- **AD-8 reset not writing `erasures/`.** This is a real question, but it belongs to the beta reset, not to today's amendment. Raise it in the `epic-launch` review.
