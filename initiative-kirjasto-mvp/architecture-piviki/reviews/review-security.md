---
type: architecture-review
lens: security and GDPR
reviews: _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md
against: _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md
date: '2026-11-02'
---

# Security and GDPR review: Piviki architecture spine

This is an engineering review, not legal advice. Legal points are listed as questions in section 6.

## 1. Verdict

**Approve with changes. The shape is right, but do not deploy core to a public address until S1 and S2 are in the spine.**

Modular monolith, one host, Cloudflare in front and minimal data are all the right choices for this team and budget. AD-14 (data rules) and AD-18 (processor list) are better than most projects this size have. No finding below calls for more infrastructure or a bigger budget. Every fix uses free features or a few hours of work.

The main weakness is that several ADs state an outcome that the chosen tools do not deliver by default:

- "Denied on the generated API" (AD-5) is not what Payload does by default.
- "Scoped key" (AD-9) is not a Payload feature.
- "State and PKCE, email verified" (AD-8) is not what `payload-oauth2` does.
- Backups have no retention rule at all (AD-16, AD-17).

Timing matters because every epic deploys to production at a non-public address (decision of 2026-11-01). A hostname behind Cloudflare is still reachable from the internet, so "before launch" is too late for S1, S2 and S7.

### Threat model used

| Who | What they want | Why it matters here |
|---|---|---|
| Internet bots | Known Next.js or Payload flaws, password stuffing | The most likely attacker by far |
| A logged-in patron | Other patrons' data through generated endpoints | Sign-up is open to anyone at launch |
| Someone hostile to the community | The patron list, or a librarian's home address | Low likelihood, but the harm is serious and physical |
| Stolen laptop or phished founder | Provider consoles, the scan key, the admin | Two people hold everything |

### How the evidence was checked

- **[source]**: read in the published npm packages `payload@3.92.1` and `payload-oauth2@1.0.21`. The code was read, not run.
- **[docs]**: from vendor documentation (section 8).
- **[judgment]**: my reasoning. Confirm these in the first relevant story.

No Piviki code exists yet, so all of this reviews the design.

## 2. Findings at a glance

| ID | Finding | Severity | Fix needed by |
|---|---|---|---|
| S1 | Generated API is open to any logged-in patron by default, and `first-register` ignores access rules | High | Before first deploy |
| S2 | Admin shares a host, cookie and allowed origins with the patron API, protected by password only | High | Before first deploy |
| S3 | Google linking allows account pre-hijacking, and the plugin does not do what AD-8 requires | High | First Google story |
| S4 | Backups have no retention, and a restore brings deleted patrons back | High | Before launch (bucket settings at creation) |
| S5 | The scan key has no scope mechanism and is not limited to drafts | Medium | Before first scan import |
| S6 | Automatic updates do not cover the software most likely to be attacked | High | Before first deploy, then ongoing |
| S7 | Origin lockdown has holes, and the host holds a zone-wide DNS token | Medium | Before first deploy |
| S8 | Secret handling is one sentence; backup keys and CI access are undefined | Medium | Before first deploy |
| S9 | Two routes can leak the pickup location or chat invitation publicly | High (harm) | Before first scan import |
| S10 | Session and CSRF design is sound but underspecified | Medium | First auth story |
| S11 | Logging rule is right but misses IP addresses, retention and several real leak paths | Medium | Before launch |
| S12 | Breach readiness is deferred and has no detection or procedure | Medium | Before launch |
| S13 | Cloudflare decrypting traffic: accept, with conditions | Medium (mostly legal) | Before launch |
| S14 | Retention and deletion do not cover every record; processor list is incomplete | Medium | Accounts story, and before launch |
| S15 | Rate limiting on the free plan is weak, and no browser security headers are specified | Low | Patron-site story |

## 3. Findings

### S1. The generated API is not denied by default [High]

AD-5 says generated REST and GraphQL are denied to patrons. Payload's defaults do the opposite:

- **Any logged-in user passes the default access check.** `defaultAccess` is `Boolean(user)`, and it is the default for create, read, update and delete on any collection without its own rule. A patron is a user. One collection that forgets its access block is readable and writable by every patron, including new ones such as `outbox`. [source]
- **`first-register` skips access control.** Every auth collection gets `POST /api/{collection}/first-register`. If the collection is empty, it creates a user with `overrideAccess: true` and logs them in. [source] Consequences:
  - Until the first librarian exists, anyone who finds the host can make themselves an admin.
  - `patrons` stays empty until `epic-launch`, so this also creates a patron outside the AD-8 gate, with whatever field values the caller supplies.
- **Other generated endpoints exist that AD-5 does not mention:** `init`, `access`, `unlock`, `verify/:id`, `refresh-token`, `:id/api-key/reveal`. [source]
- **The planned test does not cover this.** The Testing convention covers "every patron endpoint", which reads as `/api/v1`.

