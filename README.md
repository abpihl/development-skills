# development-skills — Claude Code plugins

General-purpose development skills for Claude Code — visualization, diagramming, and cross-project utilities.

---

## Plugins

| Plugin | Install command | Description |
|---|---|---|
| `dev` | `/plugin install dev@development-skills` | Excalidraw authoring rules and diagram generation |

---

## dev

Tools for generating and working with Excalidraw diagrams from use cases, agent architectures, or free-text descriptions.

### Skills

| Skill | Command | Description |
|---|---|---|
| Excalidraw | `/dev:excalidraw` | Authoring rules for working with `.excalidraw` files — element types, required fields, ID rules, arrow binding, and the subagent delegation pattern for editing. Reference skill loaded by `/dev:diagram`. |
| Diagram | `/dev:diagram` | Generates an Excalidraw `.excalidraw` file from a confirmed use case, build manifest, or free-text description. Works standalone or as a step in the `/usecase` or `/mcs:architect` workflows. |

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
