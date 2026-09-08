# Component Scaling Classification
### For Step 1 — Classify the Component

**Purpose:** Before a component gets variables/tokens, it needs to be classified by *how it
scales*. The class determines which dimensional tokens it needs (fixed size, width only, or
width + height + max constraints). This runs before token creation.

**Feeds into:** the `{component}-{element}-{property}-{modifier}` naming convention. The class
determines which `{property}` + `{modifier}` combinations a component is allowed to have.

---

## The Three Classes

### Class A — Fixed-size
Intrinsic size. Never stretches in either direction, regardless of container.

- **Behavior:** width and height are both hard-coded per size variant. No `fill`/`stretch` on either axis.
- **Needs:** a discrete `width` + `height` token pair *per size variant*. No `min`/`max` modifiers — there's nothing to constrain, because the value never changes.
- **Examples:** checkbox, radio, toggle/switch, icon button, avatar, badge dot.
- **Confirmed example — Toggle (ZEUS 3.0, node `636:14967`):**

  | Size | Width | Height |
  |------|-------|--------|
  | sm   | 36px  | 20px   |
  | md   | 44px  | 24px   |

  Both dimensions are literal pixel values with no auto-layout stretch on the track. This is the textbook Class A signature: pull up the dev-mode CSS and if width/height are hard numbers with no `flex-grow`/`w-full` on the root, it's Class A.

- **Tokens generated:**
  ```
  toggle-root-width-sm      = 36px
  toggle-root-height-sm     = 20px
  toggle-root-width-md      = 44px
  toggle-root-height-md     = 24px
  ```
  No `-min` / `-max` modifiers exist for Class A. If you find yourself wanting one, the component is probably misclassified — check whether it secretly has a text-label variant that *does* grow (see the caveat below).

- **Caveat to check per component:** some "fixed" components have an optional text-label slot (e.g. Toggle's `text=true` variant) that grows independently. In that case the *control* stays Class A, but the label wrapper next to it may be Class B. Classify the control and its label region separately — don't let one optional slot pull the whole component into a different class.

  → This is also the signature of a **composed component**. See Step 2b of the orchestrator: the control and its wrapper are one family sharing one variable collection, split by groups.

---

### Class B — Width-scaling
Height is fixed (or intrinsic to content, like line-height), width flexes with its container or with content length.

- **Behavior:** width is `fill`/`stretch`/`auto` and responds to parent width. Height stays fixed per size variant, or is driven purely by line-height/padding (not by parent).
- **Needs:** a `height` token per size variant (same as Class A), plus a `width` token that is usually **not** a fixed value — instead you define `width-max` (to stop the field from becoming absurdly wide on large screens) and sometimes `width-min` (to guarantee tap-target / readability at small container widths). A literal `width` value is only added if there's a deliberate default (e.g. inside a fixed-width form column).
- **Examples:** text field / input, select, textarea (width-scaling, height can still be fixed unless multi-line), search bar, single-line label-and-input row.
- **Tokens generated (pattern, using text field as the stand-in until audited):**
  ```
  input-root-height-sm     = 32px   (fixed, same logic as Class A)
  input-root-height-md     = 40px
  input-root-width-max     = 480px  (caps line length for readability)
  input-root-width-min     = 160px  (optional — floor for tight layouts)
  ```
- **How to verify in Figma:** select the component root → check the horizontal resizing property. `Fixed` = not this class. `Fill container` or `Hug contents` with a max-width constraint set = Class B.

---

### Class C — Omnidirectional-scaling (containers)
Both width and height flex with content and/or container. Nothing about its size is fixed.

- **Behavior:** both axes are `fill`/`fixed-by-content` and expected to be resized by whoever places it. Internal content (children) drives the practical minimum; the design usually wants to cap the practical maximum on at least one axis.
- **Needs:** `min-width`/`min-height` (almost always — prevents the container collapsing below a usable size once padding + smallest child are accounted for) and, most of the time, `max-width` and/or `max-height`. Containers are the one case where the max constraint is sometimes intentionally *absent* (e.g. a full-bleed page section) — so `max` is "usually present, confirm per component" rather than mandatory.
- **Examples:** panel, dialog/modal, card, drawer, popover, page section, accordion body.
- **Tokens generated (pattern):**
  ```
  dialog-root-width-min    = 320px
  dialog-root-width-max    = 640px   (omit if intentionally unconstrained)
  dialog-root-height-min   = 120px
  dialog-root-height-max   = 90vh    (often viewport-relative rather than px)
  ```
- **How to verify in Figma:** both horizontal and vertical resizing on the root are `Fill container` or `Hug contents`, and there's a `min width` / `min height` / `max width` / `max height` constraint set in the constraints panel (or it should be — flag it if missing).

---

## Decision Checklist

Run every component through this in order. Stop at the first "yes."

1. **Does the root have a fixed width AND a fixed height in every size variant, with no fill/stretch on either axis?**
   → **Class A — Fixed-size.** Generate the size-variant width/height pairs. Check for an optional label slot that might need separate Class B treatment.

2. **Is the height fixed (or purely content/line-height driven) while the width is set to fill/hug with a max constraint?**
   → **Class B — Width-scaling.** Generate height per size variant + `width-max` (+ `width-min` if a floor makes sense).

3. **Do both width and height flex, or is this a layout container whose job is to hold arbitrary children?**
   → **Class C — Omnidirectional-scaling.** Generate `min` for both axes; add `max` for both axes unless there's a deliberate reason it should be unconstrained (note that reason in the audit table).

4. **None of the above cleanly fits** (e.g. a component that scales in discrete steps rather than continuously, like a stepped avatar-size system, or one axis is fixed-by-variant while the other has no stated behavior yet):
   → Flag as **Needs Design Decision** rather than forcing a class. Note the open question directly in the audit table so it doesn't silently get token'd wrong.

---

## Audit Table

Fill in as components are reviewed. This is the artifact that feeds directly into variable generation.

| Component | Class | Fixed dims (if A) | Height (if B) | Min (if C) | Max | Notes |
|---|---|---|---|---|---|---|
| Toggle | A | sm 36×20, md 44×24 | — | — | — | Confirmed from node `636:14967`. Text-label variant grows independently — composed wrapper, audit the label row separately. |
| Checkbox | *pending* | | | | | Expected Class A — same shape as toggle, confirm sm/md px values. |
| Text field | *pending* | | | | | Expected Class B — confirm whether a default `width` exists for form-column contexts, or only `width-max`. |
| Panel | *pending* | | | | | Expected Class C — confirm whether `max` is intentionally omitted for full-bleed usage. |
| Dialog | *pending* | | | | | Expected Class C — check whether height uses `vh` rather than px for `max-height`. |
