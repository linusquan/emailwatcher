---
name: excalidraw-diagram
description: Edit Excalidraw diagram JSON (elements, not screenshots) for EmailWatcher system-design updates. Use when Design Sync or sync-system-design changes docs/system-design/diagrams/*.excalidraw — system context, processing pipeline, quality gate, or Notion routing.
---

# Excalidraw Diagram Creator

Generate and edit `.excalidraw` JSON files that **argue visually**, not just display information. **Do not replace diagrams with PNG/SVG screenshots** — edit the `elements` array in place.

Based on [coleam00/excalidraw-diagram-skill](https://github.com/coleam00/excalidraw-diagram-skill); adapted for EmailWatcher `docs/system-design/diagrams/`.

## EmailWatcher targets

| Diagram | Path | Update when |
|---|---|---|
| System context | `docs/system-design/diagrams/system-context.excalidraw` | Actors, integrations, automations, MCPs change |
| Processing pipeline | `docs/system-design/diagrams/processing-pipeline.excalidraw` | Fetch/classify/route/integrate/report steps or labels change |
| Quality gate | `docs/system-design/diagrams/quality-gate.excalidraw` | USEFUL / PROMOTIONAL / LOW_VALUE / UNCERTAIN logic changes |
| Notion routing | `docs/system-design/diagrams/notion-routing.excalidraw` | `email inject`, match confidence, integrate vs skip changes |

**Before editing:** read the matching markdown (`architecture.md`, `pipeline.md`) and source-of-truth repo files (`README.md`, `emailwatcher.config`, `Emailer-Agent.md`). Match labels to names used there.

**Prefer surgical edits** for Design Sync: update text labels, add/remove elements, adjust bindings — preserve existing layout, IDs, seeds, and color vocabulary unless the architecture truly changed.

---

## Core philosophy

**Diagrams should ARGUE, not DISPLAY.**

A diagram isn't formatted text. It's a visual argument that shows relationships, causality, and flow that words alone can't express.

**The Isomorphism Test**: If you removed all text, would the structure alone communicate the concept? If not, redesign.

**The Education Test**: Could someone learn something concrete from this diagram, or does it just label boxes?

---

## Design process (before generating or editing JSON)

1. **Assess depth** — simple/conceptual vs comprehensive/technical. EmailWatcher diagrams are **technical**: use real config values, automation names, MCP names, and stage labels from the repo.
2. **Understand deeply** — what does each concept *do*? What relationships and flow exist?
3. **Map concepts to patterns** — fan-out, convergence, timeline, assembly line, decision diamond, etc. Each major concept should use a distinct pattern where possible.
4. **Sketch the flow** — trace how the eye moves before touching JSON.
5. **Edit JSON** — see workflow below.
6. **Validate** — structural checks always; visual render when the renderer is available (see Render & Validate).

---

## Editing workflow (Design Sync)

1. **Read the full `.excalidraw` file** — understand existing `elements`, `boundElements`, and layout.
2. **Identify elements to change** — usually `type: "text"` nodes (`text` and `originalText` must match) or labels bound to rectangles.
3. **Preserve invariants:**
   - Keep `"type": "excalidraw"`, `"version": 2`, `"source": "https://excalidraw.com"`.
   - Keep `appState.viewBackgroundColor` and `files` unless intentionally changing.
   - Use stable string `id` values; increment `version` / `versionNonce` / `updated` on edited elements.
   - Set `"isDeleted": false` on active elements.
   - Wire arrows via `boundElements` on both ends.
4. **Match existing style** — see Color palette and Element defaults below (this repo uses `roughness: 0`, `fontFamily: 3`, `opacity: 100`).
5. **Large changes** — build one section at a time; do not rewrite an entire comprehensive diagram in one pass.

---

## Color palette (EmailWatcher system-design)

Use these fills/strokes consistently with the existing four diagrams:

| Role | Fill | Stroke |
|---|---|---|
| Default shape | `#e3f2fd` | `#1e1e1e` |
| Agent / automation | `#fff3e0` | `#1e1e1e` |
| External / MCP | `#f3e5f5` | `#1e1e1e` |
| Integration / success path | `#e8f5e9` | `#1e1e1e` |
| Alert / gate / decision emphasis | `#fce4ec` | `#1e1e1e` |
| Free-floating text / labels | `transparent` | `#1e1e1e` or `#666666` for secondary |

Always pair darker stroke (`#1e1e1e`) with lighter fill. Do not invent new palette colors unless adding a genuinely new semantic category.

---

## Element defaults

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [],
  "appState": {
    "viewBackgroundColor": "#ffffff",
    "gridSize": 20
  },
  "files": {}
}
```

**Text element** — `text` and `originalText` must contain ONLY readable words (no markup):

```json
{
  "type": "text",
  "fontSize": 16,
  "fontFamily": 3,
  "textAlign": "center",
  "verticalAlign": "middle",
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "fillStyle": "solid",
  "strokeWidth": 1,
  "roughness": 0,
  "opacity": 100,
  "text": "Label",
  "originalText": "Label"
}
```

**Rectangle** — `roundness: { "type": 3 }` for rounded corners; bind label text via `boundElements`.

**Arrow** — connect with `startBinding` / `endBinding` or `boundElements`; route `points` to avoid crossing shapes.

| Concept | Shape |
|---|---|
| Process, step | `rectangle` |
| Start / end | `ellipse` |
| Decision | `diamond` |
| Label only | free-floating `text` (no container) |

---

## Text rules

- Title: `fontSize` 24–28; body labels: 14–16; annotations: 12–14.
- Short labels aligned with `Emailer-Agent.md` terminology — no invented runtime components.
- Every relationship that matters needs an arrow or line; position alone is not enough.

---

## Render & validate

You cannot fully judge layout from JSON alone.

**When a renderer is available** (optional setup from [excalidraw-diagram-skill](https://github.com/coleam00/excalidraw-diagram-skill)):

```bash
cd .cursor/skills/excalidraw-diagram/references && uv run python render_excalidraw.py <path-to-file.excalidraw>
```

Read the output PNG and fix overlaps, clipped text, and arrow routing in a loop until clean.

**In cloud Design Sync runs (no renderer):** after editing, verify JSON parses; every `id` referenced in `boundElements`/bindings exists; no duplicate ids; `text`/`originalText` pairs match; labels reflect repo truth. Prefer minimal diffs.

---

## Quality checklist

1. Labels match current repo config and automations (e.g. `timerange`, inbox names, MCP names).
2. Diagram and prose updated together (`architecture.md` / `pipeline.md` ↔ `.excalidraw`).
3. No uniform card grids — varied patterns where the diagram teaches flow or decisions.
4. `<30%` of text inside containers where free-floating text suffices.
5. `roughness: 0`, `opacity: 100`, `fontFamily: 3` throughout.
6. Valid Excalidraw JSON — not a screenshot, not markdown, not mermaid.
