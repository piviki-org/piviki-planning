---
title: 'Technical research: email provider selection'
type: 'technical'
shape: 'select'
topic: 'Email provider selection'
decision: 'Pick the transactional email provider (SMTP, own domain, GDPR baseline, accepts an unregistered group) for Piviki story 9'
deepened: '2026-11-03 (compliance gates G6, G7 and re-weighting)'
source: 'native run'
status: complete
preset: 'standard'
validation: 'normal'
claims_verified: 17
claims_unverified: 9
claims_disputed: 1
created: '2026-11-03'
updated: '2026-11-03'
---

# Technical research: email provider selection

**Decision this research serves:** which transactional email provider a founder picks in story 9, "Mail sends from the library's domain" (`epic-librarian-catalogue`), measured against the spine's criteria in AD-12 and AD-18.

## Executive summary

**Pick Lettermint (Free plan).** After the compliance Deepen (G6: accepts an unregistered volunteer group; G7: GDPR baseline by contract), **Lettermint is the only provider whose current terms were confirmed to pass every hard gate.** It leads the re-weighted matrix 82 to 68 of 95. Scaleway TEM stays the runner-up, but only conditionally: its current DPA could not be read.

1. **Lettermint's DPA meets the whole GDPR baseline in writing** (version of 2026-10-09) [8]. Email data stays in the EU and support data in the EU/EEA, with no remote access from outside the EEA (Sec. 8). Breach notice comes within 24 hours (Sec. 10.1). New sub-processors get 30 days' notice, and we can object and terminate (Sec. 7.2–7.4). Audits can be on-site (Sec. 12). Government requests get notice and challenge (Sec. 5.8). Email data is kept 28 days and backups at most 14 days.
2. **Lettermint accepts an unregistered group.** Its terms cover natural persons. The verification form has an "individual" type with no company or VAT field, and review is "usually within a few hours" [52][53]. This removes the earlier worry that review time was unknown.
3. **Scaleway's compliance could not be confirmed.** Its current DPA and sub-processor pages returned 403. The only DPA text found is a 2021 third-party copy [58]. In that copy, Scaleway has a general authorisation to change sub-processors with no notice period or objection right, breach notice is only "as quickly as possible", and Scaleway appoints the auditor. Its sub-processor list (reviewed July 2025) also gives a general authorisation [59]. On top of that, Scaleway keeps recipient addresses and subjects for as long as the domain is active [10], and its sign-up identity check uses a volunteer's photo ID plus a face check [56].
4. **SES and SMTP2GO now fail G7.** AWS only commits to keep data in the chosen region, with exceptions "as necessary to provide the Services" and for government orders [60]. SMTP2GO's DPA and sub-processor list are not public [63][64].

**Biggest caveats:**
- **Durability.** Lettermint is a two-founder company that started in 2025, and this run found no independent user reports on it [7].
- **Deliverability is unproven for every provider.** It has to be tested in story 9.
- **The account holder.** Piviki is not registered, so the Lettermint account and DPA would be in a founder's personal name (the "individual" type). Who counts as controller until registration is a question for the Ombudsman's association guide, not this research.
- **No fallback is confirmed yet.** If Lettermint fails, no other provider is confirmed against G7 until someone reads Scaleway's current DPA.

**Reversibility is cheap.** Everything goes through one Nodemailer SMTP adapter (AD-12). Switching provider means new SMTP credentials and a new set of DNS records in OpenTofu, with no code change.

## Requirements used

Agreed with the user on 2026-11-03, from AD-12, AD-18, the initiative constraints and story 9.

| # | Hard gate | Source in project |
|---|---|---|
| G1 | SMTP relay, because all mail goes through one Nodemailer SMTP adapter | AD-12 |
| G2 | Sends from `@piviki.fi`, with SPF, DKIM and DMARC as DNS records that OpenTofu can manage | AD-12, story 9 |
| G3 | A written data processing agreement (DPA) on file before `signupOpen` | AD-18, epic decision 2026-11-03 |
| G4 | Open and click tracking fully off, with no link rewriting | AD-12 |
| G5 | Not a personal mailbox (Gmail or Proton) | initiative constraints |
| G6 | **Accepts an unregistered volunteer group** (no Y-tunnus or VAT): its terms and sign-up review allow individuals or unregistered groups | Deepen 2026-11-03 (user) |
| G7 | **GDPR baseline by contract:** Art. 28 DPA; published sub-processor list with advance notice; patron data stored and processed only in the EU/EEA by contract, not just by region choice; breach notice without undue delay; documented retention | Deepen 2026-11-03 (user), AD-18 |

Weights after the Deepen (agreed 2026-11-03). The original weights are kept in brackets.

