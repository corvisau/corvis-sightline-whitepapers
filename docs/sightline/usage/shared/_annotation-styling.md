An annotation bubble takes its colour and leading icon from three optional controls: **Style**, **Colour** and **Icon**. Tetra and Lamina share the controls. Tetra adds a fourth control, **Glow**, which lights the entities that the annotation links to. For how the bubble's text renders, see [Annotations](/docs/sightline/usage/shared/annotations).

---

## Set the style in the properties drawer

Select an annotation to open the properties drawer, or choose **Add annotation** to open the create drawer. Both show the same controls. See [Create in drawer](/docs/sightline/usage/shared/create-in-drawer) for the create flow.

- The **Style** field is a text box that suggests the presets below. It accepts any other text.
- The **Colour** field is a text box that suggests the colours below. It accepts any other text.
- The **Icon** field is a picker that lists every codicon. It also accepts free text.

All three fields are optional.

## Precedence

Each control resolves on its own. A value in **Colour** replaces the preset colour, and a value in **Icon** replaces the preset icon. A control that you leave empty takes its value from the preset.

A **Style** of `-` or `Plain`, or an empty **Style**, gives the default bubble, with no colour and no icon. A value in **Colour** or **Icon** still applies to a default bubble. The matching ignores case. An unrecognised preset name adds no colour and no icon.

## Style presets

The **Style** field offers the ten choices in the first column.

| Style | Colour | Icon |
|---|---|---|
| `-` | default | none |
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

A colour name follows the active VS Code theme, so the bubble changes colour when you change theme. The **Colour** field suggests `Grey`, `Plain`, `Green`, `Blue`, `Red` and `Yellow`.

Any other text is used as a literal CSS colour, for example `#7048e8`, `rebeccapurple` or `rgb(20 120 200)`. `Plain` and `neutral` give the default bubble.

## Icons

The **Icon** field takes the name of any VS Code codicon, for example `shield`. Browse the names in the [codicon reference](https://microsoft.github.io/vscode-codicons/dist/codicon.html). The presets use `pass`, `error`, `warning`, `info` and `note`. A name that is not a codicon shows an empty glyph.

## How it renders

The resolved colour sets a 1.5 px border, a 12% tint of the bubble background and the colour of the leader line. The icon appears as a glyph before the text.

## drawio export

The export includes annotations. A named colour exports as a fixed hex value, with an approximate lightened fill. A literal CSS colour with no hex value exports as a border only. A theme-variable colour with no hex fallback exports as the default bubble. The icon is never exported, because draw.io has no codicon font.

## Older models

A model that still carries the older severity setting shows the matching style on the canvas. To change it, set **Style** in the properties drawer.

## Tetra

### Link style

The **Style** field in the link settings dialog is a different control. Select the settings button on a link in the annotation's properties drawer to open the dialog. **Style** chooses how the bubble labels the linked entity, with `value` or `id`. It does not colour anything.

### Glow

Tick **Glow** in the annotation's properties drawer, beside **Style**, **Colour** and **Icon**, to light every entity that the annotation links to. The linked entities can be nodes, channels and flows. The ring uses the annotation's resolved colour, or a neutral yellow when the annotation has none. The glow appears after you save.

Glow is one setting for the whole annotation. The link settings dialog has no Glow control. Lamina does not show **Glow**, because Lamina does not draw it.

When several annotations glow the same target, the strongest style wins.

| Style | Rank |
|---|---|
| `High`, `Error`, `Bad` | 3 |
| `Med`, `Warn` | 2 |
| `Low`, `Info` | 1 |
| Every other style, or none | 0 |

## Lamina

### Setting the style from the canvas

Right-click an annotation and choose **Set style**. The submenu lists **None** and the presets above. **None** clears the style. The menu item does nothing, and is disabled, in a read-only session.

### Template Editor

The Template Editor shows the same **Style**, **Colour** and **Icon** controls on an annotation template.

### drawio export

**Sightline: Export Draw.io** exports each bubble as a rounded cell. The cell shows the annotation text. It shows the annotation id in id display mode, and for a `statusgrid` annotation. The leader line from a bubble to its owner is not exported. The bubble keeps the size that it has on the canvas.
