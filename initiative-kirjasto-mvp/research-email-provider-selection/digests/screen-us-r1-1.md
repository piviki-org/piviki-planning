# Screen r1-1: Postmark, Resend, MailerSend, SMTP2GO

Run date: 2026-11-03. Budget: 23 tool calls. Some sources were read only through search-engine summaries, not fetched in full. Those lines are marked "(via search snippet)" and given lower confidence. Any belief not backed by a source from this run is labelled UNVERIFIED.

## Postmark (ActiveCampaign)

- Free tier is $0/mo for 100 emails/month. "No overages allowed in this plan." Free accounts don't expire. | https://postmarkapp.com/pricing | Postmark | undated | accessed 2026-11-03 | high | pricing
- Cheapest paid tier is Basic at $15/mo for 10,000 emails, with overage at $1.80 per 1,000. | https://postmarkapp.com/pricing | Postmark | undated | accessed 2026-11-03 | high | pricing
- The pricing page lists SMTP under "Send and Receive", but its per-plan checkmarks did not render. SMTP on the free tier is therefore not confirmed from this page. | https://postmarkapp.com/pricing | Postmark | undated | accessed 2026-11-03 | low | capability
- Default retention is 45 days of full message content. Custom retention (7–365 days) is an add-on from $5/mo and requires Pro or higher. Statistics are kept indefinitely. | https://postmarkapp.com/pricing | Postmark | undated | accessed 2026-11-03 | high | compliance
- The pricing page does not mention an account approval step. UNVERIFIED belief: Postmark manually reviews accounts before lifting the restriction to sending only to your own domain. | https://postmarkapp.com/pricing | Postmark | undated | accessed 2026-11-03 | low | capability
- The DPA with the new SCCs has been incorporated into the Terms of Service since 2021-09-27, so no signature is needed. | https://postmarkapp.com/eu-privacy | Postmark | undated | accessed 2026-11-03 | high | compliance
- Data is hosted in the US, at the Deft data centre near Chicago and on AWS. Quote: "We currently don't have plans to add servers in the EU." There is no EU region. | https://postmarkapp.com/eu-privacy | Postmark | undated | accessed 2026-11-03 | high | compliance
- The transfer mechanism is SCCs. The EU privacy page does not mention the DPF. Listed sub-processors are AWS and Deft, with Zendesk mentioned separately. HQ is Chicago, IL, and the privacy contact is at activecampaign.com. | https://postmarkapp.com/eu-privacy | Postmark | undated | accessed 2026-11-03 | high | compliance
- Open tracking defaults to off (TrackOpens=false). It can be enabled per message, per server or per message stream. If the server-level setting is on, it cannot be turned off for an individual message. | https://postmarkapp.com/developer/user-guide/tracking-opens | Postmark | undated | accessed 2026-11-03 | medium (via search snippet) | capability
- Link tracking reportedly defaults to "None" ("No links will be replaced or tracked"). This comes from a third-party summary; Postmark's own doc was not fetched. | https://www.gblock.app/articles/is-postmark-tracking-your-email-block-2026 | gblock.app | 2026 | accessed 2026-11-03 | low | capability
- DKIM is a TXT record using the hostname and value from the DNS Settings page, with 1024-bit keys. The custom Return-Path (CNAME) and DMARC are covered in separate guides. Exact record names were not captured. | https://postmarkapp.com/support/article/1091-how-do-i-set-up-dkim-for-postmark | Postmark | undated | accessed 2026-11-03 | high | capability

## Resend

