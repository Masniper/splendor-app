# Production readiness checklist — Splendor Online

This document expands the prioritized roadmap from **MVP (play with friends)** to a **reliable production deployment**. It does not prescribe gameplay or game-rule changes; focus is **security, operations, reliability, and product polish**.

Use checkboxes as you complete items. Revisit **P0** before every first launch on a new domain or host.

---

## How to use this file

| Priority | Meaning |
|----------|---------|
| **P0** | Do before pointing a real domain or non-technical friends at the stack. |
| **P1** | Do soon after go-live to reduce incident pain and operational surprise. |
| **P2** | Quality, maintainability, and UX depth without changing core game logic. |
| **P3** | When the audience grows beyond a closed friend group or you need trust/compliance depth. |

**Suggested order:** finish **all P0**, then **P1** in parallel with small **P2** items. Treat **P3** as a separate milestone when you intentionally open the product up.

---

## P0 — Blockers before production

### Secrets and configuration

- [ ] **JWT_SECRET** is a long, random value (e.g. 32+ bytes from a CSPRNG). It is not committed, not baked into images as a default, and not logged.
- [ ] **Database credentials** for PostgreSQL are strong and unique per environment (dev / staging / prod).
- [ ] **`.env` is never committed** (parent `.gitignore` already lists `.env`; verify no overrides).
- [ ] **`.env.example`** (or internal runbook) documents every required variable with a **placeholder**, never a real secret.
- [ ] **Rotation plan**: you know how to rotate `JWT_SECRET` (accept that existing tokens invalidate) and DB passwords without data loss.

### TLS (HTTPS)

- [ ] The **browser** loads the app over **HTTPS** (valid certificate: Let’s Encrypt, managed LB, etc.).
- [ ] **API and Socket.io** are reachable on the same security model the client expects (typically same host behind reverse proxy, or explicit `VITE_*` build args documented).
- [ ] **HSTS** and modern TLS versions are considered at the reverse proxy (nginx, Traefik, cloud LB).

### Database lifecycle

- [ ] **Backups**: automated backups of PostgreSQL (volume snapshots, `pg_dump` cron, or managed DB backups).
- [ ] **Restore drill**: you have restored a backup to a scratch instance at least once.
- [ ] **Migrations**: production deploy runs **`prisma migrate deploy`** (or equivalent) before or as part of app start; failed migrations block rollout.
- [ ] **Connection string** uses TLS to the DB if the provider requires it (`sslmode` in `DATABASE_URL` where applicable).

### Network and application hardening

- [ ] **CORS** for HTTP API is restricted to known frontend origins (not wide open in production).
- [ ] **Socket.io CORS** is restricted to the same origins you actually use (current dev-style `*` is not appropriate for public production).
- [ ] **Admin / debug endpoints** (if any) are not exposed publicly without authentication.
- [ ] **Rate limiting** on authentication routes (`/api/auth/*`) to mitigate brute force and registration spam (even a simple limit per IP helps).

### Observability and operations

- [ ] **Structured logs** (JSON or key=value) from the API process, with log aggregation or file rotation on the host.
- [ ] **Error tracking** for the server (e.g. Sentry, OpenTelemetry exporter) — at minimum uncaught exceptions and critical paths.
- [ ] **Health endpoint**: e.g. `GET /health` returns 200 when process + DB connectivity are OK; use it for Docker `HEALTHCHECK`, Kubernetes probes, or uptime monitors.
- [ ] **Uptime check** on the public URL (external ping or synthetic check).

### Docker / Compose (if you use them in prod)

- [ ] **Images** are built from pinned base tags or digests where practical.
- [ ] **Non-root user** in containers considered for the API image (defense in depth).
- [ ] **Resource limits** (CPU/memory) set so one runaway process does not take the whole host.
- [ ] **Secrets** injected via environment or secret manager, not copied into layers.

### Legal and expectations (minimal)

- [ ] **Privacy**: short note on what you store (accounts, room metadata, bets, ephemeral chat) and retention — even a simple `PRIVACY.md` helps friends trust the deployment.
- [ ] **Terms / acceptable use** if strangers might eventually join public rooms.

