# Forkli Web App – Project Requirements Document

## 1. Vision & Scope

* Empower creators to design branching narratives, conversation FAQs, and complex decision flows through an intuitive, visual editor.
* Provide AI‑assisted guidance and content generation via OpenAI GPT‑4o (chat completions).
* Lightweight, feature‑rich UX: built‑in navigation aids, conflict checking, and fast export paths.
* Seamless deployment and onboarding; MVP in < 5 days of focused development.

## 2. Core Use Cases

1. **Narrative Architect** – Authors map large‑arc interactive storylines (games, choose‑your‑own‑adventure).
2. **FAQ Conversational Designer** – Ops teams convert static FAQs into dynamic conversation trees (chatbot intents).
3. **How‑To Coach** – Built‑in assistant explains features, suggests next steps, and drafts node content.
4. **Customer Journey Mapper** – Growth & CX teams visualize onboarding, retention, and upsell flows ready for marketing automation hand‑off.
5. **Skill‑Tree Designer** – Game and ed‑tech creators lay out abilities, prerequisites, and unlock conditions for players/learners.
6. **Clinical Pathway Builder** – Healthcare teams encode diagnostic or treatment protocols, leveraging the HIPAA guardrail and conflict detector.

## 3. Functional Requirements

* CRUD for Trees, Nodes, Edges.
* Drag‑and‑drop canvas w/ zoom, pan, **mini‑map**, and **breadcrumb path**.
* Conditional branches & metadata (probabilities, labels, **tags\[]**).
* Version history & autosave.
* Re‑usable node templates & snippets.
* Import/Export JSON & PNG snapshot **plus one‑click export to Dialogflow‑style JSON chatbot**.
* **Shareable links with view or edit mode** (query‑param‑driven).
* **Pre‑loaded tutorial tree** seeded on first login.
* GPT Assist endpoints:

  * “Suggest next branch”
  * “Rewrite node copy in tone X”
  * **“Conflict detector”** – flags unreachable nodes/loops.
* **HIPAA feature flag** to block PHI from being sent to GPT when enabled.
* User auth (email + OAuth), team projects, role‑based permissions.

## 4. Non‑Functional Requirements

* Responsive (desktop/tablet/phone).
* Real‑time collaboration (post‑MVP) via Supabase Realtime.
* P95 canvas render < 100 ms for 500 nodes.
* Basic usage analytics (events table) behind feature flag.
* OWASP top‑10 compliance.

## 5. System Architecture

Below is a layered breakdown showing *what* lives *where* and *why*—so each responsibility is explicit and testable.

```
┌─────────────────────────┐
│ 1. Client (Next.js 14)  │
└─────────────────────────┘
        │  UI Layer (Tailwind + shadcn/ui + React‑Flow)
        │     • Canvas        → node/edge render, drag‑drop, MiniMap, Breadcrumb
        │     • Toolbar       → add node, templates, undo/redo, search
        │     • Sidebar       → node inspector, tag editor, metadata JSON
        │     • Modals/Toasts → share link, export, settings
        │
        │  State Layer (Zustand)
        │     • Normalised graph + UI prefs
        │     • Undo stack & optimistic updates
        │     • Selectors for filters (tags, orphan nodes)
        │
        │  Service Hooks (SWR / fetch)
        │     • useTrees(), useOpenAI(), useAnalytics()
        ▼
┌───────────────────────────────┐
│ 2. Server (Next.js API/EZ‑Actions) │
└───────────────────────────────┘
        │  /api/trees   → REST RPC for CRUD; validates via Supabase JWT
        │  /api/openai  → suggest, rewrite, conflict‑detector; HIPAA guard
        │  /api/export  → build Dialogflow JSON, queue PNG snapshot
        │  /api/events  → ingest analytics (feature‑flagged)
        │  Middleware   → auth check + rate‑limit + feature‑flags
        ▼
┌─────────────────────────────────────┐
│ 3. Supabase (Managed Platform)      │
└─────────────────────────────────────┘
        │  PostgreSQL
        │     • trees, nodes, edges, users, events (JSON payload)
        │     • RLS policies: owner/editor/viewer, project scope
        │
        │  Edge Functions (TypeScript on Deno)
        │     • validate_ops()   → enforce graph size quota, auth
        │     • conflict_detector() → recursive CTE to mark unreachable loops
        │
        │  Storage Buckets
        │     • exports/  → JSON + PNG; public‑read via signed URLs
        │
        │  Realtime (post‑MVP)
        │     • channel <tree‑id> broadcasts node/edge deltas
        ▼
┌──────────────────────────┐
│ 4. DevOps & Deployment   │
└──────────────────────────┘
        • GitHub Actions: lint, test, Vercel preview deploy
        • Vercel prod: immutable builds, edge network caching
        • Secrets: OPENAI_API_KEY, SUPABASE_URL/ANON_KEY, FEATURE_FLAGS
        • Monitoring: Vercel Web Analytics, Supabase Logs, Sentry SDK
```

