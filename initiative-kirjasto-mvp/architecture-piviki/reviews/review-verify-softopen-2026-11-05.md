# Verify review: soft-open update, 2026-11-05

Spine: `architecture-piviki.md`. Diff: `spine-softopen.diff` (AD-6 Origin line, AD-8 sign-up gate, AD-17 Production start, Environments, Test environment, Cutover, epic table, Hosts row, diagram, closed Open Question). Memlog scope: "Production start timing (2026-11-05, user)" to the end.

Lens: was each committed decision web-researched or checked against reality, or only asserted from training data? Is anything out of date?

## Verdict

**The host decision holds, and the cost figures are now confirmed against the vendors' own price data. Three cutover and exposure statements are imprecise or don't hold up.** The memlog's AWS figures came from third-party aggregators. They now match AWS's published price files of 2026-11-04 exactly, and AWS on-demand exceeds 15 EUR. One supporting detail is stale (the 750 free IPv4 hours). OVH VPS-1 is 4.49 EUR excl. VAT month to month, from OVH's own order catalog. Retiring `beta` and switching `piviki.fi` at the soft open works with the stated stack. But "production is not public" is not accurate. "Add `piviki.fi` to Google redirects" names the wrong Google setting. Default `workers.dev` and preview URLs can get around Access and the "no patron site host" rule.

## Claims table

