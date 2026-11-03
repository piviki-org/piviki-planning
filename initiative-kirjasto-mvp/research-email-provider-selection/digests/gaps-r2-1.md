# Gaps round 2, part 1: email provider primary-source check

Accessed 2026-11-03. 23 tool calls. Claim format: claim | source URL | publisher | pub_date | accessed | confidence | class.
Classes: primary-doc, primary-legal, primary-pricing, primary-blog, secondary, search-snippet, model-belief (unverified).

## Amazon SES

- The AWS Service Terms (s.1.14.1) say "These Service Terms incorporate the AWS Data Processing Addendum" and apply it "when you use AWS Services to process Customer Data". So the DPA is automatically part of the terms, with no separate signature. The name is "AWS Data Processing Addendum", not a GDPR-specific one. | https://aws.amazon.com/service-terms/ | AWS | undated (live page) | accessed 2026-11-03 | high | primary-legal
- The GDPR Standard Contractual Clauses (s.1.14.3) apply only when GDPR applies and data is transferred outside the EEA to a country without an adequacy decision. | https://aws.amazon.com/service-terms/ | AWS | undated | accessed 2026-11-03 | medium (summarised by the fetch tool) | primary-legal
- SES Service Terms s.15.3: "we (or our third-party providers) may store and scan your SES Email and Your Content included in SES Email" to block spam, phishing and viruses. So message content may be stored and scanned. | https://aws.amazon.com/service-terms/ | AWS | undated | accessed 2026-11-03 | high | primary-legal
- Europe (Stockholm) eu-north-1 has an SES API endpoint and the SMTP endpoint email-smtp.eu-north-1.amazonaws.com (also email-smtp.eu-north-1.api.aws). There is no SMTP endpoint in Milan or Zurich. | https://docs.aws.amazon.com/general/latest/gr/ses.html | AWS | undated | accessed 2026-11-03 | high | primary-doc
- New-account sandbox quota: 200 messages per 24 hours and 1 per second, in each Region (adjustable). Leaving the sandbox needs a production access request. | https://docs.aws.amazon.com/general/latest/gr/ses.html | AWS | undated | accessed 2026-11-03 | high | primary-doc
- Easy DKIM: publish three CNAME records shown in the console. Format: {token}._domainkey.{domain}, pointing to {token}.dkim.amazonses.com (the hosted zone varies by Region; eu-north-1 uses dkim.amazonses.com). The default key is 2048-bit, with 1024-bit optional. | https://docs.aws.amazon.com/ses/latest/dg/creating-identities.html ; https://docs.aws.amazon.com/ses/latest/dg/send-email-authentication-dkim-easy.html | AWS | undated | accessed 2026-11-03 | high | primary-doc
- Custom MAIL FROM needs exactly one MX record, `10 feedback-smtp.{region}.amazonses.com` (for Stockholm: feedback-smtp.eu-north-1.amazonses.com), and one TXT record, `"v=spf1 include:amazonses.com ~all"`, on the MAIL FROM subdomain. That subdomain must not be used for sending or receiving mail. DMARC can then pass through SPF and/or DKIM. | https://docs.aws.amazon.com/ses/latest/dg/mail-from.html | AWS | undated | accessed 2026-11-03 | high | primary-doc
- Pricing: $0.10 per 1,000 outbound emails (a la carte), plus $0.12 per GB of attachments. New AWS customers get "up to $200 in AWS Free Tier credits"; the free plan lasts 6 months and credits must be used within 12 months of account creation. | https://aws.amazon.com/ses/pricing/ | AWS | undated | accessed 2026-11-03 | high | primary-pricing
- Region tracking domains (for example r.eu-north-1.awstrack.me) exist for open and click links. | https://docs.aws.amazon.com/general/latest/gr/ses.html | AWS | undated | accessed 2026-11-03 | high | primary-doc
- UNVERIFIED BELIEF: SES open and click tracking is off by default. It only happens when a configuration set with OPEN/CLICK event destinations is used, so mail sent with no configuration set gets no pixel and no link rewriting. The fetched FAQ page returned only its title, and this text came from the tool model's own knowledge. | https://docs.aws.amazon.com/ses/latest/dg/faqs-enable-open-click-tracking.html | AWS | undated | accessed 2026-11-03 | low | model-belief
- MFA on root/IAM: not researched (out of budget). | - | - | - | - | - | -

## SMTP2GO

- Click tracking "is enabled per SMTP user, authenticated IP address, or API key" on the dashboard's Tracking tab. A `no-track` attribute excludes individual links. Only https:// href links can be tracked, and plain-text mail is not. The page does not say the feature is on by default and does not describe the rewrite mechanism. | https://www.smtp2go.com/blog/new-feature-click-tracking/ | SMTP2GO | 2020-09-22 (updated 2026-06-15) | accessed 2026-11-03 | medium | primary-blog
- Open tracking embeds a 1x1 pixel. It is toggled under SMTP Users > [user] > Tracking & Status > Open Tracking. | https://support.smtp2go.com/hc/en-gb/articles/900002237106 (403 on fetch; seen in search snippet only) | SMTP2GO | undated | accessed 2026-11-03 | low | search-snippet
- Free plan: "up to 1,000 emails per month". The pricing page states no daily cap. No credit card is needed. Sign-up needs a work email ("we don't allow email addresses at public domains such as Gmail and Yahoo") plus SMS phone verification. | https://www.smtp2go.com/pricing/ | SMTP2GO | undated | accessed 2026-11-03 | high | primary-pricing
- SMTP hosts: mail-eu.smtp2go.com (EU/UK) and mail-eu2.smtp2go.com (EU only). Ports 25/2525/8025/587/80 for TLS and 465/8465/443 for SSL. The Amsterdam data center "sends emails and stores account information". European sign-ups go to Amsterdam automatically. Frankfurt and London are inbound servers only. | https://developers.smtp2go.com/docs/smtp-relay | SMTP2GO | undated | accessed 2026-11-03 | high | primary-doc
- A third-party review says the SMTP2GO privacy policy allows processors outside the EEA. | https://blocksentient.com/review/smtp2go/ (search snippet) | blocksentient | undated | accessed 2026-11-03 | low | secondary
- The vendor info article names the payment processors Chargebee and FastSpring and links to a separate sub-processor list, which was not retrieved. | https://support.smtp2go.com/hc/en-gb/articles/14855846048665-Vendor-Information (search snippet) | SMTP2GO | undated | accessed 2026-11-03 | low | search-snippet

