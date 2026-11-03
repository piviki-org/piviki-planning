# Brief: email provider selection (Piviki)

Decision: pick the transactional email provider for story 9 (epic-librarian-catalogue), "Mail sends from the library's domain".
Type: technical · Shape: select · Preset: standard (3 subagents, 8 sources/round, depth 2) · Validation: normal · Red team: off.

## Requirements (agreed 2026-11-03)
Hard gates: (1) SMTP relay (Nodemailer SMTP adapter, AD-12); (2) own domain @piviki.fi with SPF/DKIM/DMARC manageable as DNS records (OpenTofu/Cloudflare); (3) written DPA available; (4) open & click tracking fully off, no link rewriting; (5) not a personal mailbox (Gmail/Proton).
Weighted: EU residency / EU company (high); cost at ~100 emails/mo, durable free tier, stack budget ~15 EUR/mo (high); deliverability (high); free-tier traps (medium); ops simplicity for two volunteers (medium); exit cost / repricing history (medium).

## Dimensions
D1 candidate screen · D2 compliance & data · D3 cost & free-tier traps · D4 deliverability & real-world experience.
User addition: keep a separate "considered and rejected" list, stating which gate each fails.

Candidates: Brevo, Mailjet, Scaleway TEM, Lettermint, Postmark, Resend, Amazon SES (eu-north-1), SMTP2GO, Mailgun EU, MailerSend (+ wildcards found in screen).
