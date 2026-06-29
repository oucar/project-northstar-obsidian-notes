# Design Doc — React Web (`apps/web`)

> **What this is.** The big-picture architecture of the web app: how it boots, routes, authenticates, loads data, and handles failure. Read this to understand *how the pieces fit* before changing them. For the quick-start / stack list see [[react-web]]; for delivery status see `06-milestones/web.md`.
>
> **Audience.** Both the human and AI working on this repo. When adding a feature, follow the patterns and the **Core Principles** below so the app stays consistent.

---

## 1. Core Principles

These are non-negotiable defaults. New code should uphold them unless there's a documented reason not to.

1. **It's a SPA — never full-page-reload for in-app navigation.** Moving between routes swaps views client-side. The browser only does a real document load on first visit / hard refresh.
2. **Everything that can be slow loads asynchronously, and the UI stays responsive while it does.** Opening a tab/route must feel instant: render the shell immediately, then stream in code and data. If something takes a noticeable moment, show a **loading indicator** (pending bar, spinner, or skeleton) — never a frozen or blank screen.
3. **Every async boundary has three visible states: loading, success, error.** Auth-gated data adds a fourth: *unauthorized* → route the user to re-auth, don't crash.
4. **The transport layer is provider-agnostic.** Only one file knows about Clerk for API calls (`useApiClient`). Swapping auth or moving the API touches one or two files, not the whole app.
5. **Fail loud but graceful.** Misconfiguration and runtime errors produce a readable message, never a white screen.

---

## 2. Big Picture

```
                         index.html  (#root + /src/main.tsx)
                                 │
                                 ▼
   main.tsx ──► loadEnv()  ──►  <ClerkProvider>  ──►  <RouterProvider router>
   (bootstrap)   (validate         (auth/session         (client-side routing)
                  env, fail          context)
                  fast)
                                          │
                                          ▼
                                   <RootLayout/>  ── app shell: header/nav + pending bar
                                          │            + <Outlet/> for the active route
                          ┌───────────────┼───────────────┐
                          ▼               ▼               ▼
                     Landing (/)     Dashboard        SignIn / SignUp
                     [eager]         (/dashboard)     (/sign-in, /sign-up)
                                     [lazy] + guarded  [lazy]
                                          │
                                          ▼
                            useApiClient() ──► api/client.ts ──► fetch ──► Backend /me
                            (injects Clerk token)  (typed, throws ApiError)
```

**Layered responsibilities**

| Layer | Files | Responsibility |
|---|---|---|
| Bootstrap | `src/main.tsx`, `src/lib/env.ts` | Validate env, mount providers, render router. |
| Routing / shell | `src/routes/router.tsx`, `src/components/RootLayout.tsx` | Route table, code-splitting, persistent layout, pending UI. |
| Auth gating | `src/components/ProtectedRoute.tsx` | Gate routes behind Clerk; redirect signed-out users. |
| Pages | `src/pages/*` | Per-screen UI + data orchestration (loading/error states). |
| Data / transport | `src/api/client.ts`, `useApiClient.ts`, `endpoints.ts`, `types.ts` | Typed, token-injecting API access. |

---

## 3. Directory Structure

```
src/
├── main.tsx              # entry: env → ClerkProvider → RouterProvider
├── index.css            # global styles (app-shell, pending-bar, card, btn, env-error…)
├── lib/
│   └── env.ts           # loadEnv(): typed, validated env or a clear EnvError
├── routes/
│   └── router.tsx       # createBrowserRouter — route table + lazy chunks
├── components/
│   ├── RootLayout.tsx   # header/nav + <Outlet/> + route-transition pending bar
│   └── ProtectedRoute.tsx  # <SignedIn>/<SignedOut> guard + RedirectToSignIn
├── pages/
│   ├── Landing.tsx      # public "/"
│   ├── Dashboard.tsx    # protected; fetches /me with full loading/error/unauthorized states
│   ├── SignIn.tsx       # Clerk <SignIn/>
│   ├── SignUp.tsx       # Clerk <SignUp/>
│   └── NotFound.tsx     # "*" 404
└── api/
    ├── client.ts        # framework-agnostic fetch wrapper + ApiError
    ├── useApiClient.ts  # the ONLY bridge: feeds Clerk token into the client
    ├── endpoints.ts     # typed endpoint functions (getMe, …)
    └── types.ts         # shared request/response/types (User, ApiErrorBody)
```

