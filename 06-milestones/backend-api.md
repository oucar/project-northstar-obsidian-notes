# Milestones — Backend API (`apps/uapi`)

Stack reference: [[backend-api]] (boilerplate). Node.js 24 LTS · Express 5.2 · PostgreSQL · Prisma 7 · Clerk · Upstash Redis · TypeScript.

Build order is top-down. Each milestone lists **Scope** (what to build) and **Acceptance** (checks that must pass before it's done). Tick acceptance boxes only after running the check.

---

## M1 — Scaffold & Tooling `[ ]`

**Goal:** A TypeScript Express app that boots, with linting, formatting, and validated env.

**Scope**
- [ ] `npm init`, TypeScript, `tsconfig.json` (strict mode on)
- [ ] Express 5.2 app in `src/app.ts` + `src/server.ts` entrypoint
- [ ] Scripts: `dev` (watch), `build` (tsc), `start` (run built)
- [ ] ESLint + Prettier configured, `lint` and `format` scripts
- [ ] Folder structure: `src/{routes,middleware,lib,config}`
- [ ] `.env.example` with all vars; env loaded + validated at startup (zod) — process exits on missing/invalid var
- [ ] `GET /health` route returning `{ status: "ok" }`

**Acceptance**
- [ ] `npm run dev` starts the server on `PORT` with no errors
- [ ] `curl localhost:3000/health` → `200` with `{"status":"ok"}`
- [ ] `npm run build` produces `dist/` and `npm start` runs it
- [ ] `npm run lint` passes with zero errors
- [ ] Removing a required var from `.env` makes startup fail with a clear message

---

## M2 — Database & Prisma `[ ]`

**Goal:** Postgres wired through Prisma 7 with a migrated `User` model.

**Scope**
- [ ] Prisma 7 installed; `prisma/schema.prisma` with `User` model (id, `clerkUserId @unique`, email, timestamps)
- [ ] Prisma client singleton in `src/lib/prisma.ts` (no hot-reload connection leaks)
- [ ] First migration committed under `prisma/migrations/`
- [ ] DB connectivity checked at startup (fail fast if unreachable)
- [ ] `db:migrate`, `db:studio`, `db:generate` scripts

**Acceptance**
- [ ] `npx prisma migrate dev` applies cleanly against a local Postgres
- [ ] `npx prisma studio` shows the `User` table
- [ ] Server logs a successful DB connection on boot; with a bad `DATABASE_URL` it fails fast with a clear error
- [ ] A throwaway script can create + read a `User` row

---

## M3 — Auth (Clerk) `[ ]`

**Goal:** Clerk JWT verification protecting routes, with user sync to Postgres.

**Scope**
- [ ] `@clerk/express` wired; `clerkMiddleware()` mounted
- [ ] `requireAuth` middleware that rejects unauthenticated requests with `401`
- [ ] `GET /me` protected route returning the current user
- [ ] On first authenticated request, upsert a `User` row keyed by `clerkUserId`
- [ ] `req.auth` typed via module augmentation

**Acceptance**
- [ ] `GET /me` with no token → `401`
- [ ] `GET /me` with a valid Clerk token → `200` and the user payload
- [ ] First authed call creates exactly one `User` row; repeat calls don't duplicate it
- [ ] Health check remains public (no token required)

---

## M4 — Redis Caching & Rate Limiting `[ ]`

**Goal:** Upstash Redis client plus rate limiting and a reusable cache helper.

**Scope**
- [ ] `@upstash/redis` singleton in `src/lib/redis.ts`
- [ ] Global rate limiter (e.g. 100 req / 15 min per IP) via `express-rate-limit`
- [ ] Stricter limiter on auth-sensitive routes
- [ ] `cache.get/set(key, ttl)` helper; one route demonstrates cache-hit/miss
- [ ] Rate-limit store backed by Redis (so limits hold across instances)

**Acceptance**
- [ ] Exceeding the limit returns `429` with a `Retry-After` header
- [ ] A cached endpoint returns a `X-Cache: HIT` (or logged equivalent) on the second call
- [ ] Cached value is visible in Upstash and expires after its TTL
- [ ] Redis being down degrades gracefully (logged), it doesn't crash the server

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
