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

_Add new ADRs below as decisions are made._