| Weighted preference | Weight | Why |
|---|---|---|
| EU data residency and minimisation | 3 (3) | GDPR; the patron list may count as Art. 9 data (brief risk) |
| DPA strength | 3 (new) | Breach-notice time, sub-processor objection right, audit, government-access handling |
| Acceptance risk | 3 (new) | Piviki is unregistered; a refused or stalled review blocks story 9 |
| Cost at about 100 emails a month | 3 (3) | The stack runs on about 15 EUR a month (AD-17) |
| Deliverability | 2 (3) | Reset, verification and pickup emails must arrive |
| Operational load (DNS, console, MFA) | 2 (2) | Two volunteers |
| Exit cost and vendor durability | 2 (2) | Five-year regret risk |
| Free-tier traps | 1 (2) | Mostly covered now by acceptance risk |

## D1 — Candidate screen

The run looked at fifteen candidates. Four went through to scoring: **Lettermint**, **Scaleway TEM**, **Amazon SES (eu-north-1)** and **SMTP2GO (EU account)**. After the Deepen, SES and SMTP2GO fail G7 (see D6). The screening record is in "Considered and rejected" below. Lettermint and Scaleway TEM are the only two with documented EU-only processing, no tracking by default, and a free tier that covers the volume without a card (Lettermint) or a fixed fee (Scaleway) [1][9][10].

## D2 — Compliance and data

- **Lettermint.** Lettermint B.V. is a Dutch company [45]. All five sub-processors are EU companies: UpCloud (NL, hosting), OVH (DE, encrypted backups) and Postmastery (NL, deliverability) handle email; Aikido and Mistral are listed only as "Europe" and have no access to email [2]. Round 1 said "all NL/DE", which was too strong and is corrected here. The DPA applies automatically to every plan, and a countersigned copy is available on request [1][8]. Message content and recipient metadata are kept 28 days on all plans [1][4]; the DPA also allows a period the controller configures [8]. Analytics events are kept 90 days from 2026-11-08 (28 days before then), with HMAC-hashed recipients; raw tracking signals are kept 28 days [4]. The DPA allows transfers outside the EEA under SCCs but names none [2]. Confidence is medium: this comes from the vendor's own pages only.
- **Scaleway TEM.** It runs only in the Paris region (fr-par). It uses no non-EU sub-processors and "no dedicated subprocessor" for content or metadata, and data is not replicated outside the EU [9][10]. TEM "does not add tracking pixels, rewrite links, track clicks", and headers and body are sent as-is [10]. Content is deleted on delivery or hard bounce (encrypted backups 7 days). Sending metadata, including the subject and RCPT TO, is kept while the domain is active; individual records can be deleted through a support ticket [10]. The DPA (SCCs in Art. 11) is in Scaleway's contracts; this run did not read its text [10].
- **Amazon SES.** The AWS Data Processing Addendum is part of the AWS Service Terms [16]. The SES terms let AWS "store and scan your SES Email" to fight spam and malware [16]. Whether a US parent company is acceptable even with eu-north-1 hosting is a legal question this run did not research.
- **SMTP2GO.** EU accounts are hosted in Amsterdam, which "sends emails and stores account information" [23]. The sub-processor list and the transfer mechanism were not retrieved. A third party says processors outside the EEA are allowed (low confidence) [47].

## D3 — Cost and free-tier traps (about 100 emails a month)

| Provider | Cost at 100/mo | Free-tier limits | Onboarding gate |
|---|---|---|---|
| Lettermint | €0, no card [1] | 300/mo, no daily cap, 1 user on Free [1] | Account verification before sending to real addresses; "usually within a few hours" [5][53] |
| Scaleway TEM | €0 (300 free, then €0.25/1k, no fixed fee) [9][13] | Default quota 10k/mo; "new users may face stricter initial limits" [11] | Payment method needed [14]; ID may be requested depending on account and usage, and is required to raise quotas [11][14] |
| Amazon SES | about $0.01 ($0.10/1k); new accounts get up to $200 of credits over 6 months [15] | Sandbox: 200 emails per 24h, verified recipients only [17][20] | Production-access review per region, initial reply within 96h; many reports of denials with no reason given [17][18] |
| SMTP2GO | $0 [22] | 1,000/mo [22] | Work email required (Gmail rejected) plus SMS verification [22] |

Price-change history: SES's pricing page now shows only a credit-based free tier [15]; the earlier monthly email allowance and the switch date are unverified. SendGrid retired its free plan in May 2025 [40]. Mailgun cut its free tier to 100/day around October 2024 [44]. This run found no pricing-change history for Lettermint, which is too young to have one, or for Scaleway TEM.

## D4 — Deliverability and real-world experience

