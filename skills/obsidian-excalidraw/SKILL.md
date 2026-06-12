---
name: obsidian-excalidraw
description: Specialist skill for generating Excalidraw diagrams (.excalidraw.md) in an Obsidian vault — architecture, sequence, flow, and component diagrams. Normally invoked by the obsidian-documentation orchestrator. Use whenever a diagram is requested or flagged as valuable by another skill: "draw the auth flow", "add a diagram", "visualize the architecture". Produces files compatible with the obsidian-excalidraw plugin, stored next to their related notes.
---

# Excalidraw Diagram Specialist

You turn system understanding into diagrams the Obsidian Excalidraw plugin can open and edit. You own all diagram generation in this ecosystem — no other skill draws.

Read `../obsidian-documentation/references/conventions.md` first.

## Placement and naming

- File name: `<Topic> <Type>.excalidraw.md` (e.g., `Auth Flow.excalidraw.md`, `Payments Sequence.excalidraw.md`)
- Location: **next to the related note** (e.g., `Features/Auth/Auth Flow.excalidraw.md`). Only cross-cutting diagrams with no single home go in `Diagrams/`.
- After creating, report the embed line `![[<name>.excalidraw]]` to the orchestrator so the owning note embeds it.

## Diagram types and when to pick them

| Type | Use for | Shape language |
|---|---|---|
| Architecture | system/infra overviews | labeled rectangles for components, arrows for dependencies |
| Component | one area's internals | rectangles grouped by boundary, arrows for calls |
| Flow | a process with branches | rectangles for steps, diamonds for decisions, arrows top-to-bottom |
| Sequence | request/event over time | vertical actor lanes, horizontal arrows in time order |

Keep diagrams **small**: 5–12 elements. If you need more, you're drawing two diagrams — split them. Every element gets a short text label; the related note carries the explanation, not the diagram.

## File format

An `.excalidraw.md` file is markdown with frontmatter and a JSON drawing block:

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---

# Text Elements
<label> ^<elementId>
…one line per text element…

%%
# Drawing
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "obsidian-documentation-skill",
  "elements": [ … ],
  "appState": { "viewBackgroundColor": "#ffffff", "gridSize": null },
  "files": {}
}
```
%%
```

Element essentials (uncompressed JSON; the plugin accepts it and will manage compression itself):

- **Rectangle**: `{"type":"rectangle","id":"<uid>","x":0,"y":0,"width":180,"height":70,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"solid","strokeWidth":2,"roughness":1,"opacity":100,"angle":0,"groupIds":[],"seed":1,"version":1,"versionNonce":1,"isDeleted":false,"boundElements":[],"link":null,"locked":false}`
- **Text**: same base fields plus `"type":"text","text":"<label>","fontSize":16,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":14,"containerId":null,"originalText":"<label>"`
- **Arrow**: same base fields plus `"type":"arrow","points":[[0,0],[160,0]],"startBinding":{"elementId":"<id>","focus":0,"gap":4},"endBinding":{…}","startArrowhead":null,"endArrowhead":"arrow"`
- Bind a label to a shape by putting `{"type":"text","id":"<textId>"}` in the shape's `boundElements` and the shape id in the text's `containerId`.
- Layout on a simple grid: 220px horizontal, 120px vertical spacing. Unique `id` per element (any short random string).
- Each text element must also appear under `# Text Elements` as `<text> ^<id>`.

Use a consistent palette: components `#a5d8ff` (blue), external systems `#ffec99` (yellow), data stores `#b2f2bb` (green), problem/legacy areas `#ffc9c9` (red).

## Process

1. Get the structure to draw from the orchestrator/requesting skill (components and relationships — not prose).
2. Choose type, sketch the grid layout mentally, cap at 12 elements.
3. Generate the file; validate the JSON parses (run it through a JSON parser if shell access exists).
4. Report back: file path + embed line + which note should own it.

## Example

**Input**: "Sequence diagram for refunds: RefundButton → API → Refund job → Stripe → webhook → DB"

**Output**: `Features/Refunds/Refunds Sequence.excalidraw.md` — 6 actor lanes, time-ordered arrows, labels from the flow. Reports embed line `![[Refunds Sequence.excalidraw]]` for `Features/Refunds/Refunds.md`.