**Fix (about half a day):**

1. Add a path allowlist in Caddy as the first layer. On the patron host, serve only:
   - `/api/v1/*`
   - `/api/patrons/{login,logout,me,forgot-password,reset-password,refresh-token}`
   - `/api/patrons/verify/*` (see S3)
   - the Google callback paths

   Everything else returns 404, including `first-register`, GraphQL and `/admin`. This one rule also reduces exposure to future Payload and Next.js flaws.
2. Create the first librarian from a seed script over SSH, before the host has any public DNS record. Add a deploy check that `first-register` returns 404.
3. Build every collection from a helper whose default access is deny. Add one test that loops over all of `payload.config.collections`, plus internal ones, and asserts a patron session gets 403 on all operations.
4. Set `graphQL.disable: true`. Nothing in the spine uses it.
5. Add field-level access for the phone number, so `me` and `/api/v1` cannot return it to the patron unless that is decided on purpose. AD-14 says only librarians can read or set it. **Check this against access requests:** a patron asking for their data must still receive it, for example through a librarian.

### S2. Admin exposure on the same host [High]

`api.piviki.fi/admin` holds the whole patron list, and it sits behind a password alone on a public URL. Three things make that worse than it first looks:

- **The cookie is shared.** Payload uses one cookie name per install (`{cookiePrefix}-token`), not one per collection. [source] A librarian logged in on `api.piviki.fi` has that cookie sent by the patron site's `credentials: include` calls.
- **The allowed origins are shared.** `csrf` is one global list. `https://piviki.fi` is on it, so a request from the patron site carrying a librarian's cookie is accepted as that librarian. [source] One XSS or compromised npm package on the patron site, opened by a librarian in the same browser, can read the patron list. AD-7 prevents a *patron* session reaching the admin, but not this.
- **Librarian login has no second factor**, and AD-7 chooses email and password only. Payload's default minimum password length is 3. [source]

**Fix (about a day, free):**

1. Serve the librarian tool and generated API on `admin.piviki.fi`, and only the S1 allowlist on `api.piviki.fi`. Same container, with Caddy routing by host. The cookie stays host-only, so a librarian's cookie is never sent to `api.piviki.fi`.
2. Put Cloudflare Access on all of `admin.piviki.fi`, with an email one-time PIN policy that names the two or three librarians. This adds a second factor for free, and it hides the login form and the Next.js admin from bots. Free up to 50 users; Cloudflare asks for a card on file. [docs]
3. Give `piviki-scan` an Access service token for the same host, under a Service Auth policy. [docs]
4. On the admin host, have Caddy reject any request whose `Origin` is present and is not `https://admin.piviki.fi`. This is required because the `csrf` list is global and `piviki.fi` and `admin.piviki.fi` are same-site, so SameSite does not help. It is what actually closes the path above.
5. Set `minLength` to 12 or more for librarians, and at least 10 for patrons.

**Spike first:** run Payload on two hostnames with one `serverURL` and check that the admin, cookies and reset links behave. This is a half-day check in the first story. If it fails, the smaller fallback is Access on the `/admin` path **and** `/api/librarians`, **and** the collections the admin uses, plus a rule that librarians use a separate browser profile for the admin. That fallback is weaker because path rules are easy to miss.

### S3. Google login [High]

**Pre-hijacking gap in AD-8.** AD-8 checks that *Google* verified the email, but not that the *local* account did. The attack:

1. An attacker signs up with `victim@gmail.com` and a password. Nothing verifies the address.
2. The victim later uses "Sign in with Google", and the accounts are linked.
3. The attacker's password still works, so they see the victim's account from then on.

Separately, anyone can make the library email a stranger, which itself suggests that person has a link to this community.

**`payload-oauth2` 1.0.21 does not deliver AD-8.** [source] The Deferred table keeps this choice open, so it is cheap to settle now:

| AD-8 needs | Plugin behaviour |
|---|---|
| State | Passes on any `state` from the query string; the callback never checks it, so login CSRF is open |
| PKCE | Available, but off unless `pkceEnabled` is set |
| Link only if email is verified | No `email_verified` check |
| Gate and `signupOpen` | `onUserNotFoundBehavior` defaults to `create`, using `overrideAccess` by default, outside the `/api/v1` patron creation AD-5 requires |
| Minimal data | Writes whatever `getUserInfo` returns onto the user at every login |

A nonce is sent but is not checked in the callback either.

**Fix:**

1. Replace the plugin with two small core endpoints using `openid-client`, the spine's stated fallback. This avoids a dependency that handles login for a site this sensitive, and it means less code to check. Keep the plugin only if a story proves all five rows above can be met through its hooks.
2. Turn on `auth.verify` for `patrons`, with email verification before any reservation. Add `verify/:id` to AD-5's allowed operations.
3. Identify Google users by `sub`, not by email. Add `googleSub` to AD-14's list of patron fields, since the notice must cover it.
4. Link to an existing patron only if both sides have a verified email. If the local account is unverified, replace its password with a random one and end its sessions, so the Google owner takes the account.
5. Request only `openid email`. Bind `state` and the PKCE verifier to a short-lived, HTTP-only cookie. Use an exact redirect URI. Reject Google sign-in when `signupOpen` is false, and apply the same check to `/api/patrons/login`, which does not go through `/api/v1`.

