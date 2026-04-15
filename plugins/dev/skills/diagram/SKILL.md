---
name: diagram
description: "Generate an Excalidraw diagram from a confirmed use case, build manifest, or free-text description. Standalone skill — run after /usecase, /mcs:architect, or with any description."
argument-hint: "<optional: free-text description, 'usecase', or 'manifest'>"
user-invocable: true
---

# /dev:diagram — Excalidraw Diagram Generator

You generate an Excalidraw `.excalidraw` file from a use case flow, agent architecture, or any described system. You write the JSON directly — you do not rely on external MCP tools to produce the output.

## Reference

Load `../excalidraw/SKILL.md` before writing any Excalidraw JSON. All element types, required field rules, ID requirements, arrow binding rules, and the subagent delegation pattern for editing are defined there. Follow those rules without exception.

---

## Input modes

Detect the mode from `$ARGUMENTS` and conversation context:

| Mode | Triggered when | Input source |
|---|---|---|
| **usecase** | `$ARGUMENTS` is `usecase` or a confirmed use case exists earlier in the conversation | Use case description(s) from context |
| **manifest** | `$ARGUMENTS` is `manifest` or `build-manifest.md` exists in the working directory | `build-manifest.md` or manifest from conversation context |
| **free-text** | `$ARGUMENTS` contains a description, or neither of the above apply | `$ARGUMENTS` or prompt the user |

If none of the above: ask:
> What would you like to diagram? Paste a description, or point me to a file.

---

## Brand (optional)

If `../brand/SKILL.md` exists, load it and resolve the active profile. Apply the profile's colours, typography, and element presets when generating diagram elements. The profile can be set via:

1. **Explicit flag** — `/dev:diagram --brand context-and`
2. **Conversation context** — user said "use the Context& brand"
3. **Default profile** — the profile with `default: true` in its frontmatter
4. **No profile** — use the built-in defaults from `../excalidraw/SKILL.md`

---

## Step 0: Confirm scope

State what you are about to diagram and where the file will be saved. Keep it brief.

**Usecase mode** — no HITL needed if the use case was already confirmed in this conversation. Simply state:
> Generating Excalidraw diagram for: **"<Use Case Name>"**
> Output: `docs/diagrams/<use-case-name>.excalidraw`

Then proceed to Step 1 immediately.

**Manifest or free-text mode** — show a one-paragraph summary of the diagram content and output path, and ask the user to confirm or adjust before generating.

---

## Step 1: Plan the element set

Before writing any JSON, produce a planning table of all elements:

| Shorthand ID | Type | Label | Connects to |
|---|---|---|---|
| ... | rectangle / diamond / ellipse / arrow | ... | ... |

Derivation rules by mode:

**Usecase mode — map use case fields to elements:**

| Use case field | Element |
|---|---|
| Actor | `rectangle`, left column, labeled with actor name |
| Main flow steps (numbered) | `rectangle` nodes in sequence, centre column |
| Decision or branching point | `diamond` node, inline in the flow |
| Out of scope | `rectangle` with dashed stroke, greyed, right edge |
| Success metric / outcome | `rectangle` with distinct background colour, bottom right |
| System boundary | outer `rectangle` container wrapping all elements |

**Manifest mode — map manifest sections to elements:**

| Manifest field | Element |
|---|---|
| Topics | `rectangle` nodes |
| Conditions / routing | `diamond` nodes |
| Connector actions / Power Automate | `rectangle` with label |
| Knowledge sources | `rectangle`, bottom section |
| Child agents | separate `rectangle` container with dashed border |

Verify before proceeding:
- Every arrow has both a source ID and a target ID in the table
- All decision points are `diamond`
- No arrows are left unbound

---

## Step 2: Generate the `.excalidraw` file

Apply all rules from `../excalidraw/SKILL.md`:
- Nanoid-style random IDs (~21 chars, alphanumeric)
- Random `seed` and `versionNonce` — never `0`
- All required fields present on every element
- For every arrow: set `startBinding`, `endBinding`, and add the arrow ID to `boundElements` on both the source and target shapes

**Layout conventions:**

| Mode | Layout |
|---|---|
| Usecase | Actor(s) left → flow steps centre (top to bottom) → outcomes bottom-right; out-of-scope dashed box far right |
| Manifest | Entry topics left → processing centre → outputs right; knowledge sources bottom; child agents in separate container |

**Output path:**
- Usecase mode: `docs/diagrams/<use-case-name>.excalidraw` (create `docs/diagrams/` if it does not exist)
- Manifest mode: `<project-dir>/<agent-name>-architecture.excalidraw`
- Free-text mode: current working directory, filename derived from the description title

Write the file. Report: `Saved: <full path>`

---

## Step 3: Optional — Excalidraw+ cloud upload

If `EXCALIDRAW_API_KEY` is available in the environment (or the user mentions having one):
1. Call `create_scene` with the diagram name
2. Call `update_scene_content` with the file content
3. Report the scene URL

If no API key: skip this step silently.

---

## Step 4: Editing an existing diagram

If the user asks to modify an existing `.excalidraw` file, delegate to a subagent. Never read the file directly in the main agent.

Subagent prompt:
> Read `<path>`. Make these changes: `<plain-language description of changes>`. Write the updated file back to the same path. Confirm which elements were added, changed, or removed.

---

## Step 5: Completion

Report concisely:
> `<filename>.excalidraw` — <N> shapes, <M> arrows.

If called from a larger workflow (e.g., `/usecase`), continue to the next step in that workflow. If called standalone, stop here.

---

## Rules

1. Load `../excalidraw/SKILL.md` before writing any Excalidraw JSON — no exceptions. Load `../brand/SKILL.md` if it exists and resolve the active profile.
2. Usecase mode: skip the confirmation gate if the use case is already confirmed. Generate immediately.
3. Manifest / free-text mode: always confirm scope before generating.
4. Never read `.excalidraw` files directly — delegate all edits to a subagent.
5. `diamond` for every decision/condition node; `rectangle` for all other nodes.
6. Arrow bindings: `startBinding` + `endBinding` on the arrow AND `boundElements` updated on both endpoint shapes.
7. IDs must be nanoid-style random strings (~21 chars) — never sequential integers.
8. `seed` and `versionNonce` must be random integers — never `0`.
9. Output path convention: `docs/diagrams/` for usecase mode; project root for manifest mode.
10. If the MCP tools (`mcp__excalidraw__text_to_excalidraw`, `mcp__excalidraw__create_flowchart`) are available, they may be used as an alternative — but the schema rules from `../excalidraw/SKILL.md` still govern any file produced.
