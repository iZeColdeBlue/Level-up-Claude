# Cleanup Rules
### For Step 6 — Apply Variables & Clean Up

Explicit list of base-component patterns that must be stripped when adapting into ZEUS 3.0.
This document grows incrementally from real adaptation runs — add a rule whenever a cleanup
decision gets made so the next run doesn't have to re-derive it.

---

## Rule 1 — Never use whole-component opacity for disabled state

ZEUS 3.0 does not dim a whole component to express disabled. This is an old-system pattern
and must always be stripped when found on a base component.

**Correct disabled treatment:**
- Background feedback → the State Layer's own `Disabled` variant
- Borders → `outline` or `subtle-outline` tokens (see Rule 3)
- Text and icons → the disabled color tokens on their own variables

**Trap:** the `Content/full` token resolves to `0.6`, not `1.0`. The naming is misleading —
do not assume `full` means fully opaque.

---

## Rule 2 — Shadow effects are not focus rings

Base components frequently express focus with a drop shadow or elevation effect. ZEUS 3.0
expresses focus through the State Layer's focus stroke. Remove the shadow effect.

The State Layer's focus ring extends **1px outside** the host frame, so `Clip content`
(`clipsContent`) must be **off** on any variant that can show focus. If it is on, the ring
gets clipped and the focus state silently looks wrong.

All other borders in the system are set to **Inside**.

---

## Rule 3 — Border color tokens are restricted

`border-color` variables may only bind to `outline` or `subtle-outline` tokens — in **every**
mode, including `Disabled`.

`On-Surface/disabled` is **not** permitted for a border-color binding, even though it exists
and even though it may visually match.

---

## Rule 4 — Fixed sizes on non-icon elements use Sizing tokens

If a non-icon element needs a specific fixed size, bind it to a `Sizing/Height` token, not
an `Icon Size` token — even when the numeric value matches exactly. Intent over value.

---

## Rule 5 — Assistive text is never a local text layer

If the base component renders help text, hint text, or an error message as its own text
layer with local spacing and color, remove it and swap in an **AssistiveContent** instance.
Drop the local spacing and color values entirely — AssistiveContent owns them.

---

## Rule 6 — Single-value vendor properties are deleted

A Figma component property left with only one usable value is dead vendor scaffolding.
Delete it. This happens in **Step 4** (`component-property-spec.md`), not here — but if one
survives into cleanup, it still goes.

Count *usable* values, not declared ones: a four-value property where two values render
identically is a two-value property, and becomes a boolean.

---

## General principle

When uncertain whether something should be removed: **stop and ask the operator.** Never
silently remove a visual property you are unsure about. Being told "yes, remove it" costs
one message; removing something intentional costs a rework.
