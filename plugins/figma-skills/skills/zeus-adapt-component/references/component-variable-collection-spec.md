# Component Variable Collection Spec
### For creating new `{Component}.Styles` / `{Component}.Measures` collections in ZEUS 3.0

**Purpose:** This is a handoff spec for a Figma agent (via `use_figma`) to generate variable collections for a *new* component that conform to the conventions already established across Card, buttonGroup, Button, Statelayer, link, Tag, badge, InputField, Dropdown, Checkbox, AssistiveContent, and Toggle. It does not cover component/frame construction — only the variable scaffolding that should exist before binding.

Pair with the token **naming convention** (`{component}-{element}-{property}-{modifier}`, directional abbreviations `rgt`/`lft`/`top`/`btm`) from `naming-convention.md`. This spec adds: which tokens to create, in which collection, with which scope and mode structure.

---

## 1. Collection Structure

Every component gets exactly **two** collections, named `{Component}.Styles` and `{Component}.Measures`. Never mix color and dimension tokens in one collection — this file has zero exceptions to that rule.

**Exception to "every component":** a composed component that wraps a base control does **not** get its own pair. See Section 6.

---

## 2. Component Classification

Before selecting tokens, classify the component:

| Classification | Has background/border? | Example | Skip |
|---|---|---|---|
| **Container** — has its own surface | Yes | Card, Button, Tag, badge, InputField, Dropdown | — |
| **Content-only** — sits on the parent's surface | No | link, AssistiveContent | `background-color`, `border-color` |
| **Fixed-shape control** — small, non-text-driven shape | Partial (border yes, background sometimes no) | Checkbox, Toggle | `size-height` (use `box-size` instead), generic `border-radius` (use `focus-ring-*` naming) |

If unsure which bucket a new component falls into, default to **Container**.

---

## 3. Token Checklist by Category

### `{Component}.Styles` — create one variable per row that applies, per state/variant mode

| Token | Scope | Applies to | Skip if |
|---|---|---|---|
| `{component}-background-color` | `FRAME_FILL`, `SHAPE_FILL` | Container only | Content-only |
| `{component}-border-color` | `STROKE_COLOR` | Container, Fixed-shape | Content-only, unless it has a focus ring |
| `{component}-label-color` or `-text-color` | `TEXT_FILL` | Any component with text | Icon-only components |
| `{component}-icon-color` | `FRAME_FILL`, `SHAPE_FILL` | Any component with icons | No icons in any variant |
| `{component}-state-opacity` or `-box-opacity` | `OPACITY` | Components with a disabled/dimmed state | Not needed if disabled uses distinct colors instead — and never whole-component opacity, see `cleanup-rules.md` Rule 1 |
| `{component}-focus-ring-color` | `STROKE_COLOR` | Fixed-shape controls, form fields | Containers that show focus via border-color mode swap instead |

**Never create local `{component}-state-layer-*` variables** — the State Layer is a separate published component with its own shared collections (`Statelayer.Styles` / `Statelayer.Measures`). Those are not duplicated per-component. Dropdown's existing local state-layer variables are legacy debt, not a pattern to follow.

**Never create variables for properties owned by a foreign nested component** — including AssistiveContent's text color, typography, and internal spacing. See Step 2g of the orchestrator.

**Element-scoped variants:** when a component has repeatable sub-elements (icon position, list item, header/footer), prefix the variable name with the element, e.g. `LeadingIcon/button-padding-rgt`, `ListItem/listItem-icon-color`. This is a Figma variable **group** (folder), not a name segment — the underlying token name still drops the group prefix.

### `{Component}.Measures` — create one variable per row that applies

| Token | Scope | Applies to | Skip if |
|---|---|---|---|
| `{component}-padding-lft-rgt` / `-padding-top-btm` | `GAP` | Any component with internal padding | — |
| `{component}-gap` (or `-item-spacing`) | `GAP` | Any component with 2+ internal children | Single-element components |
| `{component}-border-radius` | `CORNER_RADIUS` | Container, Fixed-shape | Content-only with no visible edge |
| `{component}-border-width` | `STROKE_FLOAT` | Anywhere `border-color` exists | — |
| `{component}-size-height` | `WIDTH_HEIGHT` | Components with a fixed/controlled height (buttons, fields, tags) | Fixed-shape controls — use `{component}-box-size` instead |
| `{component}-icon-size` | `WIDTH_HEIGHT` | Any component with icons | No icons |
| `{component}-size-width-max` / `-min` | `WIDTH_HEIGHT` | Components that truncate or constrain width (buttons, tags, links, action fields) | Fixed-width or intrinsically-sized components |

---

## 4. Scope Assignment Rule

**Always set `variable.scopes` explicitly — never leave the default `ALL_SCOPES`.** Some existing collections (Card, buttonGroup, badge, InputField, Dropdown, Checkbox, AssistiveContent) were created with `ALL_SCOPES` throughout; treat that as legacy debt, not the pattern to copy. `Button.Styles`, `Button.Measures`, `Tag.Styles`, and `Statelayer.Measures` use precise scopes — **match those**.

| Property type | Scope |
|---|---|
| Background/shape fill | `["FRAME_FILL", "SHAPE_FILL"]` |
| Text color | `["TEXT_FILL"]` |
| Border/stroke color | `["STROKE_COLOR"]` |
| Border/stroke weight | `["STROKE_FLOAT"]` |
| Padding, gap, item spacing | `["GAP"]` |
| Corner radius | `["CORNER_RADIUS"]` |
| Width/height, icon/box size | `["WIDTH_HEIGHT"]` |
| Opacity | `["OPACITY"]` |
| Multi-purpose fill (e.g. state layers used on fills and strokes) | `["ALL_FILLS", "STROKE_COLOR"]` |