## Sweego

- Free plan: EUR 0, 100 emails per day by API and SMTP, unlimited domains, 7-day log retention. Paid "Individual" plan: EUR 8/month billed annually or EUR 10/month billed monthly, for 20K or 50K emails/month. "Tracking" is listed in every plan, with no detail on whether it can be disabled. | https://www.sweego.io/pricing | Sweego | undated | accessed 2026-11-03 | medium (tiering partly inferred by the fetch tool) | primary-pricing
- The pricing page footer links a "Data Privacy Agreement" and a subprocessors list. Their contents were not retrieved; /subprocessors returned 404. | https://www.sweego.io/pricing | Sweego | undated | accessed 2026-11-03 | medium | primary-pricing

## Mailjet

- Mailjet SMTP relay supports the headers `X-Mailjet-TrackOpen: 0` and `X-Mailjet-TrackClick: 0`. Each "will override your tracking options set on your user account". So both kinds of tracking can be turned off per message, and there is also an account-level setting. The account default is not stated. | https://dev.mailjet.com/smtp-relay/custom-headers/ | Mailjet | undated | accessed 2026-11-03 | high | primary-doc
- Free plan: Mailjet's documentation says the "No logo" option exists only for Email Editor templates and is not available on the Free or Starter plans. So the logo concerns editor templates, not raw SMTP mail (inferred). Free limits per Capterra: 200 per day, 6,000 per month. | https://documentation.mailjet.com/hc/en-us/articles/360043179813 (search snippet); https://www.capterra.com/p/130105/Mailjet/pricing/ | Mailjet / Capterra | undated | accessed 2026-11-03 | low-medium | search-snippet / secondary

## MailerSend

- Free plan: 500 emails/month, "Email API and SMTP relay", 1 domain. The "Tracking & analytics" column is blank for Free, activity is kept for 1 day, and "Remove MailerSend logo" is available only on Starter and above. | https://www.mailersend.com/pricing | MailerSend | undated | accessed 2026-11-03 | medium | primary-pricing
- Tracking is toggled per domain (Domains > Manage). The settings include track_opens, track_clicks and track_content. The default state for new domains was not found. | https://mailersend.com/help/domain-tracking-options (search snippet) | MailerSend | undated | accessed 2026-11-03 | low | search-snippet

## Updated gate verdicts

| Provider | SMTP | Custom domain SPF/DKIM/DMARC | Written DPA | Open+click fully off, no rewrite | EU processing | Free fit for ~100/mo |
|---|---|---|---|---|---|---|
| Amazon SES (eu-north-1) | PASS (email-smtp.eu-north-1) | PASS (3 DKIM CNAMEs + MX/SPF MAIL FROM) | PASS (DPA incorporated in Service Terms) | LIKELY PASS (off unless a configuration set enables it; unverified) | PASS for region choice; content may be stored and scanned (s.15.3) | Credits for 6 months, then about $0.01/mo; sandbox exit needed |
| SMTP2GO (EU account) | PASS | Not verified this round | Not verified | LIKELY PASS (per-user toggles; default unconfirmed) | PARTIAL (Amsterdam; sub-processors unknown) | PASS (1,000/mo), but needs a work email and phone |
| Sweego | PASS (free includes SMTP) | Not verified | Exists (link), not read | UNKNOWN | Assumed FR, unverified | PASS (100/day) |
| Mailjet | PASS | Not verified | Not verified | PASS (headers set to 0 override the account setting) | Not verified this round | PASS (6,000/mo); logo only on editor templates |
| MailerSend | PASS (free includes SMTP) | Not verified | Not verified | UNKNOWN (per-domain toggles; default unknown) | Not verified | PASS (500/mo); logo removal only on paid plans |

## Leads

- SES: https://docs.aws.amazon.com/ses/latest/dg/configure-custom-open-click-domains.html and the configuration-set event docs, to confirm tracking is off by default.
- SMTP2GO: the "Verify Data Center and Server Location" support article; the Sub-Processor List linked from the Vendor Information article; the support.smtp2go.com DPA (Zendesk returns 403 to the fetcher, so a browser is needed).
- Sweego: the footer links "Data Privacy Agreement" and "subprocessors" on sweego.io; find the real URLs in a browser.
- MailerSend: the API reference for PUT /v1/domains/{id}/settings (track_opens, track_clicks defaults).
- Mailjet: the account-level "tracking options" default in the documentation.

## Looked for, not found

- An authoritative SES statement that tracking is off by default (the FAQ page returned no body).
- AWS MFA documentation (not attempted; out of budget).
- Whether SMTP2GO tracking is on by default, how click tracking rewrites links, any daily cap on the free plan, DNS record list, and sub-processor list.
- Sweego hosting provider (OVH, Scaleway, AWS), sub-processors, DPA text, and tracking default or disable option.
- Mailjet account tracking default.
- MailerSend tracking default, and where the free-plan logo appears.
