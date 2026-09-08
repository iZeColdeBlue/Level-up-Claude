# Add State Layer
### For Step 3 — State Layer Setup

> **Status: partially reconstructed.** Component structure and the opacity ladder are
> verified against the live files. The step-by-step Plugin API sequence should be
> re-verified against the live component before a production run.

## The Component

- Lives at node **`388:824`** in file `2qBaAweE1afdX1UCMxtyId`
- **3 shape variants:** Default, Pill, Checkbox
- **5 color variants:** Primary, Black, Error, AI, White
- **5 states:** Enabled, Hovered, Active/Pressed, Focused, Disabled
- **Own collections:** `Statelayer.Styles`, `Statelayer.Measures` — shared across every
  component that uses it. **Never modified per-component. Never duplicated.**

Always instance the existing component. Never create a new State Layer.

## Shape Selection

| Shape | When |
|---|---|
| Default | Most components — buttons, cards, inputs, tags |
| Pill | Border-radius ≥ half the component height — fully rounded chips, pill buttons |
| Checkbox | Fixed-shape binary controls — checkbox, radio |

## Color Selection

| Color | When |
|---|---|
| **Primary** | Default choice for most components |
| **Black** | Host has a primary (blue) background, **or** a neutral/gray background — follow the base component's color intent, not a blanket default |
| **Error** | Host is in an error state |
| **AI** | AI-related components only. Ask the operator if unsure. |
| **White** | Rare. Only when the operator says so. |

A component may need more than one configuration across its variants.

## Insertion

1. Instance the State Layer into the component.
2. Position it as the **bottom-most content layer** — above the background fill, below all content.
3. Absolute-position it, stretched to fill via Left + Right + Top + Bottom constraints.
4. Match corner radius to the host.
5. Set the color and shape properties per above.
6. Sync its state property to the host's state mechanism. `setProperties()` accepts plain
   strings for variant properties (e.g. `{ State: 'Hovered', Color: 'Primary' }`) and works
   reliably in a loop across all variants.

**Verification:** `getMainComponentAsync()` on the instance returns the actual resolved main
component name and ID. This is the reliable way to confirm correct wiring — do not infer it
from property values alone.

## Clip Content

The focus ring extends **1px outside** the host frame. `clipsContent` must be **off** on any
variant that can show focus, or the ring is silently clipped. All other borders are Inside.

## Division of Responsibility

**The State Layer handles:**
- Background feedback via its own opacity tokens. Verified values from Foundations
  (`Opacity` collection, `State Layer/*`):

  | Token | Value |
  |---|---|
  | `State Layer/hidden` | 0 |
  | `State Layer/minimal` | 0.02 |
  | `State Layer/hovered` | 0.12 |
  | `State Layer/pressed` | 0.12 — **identical to hovered** |
  | `State Layer/dragged` | 0.16 |
  | `State Layer/active` | 0.38 |
  | `State Layer/focused` | 1.00 |

  Pressed is not a stronger tint than hovered. If a base component distinguishes them by
  opacity, that distinction does not survive into ZEUS 3.0 — don't try to reproduce it with
  a local variable.
- Focus rendering via its own visible stroke, at full opacity

**The State Layer does NOT handle** (so these still need host variables):
- Border color changes between states
- Icon color changes between states
- Text / label color changes between states
- Any non-background visual change that varies by state
