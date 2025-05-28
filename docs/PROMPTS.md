# PROMPTS.md – AI Prompt Library

> **Purpose:** Consolidate reusable ChatGPT prompt templates that align with the *Forkli Web App* development workflow buckets (A–H) and the project’s “Vibe‑Coding” rules. Copy, tweak 🔷 placeholders, paste into ChatGPT 4‑o, and commit the generated code & tests.

*Last updated 2025‑05‑28*

---

## 1 How to Use This Library

1. Locate the **bucket** and **component** you’re working on (see §8 Modular Workflow in the project plan).
2. Copy the **General Frame** (below) and the **Bucket‑specific Goal line**.
3. Replace 🔷 placeholders with concrete names and objectives.
4. Paste into ChatGPT; ensure the response includes *both* implementation **and** matching Vitest spec.
5. Review diff, run `pnpm test:changed`, commit with `Closes Motion‑<id>` footer.

---

## 2 General Prompt Frame

```md
### AI Prompt – Forkli Web App
Role: You are an expert TypeScript/React/Supabase engineer. ✨Stay within the component’s directory.✨
Context:
- Component: 🔷 <ComponentName>
- Bucket: 🔷 <A–H>
- Goal: 🔷 <One‑sentence objective>
Additional Info: TS strict, Tailwind via CVA, ≤ 100 LOC per function.
Instructions:
1. Output updated **TypeScript** code only, no explanations.
2. Keep funcs ≤ 100 LOC; extract helpers if needed.
3. Append a **Vitest** spec covering happy & error paths.
```

> **Tip:** Pre‑pend relevant `interface` and type definitions *first* so ChatGPT works interface‑first.

---

## 3 Bucket Templates & Inline Prompt Examples

| Bucket | Template Goal Prefix    | Prompt Snippet                                          |
| ------ | ----------------------- | ------------------------------------------------------- |
| **A**  | *Bootstrap environment* | `Goal: Set up Tailwind with shadcn/ui plugin enabled`   |
| **B**  | *Database & auth*       | `Goal: Create RLS policy so only tree owner can UPDATE` |
| **C**  | *Canvas UI*             | `Goal: Add Mini‑Map component with React‑Flow built‑in` |
| **D**  | *Edge validation*       | `Goal: Write recursive CTE to detect unreachable nodes` |
| **E**  | *AI assist*             | `Goal: Implement /api/openai route with HIPAA guard`    |
| **F**  | *Export & share*        | `Goal: Generate Dialogflow‑compatible JSON exporter`    |
| **G**  | *Analytics*             | `Goal: Insert event row on every tree SAVE action`      |
| **H**  | *Polish & deploy*       | `Goal: Add shadcn EmptyState for empty tree list`       |

### Example Prompt Blocks

Copy the **General Prompt Frame** from §2 and drop in one of these bucket‑specific contexts.

<details><summary>Bucket A – Bootstrap environment</summary>

```md
### AI Prompt – Forkli Web App
Role: You are an expert DevOps/React engineer. ✨Stay within the `scripts/` directory.✨
Context:
- Component: setup/tailwind-config.ts
- Bucket: A
- Goal: Set up Tailwind with shadcn/ui plugin enabled
Additional Info: TS strict, Tailwind via CVA, ≤ 100 LOC per function.
Instructions:
1. Output updated TypeScript code only.
2. Keep funcs ≤ 100 LOC.
3. Append a Vitest spec for the configuration helper.
```

</details>

<details><summary>Bucket B – Database & auth</summary>

```md
### AI Prompt – Forkli Web App
Role: You are a PostgreSQL & Supabase RLS expert.
Context:
- Component: migrations/001_create_rls.sql
- Bucket: B
- Goal: Create RLS policy so only tree owner can UPDATE
Additional Info: Show full SQL; include comment blocks for rollback.
Instructions:
1. Provide SQL only (no markdown).
2. Include accompanying Vitest that inserts dummy user & tree and asserts UPDATE rejection for non‑owner.
```

</details>

<details><summary>Bucket C – Canvas UI</summary>

```md
### AI Prompt – Forkli Web App
Role: You are an expert React/TypeScript/Tailwind engineer. ✨Stay within the component’s directory.✨
Context:
- Component: FlowCanvas
- Bucket: C
- Goal: Add Mini‑Map component with React‑Flow built‑in
Additional Info: TS strict, Tailwind via CVA, ≤ 100 LOC per function.
Instructions:
1. Output updated TypeScript code only.
2. Keep funcs ≤ 100 LOC.
3. Append a Vitest spec for zoom syncing.
```

</details>

