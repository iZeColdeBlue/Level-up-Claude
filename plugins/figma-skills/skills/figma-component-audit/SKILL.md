---
name: figma-component-audit
description: "Audit a Figma design system component for structural quality and variable/token hygiene, and produce a single compiled review report. Use this skill whenever the user asks to review, audit, QA, or check a Figma component, its layers, properties, states, slots, typography, variables, tokens, naming, or scoping — including phrases like \"review this component\", \"component review\", \"variable review\", \"check my tokens\", \"variable audit\", \"is this component ready\", \"design system check\", \"component QA\", \"are my variables correct\", or when the user pastes a Figma component link and asks whether it's correct. Also use it when the user asks for only the structural half or only the variable half — run just that section. Do not use for creating or adapting components (that's /adapt-component) or for visual design critique."
---
 
# Figma Component Audit
 
Read-only audit of a single Figma design system component. Produces one compiled report
in two sections: **A. Structure & Properties** (Checks 1–8) and **B. Variables & Tokens**
(Checks 9–13).
 
This skill audits and reports. It never modifies the file.
 
## Inputs
 
| Input | Required | How to get it |
|---|---|---|
| Component link | Yes | Figma URL to the main component or component set |
| Foundations library | Preconfigured | `FOUNDATIONS_FILE_KEY` below |
 
**If no Figma link was provided, ask for one before doing anything else.** Ask plainly:
"Paste the Figma link to the component you want audited (right-click the main component →
Copy link to selection)." Do not guess at a component, do not audit the current selection
instead, and do not proceed with a link to a frame, an instance, or a whole page.
 
If the link points to an **instance**, say so and ask for the main component instead.
If the link points to a **component set**, audit the set and note per-variant findings.
 
### Configuration
 
```
FOUNDATIONS_FILE_KEY = dULgduzwkPkKWVOhy8MSHR   # ZEUS 3.0 Foundations
STATE_LAYER_FILE_KEY = 2qBaAweE1afdX1UCMxtyId
STATE_LAYER_NODE_ID  = 388:824
```
 
Never skip Check 10 silently and never fabricate foundation token names. If the
foundations file is unreachable, say so in the report.
 
## Gathering data
 
Load the `figma-use` skill before any `use_figma` call — this is mandatory and skipping it
causes hard-to-debug failures.
 
Gather in this order, running independent reads in parallel where possible:
 
1. `get_metadata` — layer tree, names, nesting, node IDs.
2. `get_design_context` — properties, variants, applied variables, text styles.
3. `get_variable_defs` — variable names and resolved values on the node.
4. `get_screenshot` — visual reference for the report and for judging which states apply.
5. `use_figma` — for anything the read tools don't expose: variable **collection**
   membership, **modes**, **groups**, **scopes**, and alias targets. Query these; do not
   infer them from variable names.
**Ground every finding in retrieved data.** If a check cannot be verified with the data
you actually got back, mark it ⚠️ and state what you couldn't read and why — never assume
a pass and never invent a layer name, variable name, or scope value.
 
## Verdicts
 
- ✅ **Pass** — meets the requirement
- ⚠️ **Warning** — non-critical issue, needs user confirmation, or unverifiable
- ❌ **Fail** — clear violation
Collect every finding silently. Do not post results as you go. Compile one report at the
end.
 
---
 
# Section A — Structure & Properties
 
### Check 1 — Layer Naming
Inspect every layer in the tree. Flag auto-generated names (`Frame 10`, `Rectangle 4`,
`Group 3`, `Vector 12`). Every layer needs a descriptive name reflecting its role.
 
### Check 2 — Structure & Wrappers
Flag unnecessary wrapper frames: layers with a single child that add no auto-layout,
constraints, clipping, or visual styling. Ideal structure wraps content directly in the
main component frame with minimal nesting. For each flagged wrapper, note what it contains
and whether it can be collapsed.
 
### Check 3 — State Layer
- Component changes visual state (hover, focus, active, pressed, disabled) → it should
  contain a `State Layer` instance. Flag if missing.
- Component has no interactive states → flag if a State Layer is present unnecessarily.
- If present, verify it is an instance of the published `State Layer` component
  (`STATE_LAYER_NODE_ID` above), not detached or locally recreated.
Audit only *whether and how* the State Layer is used. Do not audit the internals of the
State Layer component itself — it is a dependency, not part of this component's scope.
 