---

## 5. Modes vs. Groups — Decision Criteria

Both collections support two axes of variation: **modes** (collection-wide — every variable gets a value per mode) and **groups** (a name-prefix folder, e.g. `LeadingIcon/button-padding-rgt` — every variable can belong to a different group independently). Picking the wrong one is the most common structural mistake, so decide deliberately per component.

### Measures: modes = scaling only, groups = structural variants

- **Modes** are reserved for a genuine *scaling* axis — density variants like `Default` / `Condensed`. Nothing else belongs in a Measures mode.
- **Groups** are for structural/content variants where the variant changes *which* dimension values apply — not how big everything is, but which layout the component is in. A label-only button, a leading-icon button, and an icon-only button each need their own padding/gap values, so each gets its own group: `LeadingIcon/`, `Icon/`, etc. — exactly as `Button.Measures` already does.
- **Criterion:** if the variant changes *which* value set applies, use a group. If it changes the scale of everything proportionally, use a mode.

### Styles: modes = UI states (only when the State Layer can't cover it), groups = semantic variants

| Scenario | Styles modes |
|---|---|
| State is conveyed **only** through background color | The State Layer handles it. **Do not create UI state modes.** Use modes for something else if needed (e.g. color variants `Default` / `AI` / `Error`). |
| State is conveyed through background color **and** other properties (border color, icon color, text color) the State Layer doesn't control | **Create UI state modes**: `Enabled`, `Hovered`, `Focused`, `Pressed`, `Disabled`, plus component-specific states (`Active` / `Checked` / `Unchecked` / `Indeterminate`). |
| Component has visual/color variants but no multi-property state changes | Use modes for those variants instead of UI states. |

**Styles groups** = semantic/color variants when modes are already used for UI states. Canonical example: the Card's border color in `Default` vs. `AI` (purple) — both need the full set of state-mode values, so they become groups `Default/` and `AI/`.

If modes are *not* used for UI states, color variants can be modes instead of groups — whichever axis changes more frequently should be the mode axis.

---

## 6. Composed Components — One Family, One Collection Pair

Some components ship as a pair: a **base control** (the bare UI control) and a **composed wrapper** that contains it alongside adjacent content. The wrapper is usually what product teams consume.

**They share one collection pair, named after the base control, split by groups.** Never create a second pair for the wrapper.

**Canonical example — Toggle.** The bare switch, plus a composed component containing it in a with-label and without-label variant:

```
Toggle.Measures
├── Toggle/            ← the control itself
│     toggle-track-size-width
│     toggle-track-size-height
│     toggle-border-width
└── Label/             ← the composed wrapper's added content
      toggle-label-gap
      toggle-label-padding-lft
```

- The label variant (`text=true` / `text=false`) is a **variant of the wrapper** — not a collection, not a mode.
- Values that exist only when the label is present live in the `Label/` group.
- The control's values stay in the control's group and are never duplicated into the wrapper's group.
- If the wrapper also contains a **foreign** component (AssistiveContent, State Layer), that one's variables stay entirely out of this collection.

**Distinguishing same-family from foreign:** would the nested thing appear inside components that have nothing to do with this one? If yes → foreign, hands off its variables. If it only ever appears inside this family → same-family, shared collection with groups.

---

## 7. Execution Order

1. **Classify** the component (Container / Content-only / Fixed-shape) per Section 2.
2. **Inspect first** — run a read-only script (`getLocalVariableCollectionsAsync`) to confirm no collection for this component already exists, and that mode-naming conventions haven't drifted since this spec was written.
3. **Determine family position** per Section 6 — if this is a composed wrapper whose base control already has collections, extend those with a group instead of creating new ones.
4. **Decide the mode axis for each collection before creating anything**, per Section 5: for Measures, confirm whether a density/scaling variant actually exists (if not, a single `Default` mode). For Styles, check whether any property value genuinely diverges between interaction states beyond what the State Layer covers — if nothing diverges, use a single mode and rely on the shared Statelayer component.
5. **Decide the group axis for each collection**, per Section 5: identify structural/content variants (Measures) or semantic/visual variants (Styles) that need their own full value set, and name groups after the variant driving the difference.
6. **Create the two collections** (`{Component}.Styles`, `{Component}.Measures`) and rename the default mode immediately — never leave `Mode 1`.
7. **Create Styles variables** from the Section 3 checklist, filtered by classification, excluding state-layer and foreign-owned tokens. Set explicit scopes per Section 4. Set values per mode and group.
8. **Create Measures variables** from the Section 3 checklist, filtered by classification. Set explicit scopes. Set values per mode and group.
9. **Return all created variable IDs and collection IDs** in the script's return value for downstream binding.
10. **Validate**: confirm scopes are non-default, no state-layer duplication exists, no second collection was created for a composed wrapper, and naming matches `{component}-{element}-{property}-{modifier}`. Then run `validation-checklist.md`.

## 8. Never do this:
1. **Always create the smallest amount of variables** Don't create variables that aren't used. Reduce to a minimum.
2. **Never create multiple variables, for the same value** always reduce to the lowest amount example: If the component needs padding in all four directions (lft,rgt,top,btm), don't create the variable 4 times for all directions, create just one. If the component needs a different padding for vertical (top-btm) and horizontal (lft-rgt) create separate variables (most common). Only create 4 padding variables if the padding is different for every direction (very rare)
3. **Never create opacity variables** except if you are asked for. Every component, content, etc. should be at 100% opacity.
