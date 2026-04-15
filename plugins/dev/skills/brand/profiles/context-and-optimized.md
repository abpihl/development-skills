---
name: context-and
display-name: "Context&"
default: true
---

# Context& Brand Profile

Human and professional, yet techy. Sleek, modern design mixed with approachable visuals.

Tagline: *Better connected in The Digital Neighbourhood*

## Quick-reference tokens

Use these semantic tokens when selecting colours by purpose:

- `primary-stroke`: `#000000` (Black)
- `primary-fill`: `#F1F5F7` (Very Light Grey)
- `accent`: `#5500FF` (Purple)
- `background`: `#FFFFFF` (White)
- `secondary-fill`: `#C9D6DE` (Light Grey)
- `muted-stroke`: `#3D424B` (Grey)
- `highlight`: `#00B6FF` (Sky Blue)
- `success`: `#1E8C7F` (Green)
- `warning`: `#FF922D` (Orange)
- `info`: `#043F9C` (Dark Blue)

## Colours

### Full palette

| Name | HEX | Tier | Role |
|---|---|---|---|
| White | `#FFFFFF` | primary | Primary background |
| Black | `#000000` | primary | Primary text, logotype |
| Purple | `#5500FF` | primary | Accent / CTA — used sparingly |
| Grey | `#3D424B` | primary | Dark backgrounds, secondary text |
| Light Grey | `#C9D6DE` | primary | Section backgrounds, containers |
| Very Light Grey | `#F1F5F7` | primary | Page backgrounds, subtle fills |
| Sky Blue | `#00B6FF` | secondary | Business area icons, graphic elements |
| Dark Blue | `#043F9C` | secondary | Business area icons, graphic elements |
| Green | `#1E8C7F` | secondary | Business area icons, graphic elements |
| Orange | `#FF922D` | secondary | Business area icons, graphic elements |
| Deep Teal | `#004951` | gradient-only | Green gradient endpoint |
| Deep Brown | `#9E4D36` | gradient-only | Orange gradient endpoint |

### Colour rules

- Secondary colours may be used at 20%, 40%, 60%, or 80% opacity on white backgrounds.
- Gradient-only colours must not be used outside gradients.

### Approved gradients

Only for business area icons and xPM icons.

| From | To |
|---|---|
| `#C9D6DE` | `#00B6FF` |
| `#00B6FF` | `#043F9C` |
| `#043F9C` | `#3D424B` |
| `#1E8C7F` | `#004951` |
| `#FF922D` | `#9E4D36` |

## Typography

**Font ratio rule:** Aktiv Grotesk 70% / PP Neue Machina 30%.

### Aktiv Grotesk (primary)

Authoritative, neutral. Use Light, Regular, and Medium weights.
Fallback: Neue Haas Grotesk Text Pro Roman.

| Usage | Weight | Line height | Letter spacing |
|---|---|---|---|
| Headlines | Light | 130% | 0% |
| Sub header / body text | Light | 145% | 0% |
| Box text | Regular | 160% | 0% |
| Small texts | Regular | 160% | 0% |
| Buttons | Medium | 130% | 5% |
| Text links | Medium | 160% | 5% |

### PP Neue Machina (secondary)

Monospace/geometric. 'Plain Regular' fontstyle. Use sparingly.
Fallback: Neue Haas Grotesk Text Pro Bold.

| Usage | Case | Line height | Letter spacing |
|---|---|---|---|
| Highlighted keywords in big titles | Normal | 130% | 0% |
| Highlighted numbers | Normal | — | 0% |
| Quotes | Normal | 130% | 0% |
| Alternate short titles | Normal | 160% | 0% |
| Tags & markers | UPPERCASE | 130% | 20% |

## Diagram styles (Excalidraw)

### Element defaults