### Check 4 — Content Properties & Slots
All changeable content should be exposed as component properties:
- **Text** (labels, headings, descriptions) → text properties.
- **Icons** (leading, trailing) → instance swap properties. Check preferred values are
  set; if unclear, ask which icon sets should be available.
- **Optional elements** (icons, badges, descriptions) → boolean visibility properties.
For each text property, check text-length handling (max lines, truncation). If neither is
set, flag ⚠️ and ask whether length constraints are needed for this content.
 
If slots are used instead of direct properties, defer to Check 5.
 
### Check 5 — Slot Configuration
For each instance swap property that accepts nested components:
- **Scoping** — is the slot scoped to a specific component set? If not, ask: "Should this
  slot accept any component, or be scoped to a specific set?"
- **Default value** — does it have a default, or is it empty? If empty, ask: "Should this
  slot have a default component, or is empty intended?"
**Exception:** container components (cards, lists, layout wrappers) designed to house
varied content do not need scoped slots. Do not flag these.
 
### Check 6 — Property-Driven States & Types
Verify variants and properties cover:
- **States**: enabled, hovered, active/pressed, focused, disabled — as applicable. Not
  every component needs all of them; use judgment based on component type.
- **Types/variants**: e.g. with icon / without icon / icon-only, text vs numeric input,
  outlined / contained / ghost. Meaningful variations must be exposed as properties, not
  as separate detached components.
Flag missing states or types. If unsure whether a state applies, flag ⚠️ and ask.
 
### Check 7 — Variable Assignments
Most configurable values should have variables assigned rather than hardcoded values:
 
| Property | What to check |
|---|---|
| Width & height | If not fixed: min/max width and height variables |
| Gap | Auto-layout gap references a variable |
| Padding | Both horizontal and vertical reference variables |
| Corner radius | References variable(s) — may differ per corner |
| Fill | Color fills reference color variables |
| Stroke color | References a color variable |
| Stroke weight | References a variable — may differ per side |
 
Flag any hardcoded value where a variable belongs.
 
### Check 8 — Typography Styles
Every text layer should use a published text style from the library, not raw
font/size/weight values. Flag any layer with locally set typography.
 
---
 
## Halt condition — Structural Failures
 
After Section A, if any check returned ❌ **Fail**, stop. Post Section A as a partial
report, list the failures, and ask: "Fix these first, or continue into the variable
audit anyway?" Wait for an answer.
 
If Section A has only passes and warnings, continue straight into Section B without asking.
 
---
 
# Section B — Variables & Tokens
 
### Check 9 — Collection Structure
Variables for this component must live in two collections:
- `{componentName}.Measures` — all numeric variables (spacing, sizing, radius, stroke weight)
- `{componentName}.Styles` — all color variables
Flag any variable in the wrong collection or in a collection not matching this pattern.
Component names use camelCase for multi-word names.
 
**Measures rules:** different sizes (condensed / default / comfortable) → different
**modes**. Variables specific to one type/variant → separate **groups**.
 
**Styles rules:** different UI states (enabled, hovered, active, focused, disabled) →
different **modes**. Different visual styles (outlined, contained, ghost) → different
**groups**.
 
### Check 10 — Foundation References
Every component variable must alias a variable from the foundations library
(`FOUNDATIONS_FILE_KEY`). Component variables alias **foundations only** — never
primitives. Flag:
- Any variable resolving to a raw value with no alias → possible **snowflake**. List it
  with its raw value so the user can decide: create a foundation token, or correct the value.
- Any variable aliasing a **primitive** directly, bypassing the foundations layer → ❌.
If you cannot read the foundations library, flag the affected variables ⚠️ and say the
library was unreachable. Never declare something a snowflake on the basis that you
couldn't find its reference.
 
### Check 11 — Naming Convention
Required pattern:
 
```
--{component}-{element}-{property}-{modifier}
```
 
- **component** (required) — the UI component (`button`, `inputField`, `card`).
- **element** (optional) — a child part (`icon`, `label`, `container`, `background`,
  `border`). Omit when the property applies to the component root: `--button-gap`, not
  `--button-root-gap`.
- **property** (required) — what the token controls (`color`, `size`, `padding`, `gap`,
  `border-radius`).
