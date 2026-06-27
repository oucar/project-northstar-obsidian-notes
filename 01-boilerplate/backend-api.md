# Boilerplate — Backend API

## Stack

| Layer | Choice |
|---|---|
| Runtime | Node.js (LTS) |
| Framework | Express |
| Auth | Clerk — JWT verification via `@clerk/express` |
| Database | PostgreSQL |
| ORM | Prisma |
| Cache / sessions | Upstash Redis (`@upstash/redis`) |
| Rate limiting | `express-rate-limit` |
| Security headers | `helmet` |
| CORS | `cors` |
| Language | TypeScript |

## What's Included Out of the Box

- `clerk.middleware.ts` — verifies Clerk JWT on every protected route; attaches `req.auth`
- `prisma/schema.prisma` — base schema with `User` model synced to Clerk user ID
- `redis.ts` — Upstash Redis client singleton
- `rateLimiter.ts` — global rate limiter (100 req / 15 min per IP) + stricter auth-route limiter
- `app.ts` — Express app with Helmet, CORS, JSON body parser, health-check route
- `.env.example` — all required env vars documented

## Required Environment Variables

```
DATABASE_URL=
CLERK_SECRET_KEY=
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
PORT=3000
```

## How to Run Locally

```bash
cd apps/api
cp .env.example .env   # fill in values
npm install
npx prisma migrate dev
npm run dev
```

API starts on `http://localhost:3000`. Health check: `GET /health`.
