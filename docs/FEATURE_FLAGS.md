# FEATURE\_FLAGS.md – Runtime Toggles

> **Purpose:** Describe every runtime feature flag in the Forkli Web App, how to enable/disable it, and best‑practice guidelines for adding new flags.

*Updated 2025‑05‑28*

---

## 1 Overview

Feature flags allow us to ship code dark, run A/B tests, and de‑risk risky features without long‑lived branches. All flags are read at **runtime** from a single comma‑separated environment variable:

```txt
FEATURE_FLAGS=hipaa,export
```

The value is parsed both **client‑side** (Next.js public env var) and **server‑side** (Edge & Node routes) so UI and API behaviour stay in sync.

### 1.1 Quick Reference

| Flag        | Default | Scope             | Primary Guards                                                 | Use Cases                            |
| ----------- | ------- | ----------------- | -------------------------------------------------------------- | ------------------------------------ |
| `hipaa`     | off     | Server middleware | `HIPAAGuard` rejects or redacts PHI in GPT payloads            | Clinical Pathway Builder, healthcare |
| `analytics` | off     | Server + client   | `/api/events` route and `events` table are no‑op when flag off | Usage metrics, future pricing tiers  |
| `export`    | on      | Client UI + API   | Export modal and `/api/export` endpoint hidden when flag off   | PNG + Dialogflow JSON exports        |

---

## 2 Flag Implementation Details

### 2.1 Environment Variable(s)

* **Backend**: `process.env.FEATURE_FLAGS` (Edge & Node runtime)
* **Frontend**: `process.env.NEXT_PUBLIC_FEATURE_FLAGS` (injected at build)

> CI pipelines must mirror production flags to avoid test drift.

### 2.2 Utility Helpers

```ts
// lib/featureFlags.ts
export const activeFlags = new Set<string>(
  (process.env.NEXT_PUBLIC_FEATURE_FLAGS ?? '')
    .split(',')
    .map(f => f.trim())
    .filter(Boolean)
);

/**
 * Determine if a feature flag is active either globally **or** for a specific user.
 * @param flag  lowercase flag name
 * @param user  optional user object containing `feature_flags text[]`
 */
export const hasFlag = (flag: string, user?: { feature_flags?: string[] }) =>
  activeFlags.has(flag) || (user?.feature_flags ?? []).includes(flag);
```

### 2.3 Per‑Account Flags

Global flags can be **augmented or overridden per user** through the `users.feature_flags text[]` column (see Architecture ERD).  API routes typically load the session (`/api/auth/session`) and feed the user into `hasFlag()`:

```ts
const session = await getSession();
if (hasFlag('analytics', session.user)) {
  await logEvent();
}
```

Client components can do the same when they already have the viewer record:

```tsx
if (hasFlag('export', me)) {
  return <ExportButton />;
}
```

````
This helper is used across client (`hasFlag('export')`) and server modules.

### 2.4 Server‑Side Guard Example – HIPAA
```ts
// app/api/openai/route.ts
import { hasFlag } from '@/lib/featureFlags';

if (hasFlag('hipaa')) {
  const violation = detectPHI(requestBody);
  if (violation) {
    return new Response('PHI content blocked', { status: 400 });
  }
}
````

### 2.5 Client‑Side Guard Example – Export Modal

```tsx
const ExportButton = () => {
  if (!hasFlag('export')) return null; // feature hidden
  return <Button onClick={openExportModal}>Export</Button>;
};
```

---

## 3 Adding a New Feature Flag

1. Choose a **lowercase, kebab‑case** identifier (e.g., `realtime`).
2. Update `.env.example` with placeholder: `FEATURE_FLAGS=<existing>,realtime`.
3. Add tests covering enabled and disabled paths.
4. Document behaviour in this file under **Quick Reference**.
5. Add UI badge in Settings → About that lists active flags (dev only).

### 3.1 Removing a Flag

* Merge code behind the flag into default path.
* Delete conditional branches & dead tests.
* Remove flag from `.env*` and this doc.

---

## 4 Caveats & Best Practices

| Guideline                                         | Reason                                             |
| ------------------------------------------------- | -------------------------------------------------- |
| Treat flags as **short‑lived**; sunset them early | Prevent permanent branching logic                  |
| Gate both **UI and API** paths                    | Avoid security holes when UI hides but API exposed |
| Keep flag checks **co‑located** with feature code | Maintain readability                               |
| Never include secrets in flag names or values     | Logs may expose them                               |

---

## 5 Change Log

| Date       | Change                                               | By                 |
| ---------- | ---------------------------------------------------- | ------------------ |
| 2025‑05‑28 | Added per‑account flag helper & docs                 | Abhi & AI Co‑Pilot |
| 2025‑05‑28 | Initial document with `hipaa`, `analytics`, `export` | Abhi & AI Co‑Pilot |

---

**End of FEATURE\_FLAGS.md**