```yaml
strokeColor: "#000000"
backgroundColor: "#F1F5F7"
strokeWidth: 2
roughness: 0        # clean lines — matches sleek, modern identity
fillStyle: solid
roundness: { type: 3 }  # rounded corners — matches brand framing
opacity: 100
fontFamily: 2       # Helvetica — closest to Aktiv Grotesk in Excalidraw
```

### Node presets

```yaml
standard:     { strokeColor: "#000000", backgroundColor: "#F1F5F7", strokeStyle: solid }
decision:     { strokeColor: "#000000", backgroundColor: "#C9D6DE", strokeStyle: solid }
accent:       { strokeColor: "#5500FF", backgroundColor: "#F1F5F7", strokeStyle: solid }
out-of-scope: { strokeColor: "#3D424B", backgroundColor: transparent, strokeStyle: dashed }
success:      { strokeColor: "#1E8C7F", backgroundColor: "#F1F5F7", strokeStyle: solid }
warning:      { strokeColor: "#FF922D", backgroundColor: "#F1F5F7", strokeStyle: solid }
info:         { strokeColor: "#043F9C", backgroundColor: "#F1F5F7", strokeStyle: solid }
container:    { strokeColor: "#C9D6DE", backgroundColor: transparent, strokeStyle: solid }
```

### Arrows

```yaml
strokeColor: "#3D424B"
strokeWidth: 2
strokeStyle: solid
```

### Canvas

```yaml
viewBackgroundColor: "#FFFFFF"
```

## Document styles

### Word

| Style | Font | Weight | Size | Colour |
|---|---|---|---|---|
| Heading 1 | Aktiv Grotesk | Light | 28pt | `#000000` |
| Heading 2 | Aktiv Grotesk | Light | 22pt | `#000000` |
| Heading 3 | Aktiv Grotesk | Regular | 16pt | `#000000` |
| Body | Aktiv Grotesk | Light | 11pt | `#000000` |
| Body (box) | Aktiv Grotesk | Regular | 11pt | `#000000` |
| Accent heading | PP Neue Machina | Regular | 14pt | `#5500FF` |
| Tag / label | PP Neue Machina | Regular (UPPERCASE) | 9pt | `#3D424B` |

### PowerPoint

| Element | Font | Weight | Size | Colour |
|---|---|---|---|---|
| Slide title | Aktiv Grotesk | Light | 36pt | `#000000` |
| Slide subtitle | Aktiv Grotesk | Light | 20pt | `#3D424B` |
| Body text | Aktiv Grotesk | Light | 16pt | `#000000` |
| Keyword highlight | PP Neue Machina | Regular | 36pt | `#000000` |
| Tag / section label | PP Neue Machina | Regular (UPPERCASE) | 10pt | `#3D424B` |
| CTA button | Aktiv Grotesk | Medium | 14pt | `#FFFFFF` on `#000000` bg |

### Slide backgrounds

| Slide type | Background |
|---|---|
| Default content | `#FFFFFF` |
| Section divider | `#F1F5F7` + Light Grey ampersand watermark |
| Dark section | `#3D424B` + white text |
| Accent section | `#5500FF` + white text |

## Icons

- Custom stroke design with soft corners
- Stroke colour: `#5500FF`
- Service icons (large): stroke weight 1.5
- Resource icons (small): stroke weight 1
- Decorative elements: shapes from winning behaviour icons and the ampersand 'c' — use in any primary palette colour

## Photography

- Urban, cool, authentic, unposed, casual
- Natural warm light, city environments with architectural details
- Rounded corners: 40px desktop, 30px mobile
- Never overlay/tint images — reposition text instead
- Never place logo on people from chest up

## Logo

| Background | Version |
|---|---|
| White / Very Light Grey / Light Grey | Black + Purple (two-colour) |
| Dark Grey / Black / Purple / any other colour | White (one-colour) |
| Images (strong contrast) | White or White + Purple |

Rules:
- Logotype must never appear without the ampersand.
- Ampersand icon can be used standalone as a decorative element.
