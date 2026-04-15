---
name: personal
display-name: "Personal Brand"
default: false
---

# Personal Brand Profile

Clean, high-contrast, data-forward design. Forest green palette with billboard-style typography. Optimised for social media and LinkedIn feed performance.

---

## Colours

### Primary palette

| Name | HEX | RGB | Role | Contrast on white |
|---|---|---|---|---|
| Primary Dark | `#1b4332` | 27/67/50 | Titles, filled shapes, accent bars | 10.4:1 AAA |
| Accent Green | `#52b788` | 82/183/136 | Icons, borders, active indicators | 3.2:1 (large text / decorative only) |
| Light Green | `#d8f3dc` | 216/243/220 | Dividers, lines, subtle backgrounds | Decorative only |
| Tint | `#f0faf4` | 240/250/244 | Callout backgrounds, row fills | Background only |
| Body Text | `#2d2d2d` | 45/45/45 | Description and supporting text | 15.8:1 AAA |
| Tertiary | `#767676` | 118/118/118 | Micro-labels, section tags | 4.6:1 AA |
| Alert Red | `#c53030` | 197/48/48 | Problem metrics, negative indicators | 6.3:1 AA |
| Background | `#ffffff` | 255/255/255 | Canvas | — |

### Semantic colour mapping

| Semantic role | Colour | HEX |
|---|---|---|
| `primary-stroke` | Primary Dark | `#1b4332` |
| `primary-fill` | Tint | `#f0faf4` |
| `accent` | Accent Green | `#52b788` |
| `background` | White | `#ffffff` |
| `secondary-fill` | Light Green | `#d8f3dc` |
| `muted-stroke` | Tertiary | `#767676` |
| `highlight` | Accent Green | `#52b788` |
| `success` | Accent Green | `#52b788` |
| `warning` | Alert Red | `#c53030` |
| `info` | Primary Dark | `#1b4332` |

### Accessibility

Every colour pairing must meet WCAG contrast ratios:
- Body text on white: minimum AA (4.5:1)
- Large text / headings: minimum 3:1
- Decorative elements: no minimum, but never carry semantic meaning alone

---

## Typography

### 6-Level Hierarchy

Sizes calibrated for 1080×1080px canvas. Scale proportionally for other formats.

| Level | Role | Font-size | Weight | Colour | Letter-spacing | Notes |
|---|---|---|---|---|---|---|
| L1 | Main title | 44–56px | 800 | `#1b4332` | 0.5 | Max 6 words |
| L2 | Subtitle / channel label | 20px | 600 | `#52b788` | 2 | ALL-CAPS |
| L3 | Section / column heading | 22px | 700 | `#1b4332` | 1 | ALL-CAPS, max 4 words |
| L4 | Body / description | 20–22px | 400 | `#2d2d2d` | 0 | Max 1 line (10–12 words), prefer fragments |
| L5 | Micro-label / category tag | 16px | 700 | `#767676` | 2 | ALL-CAPS |
| L6 | KPI hero number | 80–96px | 800 | `#1b4332` or `#c53030` | 0 | For statistics that tell the story alone |

### Typography rules

- Never use more than 3 font sizes in any single zone
- Bold (700+) for labels and headings only — body text is always 400
- Italic is reserved for user-provided quotes only
- Letter-spacing >= 2 applies to ALL-CAPS labels only
- Minimum line-height: 1.3x font size

### Text principles — billboard, not brochure

- L1 title: max 6 words
- L3 labels: max 4 words
- L4 body: max 1 line (10–12 words) — prefer fragments over full sentences
- No filler words — strip to the essential

---

## Grid System

### 8px Grid

All spacing, padding, and element sizing must align to the 8px grid.

| Element | Value |
|---|---|
| Canvas size | 1080×1080px (square, LinkedIn-optimised) |
| Canvas padding | 48px all sides |
| Content zone | 984×984px |
| Minimum gap between elements | 16px |
| Section spacing | 32px |
| Tight internal padding | 4px (only within a component) |

### Structural Anchors

Every output includes these fixed elements:

