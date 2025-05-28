# Contributing to **Forkli**

Welcome—whether you are Abhi, a teammate, or an open‑source newcomer, this guide explains **how to collaborate effectively, keep code quality high, and work smoothly with Copilot & ChatGPT**.

*Last updated 2025‑05‑28*  |  *Maintainer: @abhi.sarup*

---

## 1  Branch Workflow

| Branch                    | Purpose                                             | Protection Rules                 |
| ------------------------- | --------------------------------------------------- | -------------------------------- |
| **`main`**                | Production; auto‑deploy to Vercel                   | • PR only • CI pass • 1 review   |
| **`dev`**                 | Integration; nightly preview                        | • PR or direct push by core team |
| **`bucket/<A‑H>-<slug>`** | Feature branches mapped to Modular Workflow buckets | —                                |
| **`hotfix/<issue>`**      | Urgent production fixes                             | —                                |

### Creating a Feature Branch

```
# Example: bucket C task "canvas‑core"
git checkout -b bucket/C-canvas-core
```

---

## 2  Commit Message Convention *(Conventional Commits + Motion hook)*

```
<type>(scope): <subject>  |  Closes Motion-<id>
```

* **type** = feat, fix, chore, docs, refactor, test, perf
* **scope** = short subsystem (canvas, api, db, ci)
* **subject** = imperative, ≤ 72 chars

> Example

```
feat(canvas): add MiniMap component  |  Closes Motion-42
```

The footer auto‑closes the corresponding task in Motion when the PR merges.

---

## 3  Pull‑Request Checklist

1. **Branch is up‑to‑date** with `dev`.
2. **All checks green**: `lint`, `type‑check`, `test`, `build`.
3. **Vitest coverage ≥ 80 %** on new/changed files.
4. **Screenshots / GIF** for UI changes.
5. **Links to Motion task** in description.
6. For buckets **E–F** (AI & export): attach sample API payloads.
7. Request review from one core maintainer (`@abhi.sarup` default).

> PR title must mirror the **commit subject**.

---

## 4  Code Standards & Tooling

* **TypeScript strict** (`noImplicitAny`, `strictNullChecks` true).
* **Tailwind via CVA** for variants.
* **≤ 100 LOC per function** (AI context‑friendly).
* **Prettier** auto‑format on commit.
* **ESLint** rules: airbnb‑type‑script + custom project rules.
* **Husky** pre‑commit: `lint:staged` → `test:changed` → `format`.

### Testing

* Unit tests in `/tests/**/*.test.ts` (Vitest).
* Integration tests for `/app/api/*` using **supertest**.
* **`pnpm test:changed`** runs only impacted tests.

### Coverage Thresholds (Vitest)

* **Lines** ≥ 80 %  global, **Branches** ≥ 70 %.
* Fail PR if thresholds drop.

---

## 5  AI Pair‑Programming (“Vibe‑Coding”) Rules

1. **Prompt Template** — use §10.3 frame from project plan; include component, bucket, goal.
2. **Interface‑First** — paste TypeScript interfaces before asking for implementation.
3. **Ask for tests** in same ChatGPT request.
4. **Review diff** with GitLens before save; reject Copilot code that violates lint or style.
5. **Tag commits** that contain large model‑generated code with `[ai‑gen]` in the body.

See **PROMPTS.md** for ready‑made examples.

---

## 6  Local Setup & Dev Scripts

```bash
pnpm install               # install deps
cp .env.example .env.local # add secrets
supabase start             # launch local DB
pnpm dev                   # run Next.js
```

For full instructions refer to **README.md → Quick Start**.

---

## 7  Feature Flags Etiquette

* Flags live in `.env*` → `FEATURE_FLAGS` (comma list).
* Never remove a feature flag without first deleting dead code behind it.
* When adding a new flag, document it in **FEATURE\_FLAGS.md** and provide migration steps.

---

## 8  Issue Labels

| Label              | Meaning                                           |
| ------------------ | ------------------------------------------------- |
| `good‑first‑issue` | Simple, isolated tasks ideal for new contributors |
| `ai‑prompt‑needed` | Issue requires fresh GPT prompt design            |
| `tech‑debt`        | Refactor or cleanup work                          |
| `blocked`          | Waiting on external dependency                    |

---

## 9  Code of Conduct

We follow the [Contributor Covenant v2.1](https://www.contributor-covenant.org/). Be kind, inclusive, and respectful.

---

## 10  Releasing

1. Bump version in `package.json` (`npm version <patch|minor|major>`).
2. Tag release: `git tag vX.Y.Z`.
3. GitHub Action builds & publishes changelog via **Release‑Please**.
4. Vercel promotes latest build to production.

---

Thank you for helping build a powerful, AI‑driven decision‑tree platform! 🙌
