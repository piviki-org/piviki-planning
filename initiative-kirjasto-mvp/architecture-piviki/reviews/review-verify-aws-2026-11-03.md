# Verification review: 2026-11-03 amendment (AWS EC2 before the move to OVHcloud)

Reviewer lens: is every committed decision backed by web research or a live check, not asserted from training data?
Date checked: 2026-11-03. Scope: AD-17 Host, "AWS before the move", "The move", Backups and Infrastructure as code lines; AD-18 processor rule; the Structural Seed host subgraph; Deferred rows "OVH data centre", "EC2 instance size and region" and "Barcode sequence after a restore"; the Open Questions row about AWS credits.

## Verdict

**Holds, with two fixes needed before the instance is created.** The named technologies exist, are current, and work on EC2 as they do on a plain VPS. Every image in the stack ships `linux/amd64`, and OVH VPS-1 is x86_64 (AMD EPYC). The AWS DPA is incorporated in the Service Terms. IMDSv2 can be required. Stockholm is the nearest full region. Cloudflare Tunnel needs no inbound port, and a dump and restore on the same Postgres major version moves the data. Two statements are not accurate as written:

1. "Nothing on it holds AWS credentials" is false. Every EC2 instance has instance identity role credentials in IMDS, with or without an instance profile.
2. "The Tunnel is pointed at the new host" does not describe how a named Tunnel works. Running the same Tunnel on OVH while the EC2 connector is still up turns the two hosts into replicas. Cloudflare then sends each request to either host, with no guarantee which.

The rest is low-severity hardening, or cost questions that the Open Questions row already parks.

## What was checked and how

| # | Claim in the amendment | Check | Result |
|---|---|---|---|
| A1 | AWS is a processor with an agreement in place (AD-18 implies one; the Open Questions row covers only credits) | aws.amazon.com/service-terms (Last Updated: November 3, 2026), §1.14.1; Navigating GDPR whitepaper; AWS GDPR Center | Confirmed. §1.14.1 incorporates the AWS DPA "when you use AWS Services to process Customer Data". §1.14.3 adds the SCCs. No separate signature is needed. |
| A2 | IMDSv2 is required on the instance | EC2 User Guide, "Configure instance metadata options for new instances" | Confirmed. Set it at launch (`HttpTokens=required`), as the account default per Region, or by enforcement (`HttpTokensEnforced`). The docs recommend hop limit 2 "in a container environment", which lets containers reach IMDS (F1). |
| A3 | No IAM role, "so nothing on it holds AWS credentials" | EC2 User Guide, "IAM roles for Amazon EC2", section "Instance identity roles" | **Not accurate (F1).** "Each Amazon EC2 instance that you launch has an instance identity role." Its temporary credentials sit at `/identity-credentials/ec2/security-credentials/ec2-instance` "regardless of whether a service or feature that makes use of instance identity roles is enabled". They work only with integrated services (STS `GetCallerIdentity`, EC2 Instance Connect, GuardDuty, SSM Default Host Management, and a few more). |
| A4 | Stockholm is closest to Finland (Deferred row) | AWS Local Zones locations page; AWS Stockholm launch post | Confirmed for full regions: eu-north-1 is the only Nordic region. There is also a Helsinki Local Zone (`eu-north-1-hel-1a`, zone ID `eun1-hel1-az1`, parent Stockholm), which the row does not mention (F6). |
| A5 | EU AWS region | Same | Confirmed: eu-north-1 is in Sweden. |
| A6 | Public IPv4 cost (Open Questions) | aws.amazon.com/vpc/pricing | Confirmed: $0.005 per hour, in use or idle, about $3.65 a month. A live DNS check shows `ghcr.io` and `pkg-containers.githubusercontent.com` have no AAAA record. An IPv6-only instance therefore cannot pull images, so the public IPv4 address is required, not optional (F5). |
| A7 | Cloudflare Tunnel is the only route in; the host opens only SSH | Cloudflare One docs: Tunnel overview; "Tunnel with firewall" | Confirmed. `cloudflared` makes outbound-only connections and needs no inbound port. It needs egress on 7844 over TCP and UDP to `region1/region2.v2.argotunnel.com`, and optionally 443. An EC2 security group allows all egress by default. If someone tightens egress, UDP and TCP 7844 must stay open. |
| A8 | The same Tunnel runs on the new host | Cloudflare One docs: "Tunnel availability and failover" and "Deploy cloudflared replicas" | Confirmed that it works, with a caveat (F2). Running the same Tunnel (dashboard token, or `tunnel run <NAME>` with its credentials) on another host adds a replica. "Replicas do not support traffic steering", and Cloudflare picks the closest replica with "no guarantee about which one is chosen". Nothing is repointed: the old connector has to be stopped. |
| A9 | The same x86_64 images run on both hosts | Docker Hub registry manifests, queried live | Confirmed. `postgres:18.6`, `caddy:2.11.7` and `cloudflare/cloudflared:2026.10.7` all publish `linux/amd64`. The GHCR images for `core` and `backup` are built by the project's own Actions, so the build must target `linux/amd64` explicitly (it does if the runner is `ubuntu-latest` x86). |
| A10 | OVH VPS-1 is x86_64 and costs under 15 EUR a month | ovhcloud.com/en/vps; vpsbenchmarks trial of 2026-06-07; next.ink (price rise of 2026-04-01) | Mostly confirmed. The trial reports an AMD EPYC-Milan CPU (x86_64), 4 vCores, 8 GB and 75 GB at 6.49 EUR. Next.ink reports a rise from 5.39 to 7.79 EUR from 2026-04-01. Even with Finnish VAT of 25.5% on 7.79, the total is about 9.8 EUR. The OVH product page now shows a "VPS 2027" VPS-1 at 2 vCores, 4 GB and 40 GB (F4). Daily backup included: confirmed. |
| A11 | Versions in the stack table are current | GitHub releases API, npm, postgresql.org/versions.json | Confirmed. `cloudflared` 2026.10.7 (2026-10-30) and Caddy v2.11.7 are the latest releases. Payload `latest` is 3.92.1. PostgreSQL 18's latest minor is 18.6. None of these behave differently on EC2. |
| A12 | The move by `pg_dump` and restore | PostgreSQL docs (training knowledge, not re-fetched); same major version on both hosts | Holds. Both hosts run the 18.6 image, so a dump and restore needs no version handling. Writes stop before the final dump, so the dump holds the exact barcode sequence value. The margin is harmless. It matters for a restore from a nightly dump, not for the move. The Deferred row describes this correctly. |
| A13 | "The EC2 instance and its disks are deleted" | EC2 and EBS behaviour (training knowledge, not re-fetched) | Plausible. The root EBS volume has `DeleteOnTermination=true` by default. Extra volumes and any snapshots do not, and must be deleted by hand. |
| A14 | Only founder librarian accounts and catalogue data live on AWS | Spine reading (AD-8, AD-17) | Consistent. The move happens before `epic-patron-site` deploys sign-up, so no patron row can exist on AWS. Librarian accounts are still personal data, and A1 covers them. |
| A15 | AWS credits pay for the instance (Open Questions) | AWS Billing docs, "Choosing an AWS Free Tier plan"; AWS Free Tier FAQ | Not checkable without the credit terms. If the credits are new-account Free Tier credits on the Free plan, the account closes after 6 months or once the credits run out (F3). |

