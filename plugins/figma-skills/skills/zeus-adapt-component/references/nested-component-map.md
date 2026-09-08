# Nested Component Map
### Registry for Step 2 — Resolve Nested Components

**Purpose:** the lookup table the agent consults when it finds something nested inside a
base component. Answers three questions: is this an always-use system component, is it a
same-family control, and what do we call the thing the vendor called something else.

This document is expected to **grow with every adaptation run**. When a nested component
fails to match an equivalent that actually exists, add the mapping here.

---

## 1. Always-Use System Components

These are finished and published. Always instance them. Never rebuild, never fork, never
create a local equivalent, never modify their variables from the host.

| Trigger in the base component | Use | Ownership |
|---|---|---|
| Any assistive text — help text, hint, error message, character counter, description below or beside a control | **AssistiveContent** | Owns its own text color, typography, spacing, and icon. Host owns only the gap between the control and the AssistiveContent instance. Content-only: it has no background or border of its own. |
| Hover / pressed / focus / disabled background feedback | **State Layer** (node `388:824`) | Owns `Statelayer.Styles` / `Statelayer.Measures`. Three shapes (Default / Pill / Checkbox), five colors, five states. See `add-state-layer.md`. |
| An icon | System icon component | Not a pasted vendor vector. |

**Detached look-alikes count.** If the vendor renders help text as a plain text layer with
its own hardcoded spacing and color, that is still an AssistiveContent case — swap it in and
drop the local values.

---

## 2. Component Families (base control + composed wrapper)

Some components ship as a pair. The base control is the bare UI control; the composed
wrapper contains it alongside adjacent content and is usually what product teams consume.

**Both share one variable collection pair, split by groups.** Never create a second
collection for the wrapper. See Step 4 of the orchestrator.

| Family | Base control | Composed wrapper | Wrapper variants | Groups |
|---|---|---|---|---|
| Toggle | The switch itself (track + knob) | Toggle with label | `text=true` / `text=false` | `Toggle/` (control), `Label/` (wrapper content) |

*Add further families as they are adapted. Checkbox and Radio are the likely next entries —
confirm whether their label rows are composed wrappers or built into the control.*

**How to tell a family pair from foreign nesting:** would the nested thing appear inside
components that have nothing to do with this one? If yes, it is foreign. If it only ever
appears inside this family, it is same-family.

---

## 3. Vendor-Name → ZEUS Name Mappings

Vendor naming does not line up with ours. Searching only the literal vendor name and finding
nothing is **not** evidence that no equivalent exists. Match on function first, name second.

| Vendor / base library name | ZEUS 3.0 equivalent | Notes |
|---|---|---|
| Helper text, Hint, Supporting text, Caption | **AssistiveContent** | All of these collapse to one component. |
| Chip | Tag | Confirm variant coverage before swapping. |
| Icon Button | Button (icon-only variant) | Not a separate component — a Button configuration. |
| Switch | Toggle | Same control, different name. |

*Add mappings as they are discovered. An entry here saves the next run a failed search.*

---

## 4. Known ZEUS 3.0 Components

Current inventory, for matching against. Verify with `search_design_system` before relying
on this list — it drifts as the system grows.

Card, buttonGroup, Button, Statelayer, link, Tag, badge, InputField, Dropdown, Checkbox,
AssistiveContent, Toggle.

File key: `2qBaAweE1afdX1UCMxtyId`
