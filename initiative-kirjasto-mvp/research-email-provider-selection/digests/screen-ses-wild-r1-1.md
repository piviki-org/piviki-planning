# Screen: Amazon SES, Mailgun EU, wildcards (round 1, slot 1)

Accessed 2026-11-03. I used 22 tool calls. A claim marked "UNVERIFIED BELIEF" comes from prior knowledge and was not evidenced in this run.

## Amazon SES (eu-north-1 Stockholm / other EU regions)

- À la carte outbound price is "$0.10 / 1,000 emails", plus $0.12/GB of attachments. About 100 emails/month costs roughly USD 0.01/month. | https://aws.amazon.com/ses/pricing/ | AWS | undated | accessed 2026-11-03 | high | pricing
- SES now also sells bundled plans (Essentials $0.16, Pro $0.22, Enterprise $0.23 per 1,000 for 0–10M emails/month). À la carte is still listed. This plan structure is new compared to the older single-rate model. | https://aws.amazon.com/ses/pricing/ | AWS | undated | accessed 2026-11-03 | high | pricing/history
- Free tier is now credit-based: "up to $200 in AWS Free Tier credits". The free plan is "available for 6 months after account creation" and credits must be used within 12 months. The page shows no SES-specific monthly free email allowance. The old "3,000 msgs/month for 12 months" offer is not on the page. | https://aws.amazon.com/ses/pricing/ | AWS | undated | accessed 2026-11-03 | high | pricing/history
- UNVERIFIED BELIEF: the SES free tier changed from 62k/month (from EC2) to 3,000/month (Aug 2023), and AWS moved to the credit-based Free Tier around July 2025. Exact dates were not evidenced this run. | — | — | — | — | low | history
- Every new account starts in the sandbox, per region. Sandbox limits: send only to verified addresses, 200 msgs/24h, 1 msg/s. | https://docs.aws.amazon.com/ses/latest/dg/request-production-access.html | AWS | undated | accessed 2026-11-03 | high | capability
- The production-access request asks for: Marketing or Transactional, website URL, contacts, and an acknowledgement "that you have a process in place for handling bounce and complaint notifications". AWS gives an initial response "within 96 hours". The request is per region, so an eu-north-1 request is separate from other regions. | same | AWS | undated | accessed 2026-11-03 | high | capability
- Many independent reports of production-access denials with no reason given. One recurring denial cites a lack of "billing history / pattern of use of other AWS services" (users call it a catch-22). AWS staff say the SES Trust & Safety team decides and Premium Support cannot override it. | https://repost.aws/questions/QUmX1zW3R_RaikO2vGCGUdpA/ses-production-access-denied-due-to-no-billing-history-circular-limitation-issue ; https://repost.aws/questions/QUO0zYv-bwRmOHRU0DD_-Kgw/ses-production-access-request-denied-with-no-information | AWS re:Post (users) | undated (page 403 on fetch; seen via search snippets) | accessed 2026-11-03 | medium | deliverability
- Open tracking is NOT active by default. It needs a configuration set, an event destination that publishes open events, and that configuration set specified on each email. Send without a configuration set and no pixel is inserted. | https://docs.aws.amazon.com/ses/latest/dg/faqs-metrics.html | AWS | undated | accessed 2026-11-03 | high | capability
- Click tracking rewrites links only when a configuration set's event destination captures clicks. It can be disabled for the whole configuration set, or per link with `ses:no-track`. No configuration set means no rewriting. | same | AWS | undated | accessed 2026-11-03 | high | capability
- SES has an SMTP interface ("for applications that can send email via SMTP"). | https://aws.amazon.com/ses/faqs/ | AWS | undated | accessed 2026-11-03 | high | capability
- SES may pause sending for "high bounced email or complaint rates". The FAQ gives no numeric thresholds. UNVERIFIED BELIEF: review at about 5% bounce / 0.1% complaint, pause at about 10% / 0.5%. | https://aws.amazon.com/ses/faqs/ | AWS | undated | accessed 2026-11-03 | medium (threshold numbers low) | deliverability
- The data protection page recommends MFA on every account. AWS console MFA is available. It links to the AWS GDPR Center and Data Privacy FAQ, but the SES pages state no region or retention for message content. | https://docs.aws.amazon.com/ses/latest/dg/data-protection.html | AWS | undated | accessed 2026-11-03 | high (MFA) / low (residency) | compliance
- UNVERIFIED BELIEF: the AWS GDPR DPA is built into the AWS Service Terms with SCCs (no signature needed). AWS is US HQ (Seattle), and the EU contracting entity is AWS EMEA SARL (Luxembourg). Easy DKIM uses 3 CNAME records, plus optional custom MAIL FROM records (MX + SPF TXT), plus a DMARC record the customer publishes. | — | — | — | — | low | compliance/capability
- Opportunistic TLS is the default for outbound delivery. Enforced TLS is configurable. | https://docs.aws.amazon.com/ses/latest/dg/data-protection.html | AWS | undated | accessed 2026-11-03 | high | capability

