# Boilerplate — React Native App

## Stack

| Layer | Choice |
|---|---|
| Framework | React Native (latest stable) |
| Navigation | React Navigation v7 |
| Auth | Clerk RN SDK (`@clerk/clerk-expo` or `@clerk/react-native`) |
| API client | Typed fetch wrapper pointing to boilerplate API |
| iOS widgets | Swift — `ios/ProjectNameWidgets/` extension folder |
| Android widgets | Kotlin — `android/app/src/main/kotlin/.../widgets/` |
| Language | TypeScript |

## What's Included Out of the Box

- `App.tsx` — ClerkProvider wrapping NavigationContainer
- `navigation/` — Root stack with unauthenticated (Sign In / Sign Up) and authenticated (Home tab) stacks; gated by Clerk `useAuth`
- `screens/SignIn.tsx`, `screens/SignUp.tsx` — Clerk-powered auth screens
- `screens/Home.tsx` — placeholder authenticated screen
- `api/client.ts` — typed API client with Clerk session token injection
- `ios/Widgets/` — Swift widget extension scaffold (WidgetKit)
- `android/widgets/` — Kotlin Glance widget scaffold

## Required Environment Variables

```
EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY=   # or set in app.config.ts
API_BASE_URL=http://localhost:3000
```

## How to Run Locally

```bash
cd apps/mobile
npm install
npx pod-install          # iOS only
npm run ios              # or: npm run android
```

Clerk requires a real device or simulator with network access to your API.
