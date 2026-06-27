# Infrastructure

## Deployment

**Status:** TBD — evaluating Railway vs. AWS.

| Option | Pros | Cons |
|---|---|---|
| Railway | Zero-config deploys, Postgres + Redis add-ons, cheap for solo | Less control, harder to scale past mid-tier |
| AWS (ECS + RDS) | Full control, production-grade | Significant ops overhead for solo operator |

Decision will be recorded in `04-decisions/README.md` as ADR-003 once made.

## CI/CD

- GitHub Actions for all repos
- On PR: lint, type-check, tests
- On merge to `main`: deploy to staging (TBD)
- On release tag: deploy to production (TBD)

## Security

### Dependabot
Enabled on all repos. Config:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

### Security Headers
All API responses include Helmet defaults: `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`.

### Rate Limiting
Global: 100 requests / 15 min per IP.
Auth routes: 10 requests / 15 min per IP.

### Secrets Management
- Local dev: `.env` files (never committed)
- Production: environment variables injected by the deployment platform
- `.env.example` committed to every repo documenting required keys

## Monitoring

_TBD — likely Sentry for error tracking, platform-native metrics._