**Key Responsibilities by Layer**

| Layer          | Responsible For                                      | Tech                                      | Owner  | Extensibility Hooks                   |
| -------------- | ---------------------------------------------------- | ----------------------------------------- | ------ | ------------------------------------- |
| Client UI      | Rendering, interaction patterns, visual polish       | React 19, Tailwind, shadcn/ui, React‑Flow | FE     | Component props + context injection   |
| Client State   | In‑memory graph and UI state, offline queue          | Zustand, Immer                            | FE     | Persist middleware, plugin middleware |
| API Routes     | Auth, rate‑limit, OpenAI orchestration, export logic | Next.js Server Actions (Edge & Node)      | FE/BE  | Add route handlers, edge‑runtime flag |
| Edge Functions | Heavy validation, graph traversal, cost control      | Supabase Edge (Deno)                      | BE     | Versioned via `supabase/migrations`   |
| Database       | Source of truth, RLS, migrations                     | Postgres 15                               | BE     | SQL migrations, pgmeta                |
| Storage        | Binary assets (PNG, JSON)                            | Supabase Storage                          | BE     | Lifecycle rules                       |
| Realtime       | Broadcast graph edits (future)                       | Supabase Realtime                         | BE     | Presence hooks                        |
| DevOps         | CI, CD, observability, secrets                       | Vercel, GitHub Actions, Sentry            | DevOps | Environment promotion gates           |

This granular map shows exactly **which subsystem owns which concern**—so bugs have one home and new features a clear insertion point.

### 5.1 Component Responsibility Matrix

A fine‑grained map of **which runtime unit owns which feature**. Each row is an atomic component or service that can be developed and tested in isolation.

