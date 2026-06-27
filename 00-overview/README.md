# Project Northstar — Overview

A solo software company building multiple mobile and web applications, each sharing a common boilerplate stack.

## What Is This Vault

This Obsidian vault is the source of truth for all project documentation across every app under Project Northstar. It is version-controlled on GitHub so Claude can reference it as context at the start of any session.

## Folder Map

| Folder | Purpose |
|---|---|
| `01-boilerplate` | Shared backend, React Native, and React web starter — what's included, how to run |
| `02-apps` | One subfolder per product (added as each app is built) |
| `03-milestones` | Cross-project milestone tracking |
| `04-decisions` | Architecture decision records (ADRs) |
| `05-infra` | Deployment, CI/CD, Dependabot, security |

## Stack (Shared Across All Apps)

- **Backend** — Node.js, Express, PostgreSQL, Prisma ORM, Clerk (auth), Upstash Redis, express-rate-limit, Helmet, CORS
- **Mobile** — React Native (latest stable), React Navigation v6, Clerk RN SDK, native iOS (Swift) and Android (Kotlin) widget scaffolds
- **Web** — Vite + React 18, Clerk React SDK
- **Docs** — This vault, pushed to GitHub
- **Security** — Dependabot enabled on all repos
- **Deployment** — TBD (Railway or AWS)

## Active Apps

_None yet — see `02-apps/README.md` for the intake process._
