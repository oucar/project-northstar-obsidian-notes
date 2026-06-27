# Boilerplate Milestones

Per-app milestone roadmaps for building the shared boilerplate from empty scaffold → production-ready starter. Each app is built **one milestone at a time** so a single Claude Code session (one per terminal) can own an app, pick the next unchecked milestone, implement it, and verify it against concrete acceptance criteria before moving on.

These cover the **boilerplate** only — not any specific product. Product features get their own milestones under `02-apps/<product>/` once a product is defined.

## Roadmaps

| App | File | Milestones |
|---|---|---|
| Backend API (`apps/uapi`) | [[backend-api]] | M1–M7 |
| React Web (`apps/web`) | [[web]] | M1–M7 |
| React Native (`apps/mobile`) | [[mobile]] | M1–M7 |

## How to Use (per terminal)

1. Open the app's roadmap file.
2. Find the lowest-numbered milestone that is not `[x]`.
3. Implement only that milestone's scope — don't pull work forward.
4. Tick each **acceptance criterion** only when you've actually run the check and seen it pass.
5. Mark the milestone `[x]` when every acceptance criterion passes, then commit.
6. Move to the next milestone.

A milestone is **done** only when all of its acceptance criteria are checked. Acceptance criteria are written to be runnable/observable (a command, an HTTP response, a visible screen) — not subjective.

## Status Key

| Symbol | Meaning |
|---|---|
| `[ ]` | Not started |
| `[~]` | In progress |
| `[x]` | Done |
| `[!]` | Blocked |

## Milestone Theme (shared across all three apps)

| # | Theme |
|---|---|
| M1 | Scaffold & tooling |
| M2 | Core structure (DB / routing / navigation) |
| M3 | Auth (Clerk) |
| M4 | API integration |
| M5 | Hardening / UI system / native widgets |
| M6 | Production readiness |
| M7 | Testing & CI |

Building the same theme order across all three keeps the three terminals roughly in sync and makes the boilerplate coherent end-to-end (e.g. all three reach "Auth works" before any reaches "API integration").

## Progress Summary

| App | M1 | M2 | M3 | M4 | M5 | M6 | M7 |
|---|---|---|---|---|---|---|---|
| Backend API | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Web | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Mobile | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` |

_Update this grid as milestones complete._
