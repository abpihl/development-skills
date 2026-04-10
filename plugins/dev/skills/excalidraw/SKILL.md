---
name: excalidraw
description: Use when creating new Excalidraw diagrams or reading, modifying, or comparing existing .excalidraw or .excalidraw.json files
---

# Excalidraw

## Overview

Two workflows — creation and editing — with one hard rule for editing:

**Main agents NEVER read `.excalidraw` files directly. No exceptions.**

Excalidraw JSON files consume 4k–22k tokens each. Nearly all of that is visual metadata.
Always delegate editing to a subagent.

---

## Creating a New Diagram

Generate the JSON directly. No subagent needed for creation.

### File Envelope

Every `.excalidraw` file is a JSON object:

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [],
  "appState": {
    "gridSize": null,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

### Element Types

| Type        | Use for                          |
|-------------|----------------------------------|
| `rectangle` | Boxes, nodes, containers         |
| `ellipse`   | Circles, ovals                   |
| `diamond`   | Decision / branching nodes       |
| `arrow`     | Connections between shapes       |
| `line`      | Non-connecting line segments     |
| `text`      | Standalone labels                |

### Required Fields Per Element

Every element needs all of these:

```json
{
  "id": "V1StGXR8_Z5jdHi6B-myT",
  "type": "rectangle",
  "x": 100,
  "y": 100,
  "width": 200,
  "height": 80,
  "angle": 0,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid",
  "roughness": 1,
  "opacity": 100,
  "groupIds": [],
  "frameId": null,
  "roundness": null,
  "seed": 1482348421,
  "version": 1,
  "versionNonce": 391154490,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1700000000000,
  "link": null,
  "locked": false
}
```

IDs must be random alphanumeric strings (~21 chars, nanoid-style). Never use sequential integers — duplicate or predictable IDs corrupt the file silently.

`seed` and `versionNonce` must be random integers. Using `0` causes rendering glitches.

### Arrow Bindings

When an arrow connects two shapes, three elements need updating:

**Arrow:**
```json
{
  "type": "arrow",
  "startBinding": { "elementId": "<source-id>", "focus": 0, "gap": 1 },
  "endBinding":   { "elementId": "<target-id>", "focus": 0, "gap": 1 },
  "points": [[0, 0], [200, 0]]
}
```

**Source shape's `boundElements`:**
```json
"boundElements": [{ "type": "arrow", "id": "<arrow-id>" }]
```

**Target shape's `boundElements`:**
```json
"boundElements": [{ "type": "arrow", "id": "<arrow-id>" }]
```

All three must be in sync. A broken binding renders as a disconnected floating arrow.

---

## Editing an Existing Diagram

**Never use the Read tool on `.excalidraw` files.** Always delegate.

### Pattern

```
Main agent                    Subagent
─────────────────────────────────────────────────────────
Receive edit request
Create subagent task ──────→  Read .excalidraw file
                               Return semantic summary:
                               - element labels + types
                               - connection topology
                               - NO raw JSON
Specify changes ───────────→  Apply changes to JSON
                               Write updated file
                               Confirm: added/changed/removed
```

### Subagent Task Templates

**Understand a diagram:**
> Read `<path>`. Return: a list of all elements with their text labels and types, and which elements are connected to which. Do not return any JSON.

**Modify a diagram:**
> Read `<path>`. Make these changes: `<plain-language changes>`. Write the updated file. Confirm which elements were added, changed, or removed.

**Create from description (via subagent):**
> Create a new Excalidraw file at `<path>` with the following diagram: `<description>`. Confirm which elements were created and how they are connected.

**Compare two diagrams:**
> Read `<file-a>` and `<file-b>`. Return the architectural differences — which elements exist in one but not the other, and how the connection topology differs. Do not return any JSON.

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `seed` or `versionNonce` is `0` | Use random integers — `0` causes visual glitches |
| Arrow added but endpoint shapes not updated | Update `boundElements` on both source and target shapes |
| `containerId` set but container's `boundElements` missing entry | Keep `containerId` and `boundElements` in sync |
| IDs are sequential integers (`1`, `2`, `3`) | Use nanoid-style random strings |
| Main agent reads file to "quickly check" structure | No exceptions — delegate to subagent |