## Mailgun (Sinch; EU region)

- Free plan: "$0/mo", "100 emails/day included", SMTP relay included, 1 custom domain, "1 day of log retention". Basic starts at $15/mo for 10k/mo. Foundation is $35/mo (first month free). | https://www.mailgun.com/pricing/ | Mailgun | undated | accessed 2026-11-03 | high | pricing
- The free tier was cut to 100/day and pay-as-you-go/Flex removed for new users around Oct 2024. Flex was still reachable via a trial-then-cancel workaround as of mid-2024. | https://forum.ghost.org/t/mailgun-free-tier-limited-to-100-emails-per-day/51538 | Ghost Forum (users) | 2024-10-09 | accessed 2026-11-03 | medium | history
- Secondary claim: without a card, the free plan sends only to the sandbox domain and up to 5 authorized recipients, and adding a card unlocks the custom domain. The official help page (help.mailgun.com 203068914) returned 403. | https://www.costbench.com/software/email-api/mailgun/free-plan/ (via search) | third party | undated | accessed 2026-11-03 | low | capability
- Click and open tracking are set per domain (Tracking Settings / `PUT /v3/domains/{name}/tracking/click active=false`). Per-message `o:tracking*` overrides the domain setting, and per-link `disable-tracking=true` stops URL rewriting. The docs do not state the DEFAULT. | https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/tracking-clicks | Mailgun | undated | accessed 2026-11-03 | high (controls) / unclear (default) | capability
- The DPA is part of the Sinch Email terms (automatically binding). Transfers use SCCs (EU 2021/914) plus the UK Addendum. Data is deleted within 90 days of termination. Security logs are kept at least 1 year. Sub-processors include GCP (Germany & Belgium for EU clients) and Rackspace (Germany for EU clients). Mailgun Technologies (San Antonio, TX, US) is a group sub-processor, and in-group transfers are allowed. The fetched version is the old 16/01/2023 one, and a newer DPA is linked. | https://www.mailgun.com/legal/dpa/ | Sinch Email | 2023-01-16 (old version) | accessed 2026-11-03 | medium | compliance
- Many independent reports of accounts disabled on signup or after small sends (e.g., about 164 emails), with no explanation and appeals refused. Mailgun's help center has a "Disabled Account & Domains" section that includes "Business verification is required". | https://forum.ghost.org/t/mailgun-permanently-disabled-account-due-to-spam-signups/54775 ; https://discuss.write.as/t/mailgun-immediately-disabled-account-i-was-planning-to-use-with-writefreely/17000 ; https://help.mailgun.com/hc/en-us/sections/360006038154-Disabled-Account-Domains | users / Mailgun | undated | accessed 2026-11-03 | medium | deliverability
- UNVERIFIED BELIEF: the EU region uses smtp.eu.mailgun.org and api.eu.mailgun.net, and the region is chosen per domain at creation. DNS records are SPF TXT, a DKIM TXT (pdk/k1._domainkey), an optional tracking CNAME, and MX for receiving. Console MFA is available. HQ is San Antonio, TX, with parent Sinch AB (Sweden). | — | — | — | — | low | capability/compliance

## Wildcards (brief screen)

### Sweego (FR)
- Free plan: 100 emails/day "by API & SMTP". Cheapest paid tier ("Individual") is EUR 8/mo billed annually or EUR 10/mo monthly. Tracking is listed on all plans. The footer links to a DPA and a sub-processors page. The pricing page states no hosting location (only "Made in France"). | https://www.sweego.io/pricing | Sweego | undated | accessed 2026-11-03 | high (price/SMTP) / low (residency) | pricing/compliance

### Zoho ZeptoMail
- Credit model: 1 credit = 10,000 emails, credits valid for 6 months, first credit free. Available in US, EU and IN data centers. The extracted page text had no price or SMTP mention. UNVERIFIED BELIEF: about USD 2.50 per credit, and SMTP is supported. | https://www.zoho.com/zeptomail/pricing.html | Zoho | undated | accessed 2026-11-03 | medium | pricing/compliance

