# Citation check: research-email-provider-selection.md

Checked 2026-11-03 against digests screen-eu-r1-1, screen-ses-wild-r1-1, screen-us-r1-1, finalists-r2-1, gaps-r2-1. Spot-fetched 2 URLs: [4] lettermint.co/docs/platform/emails/data-retention and [45] lettermint.co/european-email. Matrix totals re-computed: 61 / 60 / 46 / 51, all correct.

## Material issues (these change a claim the decision rests on)

- [4] | "Lettermint keeps message content and recipient metadata for 28 days and then deletes it" (Exec summary point 1; D2; matrix; Rec 6 "state its 28-day retention") | (a) The source says more than the report. The digest (r1 line 43) says analytics events are kept 90 days from 8 Nov 2026, with HMAC-hashed recipients. The spot-fetch confirms this: "Analytics events: 90 days after the event. Until 8 November 2026, this period is 28 days." Raw tracking signals are kept 28 days, and IP hashes are kept after that. The report's claim that retention "fits Piviki's deletion design" and Rec 6 both leave out a 90-day tier that starts 5 days after the report date. | Fix: add "analytics events (hashed recipients) 90 days from 2026-11-08". Rec 6 should name both periods. Keep the exec summary point, but say that recipient-derived data persists 90 days in hashed form. Confidence stays medium.
- [7] | "its ISO 27001 claim contradicts its own January 2026 blog" (Strongest argument against) | (a) The text says more than the source. The blog (2026-01-02) called ISO 27001 a "2026 goal". The site now says "Certified to ISO/IEC 27001:2022" with no date (spot-fetch [45]). The certificate could have been obtained later in 2026, so the two statements may both be true. D4 phrases this correctly ("certificate was not checked"). | Fix: "its ISO 27001 claim is newer than its January 2026 blog and unverified". Drop "contradicts".
- [10] | Matrix: "EU-only, no sub-processors" (Scaleway) | (a) The text says more than the source. The source says TEM uses "no dedicated subprocessor to process the content or metadata". S3 adds a caveat: Scaleway-wide "rare, exceptional" US/CA partners exist, for example for payment and support. | Fix: "no dedicated TEM sub-processors; Scaleway-wide US/CA partners for payment/support not checked". Score 4 can stand.
- [14] (cited as [11]) | D3: "ID verification only to raise quotas" | (a) N7 says "Depending on your account and usage, Scaleway may ask you to provide an ID document". So ID can be requested outside quota raises. | Fix: "ID verification may be requested depending on account/usage; required to raise quotas".

## Wrong or missing source numbers

- [9][11] | "its docs and changelog run from 2023 to 2026" (Exec summary) | (b) The changelog evidence (2023-2026) comes from the changelog directory github.com/scaleway/docs-content/tree/main/changelog (finalists S4), which has no appendix row. [11] supports only the 2023-12-01 hourly-quota change. | Fix: add the changelog directory as a source, or cite [11] for "since 2023" only.
- [22] | D2 SMTP2GO: "A third party says processors outside the EEA are allowed" | (b) [22] is SMTP2GO's own pricing page. The claim comes from blocksentient.com/review/smtp2go (gaps line 309), which has no appendix row. | Fix: add that source as a new row (low confidence).
- [26] | D4 and Rejected (Brevo): "suspension stories for transactional-only accounts" / "reports of transactional-only accounts suspended soon after signup" | (b) [26] is Brevo's Help Center suspension policy. The user reports come from community.brevo.com/t/.../3137 and Trustpilot (r1 line 15), which have no appendix row. | Fix: add the community/Trustpilot source, or reword as "Brevo's policy allows suspension right after signup [26]".
- [9][11] | Matrix Cost: "€0, card on file" (Scaleway) | (b) "Payment method needed to order resources" is N7 = [14]. [11] covers a payment method only for raising quotas. | Fix: cite [14].
- [20] | D3: "Sandbox: 200 emails per 24h, verified recipients only" | (b) [20] supports 200/24h. "Verified recipients only" is in [17]. | Fix: cite [17][20].
- [3][6] | Matrix Ops (Lettermint): "passkeys and MFA" | (b) Neither [3] nor [6] covers MFA. The source is the pricing page [1] (L7) or docs/platform/teams/security (r1 line 47, no appendix row). | Fix: cite [1].
- [37][38][39] | Rejected (Mailjet): "it runs on Google Cloud" | (b) The source is documentation.mailjet.com/hc/en-us/articles/360042712274 (search snippet, medium; r1 line 21), which has no appendix row. | Fix: add the row.
- [41] | Rejected (Cloudflare): "SMTP on port 465 only" | (b) The source is developers.cloudflare.com/email-service/api/send-emails/smtp/ (snippet, medium), not the pricing page [41]. | Fix: add the row.
- [7] | Matrix Deliverability (Lettermint): "no independent evidence [7]" | (b) [7] is the vendor's own blog. The absence of independent reports comes from the finalists digest's "Looked for, not found" section. | Fix: drop [7] or cite the search record.

## Claims resting on UNVERIFIED BELIEF or no digest line

- [15] | D3: "SES replaced its monthly free email allowance with credits" | (c) [15] shows only the current credit model and the absence of a monthly allowance. The earlier allowance (3,000/mo) and the switch date rest on an UNVERIFIED BELIEF line (ses-wild line 87). | Fix: "SES's pricing page now shows only a credit-based free tier [15]; the earlier per-email allowance is unverified". Low confidence on the history.
- (no cite) | Matrix Ops (SES): "IAM SMTP credentials" | (c) No digest line. Background knowledge only. | Fix: mark it unverified, or drop it. The score does not change.
- (no cite) | Matrix Exit (SES): "long-running service" | (c) No digest line. Common knowledge, but uncited. | Minor. Mark it uncited.
- [6] | Matrix Ops (Lettermint): "CNAME-only DNS" | (a) N4 lists DMARC as an authentication record, and DMARC is a TXT record. Rec 3 itself includes DMARC. | Fix: "CNAMEs for DKIM/Return-Path plus a DMARC TXT".

## Minor qualifier notes (no downgrade needed)

- [10] | "Scaleway deletes content on delivery" | The source says content is deleted on delivery or hard bounce, or after 36 h of soft bounces. Spam is kept 72 h, blocked-domain mail 15 days, encrypted backups 7 days. Add "(backups 7 days)" if space allows.
- [2] | "All five sub-processors are EU companies" | This is correct. L3 also notes that transfers outside the EEA/UK are allowed via SCCs without naming any. Consider a half-sentence on that.
- [9][10][11] | Rec 7: "high on gates" for Scaleway | Gate G3 rests on a DPA that is referenced but was never read (S6, M-H). "High" for G3 is generous. Suggest "high on G1/G2/G4, medium on G3".
- [9] | D1: Scaleway has "no fixed fee" | Supported by [13]/N8 (snippet), not [9]. Add [13].
- [1] | D2: "28 days on all plans" | "All plans" comes from [1]; "not configurable" comes from [4]. Both sources hold. Cite [1][4].

All other inline citations checked out against their digest lines: [1], [2], [3], [5], [6], [8], [12], [13], [16], [17], [18], [19] (correctly labelled disputed), [21], [23], [24], [25], [27]-[36], [38]-[40], [42]-[45].