- **Lettermint:** the run searched Reddit, HN, blogs and G2 and found **no independent user reports**. The company itself reports "millions of emails per month", government and healthcare customers, and email delays while it scaled and migrated to multiple availability zones [7]. Its site says "ISO 27001 certified", but its January 2026 blog called certification a 2026 goal; the certificate may have come later and was not checked [7][45].
- **Scaleway TEM:** domains are blocked automatically when the undelivered-mail rate is high [9]. Independent deliverability reports were not searched within budget.
- **SES:** the run found repeated reports of production-access denials, including the "no billing history" catch-22, and AWS staff say Premium Support cannot override the Trust & Safety team [18].
- **Comparison base:** the run found suspension stories for transactional-only accounts at Brevo [48], Mailgun [34] and MailerSend [36]. It found none for either finalist, but because so little has been written about them, that absence is weak evidence.

## D5 — Acceptance of an unregistered group (Deepen)

| Provider | Accepts individuals or an unregistered group? | What review asks for | Review time | Acceptance risk |
|---|---|---|---|---|
| Lettermint | **Pass.** The terms define a User as "the natural person or legal entity" [52] | An "individual" or "business" type, website, email types, unsubscribe handling and volume; no company, VAT or ID field [53] | "Usually within a few hours" [53] | Low. No stated rejection criteria. The nonprofit discount needs a registered nonprofit [54], and Free doesn't need it |
| Scaleway TEM | **Pass.** There is a "Personal Project" account type, and VAT is asked only for Corporate [55] | Payment method; may ask for a government photo ID plus a face check "depending on your account and usage" [56] | No TEM sender review documented. The ID check is self-service, otherwise it goes to manual support [56] | Low to medium. A volunteer's biometric ID check; new users may face stricter limits [11]. The TEM conditions contain no professional-only clause [57] |
| Amazon SES | Unclear. The request form asks for no company number; whether AWS allows individual accounts was not checked [17] | Use case, website, and a process for bounces and complaints [17] | Up to 96 hours for a first reply [17] | Medium to high. Opaque denials, including "no billing history" [18] |
| SMTP2GO | Likely pass. It requires an own-domain sign-up address, not a registration (our reading) [62] | Ongoing review based on quota use; a full review takes about 500–700 emails [62] | Could stay pending for months at about 100/mo; the 1,000 starting quota covers the volume [62] | Low to medium |
| Sweego | Unclear. The terms are open to "toute personne physique ou morale", but the listed sign-up data includes SIREN/SIRET and VAT [65] | Not found | Not found | Medium |

## D6 — DPA strength against the GDPR baseline (Deepen)

| Item | Lettermint [8] | Scaleway (2021 DPA copy [58], list [59]) | AWS SES [60][61] | SMTP2GO [63][64] |
|---|---|---|---|---|
| Art. 28 DPA | Public; applies automatically; Sec. 3 description | Exists; current text not readable (403) | Public; supplements the Customer Agreement | Only inside the dashboard (search snippet) |
| Sub-processor notice and objection | 30 days' active notice; right to object; terminate before onboarding (Sec. 7.2–7.4) | "General authorization to add, modify or remove"; no notice period or objection right stated | 30-day website update, email alert opt-in; objecting means leaving or changing region (Sec. 6.1) | List not public |
| EU/EEA-only by contract | Yes: email data in the EU (8.1), all other data including support in the EU/EEA (8.2), no remote access from outside the EEA without authorisation (8.4) | 2021 text: no storage outside the EU without the client's express agreement (Art. 11.1b); current text unverified | **No**: region choice, with exceptions "as necessary to provide the Services" and government orders (Sec. 12.1) | EU servers only on the dedicated EU service (snippet); NZ company |
| Breach notice | Within 24 hours (10.1) | "As quickly as possible", no hour figure | "Without undue delay", no figure (9.1) | Not found |
| Retention | 28 days (or a period the controller sets), analytics ≤ 90 days, backups ≤ 14 days, 30-day exit window | Messages deleted after processing [57]; metadata kept while the domain is active [10] | Deleted via service controls; up to 90 days after termination (Sec. 14) | 35 days by default (snippet) |
| Audit | On-site possible; 30 days' notice; we choose the method (Sec. 12) | Yearly, 15 days' notice, auditor appointed by Scaleway (2021) | ISO/SOC reports under NDA; no on-site right (Sec. 10) | Not found |
| Government requests | Notify, review, challenge, disclose the minimum (Sec. 5.8) | Transfers required by EU or member-state law, with notice (2021) | Disclosure "to comply with the law or a valid and binding order"; notice unless prohibited (Sec. 3) | Not found |
| **G7 verdict** | **Pass** | **Unclear**: needs the current DPA | **Fail** (EU-only by contract) | **Fail** (no public DPA or sub-processor list) |

## Decision matrix

Re-scored after the Deepen with the new weights (maximum 95). Change the weights to re-weight: each total is the sum of weight × score. SES and SMTP2GO fail G7 and are no longer scored; their earlier scores were 46 and 51 of 75.

