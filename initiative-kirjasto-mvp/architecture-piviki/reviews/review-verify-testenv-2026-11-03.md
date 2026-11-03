# Verification review: 2026-11-03 amendment (AWS EC2 becomes the test environment after the move)

Reviewer lens: is every committed decision backed by web research or a live check, not asserted from training data? Is anything out of date?
Date checked: 2026-11-03. Scope: AD-17 "Host" (last two sentences), "Build and deploy", move step 7, "Environments" and "Test environment"; AD-12 test mail sentence; AD-8 test-environment sentence; Conventions "Hosts" row; Structural Seed `test` node; Deferred row "Test environment after the credits". The checks in `review-verify-aws-2026-11-03.md` (IMDS, instance identity role, Tunnel replicas, IPv4 price, credits, OVH sizing) are not repeated.

## Verdict

**Holds in outline. Fix three things before the test environment is built.** The platform facts the amendment relies on are current and correct. Cloudflare's free Universal SSL covers only the apex and first-level names, so `test-admin.piviki.fi` is covered and `admin.test.piviki.fi` would not be. Total TLS still needs the paid Advanced Certificate Manager. A second Tunnel, extra Access applications, a second R2 bucket, a `test.` sending subdomain with its own DMARC record, and a `workflow_dispatch` that takes an environment and a tag are all supported on the free tiers in use.

Three things will not work as written, or are not enforced by what is written:

1. **`test.piviki.fi` cannot be both the patron-site hostname and the mail domain.** The Tunnel needs a proxied CNAME at that name. Cloudflare does not support other record types, such as the SPF or provider-verification TXT records, at the name of a proxied CNAME (F1).
2. **"Its own credentials" does not by itself keep test mail off `piviki.fi`.** At most providers an SMTP key can send from every verified domain on the account (F2).
3. **Putting `test-api.piviki.fi` behind Access breaks the patron site's cross-origin API calls.** Browsers never send cookies with CORS preflight requests, so Access returns 403. Production does not have this problem, because `api.piviki.fi` is public (F3).

A fourth item is a gap in the text rather than an error. "Can be stopped at any time" changes the instance's public IPv4 address. The spine does not say how a deploy reaches the host, so it is unclear whether anything depends on that address (F4).

## What was checked and how