| # | Claim (memlog or changed line) | How checked | Result |
|---|---|---|---|
| C1 | t3.small eu-north-1 on-demand about 0.0216 USD/h, about 15.8 USD/month (memlog "AWS cost check") | AWS public price map `b0.p.awsstatic.com/.../ec2-ondemand-without-sec-sel/EU (Stockholm)/Linux/index.json`, published 2026-11-04T21:03Z | **Confirmed.** t3.small Linux 0.0216 USD/h, 2 vCPU, **2 GiB**. 730 h = 15.77 USD. (t3.medium, 4 GiB like VPS-1: 0.0432 USD/h.) |
| C2 | Public IPv4 0.005 USD/h, about 3.65 USD/month | AWS VPC price map (published 2026-11-03), aws.amazon.com/vpc/pricing (live) | **Confirmed.** 0.005 USD/h, in use or idle, Stockholm. |
| C3 | "free 750 h/month for 12 months on new accounts" (IPv4) | aws.amazon.com what's-new 2025-07 (Free Tier credits and 6-month plan); AWS Free Tier docs; VPC pricing page | **Stale.** Accounts created on or after 2025-07-15 get the credit-based Free Plan, not the 12-month 750-hour allowance. The VPC pricing page shows no IPv4 free allowance. This doesn't affect the conclusion, which is about after the credits. |
| C4 | "with EBS about 21 USD/month, above the 15 EUR cap once credits end" | AWS EBS price map (gp3 Stockholm 0.0836 USD/GB-month); ECB reference rate 2026-11-04 (1 EUR = 1.1267 USD) | **Confirmed for on-demand.** 15.77 + 3.65 + 1.67 (20 GB gp3) = 21.09 USD = 18.7 EUR excl. VAT, about 23.5 EUR with Finnish VAT at 25.5 %. A 40 GB disk adds 1.67 USD. **Not checked:** 1- or 3-year Savings Plans or Reserved Instances, which could bring t3.small near the cap. They also add a commitment, which the "month to month" rule would exclude anyway (F5). |
| C5 | OVH VPS-1 stays under 15 EUR/month, month to month, EU | OVH public order catalog API `eu.api.ovh.com/v1/order/catalog/public/vps?ovhSubsidiary=IE` (live), plan `vps-2027-model1`; ovhcloud.com/fr/vps (live) | **Confirmed.** No commitment: 4.49 EUR/month excl. VAT, installation 0. 6-month upfront: 4.26. 12-month upfront: 3.81 (the "from" price on the site). With 25.5 % VAT: 5.63 EUR. EU data centres in the plan config: DE, WAW, GRA, SBG, EU-WEST-RBX, EU-SOUTH-MIL (and UK). This closes the earlier open question about the no-commitment price and DE/PL. Stock per data centre is not checked. Secondary sites report an OVH price rise in 2026. The catalog figure above is the current one. |
| C6 | AWS production option dropped: exceeds cap after credits, and a later move is the dropped restore-move | C1 to C4; spine AD-17 | **Holds.** It also holds like for like: matching VPS-1's 4 GB needs t3.medium, about 37 USD/month all in. |
| C7 | CORS and CSRF lists hold exact origins; `piviki.fi` joins production's lists at the soft open | Payload 3.92.1 source on disk: `payload/dist/auth/extractJWT.js`, `payload/dist/utilities/headersWithCors.js`; Payload docs (config overview, cookies) | **Confirmed.** CSRF: `config.csrf.includes(origin)`. CORS: `cors.indexOf(requestOrigin)`, which echoes the origin with `Allow-Credentials: true`. Both are exact string matches. The docs don't say this; the source does. When `Origin` is missing, CSRF falls back to `Sec-Fetch-Site` ∈ {same-origin, same-site, none}. So any `*.piviki.fi` page counts as same-site. Host-only cookies limit what that exposes (Info). |
| C8 | Host-only `SameSite=Lax` session cookie on `api.piviki.fi` works for the patron site at `piviki.fi` | Registrable-domain reasoning (both under `piviki.fi`) | **Holds.** `piviki.fi` to `api.piviki.fi` is same-site, so Lax cookies go with credentialed fetches. The same applies on test (`test.piviki.fi` to `test-api.piviki.fi`). |
| C9 | Cutover adds `piviki.fi` to "Google redirects" at the soft open | Google OAuth web-server docs (live); spine AD-7 line 91, AD-8 line 128 | **Imprecise (F2).** Google's redirect URIs must match exactly (scheme, case, trailing slash), with no wildcards and HTTPS only. The spine puts the Google endpoints under `api.piviki.fi/api/v1/auth/google/*`, so the authorized redirect URI is on `api.piviki.fi`. That host exists from the production start. `piviki.fi` is the post-login return target, a core setting, not a Google console redirect. |
| C10 | Production's Google OAuth client is usable at the soft open | Google Cloud notices and forum threads (unused-client deletion); spine AD-17 line 223 already records the 6-month rule for test | **Risk not covered (F2).** Clients with no token exchange or update for about 6 months are deleted (soft delete, restorable for about 30 days). The production start can now come well before the soft open, with no patron site on production in between. The official help article was not read; forum reports and Google staff statements only. |
| C11 | Google sign-in works for the public from the soft open | support.google.com/cloud/answer/15549945 (live) | **Step missing (F2).** In "Testing" status, access is limited to listed test users (up to 100). The app must be published "In production" before `signupOpen`. With only `openid email`, publishing needs no verification. Brand verification (name and logo) is optional and needs a homepage and privacy policy, which `piviki.fi` serves only from the soft open. |
| C12 | `piviki.fi` can move from the coming-soon page to the patron site (static assets + Worker) at the cutover | developers.cloudflare.com: Workers Custom Domains, static-assets routing (live) | **Feasible, with a DNS step (F4).** A Custom Domain can be the zone apex and takes all paths. "You cannot create a Custom Domain on a hostname with an existing CNAME DNS record." The coming-soon record has to be removed in the same apply. An Advanced Certificate is issued for the hostname. |
| C13 | "Until the soft open ... production has no patron site host", and test's patron site is behind Access | developers.cloudflare.com: workers.dev routing (live) | **Can be bypassed by default (F3).** Every Worker gets a `<name>.<subdomain>.workers.dev` route when created. Wrangler turns it back on at the next deploy unless `workers_dev = false`. Version, preview and deployment URLs stay on even then. Access can protect them, but only if configured. |
| C14 | "Until the soft open, production serves only `admin.piviki.fi` behind Access, `api.piviki.fi` and its deploy SSH host, and is not public" | Spine AD-7 line 91 (`api.piviki.fi` is the public surface), AD-16 (public R2 media bucket), AD-17 line 222 (`noindex` stated only for test hosts) | **Contradicted by the spine (F1).** `api.piviki.fi` serves `/api/v1` with no Access. After the real import it answers public catalogue reads and the auth endpoints, though sign-up is allowlist-only. Cover images are in a public bucket. Production hosts have no stated `noindex`. |
| C15 | Cloudflare Access on `admin.piviki.fi` (and test patron and admin hosts) | Unchanged technology. Verified in earlier reviews (review-verify-2026-11-03/04/05) | Not re-checked. Nothing in this diff changes it. |
| C16 | Scans on test are safe in test's public bucket (founders checked them against the AD-16 checklist) | User statement in the memlog | **Not technically checkable; user-owned.** AD-16 EXIF stripping and re-encoding still apply on test, so the risk left is image content, which the founders' check covers. |

