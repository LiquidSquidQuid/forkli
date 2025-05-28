# Forkli Web App

> **Visual editor & AI assistant for branching narratives, conversation flows, and decision support trees.**

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Prerequisites](#prerequisites)
3. [Quick Start (Local Dev)](#quick-start-local-dev)
4. [Environment Variables](#environment-variables)
5. [Package Scripts](#package-scripts)
6. [Project Structure](#project-structure)
7. [Feature Flags](#feature-flags)
8. [AI Prompt Library](#ai-prompt-library)
9. [Deployment (Vercel + Supabase)](#deployment-vercel--supabase)
10. [Contributing](#contributing)
11. [License](#license)

---

## Tech Stack

| Layer              | Tech                                                      | Notes                                     |
| ------------------ | --------------------------------------------------------- | ----------------------------------------- |
| **Frontend**       | Next.js 14, React 19, Tailwind CSS, shadcn/ui, React‑Flow | App Router, strict TS                     |
| **Backend (Edge)** | Next.js API routes & server actions                       | Runs on Vercel Edge / Node                |
| **Database**       | Supabase PostgreSQL 15                                    | RLS, migrations via `supabase/migrations` |
| **Functions**      | Supabase Edge (Deno + TypeScript)                         | `validate_ops()`, `conflict_detector()`   |
| **AI**             | OpenAI GPT‑4o                                             | Suggest, rewrite, conflict checks         |
| **CI/CD**          | GitHub Actions → Vercel                                   | Lint, test, preview deploy                |

---

## Prerequisites

* **Node ≥ 20** (recommended LTS)
* **pnpm** (`corepack enable && corepack prepare pnpm@latest --activate`)
* **Supabase CLI** ≥ 1.150 (`brew install supabase/tap/supabase`)
* **Vercel CLI** (optional, for local edge runtime)
* A Supabase project with database + storage buckets
* An OpenAI API key with GPT‑4 access

---

## Quick Start (Local Dev)

```bash
# 1. Clone
$ git clone git@github.com:<your‑org>/decision‑tree.git && cd decision‑tree

# 2. Install deps
$ pnpm install

# 3. Bootstrap env
$ cp .env.example .env.local   # fill in SUPABASE_URL, ANON_KEY, OPENAI_API_KEY, etc.

# 4. Start Supabase stack (Docker)
$ supabase start
$ supabase db reset           # loads schema + seed tutorial tree

# 5. Apply migrations / functions
$ supabase link --project-ref <your‑ref>
$ supabase db push            # sync local → remote
$ supabase functions deploy   # deploy edge functions

# 6. Dev server (Next.js + Supabase hot‑reload)
$ pnpm dev

# open http://localhost:3000 🚀
```

> **Tip:** First‑time run seeds a tutorial tree so you can explore the editor immediately.

---

## Environment Variables

| Key                             | Example                   | Description                                       |
| ------------------------------- | ------------------------- | ------------------------------------------------- |
| `NEXT_PUBLIC_SUPABASE_URL`      | `https://xyz.supabase.co` | Public API URL (from Supabase → Settings → API)   |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | `eyJhbGci...`             | Front‑end anon key (API → Project API keys)       |
| `SUPABASE_SERVICE_ROLE_KEY`     | `eyJhbGci...`             | **Server‑only** key for Edge routes (kept secret) |
| `OPENAI_API_KEY`                | `sk‑...`                  | GPT‑4o access token                               |
| `FEATURE_FLAGS`                 | `hipaa,analytics,export`  | Comma‑separated list of enabled flags             |
| `NODE_ENV`                      | `development`             | Standard Node env                                 |
| `VERCEL_ENV`                    | auto                      | Provided by Vercel at build/runtime               |

`.env.example` contains the full list—copy it and fill the blanks. **Never commit real secrets.**

---

## Package Scripts

| Script                     | What it Does                           |
| -------------------------- | -------------------------------------- |
| `pnpm dev`                 | Run Next.js dev server with hot reload |
| `pnpm lint`                | ESLint + TypeScript strict checks      |
| `pnpm test`                | Vitest unit + integration tests        |
| `pnpm format`              | Prettier write                         |
| `supabase start`           | Launch local Supabase Docker stack     |
| `supabase db reset`        | Recreate DB + run migrations & seeds   |
| `supabase functions serve` | Serve edge functions locally           |
| `pnpm vercel:deploy`       | Trigger Vercel production deploy       |

CI runs `lint`, `test`, and a production build for every PR.

---

## Project Structure

```
├─ app/               # Next.js (app router)
│  ├─ (site)/         # Public pages
│  └─ api/            # Edge + Node routes
├─ components/        # React UI atoms/molecules (shadcn)
├─ lib/               # Helpers, hooks (e.g., useTrees, useOpenAI)
├─ supabase/          # SQL migrations, edge functions
│  ├─ migrations/
│  └─ functions/
├─ tests/             # Vitest specs
└─ prisma.svg         # ERD (generated)
```

---

## Feature Flags

Feature toggles are **opt‑in** via `FEATURE_FLAGS` env var:

| Flag        | Purpose                                                  | Default |
| ----------- | -------------------------------------------------------- | ------- |
| `hipaa`     | Scrub PHI + block outbound GPT when on HIPAA projects    | off     |
| `analytics` | Enable `/api/events` ingestion + Supabase `events` table | off     |
| `export`    | Enable Dialogflow JSON + PNG export UI                   | on      |

See **FEATURE\_FLAGS.md** for implementation details.

---

## AI Prompt Library

See **PROMPTS.md** for bucket‑specific templates and “vibe‑coding” guidelines outlined in §10 of the project plan. Sticking to those templates keeps ChatGPT responses small, testable, and merge‑ready.

---

## Deployment (Vercel + Supabase)

1. **Fork → Vercel Import**: Vercel auto‑detects Next.js & supplies Edge runtime.
2. **Add Environment Variables**: replicate your `.env.local` values in Vercel project → Settings → Environment Variables.
3. **Supabase → Production URL**: ensure Production branches use the remote Supabase credentials.
4. **Custom Domain**: attach in Vercel; DNS propagation may take up to 24 h.

> Production deploys happen automatically on `main` push after CI passes.

---

## Contributing

Please read **CONTRIBUTING.md** for branch workflow, commit message format (`<type>: <subject> / Closes Motion-<id>`), PR checklist, and local pre‑commit hooks.

### Good First Issues

Check the [Motion Board](https://app.usemotion.com/) for tasks tagged “good‑first‑issue.”

---

## License

MIT © 2025 Prism Health Lab & Contributors
