# Contributing to OrbitStream

We welcome contributions across all OrbitStream repositories. This guide covers how to get started, what kind of work is available, and how to submit a quality PR.

---

## Repositories

| Repo | Good for |
|------|---------|
| `orbitstream-contracts` | Rust developers, Soroban experience |
| `OrbitStream_backend` | TypeScript/NestJS developers |
| `orbitstream-frontend` | React/Next.js developers |
| `orbitstream-docs` | Technical writers, anyone |

---

## Issue Labels

| Label | Meaning |
|-------|---------|
| `good first issue` | Small, well-defined, great starting point |
| `bug` | Something is broken |
| `feature` | New functionality |
| `docs` | Documentation improvements |
| `contracts` | Soroban contract work |
| `backend` | NestJS API work |
| `frontend` | Next.js UI work |
| `help wanted` | Maintainers actively looking for contributors |

---

## Contribution Workflow

### 1. Find an issue

Browse open issues and comment to claim one before starting work. Issues with active PRs are locked to prevent duplicate submissions.

### 2. Fork and clone

```bash
git clone https://github.com/<your-username>/orbitstream-<repo>
cd orbitstream-<repo>
```

### 3. Create a branch

```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/issue-description
```

### 4. Make your changes

- For **contracts**: run `cargo test` — all 8 existing tests must pass
- For **backend**: run `npm test` — no regressions
- For **frontend**: run `npm run build` — must build clean
- For **docs**: ensure markdown renders correctly

### 5. Commit

Use [conventional commits](https://www.conventionalcommits.org/):

```bash
git commit -m "feat: add batch stream creation endpoint"
git commit -m "fix: claimable amount overflows on long-running streams"
git commit -m "docs: add rate conversion table to integration guide"
git commit -m "test: add edge case for paused stream cancellation"
```

### 6. Open a Pull Request

Your PR description must include:

```markdown
Fixes #<issue-number>

## What changed
-

## Why
-

## How to test
-
```

PRs without a linked issue are closed.

---

## Code standards

### Rust (contracts)
- No `unwrap()` in production paths — use `ok_or(Error::...)` 
- All public functions must have `#[cfg(test)]` coverage
- Follow the existing error enum pattern
- Add storage TTL on every write

### TypeScript (backend/frontend)
- No `any` types
- DTOs must use `class-validator` decorators
- Services own business logic — controllers are thin
- No raw SQL — use TypeORM query builder

### Documentation
- Keep examples runnable — test them before submitting
- Link to related functions/endpoints where relevant
- Update the relevant `README.md` if you change behaviour

---

## PR review turnaround

We aim to review all PRs within **48 hours**. You'll receive inline comments. Address them and push to the same branch — the PR updates automatically.

---

## Getting help

- Open a [GitHub Discussion](https://github.com/OrbitStream/orbitstream-docs/discussions)
- Comment on the issue you're working on
- Tag `@OrbitStream` in your PR if you're stuck

Thank you for contributing to OrbitStream. 🌊
