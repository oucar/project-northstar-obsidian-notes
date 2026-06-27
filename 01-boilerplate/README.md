# Boilerplate — Overview

The Project Northstar boilerplate is a monorepo starter that every new app forks from. It provides auth, database, API, and frontend shells so each new product starts with production-grade fundamentals already wired up.

## What's Included Out of the Box

- **Backend API** — Express server with Clerk JWT verification, Prisma + PostgreSQL, Upstash Redis session/cache, rate limiting, Helmet security headers, CORS
- **React Native app** — Expo or bare RN (TBD), React Navigation v6 shell, Clerk RN SDK, iOS Swift widget folder, Android Kotlin widget folder, typed API client
- **React web app** — Vite + React 18, Clerk React SDK, typed API client, basic routing shell

## Repo Structure

```
boilerplate/
├── apps/
│   ├── api/          # Node.js backend
│   ├── mobile/       # React Native
│   └── web/          # Vite + React
├── packages/
│   └── api-client/   # Shared typed API client (TBD)
└── ...
```

## How to Run Locally

See each sub-doc for service-specific instructions:

- [`backend-api.md`](./backend-api.md)
- [`react-native.md`](./react-native.md)
- [`react-web.md`](./react-web.md)