| # | Claim in the amendment | Check | Result |
|---|---|---|---|
| T1 | "One-level host names keep Cloudflare's free certificate" (`test-admin`, `test-api`, not `admin.test`) | developers.cloudflare.com: Universal SSL limitations; Total TLS; Total TLS error messages | **Confirmed.** On a full setup, Universal SSL covers "the root domain" and first-level subdomains. Deeper names "will not serve a valid certificate". Total TLS fixes this, but it is "available for domains that have purchased Advanced Certificate Manager", which is a paid add-on (about 10 USD a month per zone). Nothing has changed this on the free plan. |
| T2 | Every test hostname is behind Cloudflare Access on the free plan | Cloudflare One account limits; Zero Trust plans page; seat-management docs | **Confirmed.** Free covers up to 50 users. The limits are 500 Access applications per account and 50 domains per application. A seat is counted per identity, not per application, so the test applications use no new seats while only the founders sign in. A removed user frees a seat; a revoked user does not. |
| T3 | Access does not interfere with the test site | Cloudflare One docs: "Authorization cookie" and "CORS" | **Not addressed (F3).** Access "looks for a cookie specific to" each hostname. Preflight OPTIONS requests fail with 403, because "the browser never includes cookies with OPTIONS requests, by design". |
| T4 | The test environment has its own Tunnel | Cloudflare One account limits | **Confirmed.** The limit is 1,000 Tunnels per account. Inactive Tunnels count toward it. There is no per-Tunnel charge below that limit. |
| T5 | The Tunnel survives a stop and start of the instance | Prior review A7 (outbound-only connection); EC2 stop/start docs | **Holds, from training knowledge.** `cloudflared` dials out and reconnects when the Compose stack comes up at boot, if it has a restart policy and Docker is enabled at boot. The public IP change does not affect it. While the instance is stopped, the test hostnames return Cloudflare error 1033. Not live-tested. |
| T6 | The test environment has its own R2 media bucket | developers.cloudflare.com/r2/pricing | **Confirmed.** 100,000 buckets per account are included. The pricing page lists the free allowance (10 GB-month Standard storage, 1M Class A and 10M Class B operations) without saying it is per bucket, so read it as per account. A test bucket that holds a second copy of the catalogue images takes from the same 10 GB (F7). |
| T7 | Test mail goes through the same provider from `test.piviki.fi`, with its own SPF, DKIM and DMARC | Cloudflare DNS record-types reference; Brevo domain-authentication guides; Scaleway TEM docs | **Supported by providers, but it conflicts with the Tunnel hostname (F1).** Brevo, Scaleway and the others let you authenticate each sending domain or subdomain separately. Brevo asks for a `brevo-code:` TXT record at the sending domain's own name. SPF also lives at that name. Cloudflare says that for a proxied CNAME, "queries for other record types on the same name are not supported". |
| T8 | DMARC on `test.piviki.fi` works alongside `piviki.fi`'s policy | Vendor guides (Resend, MXToolbox, Suped); RFC 9989 (DMARCbis) summaries | **Confirmed.** A record at `_dmarc.test.piviki.fi` overrides the parent's `sp=` for that subdomain. `sp=` applies only to subdomains with no record of their own. DMARCbis was published as RFC 9989 in May 2026. It keeps `p` and `sp`, adds `np` (policy for non-existent subdomains) and `psd`, removes `pct`, `rf` and `ri`, and replaces the Public Suffix List with a DNS tree walk. None of this changes the outcome for `test.piviki.fi`. Any `pct=` in the production record is now ignored. |
| T9 | "Its own credentials" keep test mail from being sent as `piviki.fi` | Provider docs (Emailit, MailerSend feature request, Brevo guides) | **Not enforced at most providers (F2).** Emailit can restrict a key to one domain. No evidence was found that Brevo, Mailjet or Scaleway can, and MailerSend has an open feature request for it. Isolation needs a separate account or project. |
| T10 | A manual workflow run names one environment and one image tag | docs.github.com workflow syntax | **Confirmed.** `workflow_dispatch` input types are `boolean`, `choice`, `number`, `environment` and `string`. `jobs.<id>.environment.name` "can be an expression", with contexts `github`, `inputs`, `vars`, `needs`, `strategy` and `matrix`. So `environment: ${{ inputs.environment }}` with a tag string input is valid. |
| T11 | Separate secrets per environment (implicitly GitHub Environments) | docs.github.com "Managing environments for deployment"; "About environments" | **Confirmed, with a condition (F5).** "Users with GitHub Free plans can only configure environments for public repositories." Required reviewers on Free are public-only too. "If you convert a repository from public to private, any configured protection rules or environment secrets will be ignored." |
| T12 | The instance "can be stopped at any time" | EC2 User Guide "How EC2 instance stop and start works"; aws.amazon.com/vpc/pricing | **Confirmed, with cost and addressing consequences (F4).** On stop, EC2 releases the auto-assigned public IPv4 address, and on start it assigns a new one. An Elastic IP stays, and is charged while the instance is stopped. All public IPv4 costs 0.005 USD an hour, in use or idle. EBS volumes are billed while stopped. Compute is not, and each start bills at least one minute. The instance usually moves to new hardware. The EBS data and SSH host keys stay. |
| T13 | The test environment has its own Google redirect URIs | Google Cloud help "Manage OAuth clients" and redirect-URI rules; Google Developers blog "Usability and safety updates to Google Auth Platform" | **Confirmed. Choose separate client or same client (F6).** Redirect URIs must match exactly, use HTTPS and contain no wildcards. The project limit is 10 unique domains, counted by registered domain, and everything here is under `piviki.fi`. Since June 2025, Google deletes OAuth clients after 6 months with no token exchanges, and they can be restored for 30 days. |
| T14 | Hosts convention, AD-8 and diagram match AD-17 | Spine reading | Consistent. The Hosts row lists the same three names. AD-8 sets `patronAccountsAllowed` true in test with a founder-only allowlist. The `test` node has no data edge from OVH. |
| T15 | Move step 7: wipe, delete snapshots, rebuild as test | Spine reading; prior review A13 | Consistent. Rebuilding with the bootstrap on the same instance keeps the same public IP, unless the instance is stopped. Rotating the AWS-side credentials in step 4 makes the wipe a second safeguard, not the only one. |

## Findings

### F1 (Medium): `test.piviki.fi` cannot carry both the Tunnel CNAME and the mail records

**What the spine says.** The Hosts convention and the Test environment bullet put the test patron site at `test.piviki.fi`. AD-12 says the test environment sends "from `test.piviki.fi`, with its own ... SPF, DKIM and DMARC records".

**What is true.** A Tunnel public hostname is a proxied CNAME to `<uuid>.cfargotunnel.com`. Cloudflare's DNS reference says that for a proxied CNAME, "queries for other record types on the same name are not supported", and its dashboard refuses a TXT record next to a CNAME at a non-apex name. SPF is a TXT record at the sending domain's own name. Brevo also needs a `brevo-code:` TXT record there, and Scaleway and Mailjet need SPF TXT there. DKIM (`selector._domainkey.test...`) and DMARC (`_dmarc.test...`) sit at other names and are unaffected. So the records some providers need cannot be published at all. A provider that verifies only by DKIM CNAMEs might get by without them, but the spine promises an SPF record.

