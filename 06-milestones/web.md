# Milestones — React Web (`apps/web`)

Stack reference: [[react-web]] (boilerplate). Vite 8 · React 19 · React Router v7 · Clerk React SDK · TypeScript.

Build order is top-down. Each milestone lists **Scope** and **Acceptance**. Tick acceptance boxes only after running the check. The web app's API milestone (M4) depends on the Backend API reaching its M3 (auth).

---

## M1 — Scaffold & Tooling `[ ]`

**Goal:** A Vite + React 19 + TS app that runs and builds, with linting and validated env.

**Scope**
- [ ] Vite 8 + React 19 + TypeScript scaffold
- [ ] ESLint + Prettier; `lint` and `format` scripts
- [ ] Folder structure: `src/{pages,components,api,lib,routes}`
- [ ] `.env.example` (`VITE_CLERK_PUBLISHABLE_KEY`, `VITE_API_BASE_URL`); env validated at startup
- [ ] `vite.config.ts` with `/api` → `localhost:3000` dev proxy

**Acceptance**
- [ ] `npm run dev` serves on `http://localhost:5173` with no console errors
- [ ] `npm run build` produces a `dist/` bundle; `npm run preview` serves it
- [ ] `npm run lint` passes with zero errors
- [ ] Missing `VITE_CLERK_PUBLISHABLE_KEY` surfaces a clear error, not a blank screen

---

## M2 — Routing & Layout `[ ]`

**Goal:** React Router v7 with a layout shell and route structure.

**Scope**
- [ ] React Router v7 (`react-router`) configured in `router.tsx`
- [ ] Root layout (header/nav + `<Outlet/>`) shared across routes
- [ ] Routes: `/` (public landing), `/dashboard` (placeholder), `*` (404)
- [ ] Client-side nav (no full reloads)
- [ ] Loading/pending UI for route transitions

**Acceptance**
- [ ] Navigating between `/` and `/dashboard` updates the view without a full page reload
- [ ] An unknown URL renders the 404 page
- [ ] The layout shell (nav) persists across route changes
- [ ] Direct-loading `/dashboard` by URL works (no blank screen)

---

## M3 — Auth (Clerk) `[ ]`

**Goal:** Clerk-gated routing with sign in / sign up.

**Scope**
- [ ] `ClerkProvider` wrapping the app in `main.tsx` with publishable key from env
- [ ] `pages/SignIn.tsx`, `pages/SignUp.tsx` (Clerk components)
- [ ] `<SignedIn>` / `<SignedOut>` guard; `/dashboard` requires auth
- [ ] Unauthenticated access to a protected route redirects to sign-in
- [ ] User button / sign-out in the nav

**Acceptance**
- [ ] Visiting `/dashboard` while signed out redirects to sign-in
- [ ] Completing sign-up/sign-in lands on `/dashboard`
- [ ] Sign-out returns to public state and re-protects `/dashboard`
- [ ] Refreshing while signed in keeps the session (no flash back to sign-in)

---

## M4 — API Integration `[ ]`

**Goal:** Typed API client that injects the Clerk token, used for a real authenticated call. **Depends on Backend API M3.**

**Scope**
- [ ] `api/client.ts` typed fetch wrapper injecting the Clerk session token per request
- [ ] Base URL from `VITE_API_BASE_URL`; uses `/api` proxy in dev
- [ ] `/dashboard` fetches `GET /me` from the backend and renders the user
- [ ] Loading + error states for the request
- [ ] `401` from the API triggers a re-auth / sign-in path

**Acceptance**
- [ ] `/dashboard` displays data fetched from the backend `/me` while authenticated
- [ ] The request carries an `Authorization: Bearer <token>` header (verify in Network tab)
- [ ] Loading state shows before data; error state shows if the API is down
- [ ] Signed-out / expired token results in a handled redirect, not an unhandled crash

---

## M5 — UI System & Polish `[ ]`

**Goal:** A reusable, accessible, responsive UI foundation. (See skill: `frontend-design`.)

**Scope**
- [ ] Design tokens (colors, spacing, typography) + theming
- [ ] Base component set (Button, Input, Card, etc.)
- [ ] Responsive layout (mobile → desktop)
- [ ] Light/dark mode
- [ ] Accessibility: focus states, semantic HTML, keyboard nav

**Acceptance**
- [ ] Layout is usable at 375px and 1440px widths (no overflow/broken layout)
- [ ] Dark mode toggles and persists across reloads
- [ ] Keyboard-only navigation reaches all interactive elements with visible focus
- [ ] An automated a11y check (axe) reports no critical violations on key pages

---

## M6 — Production Readiness `[ ]`

**Goal:** Resilient, optimized, deployable build. (See skill: `production-ready`.)

**Scope**
- [ ] Route-level error boundaries + a global fallback
- [ ] Sentry capturing client errors
- [ ] Build optimization: code-splitting per route, bundle analyzed
- [ ] SPA fallback / hosting config for client-side routing
- [ ] Deploy to Railway/AWS (static hosting); deploy notes in `05-infra`

**Acceptance**
- [ ] A thrown render error shows the fallback UI, not a white screen, and reports to Sentry
- [ ] Production build code-splits (separate chunks per route, verifiable in `dist/`)
- [ ] Deep-linking a deployed route (e.g. `/dashboard`) loads correctly (SPA fallback works)
- [ ] Deployed app loads over its public URL and auth + API calls work end-to-end

---

## M7 — Testing & CI `[ ]`

**Goal:** Component/integration tests gated in CI. (See skill: `testing`.)

**Scope**
- [ ] Vitest + React Testing Library set up
- [ ] Tests: routing/guard behavior, a component, the API client (mocked) token injection
- [ ] Coverage reporting; `test` and `test:watch` scripts
- [ ] GitHub Actions: install → lint → test → build

**Acceptance**
- [ ] `npm test` runs green locally
- [ ] A test verifies signed-out users can't see protected content
- [ ] CI workflow passes on a pushed branch
- [ ] CI fails when a test is deliberately broken (guard works)