**Folder intent:** `pages/` orchestrate, `components/` are reusable, `api/` is the data boundary, `lib/` is framework-light utilities, `routes/` is the map. Keep Clerk imports out of `api/client.ts` — that's what keeps the transport swappable (Principle 4).

---

## 4. Application Bootstrap (`main.tsx`)

Order matters and is deliberate:

1. **`loadEnv()`** reads `import.meta.env`, trims, and throws a descriptive `EnvError` if `VITE_CLERK_PUBLISHABLE_KEY` is missing. `VITE_API_BASE_URL` defaults to `http://localhost:3000`.
2. On success → mount `<ClerkProvider>` (configured with `signInUrl`, `signUpUrl`, `signInFallbackRedirectUrl="/dashboard"`, `signUpFallbackRedirectUrl="/dashboard"`, `afterSignOutUrl="/"`) wrapping `<RouterProvider>`.
3. On failure → the `catch` renders a readable **"Configuration error"** page (escaped HTML) into `#root` instead of a blank screen.

This is **fail-fast config** mirrored from the backend: a broken environment is caught at startup with a clear message.

---

## 5. Routing Architecture

- **`createBrowserRouter` + `RouterProvider`** (React Router v7, library mode — *not* framework/SSR mode).
- A single root route renders **`RootLayout`** (persistent header/nav). All pages render into its `<Outlet/>`, so the shell never unmounts between navigations (Principle 1).
- **Route table:** `/` (Landing, eager) · `/dashboard` (lazy, guarded) · `/sign-in/*` & `/sign-up/*` (lazy) · `*` (NotFound, lazy).
- **Direct-linking and deep links work** — loading `/dashboard` by URL renders correctly (SPA fallback serves `index.html`, the router resolves the route).

---

## 6. Async & Loading Strategy ★

This is the behavior the app is built around (Principles 1–3). There are **two independent async axes** — code and data — and both must stay non-blocking.

### 6a. Code loading (route-level code-splitting)
- Every route except the landing page is **lazy-imported** in `router.tsx` (`lazy: async () => import('../pages/...')`). Each page ships as its **own JS chunk**, fetched on demand the first time you visit it.
- Result: the initial bundle is small, the app shell paints fast, and opening a new tab/route pulls only that route's code.
- **The shell stays mounted** during this — you never see a full reload, just the layout with the new view filling in.

### 6b. Route-transition feedback (pending UI)
- `RootLayout` reads `useNavigation()`. While a transition is `!== 'idle'` (e.g. a lazy chunk is still downloading), it renders a **`pending-bar`** (`role="status"`, `aria-label="Loading"`) across the top.
- So even if a chunk or loader is slow, the user gets immediate visual feedback that navigation is happening — never a dead click.

### 6c. Data loading (per-screen async fetch)
- Pages fetch their own data and render an **explicit state machine**. Canonical example — `Dashboard.tsx`:
  ```ts
  type State =
    | { status: 'loading' }
    | { status: 'error'; message: string }
    | { status: 'unauthorized' }
    | { status: 'success'; user: User };
  ```
  - `loading` → show a spinner / "Loading…" copy.
  - `success` → render the data.
  - `error` → show a recoverable error card ("Couldn't reach the API").
  - `unauthorized` (HTTP 401) → prompt to sign in again (link to `/sign-in`), **not** a crash.
- The effect uses an `active` flag to ignore results after unmount (avoids setting state on a gone component).

### 6d. Spinner / skeleton policy
- **Always** represent in-flight work: pending bar for navigation, spinner/skeleton for data.
- Prefer **skeletons** for content areas (layout-stable) and **spinners** for short, indeterminate waits.
- Avoid spinner flash — for very fast responses a brief delay (~150–200ms) before showing a spinner prevents flicker. (Convention to apply as we add a shared loading component.)
- Keep the surrounding shell/nav interactive while a region loads — never block the whole screen for one widget.

### 6e. Direction of travel (not built yet, but the target)
- Introduce a shared **`<Spinner/>` / `<Skeleton/>`** component set (lands with the UI system, milestone M5).
- Consider **React Router data loaders** + `<Suspense>` / `defer` to move fetching into the route layer (parallelize code+data, render-as-you-fetch) and **route-level error boundaries** (milestone M6) so a failed load shows a scoped fallback rather than bubbling.
- Prefetch likely-next chunks/data on intent (hover/focus) once we have a component library.

