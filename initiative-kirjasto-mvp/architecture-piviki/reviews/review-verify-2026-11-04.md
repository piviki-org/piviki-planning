# Verification review: 2026-11-04 update (repo split, deploy trust, Bitwarden)

Reviewer lens: is every committed decision in the changed parts backed by web research or a live check, not asserted from training data? Does each named technology still exist and fit?
Date checked: 2026-11-04. Scope: the Design Paradigm rows "Host runtime" and "Shared infrastructure", AD-17 "Build and deploy" (last sentence), "Recovery", "Infrastructure as code", AD-18 (Bitwarden holds credentials only), and the Consistency Conventions "Repositories" row.

## Verdict

The update holds up. Bitwarden's free two-user organisation still exists. The EU region offers the same features. Free accounts have two-step login, and recovery codes exist. Two owners in a free organisation is what Bitwarden's own setup guide recommends. Before this review, though, the Bitwarden decision had no research behind it: `.memlog.md` line 128 records it as a user decision, and the only earlier source is review-security's "Bitwarden's free tier, or a KeePassXC file". It is now checked. Four wording gaps remain. The biggest is that a Bitwarden "recovery code" does not recover a forgotten master password, so the spine's lockout protection is weaker than it reads.

## What was checked and how

| # | Claim | Check | Result |
|---|---|---|---|
| V1 | A free two-user Bitwarden organisation exists in 2026 | bitwarden.com/help/about-organizations, /help/password-manager-plans, /pricing/business (fetched 2026-11-04) | Confirmed. "Free organizations allow two users to securely share in up to two collections." The pricing page still lists "Free sharing with another user". |
| V2 | Collections limit | Same pages | **Two collections**, unlimited items. The free organisation has no file attachments, and Send is text only. Event logs, groups and policies are Teams/Enterprise only. Emergency access and vault health reports are Premium. |
| V3 | Both founders can be organisation owners | /help/getting-started-organizations (the free-organisation quick start), /help/user-types-access-control | Confirmed. The free-organisation quick start says "it's a good idea to add a second user with the Owner role". The roles page says to assign "at least one additional owner". No free-plan restriction on Owner or Admin. Only custom roles are Enterprise-only. |
| V4 | EU region (bitwarden.eu), with a free organisation available there | /help/server-geographies | Mostly confirmed. "Features and functions are identical across regions", and the region selector is used "to choose which Bitwarden server region to create your account or organization on". No page says "free organisation on EU" in so many words, but nothing excludes it. Regions are separate: "an account in one region can't join an organization in another", and Bitwarden cannot move accounts between regions. |
| V5 | MFA (two-step login) on free accounts | /help/password-manager-plans | Confirmed. The Free plan lists "Hardware security key, authenticator apps, or email". Duo and Yubico OTP are not listed for Free. |
| V6 | Recovery codes exist | /help/two-step-recovery-code | Confirmed, with a limit (F1). The code "disconnect[s] all two-step login methods". You need it together with the email and **master password**. It changes after use. It does not recover a forgotten master password. |
| V7 | Bitwarden holds credentials only, so it is not an AD-18 processor | Reasoning check against AD-18 and AD-17 | Consistent, provided no patron data or production dump ever goes into the vault. The free organisation has no attachments, which makes that harder to do by accident. |
| V8 | Root-owned, founder-pulled checkout of `piviki-core` provides the deploy script, Compose file and Caddy config | Reality check | Workable. The repository is public (AD-17, AD-18), so a pull needs no credential on the host. Compose variable interpolation (`${TAG}`) lets a deploy name only the image tag. This is long-standing Compose behaviour, not re-fetched here. See F4 for the ordering gap. |
| V9 | Repo split rows (Design Paradigm, Conventions, Recovery) | Consistency check within the spine | Consistent. "Recovery: rebuild from `piviki-infra`, `piviki-core` and the latest dump" now covers the runtime in `piviki-core/deploy`. No named technology is new apart from Bitwarden. OpenTofu, Caddy, Compose and GHCR were checked in earlier reviews. |

## Findings

### F1 (High): "Recovery code" does not protect against a lost master password

AD-17 says each founder "keeps their recovery code offline". The memlog says the aim is to prevent "a single founder locking out the association". A Bitwarden recovery code only turns off two-step login, and using it needs the master password. Bitwarden cannot reset a master password. Admin account recovery is Enterprise-only, and emergency access is Premium. A founder who forgets their master password loses their account for good. The other owner still has the organisation's items, so the association is not locked out. The safety comes from having two owners, not from the recovery code, and the spine should say so.

**Fix.** Reword to: "each keeps their master password and their two-step login recovery code offline, apart from their devices". Add: "a founder who loses their account is re-invited with a new account by the other owner; both founders are owners so that either alone can do this." Optionally note that emergency access needs Premium and is not used.

### F2 (Medium): Region lock-in and account creation are not stated

Accounts and organisations are bound to one region and cannot be moved. A founder who already has a bitwarden.com account cannot join an EU organisation with it. One community report says an invitation across regions quietly made a separate account.

**Fix.** In AD-17, or in the story 1 runbook step, say: "both founders' Bitwarden accounts are created on bitwarden.eu (vault.bitwarden.eu), and clients are set to the EU server before signing in".

### F3 (Medium): "Holds every secret" conflicts with the offline age key, and the two-collection limit is unplanned

AD-17 Backups says "Both founders hold the private key offline". "Infrastructure as code" says Bitwarden "holds every secret". Is the `age` private key in Bitwarden, or deliberately kept out of it? Keeping it out means a vault compromise cannot decrypt the backups. Keeping it in means a lost paper copy is not fatal. The spine has to choose. Separately, the free organisation allows only two collections, with no attachments. Two environments fit, for example `production` and `test`. A third grouping, such as "shared/provider consoles", would need the paid Teams or Families plan.

**Fix.** State whether the `age` private key is in the vault. Name the two collections, for example `production` and `test` with provider console logins in `production`, and say that secrets are stored as text fields or notes, not attachments.

### F4 (Low): Two-step method for the vault, and ordering when Compose changes

1. Email is one of Bitwarden's free two-step methods. If a founder's email account is protected by credentials kept in Bitwarden, email two-step login is circular. **Fix:** "Bitwarden two-step login uses an authenticator app or a security key, not email."
2. A deploy cannot change the Compose file. So a new image that needs a new environment variable, service or Caddy rule fails until a founder pulls. **Fix:** add "an image that needs a changed `deploy/` file is deployed only after a founder has pulled that change on the target host; the deploy check refuses a tag newer than the host checkout supports", or just the first half if no check is wanted.

### F5 (Low): The [ADOPTED] tag on Bitwarden had no recorded research

The Bitwarden facts are now confirmed (V1 to V6). Record the sources so the tag can be traced: bitwarden.com/help/about-organizations, /help/password-manager-plans, /help/server-geographies, /help/two-step-recovery-code and /help/getting-started-organizations, all checked on 2026-11-04. Put them in the research notes or the memlog entry. Free-plan terms can change. The tag rests on "two users, two collections, owner role allowed", so check those again if Bitwarden changes its plans.

## Sources

- https://bitwarden.com/help/about-organizations/
- https://bitwarden.com/help/password-manager-plans/
- https://bitwarden.com/pricing/business/
- https://bitwarden.com/help/getting-started-organizations/
- https://bitwarden.com/help/user-types-access-control/
- https://bitwarden.com/help/server-geographies/
- https://bitwarden.com/help/two-step-recovery-code/
- https://community.bitwarden.com/t/confusing-experience-adding-organization-user-solved-server-region-mismatch/90152 (community report, not official)