### Twilio SendGrid
- The free plan was retired starting 2025-05-27, with sending paused after a 60-day grace period. New accounts get a 60-day trial at 100/day, then paid plans from about $19.95/mo (secondary source). This fails on budget and has no permanent free tier. | https://www.twilio.com/en-us/changelog/sendgrid-free-plan | Twilio | 2025-05-27 | accessed 2026-11-03 | high | pricing/history

### Cloudflare Email Service
- Outbound sending is "Not available" on Workers Free. Workers Paid includes 3,000/mo, then $0.35/1,000, and "Sending to arbitrary recipients requires the Workers Paid plan". The pricing page states no data location. | https://developers.cloudflare.com/email-service/platform/pricing/ | Cloudflare | undated | accessed 2026-11-03 | high | pricing
- SMTP submission at smtp.mx.cloudflare.net:465 (implicit TLS only, no 587 STARTTLS). The username is the literal `api_token`. Nodemailer can use `secure: true` on 465. | https://developers.cloudflare.com/email-service/api/send-emails/smtp/ (search snippet) | Cloudflare | undated | accessed 2026-11-03 | medium | capability
- UNVERIFIED BELIEF: Workers Paid costs USD 5/mo, which is about a third of the 15 EUR stack budget. US company, and EU data residency is not established.

### Not screened this run: Mailtrap, Infomaniak, Elastic Email, MailPace (out of budget).

## Gate verdict table

| Provider | (a) SMTP on free/cheapest | (b) Custom domain DKIM/SPF/DMARC | (c) Written DPA | (d) Tracking fully off, no rewriting | Notes |
|---|---|---|---|---|---|
| Amazon SES (EU) | PASS: SMTP interface; pay-per-use about $0.01/mo | PASS (likely), DNS records unverified this run | UNCLEAR: AWS DPA believed built into Service Terms, not evidenced | PASS: off unless a configuration set with open/click event destination is used; default off | Sandbox exit per region, 96h review, real denial risk |
| Mailgun (EU) | PASS: free 100/day includes SMTP | PASS (likely), but the free plan may need a card for a custom domain (low-confidence source) | PASS: DPA built into Sinch terms, SCCs, EU sub-processors in DE/BE | PASS on controls (domain toggle, per-message, per-link); DEFAULT UNCLEAR | Account-disable stories; 1-day logs on free |
| Sweego (FR) | PASS: free 100/day API & SMTP | UNCLEAR (not checked) | PASS (DPA page exists, not read) | UNCLEAR | EUR 8–10/mo paid; hosting location unverified |
| Zoho ZeptoMail | UNCLEAR (SMTP not on the page) | UNCLEAR | UNCLEAR | UNCLEAR | EU DC exists; credits expire after 6 months |
| SendGrid | FAIL (no free plan; about $19.95/mo) | n/a | n/a | n/a | Drop |
| Cloudflare Email | PASS (465 implicit TLS) but needs Workers Paid | UNCLEAR | UNCLEAR | UNCLEAR | No residency statement |

## Leads worth chasing
- AWS Service Terms / GDPR DPA page: confirm automatic incorporation, SCCs, and the AWS EMEA SARL entity. Also find where SES stores message content and suppression lists, and how long, for eu-north-1.
- Read the SES easy-DKIM and custom MAIL FROM docs to confirm the DNS record list. Check whether the eu-north-1 SMTP endpoint exists (email-smtp.eu-north-1.amazonaws.com).
- Check that the SES "Essentials/Pro/Enterprise" plans don't replace à la carte for new accounts, and check plan-change dates.
- Mailgun: read the current Sinch DPA and the EU sub-processor list. Find the default tracking state for new domains (dashboard or API GET /v3/domains/{name}/tracking). Find the free-plan credit-card requirement on an official page. Find the EU-region docs.
- Sweego: read the DPA and sub-processors pages for hosting (likely French/EU clouds). Read the tracking toggle docs.
- Read the ZeptoMail SMTP docs and EU DC (zeptomail.zoho.eu) pricing in EUR.
- Screen Infomaniak (CH), MailPace, Mailtrap, and Elastic Email.

## Looked for, not found
- SES retention period for message content or logs, and an explicit EU-residency statement (not on the FAQ or data-protection pages).
- Numeric SES bounce/complaint thresholds (not on the FAQ).
- Mailgun's default tracking state for new domains.
- An official Mailgun page on free-plan card or sandbox restrictions (403).
- Inactivity-suspension policies for either SES or Mailgun.
- Reddit originals (only mirrors and re:Post surfaced).