| Element | Position | Style |
|---|---|---|
| Top accent bar | y=0, height=8 | fill `#1b4332` |
| Bottom accent bar | y=1072, height=8 | fill `#1b4332` |
| L1 title | centred, x=540, y=72–88 | — |
| L2 subtitle | centred, x=540, y=112–124 | ALL-CAPS |
| Divider | x1=240 x2=840, y=144, height=1 | stroke `#d8f3dc` |
| Scroll-stop zone | y=0 to y=380 | Headline + primary visual hook must land here |
| Callout box (optional) | Bottom, 56px above bottom accent bar | Tint rect + 6px left bar `#52b788`, L4 italic quote |

---

## Diagram Styles

Excalidraw and diagram element defaults when using the Personal brand:

### Element defaults

| Property | Value |
|---|---|
| `strokeColor` | `#1b4332` (Primary Dark) |
| `backgroundColor` | `#f0faf4` (Tint) |
| `strokeWidth` | 2 |
| `roughness` | 0 (clean lines) |
| `fillStyle` | `solid` |
| `roundness` | `{ "type": 3 }` (rounded corners, r=8) |
| `opacity` | 100 |
| `fontFamily` | 2 (Helvetica) |

### Node presets

| Node type | strokeColor | backgroundColor | strokeStyle |
|---|---|---|---|
| Standard node | `#1b4332` | `#f0faf4` | `solid` |
| Decision diamond | `#1b4332` | `#d8f3dc` | `solid` |
| Accent / CTA node | `#52b788` | `#f0faf4` | `solid` |
| Out-of-scope node | `#767676` | `transparent` | `dashed` |
| Success / outcome | `#52b788` | `#f0faf4` | `solid` |
| Warning / alert node | `#c53030` | `#f0faf4` | `solid` |
| Info node | `#1b4332` | `#d8f3dc` | `solid` |
| Container / group | `#d8f3dc` | `transparent` | `solid` |

### Arrow defaults

| Property | Value |
|---|---|
| `strokeColor` | `#1b4332` |
| `strokeWidth` | 2 |
| `strokeStyle` | `solid` |

### Canvas

| Property | Value |
|---|---|
| `viewBackgroundColor` | `#ffffff` |

---

## Layout Templates

Reusable structural templates for infographics and visual content. Each template defines element positions on the 1080×1080px grid.

### 1 — Numbered Steps

**Use when:** process, workflow, sequence, or "how it works."

| Element | Spec |
|---|---|
| Timeline spine | Horizontal, `y=540`, `x1=80` to `x2=1000`, stroke `#d8f3dc` width 3 |
| Step circles | Outer ring `r=56` stroke `#52b788` width 3, inner fill `r=46` `#1b4332`, white number 28px weight 800 |
| Max steps | 4 (3 is ideal) |

### 2 — Comparison Table

**Use when:** comparing two options, channels, tiers, or products feature-by-feature.

| Element | Spec |
|---|---|
| Column header strip | y=155, height=56px, fill `#f0faf4` |
| Active chip | fill `#1b4332`, white L3 text |
| Muted chip | fill `#e8e8e8`, `#767676` L3 text |
| Row height | 72px, max 5 rows |
| Result cells | Positive: filled circle `#52b788` / Negative: filled circle `#c53030` |

### 3 — KPI Cards

**Use when:** 2–3 statistics that tell the story on their own.

| Element | Spec |
|---|---|
| Card size | 296×200px, r=8 rounded corners |
| Card accent bar | Top, height=8px, fill `#1b4332` (or `#c53030` for alert) |
| Card gap | 24px between cards |
| Card content | L6 number → 8px gap → L3 label |

### 4 — Bold Statement

**Use when:** single powerful idea or provocation — one thing the reader should remember.

| Element | Spec |
|---|---|
| Title | 64–80px, weight 800, centred, max 5 words |
| Optical centre | y=440–500 |
| Supporting line | L4, 24px below title |
| Left accent bar | x=0, y1=360, y2=600, width=8, fill `#52b788` |

### 5 — Icon Grid

**Use when:** 4–6 standalone tips, principles, or concepts with no hierarchy.