### S4. Backups, retention and erasure [High]

AD-14 and R19 promise deletion and stated retention, yet AD-16 and AD-17 say nothing about how long dumps last. As written, every deleted patron stays in R2 forever, encrypted but recoverable by whoever holds the key. A restore also brings back deleted patrons as live accounts, who then get reminder emails. Key custody is also undefined.

**Fix (mostly settings, plus about half a day):**

1. **Retention.** Set an R2 lifecycle rule that deletes dumps after a fixed period; I suggest 30 days. State that period in the privacy notice, and manage the rule in OpenTofu. See question Q4.
2. **Erasure ledger.** On deletion, `modules/accounts` writes one object to the private bucket: `erasures/{patronId}` with the date and nothing else. A restore runbook re-deletes every listed id before the app starts. Expire these objects a little after the dump window. The ledger must live outside Postgres, or a restore would rewind it. Retention needs no ledger, because the job is safe to re-run and catches up on start.
3. **Encryption with a public key.** Use `age` or similar, so the host holds only the public key and a compromised server cannot read old backups. Both founders hold the private key offline. Test decryption with each founder's copy. Losing it means losing the backups. [judgment]
4. **Do not run the dump inside core.** The Structural Seed shows `core --> backup`, and AD-13 pushes all scheduled work there. That gives internet-facing code the backup credentials and `pg_dump` access. Use a small `backup` Compose container with its own read-only database role and its own R2 token. Record this as a named exception to AD-13; backups should also still run when core is unhealthy.
5. **Protection against deletion.** Add an R2 bucket lock of about 7 days, so an attacker with the token cannot erase recent backups. Locks override lifecycle rules, so keep the lock shorter than the retention period. [docs]
6. **EU location.** Create the private bucket with the EU jurisdiction. This cannot be changed later, so do it at creation. [docs] Confirm in the first infra story that it has no cost or feature impact you care about.
7. **Know the other copies.** The OVH VPS-1 plan includes a daily backup of the previous 24 hours (per `.memlog.md`). That is a further unencrypted copy at a named processor, so include it in the data inventory, or disable it if R2 is the plan.
8. **Failure alert.** A dump that silently stops is the usual failure. Have the backup container ping a free heartbeat service such as healthchecks.io, which receives no personal data. Repeat the restore test yearly, not only once before `epic-launch` closes.

### S5. Scan key scope [Medium]

AD-9 says "an API key scoped to `titles`, `copies` and `media`". Payload API keys have no scopes. A key authenticates *as a user document*, and that user's access rules apply. [source] This causes four gaps:

- **No home for the key.** AD-7 has only `patrons` and `librarians`. On a librarian record, the script gets librarian rights, which breaks AD-9's "never gets a human permission".
- **Draft is the caller's choice.** `?draft=true` is a request parameter. With write access, a scan client can publish directly and skip the review R13 requires.
- **Re-runs can overwrite reviewed work.** "A re-run updates rather than duplicates" also matches titles a librarian has corrected and published.
- **Images are public before review.** The media bucket is public, and uploads land there immediately (see S9).

**Fix (about half a day):**

1. Add a third auth collection, `scan-clients`, with `useAPIKey`, `disableLocalStrategy` and no admin access. Update AD-7.
2. In access rules and a `beforeChange` hook, for `scan-clients`: force `_status: draft`, allow update only on records that have a `scanId` and have never been published, and deny delete. Allow read only on the catalogue collections.
3. Add tests that the key cannot publish, overwrite a published title, or read `patrons`. The S1 loop test covers the last case.
4. Keep the key in a founder's password manager rather than a repo `.env`. Delete or rotate it after the bulk import, and mint a new one only when needed. Rotating `PAYLOAD_SECRET` invalidates it, as keys are indexed with an HMAC of that secret. [source]

### S6. Patching is the real exposure [High]

AD-17 says "automatic security updates are on". That covers the OS packages. It does not cover the pinned Node, Next.js, React, Payload, Postgres and Caddy versions in the Stack table, which is where internet-reachable flaws appear; Next.js and React server components have had serious ones in recent years. With a fixed image and a part-time team, an image can go months without an update. This is the likeliest way this system gets breached.

**Fix (an hour to set up, plus a monthly slot):**