## Findings

### Tier 1: fix before the tickets are replanned

**F1. "Production ... is not public" before the soft open is not true as the stack stands.** (C14)
`api.piviki.fi` has no Access and is the public `/api/v1` surface (AD-7). After the real import, published titles can be read there, and cover images sit in a public R2 bucket. Auth endpoints answer, though sign-up is limited to the allowlist. Only the test hosts are stated to send `noindex`. The memlog entry ("production serves only admin.piviki.fi behind Access and api.piviki.fi") is accurate. The spine's "is not public" overstates it.
**Fix (pick one):** (a) Reword: "Until the soft open, production serves only `admin.piviki.fi` behind Access, `api.piviki.fi` and its deploy SSH host. `piviki.fi` stays the coming-soon page, `api.piviki.fi` sends `X-Robots-Tag: noindex`, and sign-up stays allowlist-only." (b) Put `api.piviki.fi` behind an Access policy until the soft open (the import goes through `admin`, so nothing needs it publicly) and remove the policy at the cutover. Then list that removal as a Cutover step. Either way, add `noindex` on production `api` until the soft open.

**F2. The Google steps at the soft open name the wrong setting, and two real steps are missing.** (C9, C10, C11)
The authorized redirect URI is on `api.piviki.fi` (`/api/v1/auth/google/*`), which is registered from the production start. What `piviki.fi` needs is to be core's allowed return origin after login. At the soft open, the consent screen must also be published "In production", or only listed test users can sign in. Separately, the production start may now come long before the soft open, so a production OAuth client created then and left unused can be deleted after about 6 months.
**Fix:** Cutover line: "add `piviki.fi` to production's CORS and CSRF origins and to core's Google return origins, publish the Google consent screen to production with `piviki.fi` homepage and privacy policy links, point `piviki.fi` at the patron site, ...". AD-17 per-environment line: "production's Google OAuth client is created, or confirmed to still exist, at the soft open". Or have a founder's pilot sign-in use it at least every 5 months.

### Tier 2: should fix

**F3. Default `workers.dev` and preview URLs undercut "behind Access" on test and "no patron site host" on production.** (C13)
Test's patron Worker is reachable at its `workers.dev` name without Access unless `workers_dev = false`. Preview, version and deployment URLs stay on even then. A production Worker deployed before the soft open would make the production build public under a `workers.dev` name, without `noindex`.
**Fix:** In AD-15 or AD-17, set `workers_dev = false` and `preview_urls = false` in both environments' Wrangler config, or protect them with Access. Add a deploy check that the `workers.dev` URL does not serve the site.

**F4. Cutover DNS mechanics and the first run of the production patron build.** (C12)
A Workers Custom Domain can't be created over an existing CNAME. The coming-soon records live in production's OpenTofu state, so one apply must delete them and create the custom domain, followed by certificate issuance. With `beta` gone, the production build (with `api.piviki.fi` as its API origin) runs for the first time on public `piviki.fi`.
**Fix:** In the R21 or `epic-soft-open` runbook: one OpenTofu apply that swaps the coming-soon record for the Workers Custom Domain, plus a rollback (re-create the record). Before the swap, smoke-test the production build on an Access-protected version URL, with that origin added to CORS for the test and removed afterwards. Or accept that the first production run is public and say so.

