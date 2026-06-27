# Boilerplate — React Web App

## Stack

| Layer | Choice |
|---|---|
| Bundler | Vite 8 |
| UI framework | React 19 |
| Auth | Clerk React SDK (`@clerk/clerk-react`) |
| Routing | React Router v7 (`react-router` package) |
| API client | Typed fetch wrapper pointing to boilerplate API |
| Language | TypeScript |

## What's Included Out of the Box

- `main.tsx` — ClerkProvider wrapping the app with publishable key from env
- `router.tsx` — React Router config with public routes and a `<SignedIn>` / `<SignedOut>` guard pattern
- `pages/SignIn.tsx`, `pages/SignUp.tsx` — Clerk-hosted or embedded auth pages
- `pages/Dashboard.tsx` — placeholder authenticated page
- `api/client.ts` — typed fetch wrapper that injects Clerk session token on each request
- `vite.config.ts` — proxy `/api` to `localhost:3000` in dev so no CORS issues locally

## Required Environment Variables

```
VITE_CLERK_PUBLISHABLE_KEY=
VITE_API_BASE_URL=http://localhost:3000
```

## How to Run Locally

```bash
cd apps/web
cp .env.example .env   # fill in Clerk publishable key
npm install
npm run dev
```

App starts on `http://localhost:5173`. The Vite dev proxy forwards `/api/*` to the Express backend.