## Findings

### F1 (Medium): "Nothing on it holds AWS credentials" is not true on EC2

AD-17 says the instance has no IAM role, "so nothing on it holds AWS credentials". AWS documents that every instance has an instance identity role. Its temporary credentials are always served by IMDS. They are low-privilege, but they are AWS credentials, and they can call STS `GetCallerIdentity` (which reveals the account id) and the other integrated services listed in A3. If someone later turns on SSM Default Host Management at account level, SSM uses this identity to give the instance a role. With hop limit 2, which Amazon Linux 2023 sets and the docs recommend for containers, any container on the Docker bridge, `core` included, can fetch them.

**Fix.** Reword the rule and tighten IMDS. Suggested text: "The instance has no IAM instance profile. IMDSv2 is required with hop limit 1, so no container can reach the metadata service. SSM Default Host Management stays off." A further option: disable the IMDS endpoint once first boot has fetched the SSH key. The docs warn that disabling it at launch breaks key injection and user data.

### F2 (Medium): Repointing the Tunnel is really a replica handover, and the order matters

"The Tunnel is pointed at the new host" suggests a switch that does not exist for a named Tunnel. The public hostnames route to the Tunnel, not to a host. Starting `cloudflared` on OVH with the same token or credentials adds a second connector. Cloudflare then sends requests to either connector with no steering, so while both run, requests can land on the stopped EC2 core (502 from Caddy) or on the new host at random. The step works. It needs an explicit order.

**Fix.** Write the order into the move line, or name it as a restore-runbook step: stop `core` and `cloudflared` on EC2; take the final dump; restore on OVH and advance the sequence; start the OVH stack and its `cloudflared`; confirm one connector in `cloudflared tunnel info` or the dashboard; then delete the instance. Keep Caddy's internal upstream names the same on both hosts, so the Tunnel's ingress config (`http://caddy:…`) needs no change. Treat the Tunnel token as a secret that was on AWS. Deleting the instance is enough, and rotating the token is optional.

### F3 (Medium): Credit type can turn into an outage if the move slips

The Open Questions row asks whether the credits cover EC2, disk and IPv4, and when they expire. If they are AWS Free Tier credits on a **Free account plan**, the plan ends after 6 months or once the credits run out. The account is then closed, and its resources are deleted after 90 days unless it is upgraded to a Paid plan. Creating an AWS Organization also moves an account to the Paid plan and expires its remaining Free Tier credits. Production would be on this account during the initial load.