<details><summary>Bucket D – Edge validation</summary>

```md
### AI Prompt – Forkli Web App
Role: You are an expert Supabase Edge Function + SQL engineer.
Context:
- Component: supabase/functions/conflict_detector.ts
- Bucket: D
- Goal: Write recursive CTE to detect unreachable nodes
Additional Info: Must handle cycles; ≤ 200 ms execution.
Instructions:
1. Provide TypeScript edge function and embedded SQL string.
2. Append Vitest hitting a local Supabase Docker instance.
```

</details>

<details><summary>Bucket E – AI assist</summary>

```md
### AI Prompt – Forkli Web App
Role: You are an OpenAI SDK & edge‑middleware expert.
Context:
- Component: app/api/openai/route.ts
- Bucket: E
- Goal: Implement /api/openai route with HIPAA guard
Additional Info: Use env `OPENAI_API_KEY`; tokenize budget 150.
Instructions:
1. Code only; include HIPAA regex guard util.
2. Append Vitest for PHI detection workflow.
```

</details>

<details><summary>Bucket F – Export & share</summary>

```md
### AI Prompt – Forkli Web App
Role: You are a TypeScript & JSON schema guru.
Context:
- Component: app/api/export/dialogflow.ts
- Bucket: F
- Goal: Generate Dialogflow‑compatible JSON exporter
Additional Info: Batch intents per 20 nodes.
Instructions:
1. Exporter function + storage upload helper.
2. Vitest verifying output matches Dialogflow spec.
```

</details>

<details><summary>Bucket G – Analytics</summary>

```md
### AI Prompt – Forkli Web App
Role: You are a Supabase & telemetry engineer.
Context:
- Component: app/api/events/route.ts
- Bucket: G
- Goal: Insert event row on every tree SAVE action
Additional Info: Require `analytics` flag; enforce JWT.
Instructions:
1. Provide route handler code only.
2. Append Vitest for 200 & 403 cases.
```

</details>

<details><summary>Bucket H – Polish & deploy</summary>

```md
### AI Prompt – Forkli Web App
Role: You are a UI/UX polisher.
Context:
- Component: components/EmptyState.tsx
- Bucket: H
- Goal: Add shadcn EmptyState for empty tree list
Additional Info: Use lucide icons; support dark mode.
Instructions:
1. Output Updated TSX only.
2. Append Vitest snapshot test.
```

</details>

---

## 4 Specialized Prompt Patterns Specialized Prompt Patterns

### 4.1 GPT Suggest → “Next Branch”

Use for Bucket E when extending `useOpenAI()` or `/api/openai`.

```md
Component: openai/suggest.ts      Bucket: E
Goal: Implement function `suggestNextBranch(nodeContext)` that calls OpenAI GPT‑4o and returns 3 branch ideas.
Constraints: redact PHI if HIPAA flag enabled; max 150 tokens; retry = 1.
```

### 4.2 GPT Rewrite → “Tone Adjustment”

```md
Component: openai/rewrite.ts      Bucket: E
Goal: Write handler that rewrites node body in requested tone (professional, friendly, etc.). Provide tests for edge cases (empty input, unsupported tone).
```

### 4.3 Conflict Detector Edge Function

```md
Component: supabase/functions/conflict_detector.ts    Bucket: D
Goal: Given tree_id, return list of unreachable or looping nodes via recursive CTE. Include SQL and TS handler.
```

### 4.4 Dialogflow Exporter

```md
Component: api/export/dialogflow.ts     Bucket: F
Goal: Map internal tree schema to Dialogflow ES intents JSON. Should batch nodes, resolve duplicates, and output to Supabase Storage. Provide test verifying JSON schema.
```

### 4.5 Analytics Event Ingestion

```md
Component: api/events/index.ts     Bucket: G
Goal: POST endpoint that validates JWT, writes event row, and respects `analytics` feature flag. Provide Vitest for 200 & 403 cases.
```

---

## 5 Prompt Hygiene Checklist

* **Scope 1 thing.** If you need two functions, send two prompts.
* **Include Types.** Pasting `interface` or SQL schema ahead improves accuracy.
* **Ask for Tests.** Always request Vitest spec.
* **Stop Explanations.** Instruct: “code only” to avoid markdown clutter.
* **Review & Refactor.** Run `pnpm lint` then tune variable names before commit.

---

## 6 Adding New Templates

1. Append a new row to **Bucket Templates** with a concise goal prefix.
2. Add detailed example in **Specialized Prompt Patterns**.
3. Increment date in header.
4. Commit as `docs(prompts): add <topic>`.

---

**End of PROMPTS.md**