1. Turn on free GitHub Dependabot alerts and security updates in all four repositories, and subscribe both founders to advisories for Next.js and Payload.
2. Agree a rule: a critical flaw reachable from the internet is patched within 72 hours, or core is taken offline; everything else is handled monthly. Name who does it each month. A library can be offline for a weekend without real harm.
3. Rebuild images monthly from updated base images, even with no code changes.
4. Limit the damage of a compromise: core runs as non-root with a read-only filesystem and dropped capabilities, no Docker socket, and a Postgres role that is not superuser. All of this is a few lines of Compose.

### S7. Origin lockdown [Medium]

"The firewall accepts 80 and 443 only from Cloudflare" has three gaps:

- **Docker bypasses `ufw`.** Published container ports go through Docker's own iptables rules, so a `ufw` rule looks correct while the port stays open to everyone. [judgment; well-known behaviour, test it]
- **An IP allowlist admits all of Cloudflare, not just your zone.** Other Cloudflare customers can send traffic through Cloudflare's IPs, which would skip your Access policies and rate limits. Global Authenticated Origin Pulls has the same limit. Only a zone-level certificate solves this. [docs]
- **The DNS challenge puts a Cloudflare token on the host.** A compromised VPS could then change DNS for `piviki.fi`, including mail records, unless the token is tightly limited. Stock Caddy also lacks the DNS module, so this adds a custom image.

**Fix, recommended: replace all of this with Cloudflare Tunnel.** It is free and needs one small `cloudflared` container. It removes inbound 80 and 443 altogether, the IP list, the certificate and the DNS token. Caddy stays only for host and path routing on the private network. The host's only open port is SSH by key, and the tunnel token authorises only that tunnel. The trade-off is greater dependence on Cloudflare, which you already have.

**If you keep AD-17 as written:** use a Cloudflare Origin CA certificate, which needs no token on the host, plus zone-level Authenticated Origin Pulls, plus rules in OVH's network firewall or the `DOCKER-USER` chain. Either way, add a check that runs from outside: `nmap` the IP before launch, and confirm it reaches neither Postgres nor Caddy directly.

### S8. Secret handling [Medium]

"Secrets stay out of repositories" is right but incomplete. The repositories are public and MIT licensed, which raises the stakes.

**Fix:**

