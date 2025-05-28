# Decision Tree Web App – **User Guide**

> The friendly walkthrough for first‑time creators, product ops, and healthcare teams building branching narratives, FAQ chatbots, customer journeys, skill trees, or clinical pathways.

*Last updated • 2025‑05‑28*

---

## 1 Welcome

The Decision Tree Web App turns complex decision logic into an interactive, visual canvas—augmented by GPT‑4o so you’re never stuck for ideas or copy. Whether you’re crafting a choose‑your‑own‑adventure, mapping an onboarding funnel, or encoding a clinical protocol, this guide will get you productive fast.

---

## 2 Getting Started

1. **Sign‑up** with email, Google, or GitHub.  A magic‑link lands in your inbox.
2. The first time you log in, the app seeds a **Tutorial Tree** so you can poke around without fear.
3. Click **New Tree → Blank** (or pick a template) to start your own project.

### 2.1 The Interface at a Glance

| Area                         | What it does                                                                 |
| ---------------------------- | ---------------------------------------------------------------------------- |
| **Canvas**                   | Drag‑and‑drop nodes, zoom/pan with trackpad or ⌘/Ctrl + scroll.              |
| **Mini‑Map**                 | Top‑right mini‑map helps navigate large graphs.                              |
| **Breadcrumb**               | Shows the path from the root to your current node.                           |
| **Toolbar**                  | Add nodes, duplicate branches, undo/redo, filter by tag, run conflict check. |
| **Sidebar (Node Inspector)** | Edit node body, tags, metadata JSON; AI rewrite & suggest buttons live here. |

---

## 3 Building Your First Tree

1. **Add a Node** – click “+ Node” or double‑click the canvas.
2. **Connect Nodes** – drag the blue handle to another node.
3. **Label & Tag** – use the sidebar to add text and `tags[]` for filtering.
4. **Duplicate Branch** for quick A/B paths.
5. Press **⌘/Ctrl+S** or wait—autosave runs every 10 s.

> **Tip:** Orphan Node Scan in the toolbar highlights any node with no inbound edge.

### 3.1 Node Types & Metadata

* **Decision** (default) – multiple outbound edges.
* **Ending** – set `type = “end”`; conflict detector will ignore further branches.
* **Custom** – add fields to `metadata` JSON for probabilities, prerequisites, etc.

---

## 4 AI Assist

| Feature                 | How to use                                                                    | Best practice                                                              |
| ----------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Suggest Next Branch** | Click “✨ Suggest” with a node selected. GPT‑4o returns 3 branch ideas.        | Provide a short context paragraph in the node body for richer suggestions. |
| **Rewrite Copy**        | Choose a tone (“friendly”, “professional”, “whimsical”) then hit “↻ Rewrite”. | Use for localisation or voice consistency.                                 |
| **Conflict Detector**   | Toolbar → 🔍 Conflicts. Flags unreachable nodes or loops.                     | Run before export to ensure logical integrity.                             |

> **HIPAA Mode**: If your organisation enables the `hipaa` flag, PHI is scrubbed before any text is sent to GPT.

---

## 5 Sharing & Collaboration

* **Share Link** – click “Share” → choose **Viewer** or **Editor** → copy URL.  Append `?mode=viewer` to force read‑only.
* **Per‑Account Flags** – Admins can toggle experimental features (`analytics`, `export`) for selected users under **Settings → Labs**.
* **Real‑time Editing** *(coming soon)* – you’ll see collaborators’ cursors once the `realtime` flag ships.

---

## 6 Export & Integrations

| Format                 | How to export                   | When to use                                        |
| ---------------------- | ------------------------------- | -------------------------------------------------- |
| **PNG Snapshot**       | Export → “Canvas to PNG”        | Pitch decks, documentation, Miro import            |
| **JSON**               | Export → “Raw JSON”             | Programmatic ingestion, backups                    |
| **Dialogflow ES JSON** | Export → “Chatbot (Dialogflow)” | Deploy FAQ or support flows into Google Dialogflow |

> The **export** feature flag controls visibility; admins can hide all exporter UI if needed.

---

## 7 Analytics (Optional)

If your workspace enables the `analytics` flag:

* Events are logged each time you **save** or **export** a tree.
* View basic charts under **Dashboard → Usage** to spot popular tags or busy hours.

Disable the flag in **Settings → Labs** to pause data collection.

---

## 8 Keyboard Shortcuts

| Action                | Mac          | Windows/Linux       |
| --------------------- | ------------ | ------------------- |
| Add Node              | **N**        | **N**               |
| Duplicate Node        | **⌘D**       | **Ctrl+D**          |
| Undo / Redo           | **⌘Z / ⇧⌘Z** | **Ctrl+Z / Ctrl+Y** |
| Zoom In / Out         | **⌘+ / ⌘‑**  | **Ctrl+ / Ctrl‑**   |
| Run Conflict Detector | **⌘Shift+F** | **Ctrl+Shift+F**    |

---

## 9 Troubleshooting & FAQ

**Canvas feels slow on large graphs**
Enable **Performance Mode** in Settings → hides rich text until you zoom in.

**GPT calls are rejected**
Check if `hipaa` flag is on and your text includes PHI (emails, IDs, full names).

**Export button missing**
Admin may have disabled the `export` flag or you have viewer permissions.

For more, visit the Notion FAQ or file an issue on GitHub.

---

## 10 Glossary

| Term             | Definition                                                          |
| ---------------- | ------------------------------------------------------------------- |
| **Tag**          | Freeform label attached to a node for filtering/analytics.          |
| **Feature Flag** | Runtime switch to enable/disable a feature globally or per‑account. |
| **PHI**          | Protected Health Information (HIPAA protected).                     |
| **RLS**          | Row‑Level Security rules in Supabase PostgreSQL.                    |

---

## 11 What’s Next

* **Realtime Multi‑Cursor** (flag: `realtime`) – live editing, presence avatars.
* **Template Marketplace** – share and import node templates.
* **AI Path Optimiser** – GPT suggests optimal branching probabilities based on outcomes.

Stay tuned in **#changelog** for rollout updates and flag announcements.

---

**Happy branching!** 🚀
