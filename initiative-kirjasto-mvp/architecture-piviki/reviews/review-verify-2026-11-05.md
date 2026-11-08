# Verify review: environments change and TypeScript 7 (2026-11-05)

Spine: `architecture-piviki.md`. Scope: the five memlog entries dated 2026-11-05. These cover AD-17 Host, AWS, Production start, Backups, Environments and Test environment; the retirement of AD-8 `patronAccountsAllowed`; AD-18's AWS sentence; the Stack row for TypeScript; and the Deferred rows "Test environment after the credits", "EC2 instance size and region" and "OVH data centre". Lens: is each committed fact checked against current sources rather than asserted? Checks from earlier verify reviews are not repeated: IMDS hop limit, instance identity role, GHCR IPv6, Universal SSL, the Bitwarden plan and the AWS DPA.

Date checked: 2026-11-05. Method: inspection plus web research. Read-only use of the npm registry, vendor docs and the public GitHub Actions API for `piviki-org/piviki-core`. Nothing was installed or changed.

## Verdict

**Holds for TypeScript and OVH. The AWS free-plan sentence is under-checked.** TypeScript 7.0.2 exists and is npm `latest`. Next.js 16.4 supports it through the `tsc` CLI, Payload 3.92 does not depend on its compiler API, and the CI run the memlog cites is real and green. OVH VPS-1 is still sold, far below 15 EUR. The free-plan argument covers the data on AWS, but not the test environment's lifetime. On the free plan the account closes after **6 months from account creation**, or when the credits run out, whichever comes first. The test environment is now permanent and is needed from the first story onward. The spine tracks only "when the credits run low", and "Founders check the AWS bill monthly" does nothing on a plan that has no bill.

## Claims checked