| Criterion (weight) | Lettermint | Scaleway TEM |
|---|---|---|
| EU data and minimisation (3) | 5: EU-only sub-processors, 28-day retention [2][4][8] | 4: EU-only, no dedicated TEM sub-processors, but recipient metadata kept while the domain is active [10] |
| DPA strength (3) | 5: 24h breach notice, objection right, on-site audit, government-access clause [8] | 2: current text unreadable; the 2021 copy has no objection right, no hour figure, and an auditor Scaleway picks [58][59] |
| Acceptance risk (3) | 4: individual type, hours to review; no independent reports [52][53] | 3: Personal Project type, but a volunteer's photo ID plus face check and stricter initial limits [11][55][56] |
| Cost (3) | 5: €0, no card [1] | 5: €0, card on file [9][14] |
| Deliverability (2) | 3: no independent reports found | 3: no independent evidence |
| Ops load (2) | 5: CNAMEs for DKIM (automatic rotation) and Return-Path plus a DMARC TXT; passkeys and MFA [1][3][6][49] | 4: SPF + DKIM TXT + MX + DMARC, manual verify; MFA and passkeys [12][14] |
| Exit and durability (2) | 3: SMTP makes exit cheap; vendor under 2 years old, 2 founders [7] | 4: SMTP exit; product documented since 2023 [46] |
| Free-tier traps (1) | 3: 1 user on Free [1] | 4: no fixed fee [13] |
| **Total (of 95)** | **82** | **68** |

**Sensitivity:**
- **Lettermint keeps the lead under the most plausible downgrade.** With durability scored 1 instead of 3, it is still ahead (78 vs 68).
- **Scaleway only catches up with a much better DPA.** With a current DPA that matches Lettermint's (DPA score 5), Scaleway reaches 77, still behind.

Before the Deepen, under the original weights, the score was 61 vs 60 of 75.

SES is still the **disputed** case on tracking [19]. That no longer matters now that SES fails G7.

## Considered and rejected

As the user asked, here are all providers considered, including those that fail a gate, with the reason for each.

| Provider | Gate failed or reason cut | Evidence |
|---|---|---|
| Personal Gmail / Proton | **G5**: excluded by the initiative constraints (mixes personal and library mail) | initiative constraints |
| Brevo | **G4**: fully disabling tracking on transactional mail is "upon request and for our Enterprise customers"; still unresolved in Nov 2025. Also reports of transactional-only accounts suspended soon after signup | [25][26][48] |
| Postmark | **EU preference (heavy)**: hosted only in the US, "no plans to add servers in the EU". Free tier is a hard cap of 100/mo with no overage, which leaves no headroom. SMTP on Free not confirmed | [27][28] |
| Resend | **EU preference (heavy)**: the EU region changes only where mail is sent from; "all account data, including email metadata, logs, and API records, is stored in the United States" | [29][30][31] |
| Twilio SendGrid | **Cost**: free plan retired from 2025-05-27; paid plans only | [40] |
| Mailgun (EU) | Passes the gates on paper. **Cut** for many reports of accounts disabled on signup or after small sends, a US group company as sub-processor, and a free tier cut to 100/day in 2024 | [32][33][34][44] |
| MailerSend | **G4 unclear** (tracking default not found). **Cut** for a card plus account approval even on Free, and reports of rejections with no clear reason | [35][36] |
| Mailjet | Passes G1 and G4 (SMTP headers turn tracking off per message) [39]. **Cut** on the EU preference: it runs on Google Cloud [50], and the Sinch DPA allows transfers within the group and under the EU-US Data Privacy Framework | [37][38][39][50] |
| Cloudflare Email Service | **Cost / G1 friction**: sending needs Workers Paid [41]; SMTP on port 465 only [51]; no data-location statement | [41][51] |
| Amazon SES (eu-north-1) | **G7** (Deepen): AWS commits only to the chosen region, with exceptions "as necessary to provide the Services" and government orders; no breach-hour figure; objecting to a sub-processor means leaving. Also G6 risk: opaque sandbox denials. Scored 46/75 before the Deepen | [17][18][21][60][61] |
| SMTP2GO (EU account) | **G7** (Deepen): DPA visible only inside the dashboard, no public sub-processor list; NZ company, privacy policy from 2023; EU-only only on the dedicated EU service. Scored 51/75 before the Deepen | [24][62][63][64] |
| Sweego (FR) | **G6 unclear, G7 not checked**: the terms are open to natural persons, but the listed sign-up data includes SIREN/SIRET and VAT. Hosting, sub-processors and whether tracking can be turned off were not found | [42][65] |
| Zoho ZeptoMail | **Insufficient evidence**: has an EU data centre; SMTP, DPA and tracking not confirmed | [43] |
| Mailtrap, Infomaniak, Elastic Email, MailPace | **Not screened** (out of budget) | — |

## Cross-dimension insights