| Component / Service                                | Runtime                 | Primary Responsibilities                                                                                                            | Key Dependencies                  |
| -------------------------------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **Canvas** (`<FlowCanvas />`)                      | Browser (React)         | Render nodes & edges, drag‑drop, zoom/pan, Mini‑Map, breadcrumb path; emit CRUD intents                                             | React‑Flow, Zustand store         |
| **Node Inspector** (`<NodeSidebar />`)             | Browser                 | Display & edit node body, tags\[], metadata JSON                                                                                    | Zustand selectors                 |
| **Toolbar** (`<CanvasToolbar />`)                  | Browser                 | Add/duplicate/delete nodes, undo/redo, search/filter by tag, orphan‑node scan                                                       | Canvas actions context            |
| **Auth Pages** (`/auth/*`)                         | Browser + Next.js Route | Sign‑in, sign‑up, magic‑link, OAuth, session refresh                                                                                | Supabase Auth JS SDK              |
| **Share Modal** (`<ShareDialog />`)                | Browser                 | Generate view/edit link with role hint; copy to clipboard                                                                           | Supabase JWT verifier             |
| **Export Modal** (`<ExportDialog />`)              | Browser                 | Trigger JSON/PNG/Dialogflow export jobs; show progress & downloads                                                                  | `/api/export`                     |
| **useTrees() Hook**                                | Browser                 | Fetch & mutate trees via SWR; optimistic updates                                                                                    | `/api/trees`                      |
| **/api/trees Route**                               | Next.js Edge            | Validate JWT, forward CRUD to Supabase via service key; enforce RLS                                                                 | Supabase client, validate\_ops()  |
| **/api/openai Route**                              | Next.js Edge            | Merge global + **per‑account feature flags** via `flagResolver`, orchestrate GPT calls → suggest, rewrite; HIPAA guard & rate‑limit | OpenAI SDK, edge KV cache         |
| **/api/conflict Route**                            | Next.js Edge            | Proxy to conflict\_detector() edge function; return unreachable/loop nodes                                                          | Supabase Edge invoke              |
| **/api/export Route**                              | Next.js Node            | Collate tree→Dialogflow JSON, persist to Storage, queue PNG snapshot                                                                | Supabase Storage SDK, `puppeteer` |
| **validate\_ops() Edge Function**                  | Supabase Deno           | Enforce graph size quota, role access; returns 403/413 on violation                                                                 | DB, JWT                           |
| **conflict\_detector() Edge Function**             | Supabase Deno           | Recursive SQL CTE to flag unreachable nodes or infinite loops                                                                       | DB                                |
| **Events Table & Tracker (**\`\`**)**              | Next.js Edge            | Feature‑flagged endpoint to log usage metrics                                                                                       | Supabase insert                   |
| **HIPAA Guard Middleware**                         | Next.js Edge            | Reject or redact PHI‑flagged payloads when `HIPAA=true`                                                                             | openai‑db cache, regex filters    |
| **Realtime Channel** (`channel-treeId`) *post‑MVP* | Supabase Realtime WS    | Broadcast node/edge deltas, presence                                                                                                | Supabase JS WS                    |

### 5.2 Use‑Case Traceability Matrix

Every requirement is now traceable to an owning component, ensuring no orphaned work.

| Use Case                    | Core Features                                    | Owning Components/Services                      |
| --------------------------- | ------------------------------------------------ | ----------------------------------------------- |
| Narrative Architect         | Canvas CRUD, Node Templates, Suggest Next Branch | Canvas, Node Inspector, `/api/openai` (suggest) |
| FAQ Conversational Designer | Tags\[], Dialogflow Export, Share‑Link           | Node Inspector, `/api/export`, Share Modal      |
| How‑To Coach                | Tutorial Tree Seed, Rewrite Node Copy            | DB seed script, `/api/openai` (rewrite)         |
| Customer Journey Mapper     | Tag‑filtered views, Analytics Events             | Toolbar filter, Events Tracker                  |
| Skill‑Tree Designer         | Prerequisite metadata, Conflict Detector         | Node Inspector (metadata), `/api/conflict`      |
| Clinical Pathway Builder    | HIPAA Guard, Audit Events                        | HIPAA Middleware, Events Table                  |

> **Auditability:** every arrow in the earlier architecture diagram now maps to rows above—making scope explicit and test coverage targetable.

---

## 6. Data Model (initial)

| Table                               | Key              | Fields                                                                            |
| ----------------------------------- | ---------------- | --------------------------------------------------------------------------------- |
| **trees**                           | id (uuid, PK)    | name, description, created\_by, created\_at                                       |
| **nodes**                           | id (uuid, PK)    | tree\_id (FK), label, body, type, **tags text\[]**, metadata (jsonb), created\_at |
| **edges**                           | id (uuid, PK)    | tree\_id, from\_node\_id, to\_node\_id, condition, metadata                       |
| **events** (optional)               | id (uuid, PK)    | user\_id, tree\_id, event\_type, payload jsonb, created\_at                       |
| **users**                           | id               | email, name, **feature\_flags text\[]**, ...                                      |
| **user\_feature\_flags** (optional) | (user\_id, flag) | maps many‑to‑many feature overrides                                               |
| **projects** (optional)             | id               | owner\_id, name                                                                   |

Indexes: `(tree_id, created_at)` on nodes/edges; `(tree_id, event_type)` on events.

## 7. OpenAI Integration

* Environment var `OPENAI_API_KEY` in Vercel + Supabase edge.
* `POST /api/openai/suggest` receives `{nodeContext, mode}` returns suggestions.
* `POST /api/openai/conflict` receives `{treeId}` returns list of unreachable/looping nodes.
* Rate‑limit per user (supabase.rate\_limit).
* **HIPAA flag check** in middleware before GPT calls.

## 8. Modular Development Workflow

A workflow optimized for **Motion** task‑scheduler import: each bucket is self‑contained, sized for 1–4 h blocks, and lists its upstream dependencies so Motion can auto‑sequence work.

| Bucket ID | Name                        | Core Deliverables                                                                                                                      | Upstream Dependency | Est. Effort | Blockers / Risks                            |
| --------- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ----------- | ------------------------------------------- |
| **A**     | Environment & Repo Setup    | • Next.js 14 TS repo scaffold• Tailwind + shadcn/ui + React‑Flow install• Supabase CLI init & link• Prettier + ESLint + Husky hooks    | –                   | 3 h         | Supabase credentials access                 |
| **B**     | Database & Auth Foundation  | • SQL migration: `trees`, `nodes`, `edges`, `users`• RLS policies draft• Supabase Auth pages (email/OAuth)• `/api/trees` CRUD scaffold | A                   | 4 h         | Correct RLS policy syntax; migrate rollback |
| **C**     | Canvas Core UI              | • `<FlowCanvas/>` w/ pan/zoom• Node & edge render• Mini‑Map + breadcrumb• Zustand store + undo stack                                   | A                   | 6 h         | React‑Flow typings; large graph perf        |
| **D**     | Edge Functions & Validation | • `validate_ops()` (size quota)• `conflict_detector()` CTE• `/api/conflict` proxy route                                                | B                   | 4 h         | Recursive SQL tuning                        |
| **E**     | AI Assist                   | • `/api/openai` suggest & rewrite• HIPAA guard middleware• UI hooks (`useOpenAI`)                                                      | B, D                | 5 h         | API quota; PHI regex false positives        |
| **F**     | Export & Share              | • `/api/export` JSON + PNG• Dialogflow mapper• Share‑link modal w/ role hint                                                           | C                   | 4 h         | Puppeteer headless issues on Vercel         |
| **G**     | Analytics & Events (Flag)   | • `events` table + RLS• `/api/events` ingest• Toolbar tag filter emits events                                                          | B                   | 3 h         | Feature flag plumbing                       |
| **H**     | Polish & Deployment         | • Shadcn empty states, error boundaries• Vercel prod deploy + DNS• Smoke tests & vitest CI                                             | A‑G                 | 4 h         | Domain DNS propagation                      |

> **Parallelization Guide:** A ↦ B/C kickoff day‑0.  C & D can run concurrently after B; E builds on D. F starts once C is stable. G independent post‑B. All converge into H.

### 8.1 Branch Strategy

* `main` (protected) – production.
* `dev` – integration.
* Feature branches: `bucket/<A‑H>‑short‑slug` (e.g., `bucket/C‑canvas‑core`).

### 8.2 CI/CD & Quality Gates

1. **GitHub Actions**

   * Lint → Typecheck → Unit tests → Vercel Preview.
2. **Preview URLs** auto‑posted to PR for QA.
3. **Motion Hook**: each merged PR auto‑closes its Motion task via commit footer `Closes Motion‑<id>`.

---

## 9. Task Export for Motion

Below is a ready‑to‑copy CSV (Motion supports pasted CSV) — each row becomes a scheduled task.

```
Task,Bucket,Estimate (h),Depends On
“Scaffold Next.js repo with TS”,A,1,
“Install Tailwind + shadcn/ui + React‑Flow”,A,1,Scaffold Next.js repo with TS
“Init Supabase & link project”,A,1,Install Tailwind + shadcn/ui + React‑Flow
“Create DB schema SQL + RLS”,B,2,Init Supabase & link project
“Implement Supabase Auth pages”,B,2,Create DB schema SQL + RLS
“/api/trees CRUD route”,B,1,Implement Supabase Auth pages
“Build FlowCanvas w/ pan+zoom”,C,2,Scaffold Next.js repo with TS
“Add Mini‑Map & breadcrumb”,C,1,Build FlowCanvas w/ pan+zoom
“Set up Zustand graph store + undo”,C,3,Add Mini‑Map & breadcrumb
“validate_ops() edge function”,D,1,Create DB schema SQL + RLS
“conflict_detector() SQL CTE”,D,2,validate_ops() edge function
“/api/conflict proxy route”,D,1,conflict_detector() SQL CTE
“/api/openai suggest+rewrite”,E,2,/api/conflict proxy route
“HIPAA guard middleware”,E,1,/api/openai suggest+rewrite
“useOpenAI React hook”,E,2, HIPAA guard middleware
“/api/export JSON+PNG”,F,2,Add Mini‑Map & breadcrumb
“Dialogflow JSON mapper”,F,1,/api/export JSON+PNG
“Share‑link modal”,F,1,Dialogflow JSON mapper
“Create events table + RLS”,G,1,Create DB schema SQL + RLS
“/api/events ingest”,G,1,Create events table + RLS
“Emit events from Toolbar”,G,1,/api/events ingest
“UI polish & empty states”,H,2,useOpenAI React hook
“Smoke tests & vitest CI”,H,1,UI polish & empty states
“Vercel prod deploy + DNS”,H,1,Smoke tests & vitest CI
```

You can paste this block directly into Motion; it will recognize `Depends On` for sequencing and `Estimate` for timeboxing.

---

## 10. Coding Standards & AI Pair‑Programming “Vibe‑Coding” Rules

### 10.1 Baseline Code Conventions

* **Language**: TypeScript strict
* **Style**: Tailwind classes via class‑variance‑authority
* **Patterns**: single‑responsibility components, hooks per feature
* **Tests**: unit for utils, integration for API routes (supertest)
* **Commit Msg Footer**: `Closes Motion‑<id>` to auto‑close tasks

### 10.2 AI Pair‑Programming Guidelines (Copilot + ChatGPT)

| Guideline                              | Why                                                         | VS Code Tip                                 |           |                  |
| -------------------------------------- | ----------------------------------------------------------- | ------------------------------------------- | --------- | ---------------- |
| **≤ 100 LOC per function**             | Fits in GPT context; easy snapshot for ChatGPT              | Use *Extract to function* refactor shortcut |           |                  |
| **Interface‑First Prompts**            | Provide model with type shapes, then ask for implementation | Copy/paste relevant `interface` blocks      |           |                  |
| **“Component • Bucket • Goal” header** | Primes ChatGPT with precise scope                           | Start prompt with \`Component: FlowCanvas   | Bucket: C | Goal: add zoom\` |
| **Ask for tests in same reply**        | Keeps Copilot suggestions aligned with coverage             | Append “generate vitest spec” to prompt     |           |                  |
| **Review diff before save**            | Prevents Copilot drift; enforces conventions                | Enable *GitLens diff on save*               |           |                  |

### 10.3 Prompt Templates by Workflow Bucket

> Copy block → paste into ChatGPT → fill 🔷 placeholders → run.

```md
### General Prompt Frame
Role: You are an expert <tech>. ✨Stay within the component’s directory.✨
Context:
- Component: 🔷 <name>
- Bucket: 🔷 <A‑H>
- Goal: 🔷 <one‑sentence objective>
Instructions:
1. Show updated TypeScript code only.
2. Keep functions ≤ 100 LOC.
3. Also output matching vitest spec.
```

| Bucket | Template Goal Line      | Example One‑Sentence Goal                                |
| ------ | ----------------------- | -------------------------------------------------------- |
| **A**  | *bootstrap environment* | “Set up Tailwind with shadcn/ui plugin enabled”          |
| **B**  | *database & auth*       | “Create RLS policy for `trees` so only owner can UPDATE” |
| **C**  | *canvas UI*             | “Add Mini‑Map component with React‑Flow built‑in”        |
| **D**  | *edge validation*       | “Write recursive CTE to detect unreachable nodes”        |
| **E**  | *AI assist*             | “Implement `/api/openai` route with HIPAA guard”         |
| **F**  | *export & share*        | “Generate Dialogflow‑compatible JSON exporter”           |
| **G**  | *analytics*             | “Insert event row on every tree SAVE action”             |
| **H**  | *polish & deploy*       | “Add shadcn EmptyState for empty tree list”              |

Add the *one‑sentence goal* to the “Goal:” placeholder in the general frame. ChatGPT returns implementation + tests ready to paste.

---

## 11. Documentation Deliverables

* **README.md** – local setup, scripts, environment variables
* **ARCHITECTURE.md** – architecture diagram, data model ERD, sequence diagrams
* **CONTRIBUTING.md** – branch strategy, commit message rules, PR checklist, test coverage thresholds
* **PROMPTS.md** – curated catalogue of GPT prompt templates and examples per bucket (updates alongside §10 “Vibe‑Coding” rules)
* **FEATURE\_FLAGS.md** – docs for HIPAA, analytics, export toggles and how to enable/disable
* **User Guide** – Notion space with quick‑start tutorial and FAQ (post‑deploy)

## 12. Risks & Mitigations

| Risk                                    | Likelihood | Impact | Mitigation                                                                |
| --------------------------------------- | ---------- | ------ | ------------------------------------------------------------------------- |
| Canvas performance stalls on >500 nodes | Medium     | High   | React Flow virtualization, debounce state updates, lazy‑load node content |
| OpenAI API cost spikes                  | Medium     | Medium | Cache GPT responses per node/version, enforce token budget per user       |
| Scope creep during sprint               | High       | High   | Feature freeze after Day‑2; park extras behind feature flags              |
| PHI leak when HIPAA flag off            | Low        | High   | HIPAA guard middleware, regex PHI scrub, compliance review                |
| Vercel headless export failures         | Low        | Medium | Fallback CSR download, capture error logs in Sentry                       |

## 13. Next Steps

1. Confirm updated scope, timeline, and AI guidelines with stakeholders.
2. Initialize GitHub repo & Supabase project (Bucket A).
3. Kick off Day‑0 tasks in Motion.
4. Schedule daily stand‑ups + end‑of‑day recap to refine prompts and unblock.
