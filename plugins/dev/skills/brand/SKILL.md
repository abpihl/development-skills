---
name: brand
description: "Brand identity profiles — colours, typography, diagram styles, and document templates. Loaded by other skills (excalidraw, diagram) to apply consistent visual identity. Use standalone to list or switch profiles."
argument-hint: "<optional: profile name, 'list', or 'info'>"
user-invocable: true
---

# /dev:brand — Brand Identity Profiles

A reference skill that provides named brand profiles to any skill that produces visual or document output. Each profile defines colours, typography, diagram element styles, and document template references.

---

## Profile resolution

When loaded by another skill (e.g. `diagram`, `excalidraw`), resolve the active profile in this order:

1. **Explicit argument** — `--brand context-and` or `$ARGUMENTS` contains a profile name
2. **Conversation context** — user said "use the Context& brand" or similar
3. **Default profile** — the profile with `default: true` in its frontmatter
4. **No profile** — if no brand skill or no profile is resolved, the consuming skill uses its own built-in defaults

---

## Standalone commands

When invoked directly as `/dev:brand`:

| Argument | Action |
|---|---|
| `list` | List all available profiles with a one-line summary each |
| `info` or `info <profile>` | Show the full profile details for the active or named profile |
| `<profile-name>` | Set the active profile for the remainder of the conversation |
| _(none)_ | Show the currently active profile name, or list all if none is set |

---

## Profile file format

Each profile lives in `profiles/<name>.md` with this structure:

```markdown
---
name: <profile-name>
display-name: <Human-readable name>
default: true|false
---

## Colours
...

## Typography
...

## Diagram Styles
...

## Document Templates
...
```

Profiles may also include optional extended sections:

```markdown
## Grid System
...

## Layout Templates
...

## UX & Readability Rules
...
```

See `profiles/_template.md` for the full blank template.

---

## How consuming skills use this

When a skill like `diagram` or `excalidraw` loads this skill, it should:

1. Read `../brand/SKILL.md` (this file) to understand the resolution logic
2. Resolve the active profile using the order above
3. Read the resolved profile file from `../brand/profiles/<name>.md`
4. Apply the profile values to generated output — colours, stroke styles, typography, etc.

### Excalidraw mapping

When generating `.excalidraw` files, map profile values to element properties:

| Profile field | Excalidraw property |
|---|---|
| `colors.primary-stroke` | `strokeColor` |
| `colors.primary-fill` | `backgroundColor` |
| `colors.accent` | `strokeColor` for highlighted/CTA elements |
| `colors.background` | `appState.viewBackgroundColor` |
| `diagram.stroke-width` | `strokeWidth` |
| `diagram.roughness` | `roughness` |
| `diagram.fill-style` | `fillStyle` |
| `diagram.roundness` | `roundness` |
| `diagram.font-family` | `fontFamily` (1=Virgil, 2=Helvetica, 3=Cascadia) |
| `diagram.node-preset.*` | Combined properties for specific node types |

### Document mapping

When generating Word, PowerPoint, or Excel output, the `document-templates` section provides:
- Template file paths (`.dotx`, `.potx`, `.xltx`)
- Style name mappings (heading styles, body styles)
- Colour theme references

---

## Rules

1. Profiles are **read-only references** — never modify a profile file during generation.
2. If a requested profile does not exist, list available profiles and ask the user to choose.
3. The `_template.md` file is never used as an active profile — it is documentation only.
4. When no profile is resolved and no default exists, consuming skills fall back to their own built-in defaults. This must never cause an error.
5. Profile names are lowercase, kebab-case (e.g. `context-and`, `personal`, `customer-x`).
6. Extended sections (Grid System, Layout Templates, UX Rules) are optional. Consuming skills should use them when present and ignore them when absent.
7. Layout templates define structural positions for visual content (infographics, cards). They are not required for diagram generation but are used by skills that produce fixed-canvas output (e.g. `/li:infographic`).