- **The compliance gates narrow the field to one confirmed provider.** Contract-level EU-only processing (G7) removes the US- and NZ-based options even when they host in the EU [60][63]. What's left is a young vendor [7] with a fallback whose current DPA nobody has read [58]. That dependence on a single vendor is the price of the gate, and recommendation 7 is the cheapest way to reduce it.
- **"EU region" does not mean EU data.** Resend's EU region still stores logs and metadata in the US [29], and Mailjet's EU hosting sits under a group DPA that allows US transfers [38]. Only a provider's sub-processor list and retention page answer the residency question; the region selector does not.
- **The two finalists trade minimisation against durability.** Lettermint's strength (28-day retention, small focused EU stack) and its weakness (two founders, under two years old) both come from the same fact: it is a young company.
- **Subject lines are personal data at the processor.** Scaleway keeps them as metadata [10], and every provider logs them for some period. Story 9 and the outbox rendering in AD-12 should keep patron names and book titles out of subjects, whichever provider is chosen.

## Recommendations

1. **Story 9: choose Lettermint Free**, sending from a subdomain such as `mail.piviki.fi` as Lettermint recommends [6]. Confidence: medium. The gate facts are primary and verified [2][3][5]; durability and deliverability are unverified.
2. **Story 9: start Lettermint's account verification on day one**, as account type "individual" in one founder's name, with the website, email types (transactional), unsubscribe handling and volume [53]. The stated time is a few hours [53]. Story 9's verify step (a real librarian reset received from `@piviki.fi`) depends on it.
2a. **AD-18 / privacy notice:** keep the countersigned Lettermint DPA (version 2026-10-09) on file [8]. Record that the founder holds the account and DPA until Piviki registers. When it registers as an ry, transfer the account and re-sign the DPA in the association's name.
3. **Story 9, OpenTofu DNS:** two DKIM CNAMEs (`lm1`/`lm2._domainkey`), a Return-Path CNAME that covers SPF, and DMARC with `aspf=r`, because `aspf=s` blocks first verification [6]. Create the CNAMEs as DNS-only records in Cloudflare (not proxied).
4. **Story 9, tracking (AD-12):** Free has no tracking, but also send the `X-LM-Override-Track-Opens: false` and `X-LM-Override-Track-Clicks: false` headers from the Nodemailer adapter [3]. That keeps tracking off after any plan upgrade, and a test can assert the headers are there.
5. **AD-12 / outbox rendering:** no patron name, phone number or book title in subject lines (see insights). This holds whichever provider is chosen.
6. **AD-18 / privacy notice:** name Lettermint and state both retention periods: 28 days for content and recipient metadata, and 90 days for analytics events with hashed recipients [4]. Optionally name Scaleway as a standby processor now, so a switch does not need a new notice first.
7. **Confirm the fallback now:** a founder with a Scaleway account downloads the current DPA from the console and checks it against G7 (sub-processor notice and objection, breach timing). Until then, Scaleway is a fallback on paper only.
8. **Hedge:** keep the Scaleway TEM records (SPF include `_spf.tem.scaleway.com`, DKIM TXT, MX, DMARC [12]) as a disabled OpenTofu block. **Scaleway wins instead if:** Lettermint's review stalls past about a week or is refused; Lettermint changes its Free plan or ownership; or the founders weigh vendor durability above 28-day retention. Under the re-weighted matrix, only a disqualifying event, not a re-weighting, makes Scaleway the pick. Confidence for Scaleway as fallback: high on G1, G2, G4 and G6; **G7 unclear** until its current DPA is read [9][10][11][55][58].

**Strongest argument against the pick:** Lettermint is a two-person company that started in 2025. There are no independent user reports on it, and its ISO 27001 claim is newer than its January 2026 blog, which called certification a goal, and is unverified [7][45]. If it folds or reprices, Piviki migrates mid-service, and today the only other EU candidate has an unconfirmed DPA. The SMTP seam keeps that migration small: credentials plus DNS.

**Staleness:** this is a selection report. Refresh it before acting if it is more than two quarters old (after 2027-05).

## Open questions

- **Does Lettermint approve an "individual" account for a community library within hours, as stated?** Answer: apply in story 9 and log the result.
- **Who is controller, and who signs the DPA, while Piviki is unregistered?** Answer: the Data Protection Ombudsman's guide for associations (tietosuoja.fi), which is already a brief risk item. This is a legal question, not research.
- **What does Scaleway's current DPA say?** Answer: download it from the Scaleway console (Account > Contracts).
- **How do both finalists actually do on deliverability to Gmail, Outlook and Finnish ISPs (Elisa, Telia, DNA)?** Answer: send test resets to seed mailboxes from both during story 9, or run a Deepen focused on independent reports.
- **Is Lettermint's ISO 27001 certificate real?** Answer: check the trust centre at trust.lettermint.co.
- **What is in Scaleway's DPA text and Scaleway-wide sub-processors (payment, support)?** Answer: read scaleway.com/en/contracts and /subprocessorlist in a browser (the fetcher got 403).
- **What did the providers that were not screened offer (Mailtrap, Infomaniak, Elastic Email, MailPace) or that have thin evidence (Sweego, ZeptoMail)?** Answer: a Deepen run if neither finalist works out.

