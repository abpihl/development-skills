# development-skills — Claude Code plugins

General-purpose development skills for Claude Code — visualization, diagramming, and cross-project utilities.

---

## Plugins

| Plugin | Install command | Description |
|---|---|---|
| `dev` | `/plugin install dev@development-skills` | Brand profiles, Excalidraw authoring rules, and diagram generation |

---

## dev

Brand profiles, Excalidraw authoring, and diagram generation for Claude Code projects.

### Skills

| Skill | Command | Description |
|---|---|---|
| Brand | `/dev:brand` | Named brand identity profiles (colours, typography, diagram styles, document templates). Loaded by `excalidraw` and `diagram` to apply consistent visual identity. |
| Excalidraw | `/dev:excalidraw` | Authoring rules for working with `.excalidraw` files — element types, required fields, ID rules, arrow binding, and the subagent delegation pattern for editing. Reference skill loaded by `/dev:diagram`. |
| Diagram | `/dev:diagram` | Generates an Excalidraw `.excalidraw` file from a confirmed use case, build manifest, or free-text description. Works standalone or as a step in the `/usecase` or `/mcs:architect` workflows. |

### Brand profiles

The brand skill provides named profiles that `excalidraw` and `diagram` consume automatically. Each profile defines colours, typography, diagram element styles, and document template references.

| Profile | Default | Description |
|---|---|---|
| `context-and` | Yes | Context& visual identity — black/white/purple palette, Aktiv Grotesk + PP Neue Machina typography, clean diagram style |
| `personal` | No | Personal brand — forest green palette, weight-based typography hierarchy, 8px grid system, 8 layout templates for infographics |

To create a new profile, copy `profiles/_template.md` and fill in the values.

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

## Installation

```
/plugin install dev@development-skills
```

---

## Source

GitHub: [abpihl/development-skills](https://github.com/abpihl/development-skills)
