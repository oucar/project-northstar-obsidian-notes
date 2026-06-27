# Architecture Decision Records (ADRs)

Every significant technical choice that would be non-obvious to a future reader gets an ADR here. The goal is to capture *why*, not just *what*.

## ADR Template

```markdown
## ADR-NNN — Title

**Date:** YYYY-MM-DD
**Status:** Accepted | Superseded by ADR-NNN

### Context
What problem were we solving? What constraints existed?

### Decision
What did we choose?

### Consequences
What does this make easier? What does it make harder or require us to accept?
```

## Decisions

### ADR-001 — Clerk for Authentication

**Date:** 2026-06-27
**Status:** Accepted

#### Context
Need auth across backend API, React Native, and React web. Building auth from scratch is high risk and high maintenance for a solo operator.

#### Decision
Use Clerk across all surfaces. Backend validates Clerk JWTs via `@clerk/express`. Mobile uses the Clerk RN SDK. Web uses `@clerk/clerk-react`.

#### Consequences
- Auth UI, session management, MFA, and user management are handled by Clerk — zero maintenance overhead.
- Clerk is a paid dependency; pricing becomes a consideration at scale.
- Switching auth providers later would require migrating all user records.

---

### ADR-002 — Upstash Redis for Cache / Rate Limiting State

**Date:** 2026-06-27
**Status:** Accepted

#### Context
Need Redis for rate-limit counters and lightweight caching without running a self-hosted Redis instance.

#### Decision
Use Upstash Redis (serverless, HTTP-based). Works in any deployment environment including edge runtimes.

#### Consequences
- No infra to manage; generous free tier.
- HTTP overhead vs. a persistent TCP connection — acceptable for rate-limit and cache use cases.

---

### ADR-003 — Adopt Latest-Stable Major Versions (June 2026 Baseline)

**Date:** 2026-06-27
**Status:** Accepted

#### Context
The boilerplate was first specced against React 18, React Router v6, React Navigation v6, Express 4, and an unpinned Node/Prisma. By mid-2026 each had a newer stable major. Starting new apps on year-old majors means migrating under pressure later.

#### Decision
Pin the boilerplate to these latest-stable versions and update all stack docs:

| Tech | Was | Now (June 2026) |
|---|---|---|
| Node.js | LTS (unpinned) | **24 LTS** (Active) |
| Express | 4 | **5.2** |
| Prisma | unpinned | **7** |
| React (web) | 18 | **19** |
| Vite | — | **8** |
| React Router | v6 (`react-router-dom`) | **v7** (`react-router`) |
| React Navigation | v6 | **v7** |

#### Consequences
- **Breaking changes to absorb up front:**
  - React Router v6 → v7: package renamed `react-router-dom` → `react-router`; update all imports.
  - Express 4 → 5: async handlers that throw now forward to error middleware automatically; a few removed/renamed APIs — audit the middleware chain.
  - Prisma 6 → 7: TypeScript-based runtime (Rust engine dropped); smaller bundles, regenerate the client.
  - React Navigation v6 → v7: new static API available; existing dynamic API still supported.
- **Not adopted yet:** React Router **v8** and React Navigation **v8** exist but are skipped — v8 of React Router is a non-breaking upgrade to take when convenient; React Navigation v8 is still pre-release. Revisit in a future ADR.
- Dependabot keeps minors/patches current; majors remain a deliberate, ADR-tracked decision.

---

_Add new ADRs below as decisions are made._