## Source appendix

| # | Supports | Publisher | Pub date | Accessed | Confidence |
|---|---|---|---|---|---|
| [1] | Lettermint plans: Free 300/mo, no daily cap, SMTP, DPA all plans, 1 user; Starter €10/10k | [Lettermint](https://lettermint.co/pricing) | undated (live) | 2026-11-03 | medium (vendor only) |
| [2] | Lettermint sub-processors, all EU | [Lettermint](https://lettermint.co/subprocessors.md) | 2026-10-29 | 2026-11-03 | high |
| [3] | Lettermint tracking off by default; SMTP override headers | [Lettermint](https://lettermint.co/docs/platform/emails/tracking/introduction) | undated | 2026-11-03 | high |
| [4] | Lettermint 28-day retention of content and metadata | [Lettermint](https://lettermint.co/docs/platform/emails/data-retention) | undated | 2026-11-03 | medium |
| [5] | Lettermint compliance verification gate | [Lettermint](https://lettermint.co/docs/platform/onboarding/compliance-verification.md) | undated | 2026-11-03 | high |
| [6] | Lettermint DNS: DKIM CNAMEs, Return-Path CNAME, DMARC aspf=r, subdomain advice | [Lettermint](https://lettermint.co/docs/platform/domains/introduction.md) | undated | 2026-11-03 | high |
| [7] | Lettermint company age, founders, scale, ISO goal | [Lettermint blog](https://lettermint.co/blog/lettermint-turns-1-and-we-are-just-getting-started) | 2026-01-02 | 2026-11-03 | medium (self-reported) |
| [8] | Lettermint DPA: Sec. 3, 5.8, 7, 8, 10, 12 (24h breach, EU-only, objection, audit) | [Lettermint](https://lettermint.co/dpa) | 2026-10-09 (version) | 2026-11-03 | high |
| [9] | Scaleway TEM pricing, SMTP, no non-EU sub-processors, auto-block | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/blob/main/pages/transactional-email/faq.mdx) | 2025-09-24 (validated) | 2026-11-03 | high |
| [10] | Scaleway TEM data privacy: fr-par only, no tracking, retention | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/blob/main/pages/transactional-email/reference-content/data-privacy.mdx) | 2026-10-19 | 2026-11-03 | high |
| [11] | Scaleway TEM limits, quotas, stricter initial limits | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/blob/main/pages/transactional-email/reference-content/tem-capabilities-and-limits.mdx) | 2025-10-29 (validated) | 2026-11-03 | high |
| [12] | Scaleway TEM DNS records | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/blob/main/pages/transactional-email/how-to/authenticate-domain.mdx) | undated | 2026-11-03 | high |
| [13] | Scaleway €0.25/1k, no minimum fees | [Scaleway](https://scaleway.com/en/pricing/managed-services) | undated (search snippet) | 2026-11-03 | medium |
| [14] | Scaleway console MFA, payment method, KYC | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/tree/main/pages/account) | undated | 2026-11-03 | medium |
| [15] | SES pricing and credit-based free tier | [AWS](https://aws.amazon.com/ses/pricing/) | undated (live) | 2026-11-03 | high |
| [16] | AWS DPA in Service Terms; SES content storage and scanning | [AWS](https://aws.amazon.com/service-terms/) | undated (live) | 2026-11-03 | high |
| [17] | SES sandbox and production-access process | [AWS docs](https://docs.aws.amazon.com/ses/latest/dg/request-production-access.html) | undated | 2026-11-03 | high |
| [18] | SES production-access denials | [AWS re:Post](https://repost.aws/questions/QUmX1zW3R_RaikO2vGCGUdpA/ses-production-access-denied-due-to-no-billing-history-circular-limitation-issue) | undated | 2026-11-03 | medium |
| [19] | SES tracking only with configuration sets (disputed) | [AWS docs](https://docs.aws.amazon.com/ses/latest/dg/faqs-metrics.html) | undated | 2026-11-03 | low (disputed) |
| [20] | SES eu-north-1 SMTP endpoint; sandbox quotas | [AWS docs](https://docs.aws.amazon.com/general/latest/gr/ses.html) | undated | 2026-11-03 | high |
| [21] | SES custom MAIL FROM records | [AWS docs](https://docs.aws.amazon.com/ses/latest/dg/mail-from.html) | undated | 2026-11-03 | high |
| [22] | SMTP2GO pricing, work email + SMS sign-up | [SMTP2GO](https://www.smtp2go.com/pricing/) | undated | 2026-11-03 | medium |
| [23] | SMTP2GO EU endpoints and Amsterdam hosting | [SMTP2GO docs](https://developers.smtp2go.com/docs/smtp-relay) | undated | 2026-11-03 | high |
| [24] | SMTP2GO click tracking per SMTP user | [SMTP2GO blog](https://www.smtp2go.com/blog/new-feature-click-tracking/) | 2026-06-15 (updated) | 2026-11-03 | medium |
| [25] | Brevo tracking opt-out Enterprise-only | [Brevo Community](https://community.brevo.com/t/no-way-to-disable-by-option-tracking-in-transactional-e-mail/201/51) | 2025-11-05 | 2026-11-03 | medium |
| [26] | Brevo suspension policy | [Brevo Help Center](https://help.brevo.com/hc/en-us/articles/360017299259) | undated | 2026-11-03 | medium |
| [27] | Postmark US-only hosting, DPA in ToS | [Postmark](https://postmarkapp.com/eu-privacy) | undated | 2026-11-03 | high |
| [28] | Postmark free 100/mo, no overage | [Postmark](https://postmarkapp.com/pricing) | undated | 2026-11-03 | high |
| [29] | Resend data stored in US regardless of region | [Resend docs](https://resend.com/docs/dashboard/domains/regions) | undated | 2026-11-03 | high |
| [30] | Resend tracking off by default | [Resend docs](https://resend.com/docs/dashboard/domains/tracking) | undated | 2026-11-03 | high |
| [31] | Resend pricing and SMTP on all plans | [Resend](https://resend.com/pricing) | undated | 2026-11-03 | high |
| [32] | Mailgun free plan 100/day, 1-day logs | [Mailgun](https://www.mailgun.com/pricing/) | undated | 2026-11-03 | high |
| [33] | Mailgun DPA, US group sub-processor | [Sinch Email / Mailgun](https://www.mailgun.com/legal/dpa/) | 2023-01-16 | 2026-11-03 | medium |
| [34] | Mailgun accounts disabled on signup | [Ghost Forum](https://forum.ghost.org/t/mailgun-permanently-disabled-account-due-to-spam-signups/54775) | undated | 2026-11-03 | medium |
| [35] | MailerSend free plan, card and approval | [MailerSend](https://www.mailersend.com/pricing) | undated | 2026-11-03 | high |
| [36] | MailerSend rejection reports | [G2](https://g2.com/products/mailersend/reviews_and_filters?page=3) | undated | 2026-11-03 | low |
| [37] | Mailjet free plan 6k/mo, 200/day | [Mailjet](https://www.mailjet.com/pricing/) | undated | 2026-11-03 | high |
| [38] | Sinch DPA: group transfers, DPF | [Sinch](https://sinch.com/legal/terms-and-conditions/other-sinch-terms-conditions/data-processing-agreement/) | 2025-10-29 | 2026-11-03 | high |
| [39] | Mailjet tracking-off SMTP headers | [Mailjet dev docs](https://dev.mailjet.com/smtp-relay/custom-headers/) | undated | 2026-11-03 | high |
| [40] | SendGrid free plan retired | [Twilio](https://www.twilio.com/en-us/changelog/sendgrid-free-plan) | 2025-05-27 | 2026-11-03 | high |
| [41] | Cloudflare Email Service pricing | [Cloudflare docs](https://developers.cloudflare.com/email-service/platform/pricing/) | undated | 2026-11-03 | high |
| [42] | Sweego pricing, SMTP on Free | [Sweego](https://www.sweego.io/pricing) | undated | 2026-11-03 | medium |
| [43] | ZeptoMail credits, EU data centre | [Zoho](https://www.zoho.com/zeptomail/pricing.html) | undated | 2026-11-03 | medium |
| [44] | Mailgun free tier cut to 100/day (2024) | [Ghost Forum](https://forum.ghost.org/t/mailgun-free-tier-limited-to-100-emails-per-day/51538) | 2024-10-09 | 2026-11-03 | medium |
| [45] | Lettermint B.V., EU-only infrastructure, ISO claim | [Lettermint](https://lettermint.co/european-email) | undated | 2026-11-03 | medium (self-declared) |
| [46] | Scaleway TEM changelog 2023–2026 | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/tree/main/changelog) | 2023–2026 | 2026-11-03 | high |
| [47] | SMTP2GO privacy policy allows non-EEA processors | [blocksentient](https://blocksentient.com/review/smtp2go/) | undated (search snippet) | 2026-11-03 | low |
| [48] | Brevo transactional accounts suspended after signup | [Brevo Community](https://community.brevo.com/t/warning-about-suspicious-charges-and-sudden-account-suspensions/3137) | undated | 2026-11-03 | medium |
| [49] | Lettermint MFA, passkeys, team MFA enforcement | [Lettermint docs](https://lettermint.co/docs/platform/teams/security) | undated | 2026-11-03 | high |
| [50] | Mailjet hosts EU data on Google Cloud (Frankfurt, Saint-Ghislain) | [Mailjet docs](https://documentation.mailjet.com/hc/en-us/articles/360042712274) | undated (search snippet) | 2026-11-03 | medium |
| [51] | Cloudflare Email Service SMTP on port 465 only | [Cloudflare docs](https://developers.cloudflare.com/email-service/api/send-emails/smtp/) | undated (search snippet) | 2026-11-03 | medium |
| [52] | Lettermint terms: User = natural person or legal entity | [Lettermint](https://lettermint.co/terms-of-service.md) | 2026-04-08 (updated) | 2026-11-03 | high |
| [53] | Lettermint account verification form and "few hours" review | [Lettermint KB](https://lettermint.co/knowledge-base/account/verified-account-required-to-send-emails-this-is-why.md) | 2025-08-27 | 2026-11-03 | high |
| [54] | Lettermint nonprofit discount needs registered nonprofit | [Lettermint](https://lettermint.co/promo-codes-and-discounts/non-profits.md) | undated | 2026-11-03 | high |
| [55] | Scaleway account types incl. Personal Project | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/blob/main/pages/account/concepts/understanding-account.mdx) | undated | 2026-11-03 | high |
| [56] | Scaleway identity check: photo ID plus face check | [Scaleway docs (GitHub)](https://github.com/scaleway/docs-content/blob/main/pages/account/setup-lifecycle/verify-identity.mdx) | undated | 2026-11-03 | high |
| [57] | Scaleway TEM Specific Conditions: no professional-only clause, no DP clauses | [Scaleway](https://www-uploads.scaleway.com/Specific_Conditions_Transactionnal_Email_Services_v042026_3f5487a93c.pdf) | 2026-04-07 | 2026-11-03 | medium |
| [58] | Scaleway DPA (2021 version, third-party mirror) | [sofus.dk mirror of Scaleway DPA](https://sofus.dk/wp-content/uploads/2024/04/Scaleway-DPA.pdf) | 2021-08-13 | 2026-11-03 | low (may be outdated) |
| [59] | Scaleway sub-processor list, general authorisation | [Scaleway via Wayback](https://web.archive.org/web/20260614195105/https://www.scaleway.com/en/subprocessorlist/) | 2025-07 (reviewed) | 2026-11-03 | medium |
| [60] | AWS DPA: region clause 12.1, breach 9.1, sub-processors 6.1, audit 10 | [AWS](https://d1.awsstatic.com/legal/aws-dpa/aws-dpa.pdf) | undated | 2026-11-03 | high |
| [61] | AWS sub-processor list and service-improvement entities | [AWS](https://aws.amazon.com/compliance/sub-processors/) | 2026-10-30 | 2026-11-03 | medium |
| [62] | SMTP2GO new-account review process | [SMTP2GO support](https://support.smtp2go.com/hc/en-gb/articles/223087427-New-Account-Limits-and-the-Review-Process) | undated (search snippet) | 2026-11-03 | medium |
| [63] | SMTP2GO privacy policy (Sand Dune Mail Ltd, NZ) | [SMTP2GO](https://www.smtp2go.com/privacy/) | 2023-07-23 | 2026-11-03 | high |
| [64] | SMTP2GO vendor information: DPA in dashboard, sub-processors | [SMTP2GO support](https://support.smtp2go.com/hc/en-gb/articles/14855846048665-Vendor-Information) | undated (search snippet) | 2026-11-03 | low |
| [65] | Sweego terms: open to natural persons; sign-up data lists SIREN/VAT | [Sweego (Mindbaz)](https://www.sweego.io/fr/wp-content/uploads/sites/4/2024/05/conditions_generales_de_vente.pdf) | 2024-02-27 | 2026-11-03 | medium |

## Staleness map

Computed with `recon_kit.py staleness`, using these windows: pricing 3 months, capability 3, compliance 6, history 12.

| Claim | Class | Re-check by |
|---|---|---|
| Scaleway TEM stricter initial limits [11] | capability | 2026-01-29 (**already stale**: page last validated 2025-10; re-read before acting) |
| Brevo tracking Enterprise-only [25] | capability | 2026-02-05 (stale; Brevo is cut either way) |
| Scaleway TEM pricing [9] | pricing | 2026-12-23 |
| Lettermint company facts [7] | history | 2027-01-02 |
| Lettermint, SES and SMTP2GO pricing; Lettermint tracking and review gate [1][3][5][15][22] | pricing / capability | 2027-02-03 |
| Scaleway TEM privacy and retention [10] | compliance | 2027-04-19 |
| Lettermint DPA clauses [8] and acceptance [52][53] | compliance / acceptance | 2027-04-09 / 2027-02-03 |
| Scaleway DPA (2021 copy) [58] | compliance | **already stale**: replace with the current DPA |
| Lettermint sub-processors and retention [2][4] | compliance | 2027-04-29 to 2027-05-03 |

The earliest re-check is Scaleway's quota page, which is already stale. Re-read Lettermint's pricing on the day story 9 starts. Refresh handles all of this.
