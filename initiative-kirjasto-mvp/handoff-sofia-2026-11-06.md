# Handoff to Sofia (bmad-agent-po): plan a host-runtime layout story at the end of epic-librarian-catalogue

Written 2026-11-06 during story 1.1's first deploy to test. This file is a work request: **add one new
story at the end of `epic-librarian-catalogue`**. It is **not a decision**. The layout below is
a direction the founders want weighed; the story decides it, through `bmad-architecture` where a
spine decision changes. Read the sources it names; do not rely on its summaries where a source exists.

## What the founders asked (2026-11-06)

- Reconsider **where `deploy/` lives and how it is implemented** (today: `piviki-core/deploy`,
  AD-21).
- Consider **decoupling `cloudflared` and `postgres` from core's Compose file**. One possible
  home is `piviki-infra`. Postgres and the Tunnel would then run independently of the app, so a
  core deploy never starts, stops or recreates them.

## Why: what story 1.1 ran into

1. **First deploy, chicken and egg.** `cloudflared` is a service in core's Compose stack, which
   only a deploy starts. A deploy reaches the host over SSH through that Tunnel. On a fresh host
   the first deploy therefore failed (`websocket: bad handshake`); a founder had to start
   `cloudflared` by hand (`docker compose … up -d --no-deps cloudflared`). Neither runbook covers
   this, and production's start will hit it again.
2. **A deploy can cut its own connection.** `deploy.sh` runs `docker compose up -d --wait
   --remove-orphans` over every service. When a `deploy/` change alters `cloudflared`'s
   definition, for example a Dependabot digest bump, Compose recreates it mid-deploy. The Tunnel
   carrying the deploy's SSH session drops, `deploy.sh` is killed between "start" and "record",
   and the run fails with the tag unrecorded. A Postgres digest bump likewise restarts the
   database inside an app deploy.
3. **Host steps per change.** Fixed on 2026-11-06 (piviki-core `1e11a5a…`, plan
   `plan-1-1-deploy-tree-label.md`). The host now compares a fingerprint of the `deploy/` files it
   reads (`deploy/fingerprint.sh`, image label `fi.piviki.host-runtime`). An app-only tag
   therefore needs no `--move-core`, but any change to Compose, Caddy or `deploy.sh` still does.

## What the story must weigh (not decided here)

- **Which services belong to the app's release and which to the host.**
  - `core` and its migrations are the app.
  - `caddy` carries core's path allowlist and the first-register block, so it is coupled to
    the app's routes.
  - `cloudflared` and `postgres` are host infrastructure.
- **Where each part's Compose file and env files live.**
  - The services in core may be a separate Compose project from those in infra, on the
    same host.
  - The networks they share (`edge`; Postgres's internal network) need one owner.
  - The env-file manifest (AD-21) may need a split.
- **How infrastructure services get updated.** Who runs it (a founder or a workflow), whether it
  is separate from a core deploy, and how an update to `cloudflared` itself avoids cutting its own
  session.
- **What AD-21 keeps.**
  - One recorded version per host. It may become two: a core tag and an infra commit.
  - Who controls root commands on the host.
  - A deploy never fetches.
  - Pinned digests.
  - How a restore or rebuild names the whole runtime (AD-21, the dump records the core tag).
- **What it means for the deploy checks.** The one-connector check, and core's start-up refusal
  when host names disagree (AD-17).

## Constraints for the ticket

- **Changes to the spine.** It changes AD-21, and possibly AD-17 ("piviki-infra holds only what is
  shared"). AD-21 was set on 2026-11-04 by moving the host files *into* core, and the memlog
  holds why. Its story runs `bmad-architecture` (Update) before building, so it is `hitl`.
- **Placement.** The founders asked for it at the end of this epic. It must still land
  **before epic-soft-open's production start**, so that production is set up once, in the final
  layout. The production host is the OVH VPS-1. Place it relative to story 14, "Refactor sweep";
  the sweep is currently `after` every other story.
- **Open findings.**
  - The first-deploy Tunnel gap (1) and the mid-deploy recreate (2) are open findings, and
    the story must close both whatever layout it picks.
  - The first-deploy gap may need a stopgap runbook step on test sooner. Ask the founders
    whether story 1.1's closing should carry that.
- **Test runs it first.** Test is rebuilt by its runbooks, so the new layout is proven on a
  fresh test host or by migrating the existing one, with no data to keep.

## Sources (authoritative)

- Spine: `architecture-piviki/architecture-piviki.md`: AD-17 (environments, IaC, deploy), AD-21
  (host runtime), and their `.memlog.md` entries of 2026-11-04 and 2026-11-06.
- Story 1.1's plan:
  `tickets/epic-librarian-catalogue/tickets/story-a-librarian-signs-in-at-admin-piviki-fi-in-production-plan.md`
  (resume point and Plan Change Log, 2026-11-06).
- Fix plan: `plan-1-1-deploy-tree-label.md`.
- Code: `piviki-core/deploy/` (`compose.yml`, `deploy.sh`, `fingerprint.sh`, `env-manifest`,
  `README.md`), `piviki-core/.github/workflows/{build,deploy}.yml`, `piviki-infra/bootstrap/`,
  `piviki-infra/docs/runbook-{host-setup,first-deploy}.md`.
