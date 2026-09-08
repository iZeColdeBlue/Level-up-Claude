# Foundation Token Map
### For Step 5 — Assign Foundation Tokens

**Source:** live inspection of the ZEUS 3.0 Foundations file, `dULgduzwkPkKWVOhy8MSHR`.
5 collections, 102 variables. Regenerate when Foundations changes.

**Core rule:** match by **intent**, not by value. A token whose number happens to match is
not the right token if its category is wrong. A wrong-category binding is worse than a
snowflake.

---

## 1. Collections

| Collection | Modes | Vars |
|---|---|---|
| Color | Light, Dark, Light - High Contrast, Dark - High Contrast | 41 |
| Spacing | Compact, Comfortable | 20 |
| Sizing | *(single, unnamed — `Mode 1`)* | 16 |
| Opacity | *(single, unnamed — `Mode 1`)* | 13 |
| Border | *(single, unnamed — `Mode 1`)* | 12 |

Every Foundations variable aliases a primitive in an external primitives library. The
numeric value is encoded in the token name (`Padding/12` = 12px, `Radius/8` = 8px), so the
name is the value for everything except Opacity.

---

## 2. Color

**Semantic scale** — `lightest` → `light` → `medium` → `dark` → `darkest`, available on:
`Brand/`, `AI/`, `Success/`, `Amber/` (named `warning--*`), `Error/`, `Information/`.

Mapping intent across the scale: **lightest** → backgrounds; **medium** → icons and borders;
**dark** / **darkest** → text.

Plus `Brand/primary` as a standalone.

**Neutral** has its own structure and is the only color group with precise scopes:

| Token | Use for |
|---|---|
| `Neutral/neutral--background` | Page/app background |
| `Neutral/neutral--surface` | Component surface |
| `Neutral/On-Surface/base` | Base content on a surface |
| `Neutral/On-Surface/high-emphasis` | Primary text |
| `Neutral/On-Surface/high-medium-emphasis` | Secondary text |
| `Neutral/On-Surface/medium-emphasis` | Tertiary text, icons |
| `Neutral/On-Surface/low-high-emphasis` | De-emphasized content |
| `Neutral/On-Surface/disabled` | Disabled text and icons — **never borders** |
| `Neutral/On-Surface/outline` | Borders |
| `Neutral/On-Surface/subtle-outline` | Subtle borders, dividers |

**Border-color bindings may only use `outline` or `subtle-outline`** — in every mode,
including Disabled. `On-Surface/disabled` is not permitted for a border, even though it
exists and may visually match. (`cleanup-rules.md` Rule 3.)

The purchased library's purple brand color maps to `Brand/primary` (blue).

---

## 3. Spacing → padding and gap

| Group | Available steps |
|---|---|
| `Padding/` | 0, 1, 2, 4, 5, 6, 8, 10, 12, 16, 20 |
| `Gap/` | 0, 2, 4, 6, 8, 10, 12, 16, 20 |

Both scoped `GAP`. **Padding tokens are for padding; Gap tokens are for gap.** They overlap
numerically — do not substitute one for the other.

`Padding/` has 1 and 5; `Gap/` does not. A 1px or 5px gap has no token.

> ⚠️ **The Compact and Comfortable modes currently resolve to identical primitives.** Every
> Spacing variable aliases the same primitive in both modes, so the density axis exists
> structurally but carries no difference today. Consequence for Step 4: adding a `Condensed`
> mode to a component's Measures collection gains nothing from Spacing tokens until
> Foundations differentiates these modes. Flag to the operator rather than working around it.

---

## 4. Sizing → heights and icon sizes

| Group | Available steps |
|---|---|
| `Height/` | 16, 24, **30 (default)**, 32, 40, 48, 64, 96 |
| `Icon Size/` | 12, 16, 20, 24, 28, 32, 36, 64 |

- `Height/30 (default)` — the name includes the parenthetical. Match it exactly.
- **Non-icon elements needing a fixed size use `Height/`, never `Icon Size/`** — even when
  the number matches. (`cleanup-rules.md` Rule 4.)
- **There are no width tokens.** No `Width/` group exists in Sizing. Any
  `size-width` / `size-width-min` / `size-width-max` variable has no foundation token and is
  a snowflake by default. Do not reach for `Icon Size/36` to fill a 36px width.

---

## 5. Border → radius and stroke width