---

## P1 — Soon after go-live (stability and hygiene)

### Monitoring and alerts

- [ ] **Alert** when health check fails or error rate spikes.
- [ ] **Disk and DB storage** alerts (Postgres volume growth).
- [ ] **Process restart policy** documented (systemd, Docker `restart`, k8s `Deployment`).

### Dependency and supply chain

- [ ] **Regular `npm audit`** (or equivalent) with a policy: when to patch, when to accept risk.
- [ ] **Lockfiles** committed; CI uses `npm ci` for reproducible builds.

### Deployment runbook

- [ ] One-page **deploy steps**: pull/build, migrate, restart, verify smoke test.
- [ ] **Rollback**: previous image/version + compatible DB migration strategy (forward-only migrations preferred; document exceptions).

### Smoke testing

- [ ] Repeatable **smoke checklist** after each deploy: login or guest → create/join room → one game action → chat message → graceful leave.
- [ ] Optional: automate the above with **Playwright** or API-only scripts.

### Data and migrations communication

- [ ] If you ever shipped **SQLite** to early testers, document that **Postgres is now canonical** and old local DBs are not migrated automatically.

---

## P2 — Product and engineering quality (no rule-engine changes)

### Onboarding and help

- [ ] Short **“How to play with friends”** flow: host creates room → shares code/link → guests join.
- [ ] In-app **Help / Rules** entry (link to official Splendor rules or your own summary) so new players are not blocked on external search.

### Resilience UX

- [ ] Clear **disconnect / reconnect** messaging; visible **retry** for failed API calls where it makes sense.
- [ ] **Loading and empty states** audited on main screens (lobby, public rooms, profile).

### Automated quality gates

- [ ] **CI** (e.g. GitHub Actions): on PR — backend `npm test` + `tsc`, frontend `tsc` / lint as you define.
- [ ] **Expand backend tests** beyond a few units (auth edge cases, room join errors, bet settlement paths — without changing game rules).

### Accessibility

- [ ] Keyboard **focus order** and **Escape** to close modals where applicable.
- [ ] Critical controls have **labels** / `aria-*` where missing (build on existing patterns in `RoomChatPanel`, modals, etc.).

### Performance (web)

- [ ] **Bundle size** sanity check after major dependency changes.
- [ ] Large static assets (images) **compressed** and appropriate formats (re-check after art changes).

---

## P3 — Broader audience and trust depth

### Chat and social safety

- [ ] **Per-IP or per-user rate limits** on chat socket events.
- [ ] **Host mute / kick** (product decision + implementation).
- [ ] **Report user** flow if public rooms are open to strangers.

### Account lifecycle

- [ ] **Password reset** (email token flow) if accounts are email-based long term.
- [ ] **Email verification** optional but valuable before any payments or reputation.

### Meta features (persistence only)

- [ ] **Match history** or post-game summaries stored for the user (design so it does not require changing core move validation — snapshots or summaries only).
- [ ] **Seasons / ranked** only if you explicitly want competitive meta; ties into existing stats fields in the schema.

### Internationalization

- [ ] **i18n** pipeline if non-English players are a goal (UI strings extracted, locale switching).

---

## Quick reference — environment variables (production mindset)

| Area | Typical concerns |
|------|------------------|
| `JWT_SECRET` | Strong, unique, rotated with a plan. |
| `DATABASE_URL` | Postgres with TLS if required; correct network (not wrong `localhost` inside containers). |
| `PORT` | Matches reverse proxy upstream. |
| Frontend `VITE_*` | Set at **build time** for production; document exact values per environment. |

---

## Related files in this repository

- Parent **`README.md`** — clone, submodules, Docker Compose overview.
- **`docker-compose.yml`** — Postgres + API + Nginx web; requires `.env` with `JWT_SECRET`.
- **`back-end/README.md`** — Prisma, API, Socket.io chat reference.
- **`front-end/README.md`** — PWA, Vite env, client architecture.

---

## Revision

Update this checklist when your threat model changes (friends-only → public lobbies → payments). Last structured for a **friends-first** MVP moving toward **hosted production**.