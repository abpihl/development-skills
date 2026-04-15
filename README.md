# development-skills — Claude Code plugins

Brand identity management, Excalidraw authoring, and diagram generation for Claude Code.

---

## Plugins

| Plugin | Install command | Description |
|---|---|---|
| `dev` | `/plugin install dev@development-skills` | Brand profiles, Excalidraw authoring rules, and diagram generation |

---

## dev

### Skills

| Skill | Command | Description |
|---|---|---|
| Brand | `/dev:brand` | Named brand identity profiles — colours, typography, diagram styles, document templates. Single source of truth consumed by `excalidraw`, `diagram`, and cross-plugin skills like `/li:infographic`. |
| Excalidraw | `/dev:excalidraw` | Authoring rules for `.excalidraw` files — element types, required fields, ID rules, arrow binding, subagent delegation pattern. Optionally loads a brand profile for styling. |
| Diagram | `/dev:diagram` | Generates Excalidraw diagrams from use cases, build manifests, or free-text. Optionally loads a brand profile for styling. |

---

## Brand Profiles

The brand skill provides named profiles that any output-producing skill can consume. Each profile defines colours, typography, diagram element styles, layout templates, and document template references.

### Available profiles

| Profile | Default | Palette | Description |
|---|---|---|---|
| `context-and` | Yes | Black / White / Purple `#5500FF` | Context& visual identity — Aktiv Grotesk + PP Neue Machina typography, clean diagram style, logo/photography/icon guidelines |
| `personal` | No | Forest green `#1b4332` / `#52b788` | Personal brand — weight-based 6-level typography hierarchy, 8px grid system, 8 infographic layout templates, WCAG contrast ratios |

### Profile contents

Each profile can define:

| Section | Required | Description |
|---|---|---|
| Colours | Yes | Primary + secondary palette, semantic role mapping (stroke, fill, accent, background, success, warning, info) |
| Typography | Yes | Font families, weights, sizes, line heights, letter spacing |
| Diagram Styles | Yes | Excalidraw element defaults, node presets by type, arrow defaults, canvas settings |
| Document Templates | Yes | Word/PowerPoint/Excel style mappings, template file references |
| Grid System | Optional | Canvas dimensions, padding, spacing rules (used by infographic/fixed-canvas skills) |
| Layout Templates | Optional | Named structural templates with element positions (e.g. KPI Cards, Timeline, Bold Statement) |
| UX & Readability Rules | Optional | Gestalt principles, contrast rules, text density limits |

### Creating a new profile

Copy `plugins/dev/skills/brand/profiles/_template.md` and fill in the values:

```
plugins/dev/skills/brand/profiles/customer-x.md
```

### Using brand with diagrams

```
# Apply a brand profile explicitly
/dev:diagram --brand context-and <description>

# Set the brand for the conversation
/dev:brand context-and
/dev:diagram <description>

# Without brand — uses built-in defaults
/dev:diagram <description>
```

### Cross-plugin integration

Brand profiles are designed to be consumed across plugins:

| Plugin | Skill | How it uses brand |
|---|---|---|
| `dev` | `/dev:diagram` | Applies colours, node presets, arrow styles to generated Excalidraw files |
| `dev` | `/dev:excalidraw` | Applies element defaults when creating or editing `.excalidraw` files |
| `communication` | `/li:infographic` | References `personal` profile for colour palette, typography, grid, layout templates, and UX rules |

The integration is always optional — skills fall back to built-in defaults when no brand profile is available.

### Excalidraw style guidance

For diagrams that look hand-drawn and human (the whole point of Excalidraw):

| Property | Value | Why |
|---|---|---|
| `roughness` | `1` | Hand-drawn edges — `0` looks like PowerPoint |
| `fontFamily` | `1` (Virgil) | The signature Excalidraw hand-drawn font — `2` (Helvetica) kills the feel |
| `fillStyle` | `"solid"` | Clean fills with light colours — `"hachure"` is too noisy for most diagrams |
| `roundness` | `{ "type": 3 }` | Rounded corners on all rectangles |
| `strokeWidth` | `2` | Standard weight for shapes |

Containers (grouping boxes) should use a subtle solid background fill with a dashed or solid border — see the `Container / group` node preset in each brand profile.

---

## Diagram Generation

### Input modes

`/dev:diagram` detects its input automatically:

| Mode | Triggered by | Source |
|---|---|---|
| `usecase` | Confirmed use case in conversation context | Use case fields → actor, flow steps, decisions, outcomes |
| `manifest` | `build-manifest.md` in working dir, or `manifest` argument | MCS build manifest → topics, actions, knowledge, child agents |
| `free-text` | Any other description | `$ARGUMENTS` or prompted |

### Workflow

```
# Standalone — from a description
/dev:diagram <description>

# After /usecase
/usecase
  ↓ confirmed use case + mermaid diagram
/dev:diagram usecase
  ↓ docs/diagrams/<name>.excalidraw

# After /mcs:architect
/mcs:architect
  ↓ build manifest approved
/dev:diagram manifest
  ↓ <agent-name>-architecture.excalidraw
```

### Output locations

| Mode | Output path |
|---|---|
| usecase | `docs/diagrams/<use-case-name>.excalidraw` |
| manifest | `<project-dir>/<agent-name>-architecture.excalidraw` |
| free-text | Current working directory |

---

## Project Structure

```
development-skills/
├── .claude-plugin/marketplace.json
├── plugins/
│   └── dev/
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           ├── brand/
│           │   ├── SKILL.md                  ← skill definition + profile resolution
│           │   └── profiles/
│           │       ├── context-and.md        ← Context& identity (default)
│           │       ├── personal.md           ← Personal brand + layout templates
│           │       └── _template.md          ← Blank template for new profiles
│           ├── diagram/
│           │   └── SKILL.md                  ← diagram generation skill
│           └── excalidraw/
│               └── SKILL.md                  ← Excalidraw authoring rules
└── README.md
```

---

## Installation

```
/plugin install dev@development-skills
```

---

## Source

GitHub: [abpihl/development-skills](https://github.com/abpihl/development-skills)