Production does not have this conflict while the patron site is on `beta.piviki.fi` and mail comes from the apex, `piviki.fi`.

**Fix.** Send test mail from a name with no web record, for example `testmail.piviki.fi`, with SPF, DKIM and DMARC there. Or move the test patron site to another first-level name, such as `test-www.piviki.fi`, and keep `test.piviki.fi` for mail only. Change AD-12 and the Hosts row together.

### F2 (Medium): "its own credentials" does not keep test mail off `piviki.fi`

**What the spine says.** The test environment uses the same provider with its own credentials "and never from `piviki.fi`".

**What is true.** At Brevo, Mailjet and Scaleway, no source found shows a way to tie an SMTP key to one sending domain. A key on an account can send as any domain verified on that account. Emailit is one provider that can restrict a key to a domain. MailerSend has an open request for it. If the test credentials sit on the production account, a misconfigured `From` or a leaked test secret can send as `piviki.fi`, and test volume uses the production account's daily quota (Brevo free: 300 a day per account).

**Fix.** Write the mechanism in AD-12: a separate provider account or project for test that holds only the test sending domain (a Scaleway project scopes domains and keys this way), or a provider with domain-restricted keys. Then "never from `piviki.fi`" is enforced, not just intended.

### F3 (Medium): Access on `test-api.piviki.fi` breaks the patron site's API calls

**What the spine says.** All three test hosts are behind Cloudflare Access. In production only `beta` and `admin` are, and `api.piviki.fi` is public. The patron site calls the API cross-origin.

**What is true.** Cloudflare's CORS page says a CORS request "must include a valid `CF-Authorization` cookie" for the target hostname, and that preflight OPTIONS requests get 403 because browsers never send cookies with them. The test site's credentialed `fetch` with a JSON body therefore fails, which production never shows. The test environment then tests a different setup from production.

**Fix.** Put the three test hostnames in **one** Access application. Access then issues the cookie for all of them at sign-in (the "Eager redirect cookie" setting from the 2026-08-03 changelog, on by default for new applications). Also either let Access answer preflights with the application's CORS settings, set to the exact `test.piviki.fi` origin, or turn on "Bypass options requests to origin", which works only because core already enforces CORS. State it in the Test environment bullet. A Preview Worker in test, if there is one, would also need an Access service token to read `test-api`.

### F4 (Medium): stopping changes the public IP, and the spine does not say how deploys reach the host

**What the spine says.** "It can be stopped at any time." "A manual workflow run names one environment and one image tag, and each host only pulls, through its own limited deploy user." The host is reachable only by SSH by key.

**What is true.** A stop releases the auto-assigned public IPv4 address, and a start brings a new one (T12). If the workflow SSHes from a GitHub-hosted runner to the deploy user:

- the host address in the environment's secrets and its `known_hosts` entry go stale after every stop and start;
- SSH must be open to GitHub's runner ranges, which in practice means `0.0.0.0/0`.

An Elastic IP fixes the first point, and it is charged even while stopped. EBS is charged while stopped in any case. The Deferred row's "delete it or move it when the credits run low" covers the cost. It does not cover the IP.

**Fix.** Name the deploy path in "Build and deploy". The option that needs no fixed IP is SSH through the host's own Tunnel (`cloudflared access ssh` with an Access service token per environment, and `known_hosts` pinned to the Tunnel hostname). It also lets the security group drop public SSH. Otherwise, use an Elastic IP on the test instance and accept the cost of about 3.65 USD a month.

### F5 (Low): GitHub Environments depend on the deploying repository staying public

The `workflow_dispatch` design is valid as written (T10). If per-environment secrets or required reviewers come from GitHub Environments, they exist only because the repositories are public on the Free plan (T11). If the deploying repository ever goes private, the protection rules and environment secrets are silently ignored. Add one line to "Build and deploy": "Environment secrets and reviewers rely on the repository being public on GitHub Free".

### F6 (Low): name whether test uses its own Google OAuth client

"Own Google redirect URIs" fits either one client with extra URIs or a second client. A separate test client keeps the production client's configuration free of test hosts, and its secret leaking does nothing to production. The trade-off: since June 2025, Google deletes a client with no token exchanges for 6 months, so a test environment stopped that long loses its client and has 30 days to restore it. Either way the 10-domain project limit is not a concern. Suggest: "a separate OAuth client in the same project".

### F7 (Low): the second bucket shares R2's free storage

