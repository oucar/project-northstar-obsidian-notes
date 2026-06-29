# Milestones — Backend API (`apps/uapi`)

Stack reference: [[backend-api]] (boilerplate). Node.js 24 LTS · Express 5.2 · PostgreSQL · Prisma 7 · Clerk · Upstash Redis · TypeScript.

Build order is top-down. Each milestone lists **Scope** (what to build) and **Acceptance** (checks that must pass before it's done). Tick acceptance boxes only after running the check.

---

## M1 — Scaffold & Tooling `[x]`

**Goal:** A TypeScript Express app that boots, with linting, formatting, and validated env.

**Scope**
- [x] `npm init`, TypeScript, `tsconfig.json` (strict mode on)
- [x] Express 5.2 app in `src/app.ts` + `src/server.ts` entrypoint _(entrypoint is `src/index.ts`)_
- [x] Scripts: `dev` (watch), `build` (tsc), `start` (run built)
- [x] ESLint + Prettier configured, `lint` and `format` scripts
- [x] Folder structure: `src/{routes,middleware,lib,config}`
- [x] `.env.example` with all vars; env loaded + validated at startup (zod) — process exits on missing/invalid var
- [x] `GET /health` route returning `{ status: "ok" }`

**Acceptance**
- [x] `npm run dev` starts the server on `PORT` with no errors
- [x] `curl localhost:3000/health` → `200` with `{"status":"ok"}`
- [x] `npm run build` produces `dist/` and `npm start` runs it
- [x] `npm run lint` passes with zero errors
- [x] Removing a required var from `.env` makes startup fail with a clear message

---

## M2 — Database & Prisma `[x]`

**Goal:** Postgres wired through Prisma 7 with a migrated `User` model.

**Scope**
- [x] Prisma 7 installed; `prisma/schema.prisma` with `User` model (id, `clerkUserId @unique`, email, timestamps)
- [x] Prisma client singleton in `src/lib/prisma.ts` (no hot-reload connection leaks)
- [x] First migration committed under `prisma/migrations/`
- [x] DB connectivity checked at startup (fail fast if unreachable)
- [x] `db:migrate`, `db:studio`, `db:generate` scripts

**Acceptance**
- [x] `npx prisma migrate dev` applies cleanly against a local Postgres
- [x] `npx prisma studio` shows the `User` table
- [x] Server logs a successful DB connection on boot; with a bad `DATABASE_URL` it fails fast with a clear error
- [x] A throwaway script can create + read a `User` row _(verified via the `/me` upsert: one `User` row exists)_

---

## M3 — Auth (Clerk) `[~]`

**Goal:** Clerk JWT verification protecting routes, with user sync to Postgres.

**Scope**
- [x] `@clerk/express` wired; `clerkMiddleware()` mounted
- [x] `requireAuth` middleware that rejects unauthenticated requests with `401`
- [x] `GET /me` protected route returning the current user
- [x] On first authenticated request, upsert a `User` row keyed by `clerkUserId`
- [ ] `req.auth` typed via module augmentation _(uses `getAuth(req)` instead — no augmentation yet)_

**Acceptance**
- [x] `GET /me` with no token → `401`
- [x] `GET /me` with a valid Clerk token → `200` and the user payload
- [x] First authed call creates exactly one `User` row; repeat calls don't duplicate it
- [x] Health check remains public (no token required)

> [!note] Hardening follow-up
> `/me` throws an unhandled `ClerkAPIResponseError` (→ 500) when a token's user no longer exists in the Clerk instance (e.g. a stale session). Catch the 404 and return `401` instead. Tracked under M5 (centralized error handling).

---

## M4 — Redis Caching & Rate Limiting `[~]`

**Goal:** Upstash Redis client plus rate limiting and a reusable cache helper.

**Scope**
- [x] `@upstash/redis` singleton in `src/lib/redis.ts`
- [x] Global rate limiter (100 req / 15 min per IP) via `express-rate-limit` (`src/middleware/rate-limit.ts`)
- [x] Stricter limiter on auth-sensitive routes (`/me`: 20 req / 15 min)
- [x] `cache.get/set(key, ttl)` helper (`src/lib/cache.ts`); `GET /stats` demonstrates cache-hit/miss
- [x] Rate-limit store backed by Redis — custom `express-rate-limit` store over Upstash (atomic Lua `INCR`+`PEXPIRE`), shared across instances

**Acceptance**
- [ ] Exceeding the limit returns `429` with a `Retry-After` header _(needs real Upstash creds — fails open with placeholder creds)_
- [ ] A cached endpoint returns a `X-Cache: HIT` (or logged equivalent) on the second call _(needs real Upstash creds)_
- [ ] Cached value is visible in Upstash and expires after its TTL _(needs real Upstash creds)_
- [x] Redis being down degrades gracefully (logged), it doesn't crash the server _(verified: placeholder creds → all routes serve, cache→miss, limiter→open, both logged)_

> [!warning] Blocked on credentials
> `apps/uapi/.env` has placeholder Upstash values (`https://placeholder.upstash.io`). Code is complete and builds/lints clean; the three live acceptance checks need a real Upstash Redis (free tier) URL + token.

---

## M5 — Security & Hardening `[ ]`

**Goal:** Standard hardening, validated input, and consistent error handling.

**Scope**
- [ ] `helmet` security headers
- [ ] `cors` with an allowlist from env (not `*` in production)
- [ ] Request body validation via zod on every write route; invalid → `400` with field errors
- [ ] Centralized error handler → consistent JSON error shape; no stack traces leaked in production
- [ ] `404` handler for unknown routes
- [ ] Request body size limit set

**Acceptance**
- [ ] Response includes Helmet headers (`curl -I` shows `X-Content-Type-Options`, `Strict-Transport-Security`, etc.)
- [ ] A cross-origin request from a non-allowlisted origin is rejected
- [ ] Malformed JSON / invalid body → `400` with a structured error, not a `500`
- [ ] An unhandled thrown error returns a clean JSON `500` (no stack trace in prod mode)
- [ ] Unknown route → `404` JSON

---

## M6 — Production Readiness `[ ]`

**Goal:** Observable, deployable service with safe lifecycle. (See skill: `production-ready`.)

**Scope**
- [ ] Structured logging with `pino` (request id, method, path, status, latency)
- [ ] Sentry (or equivalent) capturing unhandled errors
- [ ] Graceful shutdown on `SIGTERM`/`SIGINT` (drain server, close Prisma + Redis)
- [ ] `GET /health` (liveness) and `GET /ready` (readiness: checks DB + Redis)
- [ ] `Dockerfile` (multi-stage) + `.dockerignore`
- [ ] Railway/AWS deploy notes in `05-infra`

**Acceptance**
- [ ] Logs are structured JSON with a per-request id
- [ ] A forced error appears in the Sentry dashboard
- [ ] `SIGTERM` drains in-flight requests and closes connections cleanly (no forced kill)
- [ ] `/ready` returns `503` when DB or Redis is down, `200` when both are up
- [ ] `docker build` succeeds and the container serves `/health`
- [ ] Deployed instance answers `/health` over the public URL

---

## M7 — Testing & CI `[ ]`

**Goal:** Automated tests against a real Postgres, gated in CI. (See skill: `testing`.)

**Scope**
- [ ] Vitest + Supertest set up
- [ ] Integration tests against a real (ephemeral) Postgres — health, auth (401/200), a CRUD route
- [ ] Test DB spun up + migrated in CI (service container)
- [ ] Coverage reporting; `test` and `test:watch` scripts
- [ ] GitHub Actions workflow: install → lint → migrate test DB → test → build

**Acceptance**
- [ ] `npm test` runs green locally against a real Postgres
- [ ] Auth tests cover both `401` (no token) and `200` (valid token)
- [ ] CI workflow passes on a pushed branch
- [ ] CI fails when a test is deliberately broken (guard works)