- Free tier: 3,000 emails/month, 100/day, 3 custom domains. | https://resend.com/pricing | Resend | undated | accessed 2026-11-03 | high | pricing
- Pro is $20/mo for 50,000 emails, 10 domains and no daily limit, with overage at $0.90 per 1,000. | https://resend.com/pricing | Resend | undated | accessed 2026-11-03 | high | pricing
- SMTP is available on every plan, including Free. | https://resend.com/pricing | Resend | undated | accessed 2026-11-03 | high | capability
- Data retention is 30 days on Free, Pro and Scale. | https://resend.com/pricing | Resend | undated | accessed 2026-11-03 | high | compliance
- Sending regions are us-east-1, eu-west-1 (Ireland), sa-east-1 and ap-northeast-1. The region controls sending only. Quote: "All account data, including email metadata, logs, and API records, is stored in the United States regardless of the sending region you select." | https://resend.com/docs/dashboard/domains/regions | Resend | undated | accessed 2026-11-03 | high | compliance
- The DPA is attached to and incorporated into the ToS, and an executed copy is available in the dashboard. Transfers use EU SCCs (2021/914) and the UK Addendum, and Resend is DPF-certified. Data is deleted within 90 days of termination. Sub-processors are listed at /legal/subprocessors with 14 days' notice of changes. | https://resend.com/legal/dpa | Resend | undated | accessed 2026-11-03 | high | compliance
- The entity is Plus Five Five, Inc., San Francisco, CA. Primary processing takes place in the US. | https://resend.com/legal/dpa | Resend | undated | accessed 2026-11-03 | high | compliance
- Quote: "Open and click tracking is disabled by default for all domains." Tracking is set per domain (dashboard, or the API's open_tracking/click_tracking booleans). When click tracking is on, links are rewritten through a tracking subdomain. The doc does not say whether the setting applies to SMTP. | https://resend.com/docs/dashboard/domains/tracking | Resend | undated | accessed 2026-11-03 | high | capability
- Domain verification is required, and sending subdomains are recommended. A custom Return-Path subdomain is supported. The intro page does not give exact DNS record names. UNVERIFIED belief: a DKIM TXT record at resend._domainkey plus an MX and an SPF TXT record on the "send" subdomain. | https://resend.com/docs/dashboard/domains/introduction | Resend | undated | accessed 2026-11-03 | medium | capability
- Trustpilot reviews describe sudden "temporarily suspended" notices, including one paid account disabled for about 3 weeks, and slow support. These reports are anecdotal. | https://www.trustpilot.com/review/resend.com?page=1 | Trustpilot | undated | accessed 2026-11-03 | low (via search snippet) | deliverability

## MailerSend (MailerLite group)

- Free tier: 500 emails/month, 100 API requests/day, 1 sending domain, 1 template, 1 day of activity retention. A credit card and account approval are required. | https://www.mailersend.com/pricing | MailerSend | undated | accessed 2026-11-03 | high | pricing
- Hobby is $7/mo ($5.60/mo billed yearly) for 5,000 emails with overage at $1.50 per 1,000, 1 domain and 1-day retention. Every plan requires account approval. | https://www.mailersend.com/pricing | MailerSend | undated | accessed 2026-11-03 | high | pricing
- "Email API and SMTP relay" is listed as included on Free. The page does not give a separate SMTP quota. | https://www.mailersend.com/pricing | MailerSend | undated | accessed 2026-11-03 | medium | capability
- Pricing history: the free tier was historically larger. UNVERIFIED belief: 12,000/month at launch, later 3,000. A competitor claimed the free plan was killed on 2025-10-21. AgentDeals logged that change and then retracted it, citing archive captures from 2026-08 and 2026-09 that show "Free 500 emails/month". | https://agentdeals.dev/vendor/mailersend | AgentDeals | 2026 | accessed 2026-11-03 | low (via search snippet) | history
- The competitor's claim that the free plan was killed comes from Maileroo, a rival, so it is weak evidence. | https://maileroo.com/blog/mailersend-kills-its-free-plan-why-maileroo-beats-their-7-tier/ | Maileroo | 2025 | accessed 2026-11-03 | low | history
- The DPA is part of the Terms of Use, so no signature is needed. Sub-processors are in an annex of the DPA. | https://mailersend.com/legal/how-mailersend-stays-gdpr-compliant | MailerSend | undated | accessed 2026-11-03 | medium (via search snippet) | compliance
- Services are hosted in an ISO 27001-certified data centre in the EU. MailerSend, Inc. participates in the EU-US DPF, Swiss DPF and UK Extension. | https://www.mailersend.com/legal/security-statement | MailerSend | undated | accessed 2026-11-03 | medium (via search snippet) | compliance
- Tracking is toggled per domain under Domains > Manage, with separate options for opens, link content and unsubscribes. The default state was not confirmed. | https://mailersend.com/help/domain-tracking-options | MailerSend | undated | accessed 2026-11-03 | low (via search snippet) | capability
- Anecdotal reports describe opaque approval rejections (including for "project not fully implemented") and suspensions before any email was sent. One account was put on hold at a 0.4% spam rate. | https://g2.com/products/mailersend/reviews_and_filters?page=3 ; https://uk.trustpilot.com/review/mailersend.com?page=5 | G2 / Trustpilot | undated | accessed 2026-11-03 | low | deliverability

## SMTP2GO

- Free tier: 1,000 emails/month, usable indefinitely. Live chat and phone support are limited to the first 14 days. Sign-up requires a work email (Gmail, Yahoo and similar are rejected). | https://www.smtp2go.com/pricing/ | SMTP2GO | undated | accessed 2026-11-03 | high | pricing
- Free tier caps from the FAQ: 200 emails/day, or 25/hour if no sender domain is verified. Up to 5 verified senders and 5 days of sending history. | https://www.smtp2go.com/pricing/ (FAQ) | SMTP2GO | undated | accessed 2026-11-03 | medium (via search snippet) | pricing
- Starter is $10/mo ($100/yr) for 10,000 emails with overage at $1 per 1,000 and 30 days of reporting. | https://www.smtp2go.com/pricing/ | SMTP2GO | undated | accessed 2026-11-03 | high | pricing
- HQ is Christchurch, New Zealand, with a US entity in Keller, TX. | https://www.smtp2go.com/about | SMTP2GO | undated | accessed 2026-11-03 | medium (via search snippet) | compliance
- The DPA is self-serve in the account settings ("Review Agreement"). Account data is stored in a data centre chosen by account location, and EU accounts show "Hosted in the EU". EU-only SMTP endpoint: mail-eu2.smtp2go.com. | https://support.smtp2go.com/hc/en-gb/articles/12974008254873 ; https://support.smtp2go.com/hc/en-gb/articles/4741818447129 | SMTP2GO | undated | accessed 2026-11-03 | medium (via search snippet; vendor page returned 403) | compliance
- The privacy policy allows transfers outside the EEA "with safeguards". The specific mechanism was not captured. | https://smtp2go.com/privacy | SMTP2GO | undated | accessed 2026-11-03 | low | compliance
- Open and click tracking are separate on/off toggles per SMTP user, IP or API key. Single links can be excluded with a data-no-track attribute. The default state was not confirmed. | https://support.smtp2go.com/hc/en-gb/articles/900002237106 | SMTP2GO | undated | accessed 2026-11-03 | low (via search snippet) | capability
- SMTP relay is the core product, available on all tiers. | https://developers.smtp2go.com/docs/smtp-relay | SMTP2GO | undated | accessed 2026-11-03 | medium | capability

## Gate verdicts

| Gate | Postmark | Resend | MailerSend | SMTP2GO |
|---|---|---|---|---|
| (a) SMTP on free/cheapest tier | unclear: listed, but pricing checkmarks unreadable. The 100/mo free tier also leaves no headroom for about 100/mo of mail. | pass: "all plans" | pass: listed on Free | pass: core product |
| (b) Custom domain DKIM/SPF/DMARC | pass: DKIM TXT plus custom Return-Path CNAME. Exact names not captured. | pass: domain verification required, custom Return-Path. Exact records not captured. | unclear: not researched this run. UNVERIFIED belief: pass. | unclear: not researched this run. 25/hr cap without a verified domain implies domain verification exists. |
| (c) Written DPA | pass: in ToS since 2021 | pass: incorporated into ToS, signed copy in dashboard | pass: part of Terms of Use (snippet) | pass: self-serve in dashboard (snippet) |
| (d) Opens and clicks fully off | pass (likely): opens default off, links default "None" (third-party) | pass: both off by default, per domain | unclear: per-domain toggle exists, default unknown | unclear: per-user toggles exist, default unknown |
| EU residency | fail: US only, no EU plans | fail: EU sending region, but all logs and metadata stored in the US | pass (snippet): EU data centre | pass (snippet): EU-hosted accounts and EU-only endpoint |
| Volume fit (about 100/mo, free) | marginal: 100/mo hard cap, no overage | pass | pass, but needs a card and approval | pass |

## Leads worth chasing
- Postmark: read the per-plan SMTP row on the pricing page in a browser. Fetch the link-tracking doc to confirm "None" is the default. Check whether ActiveCampaign's DPA or sub-processor list adds a DPF claim.
- Resend: find out whether the regions page means email bodies are stored in the US, or only metadata. Read /legal/subprocessors. Check whether SMTP sends follow the per-domain tracking settings.
- MailerSend: fetch the full DPA at https://mailersend.com/legal/data-processing-addendum to get the hosting provider and region (believed to be EU) and the sub-processor annex. Confirm the tracking default and the exact DNS records. Note that 1-day activity retention on Free is a GDPR plus.
- SMTP2GO: fetch the vendor-information article (it returned 403) and the privacy policy for the transfer mechanism and sub-processors. Confirm the tracking defaults (UNVERIFIED belief: both on by default on new accounts). Check whether a nonprofit's own domain email address is accepted at sign-up.
- Not checked for any provider: MFA on the console and inactivity suspension policies.

## Looked for, not found
- MFA documentation for all four providers (none found within budget).
- Hacker News or Reddit threads on Resend suspensions (only Trustpilot turned up). No Reddit hits for MailerSend approval problems.
- A DPF statement on Postmark's EU privacy page.
- An exact DNS record list for Postmark or Resend (the intro and DKIM pages only point to sub-guides).
- Postmark and Resend pricing-change history within the last 3 years (not searched within budget).
- An official statement of SMTP2GO's transfer mechanism.