| Group | Available steps | Scope |
|---|---|---|
| `Radius/` | 0, 2, 4, 6, 8, 10, 12, 14, 16 | `CORNER_RADIUS` (0 and 2 are `ALL_SCOPES` — legacy) |
| `Width/` | 0, 1, 2 | `STROKE_FLOAT` |

Only three stroke widths exist. A 0.5px border has no token — bind to `Border/Width/1` or
flag it.

---

## 6. Opacity

Values are percentages in the primitives library (`opacity-60` = 60 → 0.6).

| Token | Resolves to | Notes |
|---|---|---|
| `Content/hidden` | 0 | |
| `Content/subtle` | 0.06 | |
| `Content/disabled` | 0.38 | |
| `Content/overlay` | 0.48 | |
| `Content/medium` | 0.60 | |
| `Content/full` | **0.60** | ⚠️ **Misnamed.** `full` does not mean 1.0 — it is identical to `medium`. Never bind it expecting full opacity. |
| `State Layer/hidden` | 0 | |
| `State Layer/minimal` | 0.02 | |
| `State Layer/hovered` | 0.12 | |
| `State Layer/pressed` | **0.12** | Same as hovered |
| `State Layer/dragged` | 0.16 | |
| `State Layer/active` | 0.38 | |
| `State Layer/focused` | 1.00 | Full — the focus ring is opaque |

`State Layer/*` tokens belong to the State Layer's own collections. **Never bind them from a
host component.** They are listed here for reference only.

Remember ZEUS 3.0 never uses whole-component opacity for disabled state
(`cleanup-rules.md` Rule 1).

---

## 7. Scope Debt in Foundations

Some Foundations variables are still on `ALL_SCOPES`. Do not treat this as licence to leave
component variables unscoped — component collections always set scopes explicitly.

Currently unscoped: all `Brand/`, `AI/`, `Success/`, `Amber/`, `Error/`, `Information/` colors;
`Height/16`; `Icon Size/28`, `/32`, `/36`; `Content/medium`; `Content/full`; `Radius/0`; `Radius/2`.

Correctly scoped, use as the reference pattern: `Neutral/On-Surface/*`
(`ALL_FILLS` + `STROKE_COLOR`), all `Padding/` and `Gap/` (`GAP`), most `Icon Size/` and
`Height/` (`WIDTH_HEIGHT`), `Radius/4`+ (`CORNER_RADIUS`), all `Width/` (`STROKE_FLOAT`),
most `Opacity` (`OPACITY`).

---

## 8. Property → Token Quick Reference

| Component variable | Foundation token |
|---|---|
| `-background-color` | `Neutral/neutral--surface`, or a semantic `--lightest` |
| `-border-color` | `Neutral/On-Surface/outline` or `subtle-outline` **only** |
| `-label-color` / `-text-color` | `Neutral/On-Surface/high-emphasis` (or lower emphasis / `--dark`) |
| `-icon-color` | `Neutral/On-Surface/medium-emphasis` or a semantic `--medium` |
| `-padding-*` | `Padding/{n}` |
| `-gap` | `Gap/{n}` |
| `-border-radius` | `Radius/{n}` |
| `-border-width` | `Width/{0,1,2}` |
| `-size-height` | `Height/{n}` |
| `-icon-size` | `Icon Size/{n}` |
| `-box-size` | `Height/{n}` — no dedicated box token |
| `-size-width` / `-min` / `-max` | **none — snowflake** |
| `-opacity` | `Content/{...}` — never for whole-component disabled |

---

## 9. Known Gaps

| Gap | Consequence |
|---|---|
| No width tokens at all | Every width variable is a snowflake |
| No 1px or 5px gap token | `Padding/1` and `/5` exist; `Gap/` equivalents don't |
| No 0.5px stroke width | Round to `Width/1` or flag |
| No `Height/20` | 20px control heights are snowflakes (`Icon Size/20` is the wrong category) |
| Spacing Compact ≡ Comfortable | Density modes carry no real difference yet |
| `Content/full` = 0.6 | Misleading name; do not use for full opacity |

## Confirmed Snowflakes

| Component | Variable | Value | Why no token |
|---|---|---|---|
| Toggle | `toggle-knob-color` | white | No token for the knob fill |
| Toggle | `toggle-track-size-height` | 20 | No `Height/20`; `Icon Size/20` is wrong category |
| Toggle | `toggle-track-size-width` | 36 | No width tokens exist; `Icon Size/36` is wrong category |