**Fix.** Extend the Open Questions row: "...and is the account on the Free plan (closes at 6 months) or a Paid plan with promotional credits?" If it is Free, upgrade to Paid before the first data, set an AWS Budgets alert at 0 USD actual spend, and do not create an Organization.

### F4 (Low): OVH VPS-1 specs and price changed in 2026

On 2026-04-01 the price rose from 5.39 to 7.79 EUR (next.ink). The product page now lists a "VPS 2027" VPS-1 at 2 vCores, 4 GB and 40 GB NVMe. Earlier 2026-range units had 4 vCores, 8 GB and 75 GB. The 15 EUR ceiling still holds. This amendment does not size the move target, however, and the VPS-1 on sale at move time may have half the RAM of the one researched earlier. That would also be smaller than a typical EC2 test size.

**Fix.** Add to the "OVH data centre" Deferred row, or to a new row: "confirm the VPS-1 RAM and disk when ordering, and size the EC2 instance no larger, so the stack is proven on the smaller box."

### F5 (Low): Public IPv4 on EC2 is mandatory, not optional

GHCR has no IPv6, as the live DNS check shows. Without a public IPv4 address the host would need a NAT gateway, which is another AWS service, breaks the "only EC2, disk and security group" rule and costs far more. Read this way, the Open Questions row already covers it.

**Fix.** Optional wording in "AWS before the move": "with one public IPv4 address (needed for GHCR and SSH)".

### F6 (Low): Small unconfirmed or missing points

- **EBS encryption at rest.** EBS volumes are not encrypted by default unless account-level "EBS encryption by default" is on (training knowledge, not re-fetched). It is free with the AWS-managed key. Turning it on adds no service that holds data. Worth one clause.
- **T3 and T3a launch in Unlimited mode by default** (EC2 docs). Image re-encoding during the bulk import could run past baseline and be billed as surplus credits. That is cheap, but it draws on the credits. Choose `standard` credit mode, or a non-burstable size.
- **Helsinki Local Zone.** `eu-north-1-hel-1a` exists. Stockholm is still the right default, because Local Zones offer fewer instance types at higher prices. The Deferred row could note that the option was considered.
- **Extra EBS volumes and snapshots** do not delete with the instance (A13). The move step "and its disks are deleted" is right, as long as the runbook checks for leftover volumes and snapshots.

## Sources

- AWS Service Terms, §1.14 Data Protection (Last Updated November 3, 2026): https://aws.amazon.com/service-terms/
- AWS whitepaper, Navigating GDPR compliance, "AWS Data Processing Addendum": https://docs.aws.amazon.com/whitepapers/latest/navigating-gdpr-compliance/aws-data-processing-addendum-dpa.html
- AWS Security Blog, AWS GDPR Data Processing Addendum: https://aws.amazon.com/blogs/security/aws-gdpr-data-processing-addendum
- AWS GDPR Center: https://aws.amazon.com/compliance/gdpr-center/
- EC2 User Guide, Configure instance metadata options for new instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-IMDS-new-instances.html
- EC2 User Guide, IAM roles for Amazon EC2 and instance identity roles: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html
- EC2 User Guide, Unlimited mode concepts for burstable instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-unlimited-mode-concepts.html
- Amazon VPC pricing (public IPv4): https://aws.amazon.com/vpc/pricing/
- AWS Local Zones locations: https://aws.amazon.com/about-aws/global-infrastructure/localzones/locations/
- AWS News Blog, Now open: AWS Europe (Stockholm) Region: https://aws.amazon.com/blogs/aws/now-open-aws-europe-stockholm-region
- AWS Billing, Choosing an AWS Free Tier plan: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html
- AWS Free Tier FAQs: https://aws.amazon.com/free/faqs/
- Cloudflare One, Cloudflare Tunnel: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/
- Cloudflare One, Tunnel with firewall: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-with-firewall/
- Cloudflare One, Tunnel availability and failover: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-availability/
- Cloudflare One, Deploy cloudflared replicas: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-availability/deploy-replicas/
- OVHcloud VPS: https://www.ovhcloud.com/en/vps/
- vpsbenchmarks, OVHcloud VPS-1 trial of 2026-06-07: https://www.vpsbenchmarks.com/trials/ovhcloud_performance_trial_07Jun2026
- next.ink, OVHcloud raises VPS 2026 and IPv4 prices: https://next.ink/225720/ovhcloud-augmente-fortement-le-prix-de-ses-vps-2026-et-ipv4/
- Docker Hub registry manifests for `library/postgres:18.6`, `library/caddy:2.11.7` and `cloudflare/cloudflared:2026.10.7` (live query)
- GitHub releases API for `cloudflare/cloudflared` and `caddyserver/caddy`; npm registry for `payload`; https://www.postgresql.org/versions.json (live queries)
- DNS AAAA lookups for `ghcr.io`, `pkg-containers.githubusercontent.com` and `region1.v2.argotunnel.com`, through dns.google and the local resolver (live)
