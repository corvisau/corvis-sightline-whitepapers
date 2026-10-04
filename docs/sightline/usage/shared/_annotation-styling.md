An annotation bubble takes its colour and leading icon from three optional fields: `style`, `colour` and `icon`. All three are free text in YAML. Tetra and Lamina share the fields and the drawer controls. For how the bubble's text renders, see [Annotations](/docs/sightline/usage/shared/annotations).

---

## Schema

```yaml
annotations:
  - id: A-0001
    type: text
    value: Example annotation text.
    style: Warn        # optional: a preset name
    colour: Red        # optional: overrides the preset colour
    icon: shield       # optional: overrides the preset icon
```

| Field | Required | Notes |
|---|---|---|
| `style` | No | A preset name from the table below. Matched without regard to case. An unrecognised name adds no colour and no icon. |
| `colour` | No | A colour token or any CSS colour. Overrides the colour that the preset supplies. |
| `icon` | No | A codicon id. Overrides the icon that the preset supplies. |

Each app's section below shows the same example with a `links` entry that suits that app.

## Precedence

Each field resolves on its own. An explicit `colour` replaces the preset colour, and an explicit `icon` replaces the preset icon. A field that you leave out takes its value from the preset.

`style: -`, `style: Plain` or no `style` gives the default bubble, with no colour and no icon. An explicit `colour` or `icon` still applies to a default bubble.

## Style presets

The style control offers the ten choices in the first column. `plain` is an accepted alias of `-` that the control does not offer.

| Style | Colour | Icon |
|---|---|---|
| `-` (alias `plain`) | default | none |
| `Low` | Blue | none |
| `Med` | Yellow | none |
| `High` | Red | none |
| `Good` | Green | `pass` |
| `Bad` | Red | `error` |
| `Warn` | Yellow | `warning` |
| `Info` | Blue | `info` |
| `Error` | Red | `error` |
| `Note` | Grey | `note` |

## Colours

A colour token maps to a VS Code theme colour, so the bubble follows the active theme. The fallback applies where the theme does not define the colour.

| Token | Theme colour | Fallback |
|---|---|---|
| `Blue` | `--vscode-editorInfo-foreground` | `#3794ff` |
| `Yellow` | `--vscode-editorWarning-foreground` | `#f5a623` |
| `Red` | `--vscode-editorError-foreground` | `#d83b3b` |
| `Green` | `--vscode-testing-iconPassed` | `#2ea043` |
| `Grey` (alias `gray`) | `--vscode-descriptionForeground` | `#808080` |

`Plain`, `neutral` and `-` give the default bubble. Any other value is used as a literal CSS colour, for example `#7048e8`, `rebeccapurple` or `rgb(20 120 200)`. The drawer suggests `Grey`, `Plain`, `Green`, `Blue`, `Red` and `Yellow`.

## Icons

`icon` takes the id of any VS Code codicon, for example `shield`. Browse the ids in the [codicon reference](https://microsoft.github.io/vscode-codicons/dist/codicon.html). The presets use `pass`, `error`, `warning`, `info` and `note`. An id that is not a codicon renders an empty glyph.

## How it renders

The resolved colour sets a 1.5 px border, a 12% tint of the bubble background and the colour of the leader line. The icon appears as a glyph before the text.

## Authoring in the properties drawer

Select an annotation to open the properties drawer, or choose **Add annotation** to open the create drawer. Both show the same three controls. See [Create in drawer](/docs/sightline/usage/shared/create-in-drawer) for the create flow.

- The **Style** field is a text box that suggests the presets above. It accepts any other text.
- The **Colour** field is a text box that suggests the colours above. It accepts any other text.
- The **Icon** field is a picker that lists every codicon. It also accepts free text.

## drawio export

The export includes annotations. A colour token exports as its fallback hex value, with an approximate lightened fill. A literal CSS colour with no hex value exports as a border only. A `var()` colour with no hex fallback exports as the default bubble. The icon is never exported, because draw.io has no codicon font.

## Legacy `severity`

A model that still carries `severity` renders as if it had the matching `style`. Replace `severity` with `style` by hand.

| `severity` | Renders as `style` |
|---|---|
| `low`, `info` | `Low` |
| `med`, `warn`, `warning` | `Med` |
| `high`, `crit` | `High` |

## Tetra

This example links the annotation to a device.

```yaml
annotations:
  - id: A-0001
    type: text
    value: Patch window agreed with the vendor.
    style: Warn        # optional: a preset name
    colour: Red        # optional: overrides the preset colour
    icon: shield       # optional: overrides the preset icon
    links:
      - ref: D-00042
```

### Link style

The **Style** field on a link inside the annotation is a different field. It chooses how the bubble labels the linked entity, with `value` or `id`. It does not colour anything.

### Glow

A link with `glow: true` lights its target, which can be a node, a channel or a flow. The ring uses the annotation's resolved colour, or a neutral yellow when the annotation has none.

When several annotations glow the same target, the strongest style wins.

| Style | Rank |
|---|---|
| `High`, `Error`, `Bad` | 3 |
| `Med`, `Warn` | 2 |
| `Low`, `Info` | 1 |
| Every other style, or none | 0 |

## Lamina

This example links the annotation to a cause.

```yaml
annotations:
  - id: A-0001
    type: text
    value: Barrier is out of service for maintenance.
    style: Warn        # optional: a preset name
    colour: Red        # optional: overrides the preset colour
    icon: shield       # optional: overrides the preset icon
    links:
      - ref: C-0007
```

### Setting the style from the canvas

Right-click an annotation and choose **Set style**. The submenu lists **None** and the presets above. **None** clears the `style` field. The menu item does nothing, and is disabled, in a read-only session.

### Template Editor

The Template Editor shows the same **Style**, **Colour** and **Icon** controls on an annotation template.

### drawio export

**Sightline: Export Draw.io** exports each bubble as a rounded cell. The cell shows the annotation text. It shows the annotation id in id display mode, and for a `statusgrid` annotation. The leader line from a bubble to its owner is not exported. The bubble keeps the size that it has on the canvas.