### Tier 3: tidy-up

**F5. Update the memlog AWS cost entry.** (C1 to C4)
"Third-party aggregators, to confirm on the AWS pricing page" can now say: confirmed 2026-11-05 against AWS price files published 2026-11-04 (t3.small 0.0216, IPv4 0.005, gp3 0.0836 USD/GB-month, Stockholm). That is 21.1 USD, or about 18.7 EUR excl. VAT. Drop "free 750 h/month for 12 months on new accounts", which belongs to the free tier that ended on 2025-07-15. Note that commitments (Savings Plans or RIs) were not priced, and that month to month excludes them. Note too that t3.small has half of VPS-1's memory.

**F6. Record the OVH no-commitment price.** (C5)
OVH catalog: 4.49 EUR/month excl. VAT, no setup fee, about 5.63 EUR with VAT, EU data centres including DE and WAW. This closes the open item in review-verify-2026-11-05. The 15 EUR cap is the whole running cost, but the host is the only line here checked against a vendor price. Mail provider, R2 beyond the free tier and the domain are not totalled in this update.

### Info

- Payload 3.92.1 matches CORS and CSRF origins exactly (`includes` / `indexOf`). This is in the source, not stated in the docs. A requirement test for "exact origins" is cheap and would catch upstream changes.
- Without `Origin`, Payload's CSRF fallback accepts `Sec-Fetch-Site: same-site`, so every `*.piviki.fi` subdomain, including test's, counts as same-site for cookie requests. Host-only cookies keep test and production apart. Keep cookies host-only (AD-6 already requires it).

## Could be out of date and was not confirmed

- AWS Savings Plan or RI prices for t3.small in eu-north-1 (not fetched).
- Google's official help article on deleting unused OAuth clients (forum and staff statements only).
- Whether Wrangler's `preview_urls` defaults to on for the version in use (the docs page says workers.dev being off does not turn preview URLs off; the default was not read).
- OVH stock of VPS-1 2027 in a given EU data centre at order time.
- Finnish VAT rate at 25.5 % (not re-checked).

## Sources

- AWS price files: https://b0.p.awsstatic.com/pricing/2.0/meteredUnitMaps/ec2/USD/current/ec2-ondemand-without-sec-sel/EU%20(Stockholm)/Linux/index.json, .../ec2/USD/current/ebs.json, .../vpc/USD/current/vpc.json
- https://aws.amazon.com/vpc/pricing/
- https://aws.amazon.com/about-aws/whats-new/2025/07/aws-free-tier-credits-month-free-plan/
- ECB reference rates: https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml
- OVH order catalog: https://eu.api.ovh.com/v1/order/catalog/public/vps?ovhSubsidiary=IE ; https://www.ovhcloud.com/fr/vps/ ; https://www.ovhcloud.com/en-ie/vps/
- https://developers.google.com/identity/protocols/oauth2/web-server
- https://support.google.com/cloud/answer/15549945
- Unused OAuth client deletion: r/googlecloud and r/Firebase threads "Automatic deletion of unused OAuth clients" (2025)
- https://developers.cloudflare.com/workers/configuration/routing/custom-domains/
- https://developers.cloudflare.com/workers/static-assets/routing/worker-script/
- https://developers.cloudflare.com/workers/configuration/routing/workers-dev/
- https://payloadcms.com/docs/configuration/overview ; https://payloadcms.com/docs/authentication/cookies
- Payload 3.92.1 source: `piviki-core/node_modules/.pnpm/payload@3.92.1_.../payload/dist/auth/extractJWT.js`, `.../dist/utilities/headersWithCors.js`
