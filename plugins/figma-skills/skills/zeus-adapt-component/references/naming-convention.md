# Token Naming Convention
### ZEUS 3.0 component variable names

> **Status: reconstructed.** The original of this document was written in an earlier
> session and is not recoverable in full. The pattern and rules below are correct, but
> verify the reference examples against the live file before treating it as final.

## The Pattern

```
{component}-{element}-{property}-{modifier}
```

No `--` prefix. Four segments, in this order, always.

| Segment | Meaning | Example |
|---|---|---|
| `{component}` | The component name, casing matched to the Figma frame name exactly | `toggle`, `button`, `inputField` |
| `{element}` | The sub-element the value applies to. Omitted when the value applies to the component root | `track`, `knob`, `label`, `icon` |
| `{property}` | What is being set | `background-color`, `border-width`, `size-height`, `gap`, `padding` |
| `{modifier}` | Direction, bound, or dimension qualifier. Omitted when not applicable | `rgt`, `lft`, `top`, `btm`, `min`, `max` |

## Rules

1. **Segments may be omitted, never reordered.** A missing element or modifier closes up;
   the surviving segments keep their relative order.
2. **Directional abbreviations are always abbreviated:** `rgt`, `lft`, `top`, `btm`.
   Never `right`, `left`, `bottom`.
3. **Never encode variant context in the name.** No `-hover`, `-disabled`, `-checked`,
   `-ai`, `-condensed` in the token name. Variant context is expressed externally, through
   **modes** and **groups**. A name describes structure; a mode or group describes context.
4. **Compound properties keep their full property name:** `background-color`, not `bg-color`;
   `border-radius`, not `radius`.
5. **Dimensional sizing uses `size-`:** `size-height`, `size-width-max`, `size-width-min`.
   Fixed-shape controls use `box-size` instead of `size-height`.
6. **Group prefixes are not name segments.** A variable inside a Figma group appears as
   `Label/toggle-label-gap`, but the token name itself is `toggle-label-gap` — the group
   prefix is a folder, not part of the name.

## Worked Examples

| Variable | component | element | property | modifier |
|---|---|---|---|---|
| `toggle-track-size-width` | toggle | track | size-width | — |
| `toggle-label-padding-lft` | toggle | label | padding | lft |
| `button-background-color` | button | — | background-color | — |
| `button-size-width-max` | button | — | size-width | max |
| `inputField-icon-color` | inputField | icon | color | — |
| `checkbox-box-size` | checkbox | — | box-size | — |

## Anti-Examples

| Wrong | Why | Right |
|---|---|---|
| `toggle-track-width-size` | Segments reordered | `toggle-track-size-width` |
| `button-padding-right` | Direction not abbreviated | `button-padding-rgt` |
| `button-background-color-hover` | Variant context in the name | `button-background-color` + a `Hovered` mode |
| `tag-ai-border-color` | Variant context in the name | `tag-border-color` + an `AI/` group |
| `card-radius` | Property name truncated | `card-border-radius` |