The R2 free allowance is not stated per bucket (T6). A test media bucket that receives the import script's output holds a second copy of every catalogue image. Usage above 10 GB-month is billed, not blocked. No change needed. Note it beside the Deferred row, or make the test import a subset.

## Not re-checked here (covered by `review-verify-aws-2026-11-03.md`)

IMDS hop limit and the instance identity role; Tunnel replica behaviour during the move; public IPv4 price and the GHCR IPv6 gap; credit and Free-plan account closure; OVH VPS-1 sizing.

## Sources

- Cloudflare, Universal SSL limitations: https://developers.cloudflare.com/ssl/edge-certificates/universal-ssl/limitations/
- Cloudflare, Universal SSL: https://developers.cloudflare.com/ssl/edge-certificates/universal-ssl
- Cloudflare, Total TLS: https://developers.cloudflare.com/ssl/edge-certificates/additional-options/total-tls/
- Cloudflare, Total TLS error messages: https://developers.cloudflare.com/ssl/edge-certificates/additional-options/total-tls/error-messages/
- Cloudflare One, account limits: https://developers.cloudflare.com/cloudflare-one/account-limits/
- Cloudflare, Zero Trust plans: https://www.cloudflare.com/plans/sase-zero-trust.md
- Cloudflare docs source, seat management: https://github.com/cloudflare/cloudflare-docs/blob/production/content/cloudflare-one/identity/users/seat-management.md
- Cloudflare One, Access and CORS: https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/authorization-cookie/cors/
- Cloudflare One, authorization cookie: https://developers.cloudflare.com/cloudflare-one/identity/authorization-cookie
- Cloudflare changelog, eager redirect cookie setting (2026-08-03): https://developers.cloudflare.com/changelog/post/2026-08-03-eager-redirect-cookie-setting/
- Cloudflare blog, Access wildcard and multi-hostname: https://blog.cloudflare.com/access-wildcard-and-multi-hostname/
- Cloudflare, Tunnels for Platforms: https://developers.cloudflare.com/tunnel/features/tunnels-for-platforms/
- Cloudflare, R2 pricing: https://developers.cloudflare.com/r2/pricing/
- Cloudflare, DNS record types (proxied CNAME): https://developers.cloudflare.com/dns/manage-dns-records/reference/dns-record-types/
- Cloudflare, records with the same name: https://developers.cloudflare.com/dns/manage-dns-records/troubleshooting/records-with-same-name/
- Brevo, authenticate your domain: https://help.brevo.com/hc/en-us/articles/12163873383186
- Brevo developers, domain authentication: https://developers.brevo.com/docs/domain-authentication-and-verification
- alwaysdata, Brevo records: https://help.alwaysdata.com/en/docs/emails/third-party-software/brevo/
- Scaleway TEM, authenticate domain: https://www.scaleway.com/en/docs/transactional-email/how-to/authenticate-domain/
- Scaleway TEM, troubleshooting (multiple domains): https://www.scaleway.com/en/docs/transactional-email/troubleshooting/fix-common-issues.md
- Emailit, API keys (domain-restricted keys): https://emailit.com/docs/developers/api-keys/
- MailerSend feature request, restrict SMTP user to one domain: https://feedback.mailersend.com/feature-requests/p/restrict-smtp-user-to-one-domain
- Brevo SMTP free-plan cap (third party): https://meshworld.in/blog/devops/email/smtp/brevo-smtp-server/
- Resend, How DMARC applies to subdomains: https://resend.com/blog/how-dmarc-applies-to-subdomains
- MXToolbox, DMARC sp tag: https://mxtoolbox.com/dmarc/details/dmarc-tags/dmarc-sp
- Suped, subdomain DMARC overrides: https://www.suped.com/learn/dmarc/how-do-dmarc-records-on-subdomains-override-root-domain-dmarc-policies
- Palisade, RFC 9989 explained: https://www.palisade.email/resources-post/what-is-dmarcbis-the-future-of-dmarc.md
- CaptainDNS, DMARCbis changes: https://www.captaindns.com/en/blog/dmarcbis-2025-whats-new
- GitHub, workflow syntax: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- GitHub, manage environments: https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments
- GitHub, about environments: https://docs.github.com/actions/reference/environments
- AWS, How EC2 instance stop and start works: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/how-ec2-instance-stop-start-works.html
- AWS, VPC pricing (public IPv4): https://aws.amazon.com/vpc/pricing/
- Google Cloud, Manage OAuth clients: https://support.google.com/cloud/answer/15549257?hl=en
- Google Cloud, OAuth domain limits: https://support.google.com/cloud/answer/7650096
- Google Developers blog, Google Auth Platform updates (unused client deletion): https://developers.googleblog.com/en/usability-and-safety-updates-to-google-auth-platform/
