# Milestones — React Native (`apps/mobile`)

Stack reference: [[react-native]] (boilerplate). React Native (latest stable) · React Navigation v7 · Clerk RN SDK · iOS Swift widget · Android Kotlin widget · TypeScript.

Build order is top-down. Each milestone lists **Scope** and **Acceptance**. Tick acceptance boxes only after running the check on a simulator/device. The API milestone (M4) depends on the Backend API reaching its M3 (auth).

---

## M1 — Scaffold & Tooling `[ ]`

**Goal:** A React Native + TS app that runs on both iOS and Android.

**Scope**
- [x] React Native project initialized with TypeScript
- [x] ESLint + Prettier; `lint` and `format` scripts
- [x] Folder structure: `src/{screens,navigation,api,components,lib}`
- [x] Env config (`.env` / `app.config.ts`) for Clerk key + API base URL
- [ ] Metro + native builds working — Metro bundle verified; native build (prebuild/EAS) not yet run

**Acceptance**
- [ ] `npm run ios` boots the app on an iOS simulator with no red screen
- [ ] `npm run android` boots the app on an Android emulator with no crash
- [ ] `npm run lint` passes with zero errors
- [ ] Editing a screen hot-reloads on both platforms

---

## M2 — Navigation `[ ]`

**Goal:** React Navigation v7 with auth and app stacks.

**Scope**
- [x] React Navigation v7 + `NavigationContainer` in `App.tsx`
- [x] Root navigator switching between an **Auth** stack and an **App** stack
- [x] App stack with a bottom tab navigator (Home + one more tab)
- [x] Typed navigation params
- [x] Placeholder screens for each route

**Acceptance**
- [ ] App launches into the Auth stack by default
- [ ] Navigating between tabs in the App stack works
- [ ] Back navigation behaves correctly (hardware back on Android included)
- [ ] Switching the auth flag flips the whole app between Auth and App stacks

---

## M3 — Auth (Clerk) `[ ]`

**Goal:** Clerk-gated navigation with sign in / sign up and persisted session.

**Scope**
- [x] `ClerkProvider` wrapping `NavigationContainer` with a secure token cache
- [x] `screens/SignIn.tsx`, `screens/SignUp.tsx` (Clerk RN flows, incl. email verification)
- [x] Root navigator gated by Clerk `useAuth` (`isSignedIn`)
- [x] Sign-out action in the App stack
- [x] Token cache backed by secure storage (Keychain / Keystore)

**Acceptance**
- [ ] Signed-out users see only the Auth stack
- [ ] Completing sign-up/sign-in switches to the App stack
- [ ] Killing and reopening the app keeps the session (token cache works)
- [ ] Sign-out returns to the Auth stack and clears the session

---

## M4 — API Integration `[ ]`

**Goal:** Typed API client injecting the Clerk token for a real authenticated call. **Depends on Backend API M3.**

**Scope**
- [ ] `api/client.ts` typed fetch wrapper injecting the Clerk session token
- [ ] Base URL from env (works with a device → local API, e.g. LAN IP or tunnel)
- [ ] Home screen fetches `GET /me` and renders the user
- [ ] Loading + error states
- [ ] `401` handling → sign-in

**Acceptance**
- [ ] Home screen shows data fetched from the backend `/me` while signed in
- [ ] The request carries `Authorization: Bearer <token>` (verify via API logs / proxy)
- [ ] Loading indicator shows before data; error UI shows if the API is unreachable
- [ ] Expired/invalid token routes the user back to sign-in (no crash)

---

## M5 — Native Widgets `[ ]`

**Goal:** Home-screen widget scaffolds on both platforms showing app data.

**Scope**
- [ ] iOS: Swift WidgetKit extension in `ios/Widgets/` (target added to Xcode project)
- [ ] Android: Kotlin Glance widget in `android/widgets/`
- [ ] Shared data bridge: app writes a value (App Group / shared prefs) the widget reads
- [ ] Small + medium widget sizes
- [ ] Documented how to add/build the widget targets

**Acceptance**
- [ ] iOS widget can be added to the home screen and renders app-provided data
- [ ] Android widget can be added to the home screen and renders app-provided data
- [ ] Updating the value in the app updates the widget (after refresh)
- [ ] Both widget targets build as part of the normal release build

---

## M6 — Production Readiness `[ ]`

**Goal:** Release-buildable app with error handling and store assets. (See skill: `production-ready`.)

**Scope**
- [ ] Error boundary + global JS error handling
- [ ] Sentry (RN SDK) capturing JS + native crashes
- [ ] App icon, splash screen, app name configured for both platforms
- [ ] Build config (EAS or native release config); env per build profile
- [ ] Release build notes in `05-infra`

**Acceptance**
- [ ] A thrown render error shows a fallback screen and reports to Sentry
- [ ] A release (not debug) build runs on a device on both platforms
- [ ] App icon + splash + name appear correctly on device
- [ ] Production build points at the production API base URL (not localhost)

---

## M7 — Testing & CI `[ ]`

**Goal:** Unit + E2E tests gated in CI. (See skill: `testing`.)

**Scope**
- [ ] Jest + React Native Testing Library set up
- [ ] Unit tests: a component, the API client (mocked) token injection, navigation gating
- [ ] Maestro E2E flow: sign-in → see Home data
- [ ] `test` and `test:watch` scripts
- [ ] GitHub Actions: install → lint → test (+ Maestro if runnable in CI)

**Acceptance**
- [ ] `npm test` runs green locally
- [ ] A test verifies signed-out users only reach the Auth stack
- [ ] The Maestro flow passes locally against a simulator
- [ ] CI workflow passes on a pushed branch and fails when a test is deliberately broken