| Element | Spec |
|---|---|
| Grid | 2×2 (4 items) or 2×3 (6 items) |
| Per cell | Icon 72×72px `#52b788` → L3 label → L4 one-liner |
| Cell backgrounds | Alternating `#ffffff` / `#f0faf4` |

### 6 — Timeline

**Use when:** chronological sequence with dates, milestones, or before/after arc.

| Element | Spec |
|---|---|
| Vertical spine | x=540, y1=155 to y2=960, stroke `#d8f3dc` width 3 |
| Milestone dot | r=12, fill `#52b788` |
| Layout | Odd items left, even items right |
| Max milestones | 4 |

### 7 — Magazine Split

**Use when:** one dominant visual or stat paired with a short supporting list (2–4 items).

| Element | Spec |
|---|---|
| Left panel | 488px wide, fill `#1b4332`, hero element (L6 number or 120px white icon) |
| Right panel | 592px wide, `#ffffff`, list items with dot + L3 label + L4 one-liner |

### 8 — Quote Card

**Use when:** the user explicitly provides a quote that IS the message. Never generate a quote.

| Element | Spec |
|---|---|
| Opening quote mark | 120px, `#d8f3dc`, top-left at x=80, y=200 |
| Quote text | 28–36px, weight 400, italic, centred, `#1b4332`, max 3 lines |
| Background | `#f0faf4` full canvas tint, 8px left bar `#52b788` |

### Template Selection Guide

| Content type | Best layout |
|---|---|
| Process / steps | Numbered Steps |
| Feature comparison | Comparison Table |
| 2–3 statistics | KPI Cards |
| Single strong insight | Bold Statement |
| 4–6 unranked tips | Icon Grid |
| Chronological / milestones | Timeline |
| One stat + short list | Magazine Split |
| User provides a quote | Quote Card |

---

## UX & Readability Rules

### Gestalt Principles

Apply to every layout:

- **Dominance**: one element must be the clear visual leader
- **Proximity**: related elements <=16px apart; >=32px separates sections
- **Alignment**: strict grid alignment
- **Repetition**: one visual motif (circle, card, pill, dot) applied consistently
- **Contrast**: every zone must have one element with clear visual weight

### The Scroll-Stop Zone

The top 35% of the canvas (top ~380px) is the scroll-stop zone:
- The L1 headline and one strong visual anchor must both land here
- The headline must be legible when the image is previewed at ~270×270px
- If the key message isn't clear at a glance — shorten, don't add more elements

### The Squint Test

Squint until the image blurs. One element should stand out. If nothing dominates, or the wrong thing does — increase contrast or size of the primary element.

---

## Document Templates

### Template references

| Format | Template path | Use for |
|---|---|---|
| Word (.dotx) | _(to be added)_ | Personal reports, blog drafts |
| PowerPoint (.potx) | _(to be added)_ | Personal presentations, talks |

### Word style mapping

| Style name | Font | Weight | Size | Colour |
|---|---|---|---|---|
| Heading 1 | System sans-serif | 800 | 28pt | `#1b4332` |
| Heading 2 | System sans-serif | 700 | 22pt | `#1b4332` |
| Heading 3 | System sans-serif | 600 | 16pt | `#52b788` |
| Body | System sans-serif | 400 | 11pt | `#2d2d2d` |
| Label / tag | System sans-serif | 700 (UPPERCASE) | 9pt | `#767676` |

### PowerPoint style mapping

| Element | Font | Weight | Size | Colour |
|---|---|---|---|---|
| Slide title | System sans-serif | 800 | 36pt | `#1b4332` |
| Slide subtitle | System sans-serif | 600 | 20pt | `#52b788` |
| Body text | System sans-serif | 400 | 16pt | `#2d2d2d` |
| KPI number | System sans-serif | 800 | 72pt | `#1b4332` |
| Tag / label | System sans-serif | 700 (UPPERCASE) | 10pt | `#767676` |

### Slide backgrounds

| Slide type | Background |
|---|---|
| Default content | `#ffffff` |
| Section divider | `#f0faf4` |
| Dark section | `#1b4332` with white text |
| Accent section | `#52b788` with `#1b4332` text |