- **modifier** (optional) — narrows by direction, dimension, or constraint. Chainable:
  `lft-rgt`, `top-btm`, `width`, `height`, `max`, `min`.
Rules:
- All lowercase (except camelCase component names), single hyphens, `--` prefix.
- Reads left to right, broad to narrow.
- Directions abbreviate to `lft`, `rgt`, `top`, `btm`. Pair symmetric axes: `lft-rgt`,
  `top-btm`.
- Dimensional constraints prefix with `size`: `size-height`, `size-width-max`.
- Compound properties stay one unit — `border-radius`, `border-color` keep `border` in the
  property, unless there is an explicit border element
  (`--button-container-border-radius`).
- Variant overrides are **never** baked into names. The same names repeat across variant
  groups and modes.
Valid references:
`--button-background-color` · `--button-label-color` · `--button-border-color` ·
`--button-gap` · `--button-size-height` · `--button-border-radius` ·
`--button-padding-lft-rgt` · `--button-size-width-max` · `--button-icon-size` ·
`--tag-padding-top-btm` · `--inputField-border-radius-lft` ·
`--inputField-text-placeholder-color`
 
Flag: missing component prefix · redundant segments (`root`, `default`) · unabbreviated
directions (`right`) · variant names baked into the name. List each violation with the
current name and a suggested correction.
 
### Check 12 — Variable Scoping
Scopes must always be set explicitly. Flag any variable left at `ALL_SCOPES` or default
scope, and any scope that doesn't match the property type:
 
| Variable purpose | Expected scope |
|---|---|
| Container / background color | `FRAME_FILL`, `SHAPE_FILL` |
| Text color | `TEXT_FILL` |
| Border color | `STROKE_COLOR` |
| Border width / stroke weight | `STROKE_FLOAT` |
| Corner radius | `CORNER_RADIUS` |
| Padding / gap | `GAP` |
| Width / height / min / max | `WIDTH_HEIGHT` |
 
List each over-scoped variable with its current and recommended scope.
 
### Check 13 — Cross-Assignment
Final recheck across the whole component. Verify:
- No variable from one variant/style applied to another (a `contained` color on an
  `outlined` layer).
- No variable from a **different component** used here (an input field measure on a
  button layer).
- No variable used for the wrong property type (a radius variable on a gap, a fill
  variable on a stroke).
This check catches the errors that are hardest to spot and most damaging in production.
Never skip it.
 
---
 
## Report Format
 
Post one report:
 
```markdown
# Component Audit: {componentName}
 
**Component:** {link} · **Type:** {component / component set, N variants}
 
## Summary
{N} passed · {N} warnings · {N} failures
{One-line verdict: ready to publish / needs fixes before publishing / blocked}
 
---
 
## A. Structure & Properties
 
### 1. Layer Naming — {✅|⚠️|❌}
{Findings, or "All layers named descriptively."}
 
### 2. Structure & Wrappers — {✅|⚠️|❌}
...through Check 8
 
## B. Variables & Tokens
 
### 9. Collection Structure — {✅|⚠️|❌}
...through Check 13
 
---
 
## Open Questions
{Every question the checks raised, numbered. Omit the section if there are none.}
 
## Manual Testing Required
Cannot be verified programmatically — test these before publishing:
1. **Long text** — enter a very long label. Does it truncate, wrap, or break layout? Is
   that intentional?
2. **Resizing** — resize to min and max width/height. Does the layout hold?
3. **Property combinations** — toggle every property and combination. Any broken states?
4. **Content extremes** — 1 character and maximum content; with and without optional
   elements.
```
 
Order findings within each section by severity: failures first, then warnings, then
passes.
 
## Don'ts
 
- ❌ Don't post findings one at a time — compile one report.
- ❌ Don't auto-fix anything. This skill audits and reports; fixes are a separate task.
- ❌ Don't write to the Figma file. All operations are read-only.
- ❌ Don't audit the internals of the `State Layer` component — only how it's used here.
- ❌ Don't assume a missing state or variant is wrong — flag ⚠️ and ask.
- ❌ Don't flag container components (cards, layout wrappers) for unscoped slots.
- ❌ Don't call a variable a snowflake just because you couldn't find its reference —
  flag ⚠️ and ask the user to confirm.
- ❌ Don't infer collection, mode, group, or scope from a variable's name — query it.
- ❌ Don't report a check as passing when you couldn't read the data to verify it.