| # | Claim in spine or memlog | Source | Result |
|---|---|---|---|
| C1 | TypeScript 7.0.2 exists and is current | npm registry dist-tags (live): `latest: 7.0.2`, `rc: 7.0.1-rc`, `beta: 7.1.0-beta`, `next: 7.1.0-dev.20261105.1`; TypeScript blog "Announcing TypeScript 7.0" (2026-07-08) | **Confirmed.** 7.0.2 is the newest stable release and 7 is the current major. 7.1 is in beta. 7.0 is the native (Go) port and ships **no JavaScript compiler API**; the blog says a new API is planned for 7.1. `@typescript/typescript6` provides `tsc6` for tools that still need the 6.x API. |
| C2 | Next.js 16.4.4 supports TypeScript 7 in `next build` | nextjs.org/blog/next-16-3 (2026-08-03); Next.js docs "TypeScript", section "Using TypeScript 7" (updated 2026-08-25) | **Confirmed.** Since 16.3, `next build` type-checks with the project-local `tsc` CLI by default; `experimental.useTypeScriptCli: false` switches back to the compiler API, which TS 7 lacks. Caveats: CLI diagnostics have no Next.js-specific rewriting, and the whole `tsconfig` project is checked, tests included. `piviki-core/next.config.ts` sets neither `useTypeScriptCli` nor `ignoreBuildErrors`, so the image build type-checks with TS 7. Next.js 15.5 and 16.2 refuse TS 7 (vercel/next.js PR #96110), so a downgrade below 16.3 would break the build. |
| C3 | Payload 3.92.1 works with TypeScript 7 | npm `payload@latest` manifest (3.93.0); `piviki-core/pnpm-lock.yaml` | **Confirmed by inspection and by CI.** Payload has no `typescript` peer or dependency. It uses `json-schema-to-typescript`, `tsx` and `ts-essentials`, none of which needs the TS compiler API at run time (`ts-essentials` is types-only). The lockfile resolves `payload@3.92.1(…)(typescript@7.0.2)`. No official Payload statement on TS 7 was found. Support rests on the green build, not on a vendor claim. |
| C4 | "CI build green (typecheck, 32 tests, next build, Compose stack) as tag ba75aa21e5be-20261105" | GitHub Actions API, run 42000342225 (push to `main`, head `ba75aa21e5be`, 2026-11-05T09:11Z) | **Confirmed.** Jobs `test` (`pnpm install --frozen-lockfile`, `pnpm typecheck`, `pnpm test`, the three deploy-script tests), `stack` (Compose build, migrate, checks through Caddy, in-network checks, non-root/read-only check) and `image` (immutable tag, refuse overwrite, push) all succeeded. The Dockerfile runs `pnpm install --frozen-lockfile` then `pnpm build`, so `next build` ran on TS 7. The test count of 32 was not checked (step logs need auth). Note: `ba75aa2` is the merge of PR #3 (vitest 4.1.12 → 5.0.5), which came after the TypeScript merge `f50ac28` (PR #1, also green, run 41999511395). The tagged build therefore proves TS 7 **together with vitest 5**. The memlog credits it to PR #1 only. |
| C5 | AWS free plan acceptable "because AWS holds only test data" | AWS Billing User Guide "Choosing a plan"; EC2 User Guide "Track your Free Tier usage for Amazon EC2" | **Holds for the data, not for continuity (F1, F2).** Free plan: "ends after six months or when your credits are fully used – whichever occurs first … your account closes automatically, and you lose access to your resources and data. AWS retains your content for 90 days." There are no charges. Other promotional credits are not eligible. The account is upgraded automatically if it joins AWS Organizations. Remaining credits carry over on upgrade to the Paid plan. |
| C6 | Test instance: x86_64, EU region, no larger than VPS-1, standard CPU credits | EC2 Free Tier usage page; third-party price aggregators (devzero.io, aws-pricing.com, doit.com) | **Partly confirmed (F3).** Free-tier-eligible types for accounts created on or after 2025-07-15: `t3.micro`, `t3.small`, `t4g.micro`, `t4g.small`, `c7i-flex.large` and `m7i-flex.large`. The `t4g` types are Arm and excluded by the x86_64 rule. `c7i-flex.large` (2 vCPU, 4 GiB) matches VPS-1 2027 exactly; `m7i-flex.large` (8 GiB) is larger than VPS-1. Flex types are not burstable, so "standard CPU credits, not Unlimited" applies only if a `t3` is chosen. Stockholm on-demand prices (third-party, not checked against AWS): `c7i-flex.large` about 0.091 USD/h, `t3.small` about 0.0216 USD/h. |
| C7 | OVHcloud VPS-1, month to month, in an EU data centre, within 15 EUR/month | ovhcloud.com/fr/vps and /en-ie/vps (live); OVHcloud blog "VPS 2027" (2026-06-17); vpsbenchmarks trial of OVHcloud US VPS-4 2027 (2026-09-18) | **Confirmed, with the price for no commitment still open.** VPS-1 is now in the **VPS 2027** range: 2 vCores, 4 GB RAM, 40 GB NVMe, unlimited traffic at 500 Mbit/s, daily backup of the last 24 h included. The price is "à partir de 3,81 € HT/mois soit 4,57 € TTC/mois" (French VAT). The pages do not say whether the from-price needs a 6- or 12-month commitment. US listings show about a 15 % premium without commitment. Even with that premium and Finnish VAT at 25.5 %, VPS-1 costs about 5.5 EUR, well within 15 EUR. A 2027-range trial reports an x86_64 CPU (Intel, Haswell-class), so the x86_64 images rule holds. EU locations for the 2027 range could not be read (the configurator failed to load). OVH runs EU VPS sites in France, Germany (Frankfurt) and Poland (Warsaw), but whether VPS-1 2027 can be ordered in Germany or Poland is **unconfirmed**. |
| C8 | AD-8 `patronAccountsAllowed` retired; `signupOpen` and the beta allowlist are the gates | Spine inspection; ticket files | **Spine consistent:** the name no longer appears in the spine. **Downstream still stale (F5).** |
| C9 | "Founders check the AWS bill monthly" | AWS plan comparison | **Ineffective on the free plan (F2).** A free-plan account incurs no charges, so the bill is always zero. What matters is the credit balance and the plan end date. |

## Findings

### F1 (High): the free plan caps the test environment at 6 months from account creation, and nothing in the spine tracks that

The update makes AWS the only test environment from its first bootstrap. `epic-librarian-catalogue` stories 2 to 13 are built and checked there, `epic-patron-site` tests there, and test continues after launch. On the free plan the account closes 6 months after the **account was created**, not after the instance was created, even if credits remain. The Deferred row "Test environment after the credits … when the credits run low" and the AD-17 phrase "paid by the founders' credits" track only credit exhaustion. The 2026-11-03 memlog line "credits confirmed sufficient until the move" was checked against a move that no longer exists. Nobody has re-checked the credits against a test environment with no end date.

The data argument is sound: test data is recreatable, and production depends on nothing in it. The continuity argument is not stated. A closure mid-epic stops every story that "is built and checked on test" until a new account or host exists.

**Fix.** In AD-17 AWS: "The free plan ends 6 months after the account was created, or when the credits are used, whichever is first; the account then closes. Before that date, the founders upgrade to the Paid plan, with a budget alert, or move test elsewhere." In the Deferred row, make the decide-by "when the credits run low or a month before the free plan's end date, whichever is first". Record the account creation date and plan end date in the secrets inventory or runbook.

### F2 (Medium): "check the AWS bill monthly" means nothing on the free plan

Line 222 has the founders checking the AWS bill monthly. That line came from the paid-plan design. On the free plan nothing is ever billed, so the check always passes. The risks now are the credit balance and the plan end date.

**Fix.** Change it to: "Founders check the AWS credit balance and the free plan's end date monthly, in the Billing console." If the account is ever upgraded, add a 0 USD actual-spend budget alert. Also confirm that the founders' credits are the free plan's own 100 + 100 USD. The 2026-11-03 memlog calls them "promotional credits", and AWS says "Free account plans are not eligible for other promotional credits". If the credits are of another kind, they apply only on the Paid plan, and dropping the paid-plan rule loses them.

### F3 (Medium): EC2 size choice is now constrained by the free plan, and the likely choice burns the credits in about 3 months

Under the free plan, only the free-tier-eligible types above can be used. x86_64 and "no larger than VPS-1" (now confirmed at 2 vCores, 4 GB) leave `t3.micro`, `t3.small` and `c7i-flex.large`. `c7i-flex.large` matches VPS-1. Running all the time it costs roughly 66 USD a month, plus about 3.65 USD for public IPv4 and a little for EBS (prices from third-party aggregators). 200 USD then lasts about 3 months, or longer if the instance is often stopped. `t3.small` (2 GiB) stretches the credits past the 6-month cap but has half VPS-1's RAM, so "what runs on test runs on production" holds in the wrong direction only if memory is tight. 2 GiB for Postgres, Payload/Next, Caddy and cloudflared is a risk, not a fact; it was not measured.

**Fix.** Update the Deferred row "EC2 instance size and region": "VPS-1 (2027 range) is 2 vCores, 4 GB, 40 GB, confirmed 2026-11-05. Free-plan x86_64 choices are `t3.small` and `c7i-flex.large`; `c7i-flex.large` matches VPS-1 but uses about 70 USD of credits a month if left running." Drop the now-done "confirm its specs first" clause. Also note that the 40 GB disk on VPS-1 limits Postgres plus images plus logs. Media is on R2, so this is probably fine, but it is unverified.

### F4 (Low): the TypeScript 7 version claim is proven, but its conditions are not written down

TS 7 works only because Next.js ≥ 16.3 type-checks with the `tsc` CLI. Three things would break it: setting `experimental.useTypeScriptCli: false`, pinning Next.js below 16.3, or adding any tool that imports the `typescript` API. Examples are typescript-eslint and `ts-morph`; Payload plugins that use the compiler API would count too. The tsconfig's `plugins: [{ "name": "next" }]` language-service plugin does nothing under TS 7, so editors lose the Next.js checks. Separately, the local `piviki-core/node_modules` still holds `typescript@6.0.3` (lockfile: 7.0.2). Typecheck results on this machine therefore differ from CI until `pnpm install` runs. That is a working-copy note, not a spine issue. Installing is left to the founders.

**Fix (optional).** One clause in the Stack table or conventions: "TypeScript 7 has no compiler API; Next.js type-checks with the `tsc` CLI (≥ 16.3); tools needing the TS API use `@typescript/typescript6`." The memlog's (version) line could also name the merged commit range (`f50ac28`, `ba75aa2`) and note that vitest 5.0.5 landed in the same tagged build.

### F5 (Low, outside the spine): tickets still describe the move and `patronAccountsAllowed`

`tickets/epic-initial-load/epic-initial-load.md:28` still requires "Production has moved to OVH by spine AD-17's seven steps". `tickets/epic-patron-site/epic-patron-site.md:37` still extends "`patrons` and `patronAccountsAllowed` from epic-librarian-catalogue 1.2". `tickets/epic-librarian-catalogue/tickets/tickets.toml` also still matches the name. The spine change is consistent internally, but these tickets now contradict it. Hand them to Sofia's replan; this review changed none of them.

## Informational (not changed by this update)

- npm `latest` is ahead of the Stack table: `next` 16.4.5 (table 16.4.4) and `payload` 3.93.0 (table 3.92.1). Dependabot will raise both. No action is needed for this update.
- Joining AWS Organizations upgrades the free plan automatically. This is harmless now that a paid plan is acceptable, but it would start charges.

## Not confirmed

- The 6-month free-plan end date for the founders' account (needs the account creation date).
- Whether the founders' credits are free-plan credits or other promotional credits.
- The OVH VPS-1 2027 price without commitment, in EUR, and whether it can be ordered in Germany or Poland.
- EC2 prices in eu-north-1 (third-party aggregators only).
- The "32 tests" count in the CI log.
- Whether 2 GiB (`t3.small`) is enough for the stack.

## Sources

- npm registry, typescript dist-tags: https://registry.npmjs.org/-/package/typescript/dist-tags
- TypeScript blog, Announcing TypeScript 7.0: https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/
- Next.js 16.3 blog: https://nextjs.org/blog/next-16-3
- Next.js docs, TypeScript (Using TypeScript 7): https://nextjs.org/docs/app/api-reference/config/typescript
- vercel/next.js PR #96110, reject TypeScript ≥ 7 on 15.5: https://github.com/vercel/next.js/pull/96110
- vercel/next.js discussion #95633: https://github.com/vercel/next.js/discussions/95633
- npm registry, payload latest: https://registry.npmjs.org/payload/latest
- npm registry, next latest: https://registry.npmjs.org/next/latest
- GitHub Actions runs, piviki-org/piviki-core: https://api.github.com/repos/piviki-org/piviki-core/actions/runs and run 42000342225 jobs
- AWS Billing, Choosing a plan: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html
- EC2 User Guide, Track your Free Tier usage: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html
- c7i-flex.large and t3.small prices (third party): https://www.devzero.io/instances/aws/c7i-flex.large, https://aws-pricing.com/c7i-flex.large.html, https://www.doit.com/compute/spot/eu-north-1/t3.small
- OVHcloud VPS: https://www.ovhcloud.com/fr/vps/ and https://www.ovhcloud.com/en-ie/vps/
- OVHcloud blog, VPS 2027: https://blog.ovhcloud.com/vps-2027/
- vpsbenchmarks, OVHcloud US VPS-1 2027 plan and VPS-4 2027 trial: https://www.vpsbenchmarks.com/hosters/ovhcloud_us/plans/vps-1-2027, https://www.vpsbenchmarks.com/trials/ovhcloud_us_performance_trial_18Sep2026