1. **Inventory in `piviki-infra`, names only:** `PAYLOAD_SECRET`, database password, SMTP, two R2 tokens, Google client secret, scan key, Access service token, tunnel token, registry token, OpenTofu state passphrase, backup private key. Against each, list where it lives, who holds it and how to rotate it.
2. **On the host:** a root-owned `chmod 600` env file per service, so each container only gets its own values. Postgres does not need `PAYLOAD_SECRET`, and Caddy needs almost nothing.
3. **Shared store:** a free two-user password manager organisation (Bitwarden's free tier, or a KeePassXC file). Without it, a founder being unavailable can lock out the association, and that is also a breach-response failure.
4. **MFA on every console:** GitHub organisation (enforce it), Cloudflare, OVH, Google Cloud, email provider and the domain registrar. Use each founder's own login, not a shared one, and a library-owned role address as the account contact. A Cloudflare or GitHub takeover beats everything else here.
5. **CI, which is effectively root on the host:**
   - Check that `next build` does not bake secrets into image layers, since images may be public.
   - Pin third-party Actions to a commit SHA.
   - Never expose deploy secrets to pull requests from forks; use a protected GitHub environment.
   - Prefer a limited deploy user that can only run one deploy script. Membership of the `docker` group is equal to root.
6. **Secret scanning:** turn on GitHub's free secret scanning and push protection.
7. **After a suspected compromise:** rotating `PAYLOAD_SECRET` ends sessions and invalidates API keys, so write that into the runbook (S12).

### S9. Pickup location and chat invitation can leak [High for harm, cheap to fix]

R16 and AD-14 treat the pickup address as the most protected fact in the system. It is a librarian's home. Two paths are not closed:

- **Photo metadata.** Scans made with a phone at home can carry GPS coordinates. AD-16 says images are "resized for web", but Payload stores the uploaded original unchanged unless `resizeOptions`, `formatOptions` or similar are set; only then is it re-encoded without metadata by default. [source] A public bucket would then serve a home location. Backgrounds in photos are a similar risk, and so are names written or stamped inside donated children's books. Those are child data, which requirement R2 rules out, and they would also go to the vision LLM.
- **Chat link in a public repo.** AD-6 says the chat group link lives in `tenant-policy` and changes through a versioned migration. Migrations live in `piviki-core`, which is public under AD-18. So the invitation R2 reserves for signed-up adults would be published. Librarian approval limits the damage but does not remove it, and it gives a hostile person a way to reach the group.

**Fix:**

1. Strip metadata in `piviki-scan` before upload *and* set `formatOptions` on the media collection, so the stored original is re-encoded too. Add a test that uploads a GPS-tagged image and checks that no EXIF data remains in the bucket.
2. Crop to the book. Add "no handwritten names, no background" to the librarian's review checklist, and review images before sending scans to the LLM when an inscription is visible.
3. Keep images private to review where it is cheap, for example random file names and no public listing. Moving draft media to a separate bucket is optional.
4. Load the chat link from a server environment value rather than a migration. Serve it only through `/api/v1` after the gate is passed. Keep the other policy values in migrations.
5. Also check that nothing in `_bmad-output`, `tulossa` or repo history carries the address, or a founder's personal details, before any repository goes public.

### S10. Session and CSRF design [Medium]

**The core design holds.** `piviki.fi` and `api.piviki.fi` are same-site, so a host-only, HTTP-only `SameSite=Lax` cookie works with `credentials: include` and needs no CORS tricks. Payload refuses the cookie when the `Origin` is not on the `csrf` list, and refuses cross-site requests without an `Origin`. [source] Keep this design, and add these to AD-7:

| Setting | State explicitly | Why |
|---|---|---|
| `cookies.secure` | `true` in production | Payload's default is `false` [source] |
| `cookies.sameSite` | `Lax`, never `None` | `None` is the usual copy-paste fix when CORS misbehaves |
| `cookies.domain` | Unset (host-only) | `.piviki.fi` would send it to every subdomain |
| `removeTokenFromResponses` | `true` | Otherwise login returns the JWT in the JSON body, readable by scripts, undoing much of HTTP-only [source] |
| `tokenExpiration` | Patrons about 7 days with refresh; librarians about 8 hours | Default is 2 hours, and `refresh-token` is missing from AD-5's allowed operations, so patrons would be logged out constantly |
| `useSessions` | Stay `true`; test that logout, password reset and deletion end the session | Default is `true`. Without it, a token from before a logout or password reset works until it expires [judgment; the test confirms it] |
| `csrf` and CORS | Same exact list, and non-empty | An empty `csrf` list disables the check; `serverURL` is added automatically [source] |

**Further rules:**

- **SameSite does not protect against subdomains.** `beta.piviki.fi` and any future subdomain are same-site, so only the `Origin` list protects you. Remove `beta.piviki.fi` from the list at launch. Keep DNS in OpenTofu so a forgotten record cannot be taken over.
- **`/api/v1` must go through Payload's auth.** Handlers must call `payload.auth({ headers })` so the `Origin` check runs, and they must set their own CORS headers if built as plain Next.js routes. Add one test that a wrong `Origin` gets an unauthenticated response.
- **No state change on GET**, and writes require `Content-Type: application/json`. Requests with no `Origin` are accepted when `Sec-Fetch-Site` is same-site, so this rule matters. [source]
- **Membership disclosure (user enumeration).** Here, "this email has an account" is itself the sensitive fact. Sign-up must give the same response whether or not the email exists and send an email either way. Payload's `forgot-password` already returns a generic success. [source] Login errors must not differ, and neither may Google failure redirects.
- **Reset tokens in URLs.** The token will sit in a `piviki.fi` URL. Set `Referrer-Policy: no-referrer`, and see S11 on logs.
- **Cache safety.** Authenticated `/api/v1` responses send `Cache-Control: private, no-store`, and CORS responses send `Vary: Origin`. Never apply a "cache everything" rule to the API host. Cloudflare serving one patron's account to another is the failure to avoid here.

### S11. Logs [Medium]

The AD-14 rule is right. These are the gaps:

- **IP addresses and retention.** Access logs hold IP addresses, which the notice must cover, and AD-14 sets no retention. Docker's default `json-file` driver does not rotate logs, so they grow until they fill the 40 GB disk, and retention is effectively forever.
- **Tokens in query strings.** Caddy's access log records the full URI, including Google `code` and `state`, and any `verify` or reset token that reaches core. Caddy redacts cookie and authorization headers by default, as far as I know, but not query strings. [judgment]
- **Postgres logs values.** A unique-constraint error includes `DETAIL: Key (email)=(...)`. Set `log_error_verbosity = terse`.
- **ORM and mail errors.** Database driver errors can include query parameters in their messages, and Payload logs errors. Payload's jobs table stores errors and task input, and failed jobs stay. Nodemailer errors can contain recipient addresses.
- **Email provider.** Its logs hold recipient and subject, usually for weeks. Book titles in subjects add detail. Turn off open and click tracking, because click tracking also sends reset links through the provider's redirect.

**Fix (under a day):**

1. Set Docker log rotation (`max-size`, `max-file`) to hold about 30 days. State that period in the notice.
2. Have Caddy drop the query string from logged URIs, or log only the path and status.
3. Put a redaction function in one shared logger, including removing `params` from database errors. Test it by forcing a duplicate-email error and searching the output for the address.
4. Keep the outbox and job payloads to ids. Render content at send time and delete sent rows after a short period; see S14.
5. **Add one thing:** a small, record-ids-only log of librarian logins and generated API writes. Breach assessment needs it (S12), and a log with no personal content is compatible with AD-14. Keep it about 90 days if the notice allows.

### S12. 72-hour breach readiness [Medium]

The initiative requires that "a breach can be notified within 72 hours". The spine defers "uptime alert and breach contact" as needing only a named owner. A name is necessary but not enough: the clock runs from when you become aware, including weekends and holidays, and both people are part-time.

Finland uses an electronic form on tietosuoja.fi, which allows a preliminary notification that you complete later. Every breach must be documented even if not reported, and for attacks on information systems the record should include log data from the time of the breach. High-risk breaches also require telling the people affected. [docs]

**Fix: one page in `piviki-infra`, plus a 30-minute drill. No tooling needed.**

1. **Roles:** primary and backup, by name. Agree that when one founder travels the other is reachable. If both are away, the fallback is taking core offline.
2. **Inbox:** one monitored address, such as `tietosuoja@piviki.fi`, that reaches both founders. Give it to processors as the breach contact and publish it in the notice. Personal inboxes will miss a processor's notice.
3. **First hour:** take core offline (`docker compose down`, or disable the tunnel), snapshot the disk and copy logs off the host **before** rotating anything. Rotate the secrets (S8).
4. **Assess:** what data, how many people, was it encrypted? A prepared note helps: "the patron list may reveal ethnic origin, so assume a higher risk rating". Stolen encrypted backups differ from a live-host compromise, where you should assume all patron data was exposed.
5. **Notify:** link to the form, and a draft message to patrons in Vietnamese and English. If the server is gone, emails come from the latest restored dump, so test this with the restore test and from a provider that doesn't depend on core.
6. **Record:** a simple breach log kept outside the VPS, including non-reportable incidents.
7. **Detection, cheaply:**
   - free uptime monitor;
   - backup heartbeat (S4);
   - Dependabot and advisory alerts (S6);
   - email on each librarian login, or a weekly glance at the S11 log;
   - a contact on the site for reports, and a `security.txt`, since the code is public.

   Be honest about the limit: with this budget, a careful intruder may go unnoticed. That is a reason to keep data minimal, not a reason to buy monitoring.

Move this from Deferred to a required item with an owner, and make it a done condition of `epic-launch`.

### S13. Cloudflare decrypting traffic [Medium, mostly legal]

With a proxied API, Cloudflare sees all traffic in clear text: passwords at login, session cookies, patron names, emails and loans. Full (strict) protects only the Cloudflare-to-origin leg. AD-18 names it as a processor, which is the right start, but the spine doesn't show it as a decision made with awareness of this.

| Option | Cloudflare sees | Lost | Cost |
|---|---|---|---|
| A. Keep the API proxied (spine) | All traffic | Nothing | Free |
| B. API DNS-only; site stays on Cloudflare | Site visits and IPs only | Access (S2), Tunnel (S7), rate limits, DDoS protection; S2 and S7 need other answers | Free, but more to run yourselves |

**Recommendation: A, unless the Art. 9 answer or the Ombudsman's guidance says otherwise.** For a two-person team, S1, S2 and S7 are worth more in practice than removing a contractually bound processor from the path. Revisit this if Q1 and Q2 come back badly. Moving to B is a few days' work, so it is not a one-way door.

**Conditions:**

- Don't claim in the notice that "all data stays in the EU". Traffic may be handled outside it, and the free plan has no regional restriction; Data Localization is an enterprise feature and out of proportion.
- Keep the Cloudflare DPA on file (Q3), and name Cloudflare's role in the notice in plain words.
- No Web Analytics, Zaraz, or injected scripts on the patron site, or AD-18 is broken without anyone noticing.
- Worker logging stays off or free of personal data. The Worker reads only public data, so its risk is small.

### S14. Retention, deletion and processor list [Medium]

**Records the rules do not reach.** AD-14 mentions loans and donations, and retention "counted from a loan's return".

- **Terminal reservations and queue places.** Those cancelled, expired or left never have a loan to count from, so they could keep a patron link forever.
- **Accounts that never borrow, or stop borrowing.** No retention event exists for them. See Q5.
- **`outbox` rows and job records.** They can hold a recipient or link a patron to a book.
- **Payload internal tables.** `payload-preferences` and locked-documents records have relationships to auth users. [source] Confirm no collection holding patron references uses `versions`, because version tables keep old values after the live row changes. `titles` having drafts is fine; `patrons`, `loans` and `donations` should not.
- **"One test per record type" is only as good as the list.** It misses whatever people forget.

**Fix (cheap and strong): the canary test.**

1. Create a patron with a unique email and name, and put them through a reservation, loan, email and donation.
2. Delete the account.
3. Run `pg_dump` and search the output for the canary email, name and patron id.
4. Fail the build on any match. Repeat the same test for retention by moving the clock forward.

This catches future tables automatically. Add a schema query that fails when a new foreign key to `patrons` lacks a test.

**Also:**

- **Anonymisation in a small community.** With about 30 families, a null patron plus date and copy may still point to someone (a rare book, a known pickup). It is probably fine, but consider coarsening old loan dates to the month. See Q6.
- **`gateAcceptedAt` and `policyVersion`** are proof of acceptance, and consent flags must survive changes. Deletion removes them with the account; confirm this is acceptable (Q5).
- **Access and portability.** R5 and R19 don't describe how a patron gets a copy of their data. A manual process via a librarian, documented in the notice, is proportionate. Add an operation to `modules/accounts` so it stays complete.

**The processor list in AD-18 is incomplete, or describes the wrong relationship:**

- **WhatsApp and Zalo.** The addendum has librarians contacting patrons there, using phone numbers from the system, and the chat group is itself a list of members of the community. AD-18 says patron data may reach "only" four providers. See Q7.
- **Google.** AD-18 requires a data processing agreement on file for each, but Google sign-in may not be a processor relationship and may have no such agreement to sign. As written, the rule might be impossible to satisfy. See Q3.
- **Vision LLM.** "Receives only book scans" is true only if S9's inscription check is done.
- **Founders' own devices and accounts.** Laptops with admin sessions: require disk encryption and a screen lock. This costs nothing, and it is a small written rule. Do not install any Payload export plugin, and do not keep CSV copies of patrons.

### S15. Rate limits and browser headers [Low]

**Rate limits.** AD-17 relies on Cloudflare limits for login and reset. The free plan gives 1 rule, path and verified-bot fields only, IP counting only, a 10-second period and a 10-second timeout. [docs]

- This slows bots a little but won't stop patient ones. Use the single rule for all sensitive paths: patron and librarian login, forgot and reset, sign-up, Google start. AD-17 leaves out sign-up.
- Payload's lockout (default 5 attempts, 10 minutes) [source] is per account, so it allows a nuisance lockout of a known email. Accept that.
- Forgot-password and sign-up send email and can burn the free email quota or bother a stranger. Add a per-address send limit in `modules/notifications`. Payload's `minRequestInterval` may help with forgot-password; check it. [source]
- Consider Cloudflare Turnstile on sign-up only if abuse appears. Don't add it upfront, since it means another notice item.

**Headers.** The patron site is what holds the session's power, so set these once in Cloudflare's static `_headers` file:

- a Content-Security-Policy with no third-party scripts, and `connect-src` limited to the API and `frame-ancestors 'none'`;
- HSTS and `X-Content-Type-Options: nosniff`;
- `Referrer-Policy: no-referrer`.

Make Mantine and Next.js work under it during the first patron story, because retrofitting CSP later is much more painful. Keep `npm ci` with a lockfile on patron CI, which limits the supply-chain route described in S2.

**Password checks.** Payload now hashes with PBKDF2 at 600,000 iterations [source], so no change is needed. Only minimum length (S2) needs setting. No forced rotation or complexity rules.

## 4. What should stay

- AD-1 and AD-2: one place for state and rules. This makes the rest of this review short.
- AD-14's minimisation: no child data, phone number not requested at sign-up, no address in the system, no free-text donor identity. The smallest possible data set is the strongest control you have.
- AD-15's Worker: no secrets, public data only, attribute setters that resist injection, fails open.
- AD-16: public and private buckets kept apart.
- AD-17: nothing built on the host, Postgres with no published port, restore test.
- AD-18: naming processors and blocking additions.
- The closed `signupOpen` until launch.

## 5. Deliberately not recommended

These would not be proportionate at 15 EUR a month and two part-time people: SIEM or log shipping, paid WAF or Cloudflare Enterprise and Data Localization, Vault or a KMS, a dedicated bastion or VPN, a separate VPS for admin, database field encryption, a paid penetration test, formal certification, MFA built into Payload for patrons, forced password rotation.

Disk encryption on the VPS is also not recommended: the key would need to be on the host to reboot unattended, so it adds work and little protection against the realistic threats.

## 6. Legal questions

For the founders to take to tietosuoja.fi's guidance or a qualified adviser. I am not answering them here.

- **Q1. Art. 9.** Is a Vietnamese community library's patron list data revealing ethnic origin? This is already an open question in the initiative. Note that the answer may depend on how the service is described and on whether speaking Vietnamese or borrowing Vietnamese books is enough. **Treat the list as sensitive in the design regardless.** If yes, which Art. 9(2) condition applies? Explicit consent changes the sign-up flow, and a condition for not-for-profit bodies may depend on the association's status and on whether patrons count as members. The answer affects the AD-8 gate fields, so ask before `epic-launch` starts and not at its end.
- **Q2. DPIA.** Is a data protection impact assessment needed, or advisable as a short written exercise, if Q1 is yes? This review may supply most of the content.
- **Q3. Processor or controller.** Is Google a processor or an independent controller for "Sign in with Google", and what document meets AD-18 in that case? Do Cloudflare's and OVH's standard online terms meet the "written agreement" requirement at the plan level used? Which transfer mechanism applies to US providers, including Cloudflare, Google and possibly the email provider?
- **Q4. Backups and erasure.** Is a fixed backup retention period with re-deletion on restore (S4) an acceptable way to handle erasure, and how should the notice describe it?
- **Q5. Inactive accounts and proof.** What retention applies to accounts with no loans or no recent activity? The 3-year rule starts from a loan's return, so nothing starts for them. How long should proof of consent or policy acceptance be kept after deletion, if at all? Does a right to erasure request have limits while a book is out (R19 refuses patron self-deletion then)?
- **Q6. Anonymisation.** Do null-linked loan records count as anonymous in a community this small, or are they still personal data subject to retention?
- **Q7. Chat groups.** Who is the controller for the WhatsApp and Zalo groups and the librarians' contact with patrons there? Is it the association, the founders personally or the group members? Does that need to be in the notice, and what about transfers to Zalo's operator outside the EU?
- **Q8. Controller identity and duties.** Is the association the registered controller, and who signs agreements on its behalf? Is a record of processing activities required (the addendum infers yes)? The founders should keep one regardless; it is a single page.
- **Q9. The 18+ tick.** Is a self-declared tick enough to support the statement that the system holds no child data, particularly if a parent's account or loan history indirectly suggests they have a child of a certain age? Loans of children's books inherently imply this, so the notice should perhaps say so plainly.
- **Q10. Library staff's phone numbers and home.** Does the association have duties to volunteers whose home address is shared with the chat group? It is outside the system but sits within the same risk.

## 7. Suggested spine changes

| AD | Change |
|---|---|
| AD-5 | Add path allowlist at the edge, default-deny helper, loop test, GraphQL off; add `refresh-token` and `verify/:id` to allowed operations; block `first-register` |
| AD-6 | Chat link comes from server configuration, not a migration |
| AD-7 | Add `scan-clients`; explicit cookie settings; separate admin host; Access in front |
| AD-8 | Require verified email on both sides, `sub` identity; choose `openid-client`; `signupOpen` applies to all login paths |
| AD-9 | Scan key is a limited service identity; draft forced server-side; no overwrite of published records |
| AD-13 | Named exception for the backup container |
| AD-14 | Add `googleSub`, IP addresses and log retention; all record types in retention; canary test |
| AD-16 | Strip metadata and re-encode originals; EU jurisdiction on the private bucket |
| AD-17 | Tunnel in place of IP allowlist and DNS token; backup retention, public-key encryption, erasure ledger and lock; patch rule; secret inventory; security headers |
| AD-18 | Resolve the Google and chat app wording after Q3 and Q7 |
| Deferred | Move "uptime alert and breach contact" out; make the runbook a done condition of `epic-launch` |

Suggested order: S1, S2, S7 and S6 in the first infra and core stories; S9 and S5 before the import; S3 and S10 in the auth stories; S4, S11, S12, S13 and S14 before launch; S8 from the start and revisited at launch. Total effort is about 4 to 6 working days spread across the epics, with no change to the monthly cost.

## 8. Sources

- Payload 3.92.1 package, `dist/`: `auth/defaultAccess.js`, `auth/extractJWT.js`, `auth/cookies.js`, `auth/operations/registerFirstUser.js`, `auth/strategies/apiKey.js`, `auth/strategies/local/generatePasswordSaltHash.js`, `auth/getAccessResults.js`, `collections/config/defaults.js`, `config/sanitize.js`, `fields/validations.js`, `uploads/generateFileData.js`
- payload-oauth2 1.0.21 package, `dist/`: `authorize-endpoint.js`, `callback-endpoint.js`, `types.d.ts`
- Office of the Data Protection Ombudsman, personal data breaches: https://tietosuoja.fi/en/personal-data-breaches
- Cloudflare WAF rate limiting rules: https://developers.cloudflare.com/waf/rate-limiting-rules/
- Cloudflare Authenticated Origin Pulls: https://developers.cloudflare.com/ssl/origin-configuration/authenticated-origin-pull/
- Cloudflare Access policies: https://developers.cloudflare.com/cloudflare-one/access-controls/policies/
- Cloudflare Zero Trust limits and pricing: https://developers.cloudflare.com/cloudflare-one/account-limits/ and https://www.cloudflare.com/plans/
- Cloudflare R2 bucket locks: https://developers.cloudflare.com/r2/buckets/bucket-locks/
- Cloudflare R2 data location: https://developers.cloudflare.com/r2/reference/data-location/

**Not verified, confirm when building:** Payload running on two hostnames (S2); Caddy's exact logging behaviour; Docker and `ufw` on the actual host; whether R2 EU jurisdiction carries any cost; `defaultUnlockAccess` and `minRequestInterval` behaviour; the OVH included backup's exact terms; the content of Cloudflare's, OVH's and Google's agreements.