---

## 7. Authentication & Session

- **`ClerkProvider`** (in `main.tsx`) holds session state app-wide; configured redirect URLs send users to `/dashboard` after auth and `/` after sign-out.
- **`ProtectedRoute`** wraps protected pages: `<SignedIn>{children}</SignedIn>` + `<SignedOut><RedirectToSignIn/></SignedOut>`. Signed-out access to `/dashboard` bounces to sign-in.
- **Nav controls** (`RootLayout`): `<SignedOut>` shows Sign in / Sign up; `<SignedIn>` shows `<UserButton/>` (account + sign-out).
- **Session persistence** is Clerk's responsibility — a refresh while signed in keeps the session (no flash back to sign-in).
- Same Clerk instance as the backend (`sound-pika-92` dev) so tokens verify server-side. See backend design / `06-milestones/backend-api.md`.

---

## 8. Data / API Layer

Three files, one clear seam:

- **`api/client.ts`** — a typed `fetch` wrapper. Imports **neither React nor Clerk**. It:
  - calls the injected `getToken()` and sets `Authorization: Bearer <token>` when present,
  - prefixes the path with `baseUrl`,
  - parses JSON, and on non-2xx **throws a typed `ApiError`** (with `status` and `isUnauthorized`),
  - extracts the backend's `{ error }` envelope for the message.
- **`api/useApiClient.ts`** — the **only** Clerk-aware file in `api/`. It pulls `getToken` from `useAuth()` and builds the client, memoized. Base URL: `/api` (Vite proxy) in dev, `VITE_API_BASE_URL` in prod.
- **`api/endpoints.ts` / `types.ts`** — typed endpoint functions (`getMe`) and shared types (`User`, `ApiErrorBody`).

**Why the split:** changing auth provider or API host is a one-file edit. Pages depend on `endpoints` + `types`, never on `fetch` or Clerk directly.

---

## 9. Environment & Configuration

| Variable | Use |
|---|---|
| `VITE_CLERK_PUBLISHABLE_KEY` | Required. Validated at startup; missing → config error page. |
| `VITE_API_BASE_URL` | Backend base URL (default `http://localhost:3000`). Used directly in prod. |

- **Dev:** `vite.config.ts` proxies `/api/*` → `http://localhost:3000`, stripping the `/api` prefix (`/api/me` → `/me`). This means **no CORS in dev** — the browser only talks to the Vite origin.
- **Prod:** the client calls `VITE_API_BASE_URL` directly; the backend must then allow the web origin via CORS (a backend concern, not yet configured).
- Keys are pulled via the Clerk CLI into gitignored `.env.local`; never commit them.

---

## 10. Error Handling & Resilience

| Failure | Behavior |
|---|---|
| Missing/invalid env | Startup catch renders a readable "Configuration error" page. |
| API non-2xx | `client.ts` throws `ApiError`; the page renders an error state. |
| 401 (expired/no session) | Page shows an "unauthorized" state → link to re-auth. No crash. |
| Unknown route | `*` route renders the 404 page. |
| Slow code/data | Pending bar (nav) + per-screen loading state (data). |

**Gaps to close (M6):** route-level error boundaries + a global fallback, and Sentry for client error reporting. Today a thrown render error isn't yet caught by a boundary.

---

## 11. Conventions for New Features

- **New page:** add a **lazy** route in `router.tsx`; render under `RootLayout`; wrap in `ProtectedRoute` if auth-gated.
- **New data call:** add a typed function in `endpoints.ts` + types in `types.ts`; call it via `useApiClient()`; render explicit loading/error(/unauthorized) states (Principle 3).
- **Never** import `fetch` or Clerk directly in a page — go through `endpoints` + `useApiClient`.
- **Always** show progress for anything async (Principle 2); keep the shell interactive.
- Keep `api/client.ts` provider-agnostic.

---

## 12. Out of Scope Today (tracked in milestones)

- **UI system / design tokens / shared Spinner & Skeleton, dark mode, a11y pass** → M5.
- **Error boundaries, Sentry, bundle analysis, deploy/SPA hosting config** → M6.
- **Vitest + RTL tests, CI** → M7.

See `06-milestones/web.md` for status.
```
