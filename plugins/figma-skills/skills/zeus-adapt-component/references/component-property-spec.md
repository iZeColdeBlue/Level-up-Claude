# Component Property Spec
### For Step 4 — Normalize Component Properties

Figma **component properties** (variant / boolean / instance swap / text), not variables.
This is a separate axis from `naming-convention.md`, which governs variable names. A
component's property structure must be settled before variable collections are created,
because the property axes decide the mode and group axes in Step 5.

Purchased base components arrive with the vendor's property structure. Almost none of it
survives unchanged.

---

## 1. Property type decision

Run this against every existing property, and against every visual difference that
*should* be a property but isn't yet.

| Number of distinct, actually-used values | Property type |
|---|---|
| 1 | **Delete it.** A property with a single value is dead vendor scaffolding. |
| 2 | **Boolean** |
| 3 or more | **Variant** (dropdown) |
| A swappable icon slot | **Instance swap** |
| A text layer | **Text** |

**"Actually-used" matters.** A vendor property with four values where two render
identically is a two-value property. Verify visually before counting.

**Trap — don't collapse a state property just because it currently has two values.** If the
component is interactive and only ships `Default` and `Disabled`, it still becomes the
`State` variant property, not a boolean. States are always a dropdown (Section 3).

---

## 2. Boolean properties

**Format:** PascalCase, `Is` or `Has` prefix, `?` suffix. Values are `true` / `false`.

| Prefix | Use for | Examples |
|---|---|---|
| `Is` | A condition of the component itself | `IsPressed?`, `IsSelected?`, `IsChecked?`, `IsRequired?` |
| `Has` | The presence of a sub-element or slot | `HasIcon?`, `HasLabel?`, `HasOverline?`, `HasToolbar?`, `HasTitle?` |

Examples of the two-value collapse:

| Vendor property | Becomes |
|---|---|
| `pressed` = Pressed / Unpressed | `IsPressed?` |
| `Label` = Show / Hide | `HasLabel?` |
| `icon` = True / False | `HasIcon?` |

### Boolean cluster vs. variant explosion

When a component has several **independent, optional** content slots, use one boolean per
slot rather than one variant property enumerating every combination. A header component
that can carry an overline, a title, and a toolbar is three booleans —
`HasOverline?` / `HasTitle?` / `HasToolbar?` — not an eight-value dropdown.

Use a variant instead when the options are **mutually exclusive** (only one can be true at
a time) or when the combinations aren't all valid.

**This is a decision gate, not a rule.** Complex components — headers, input fields, list
rows — sit close to the line. Present the proposed split to the operator with the reasoning
and confirm before building it. Do not guess.

---

## 3. Variant properties

**Format:** Title Case, singular. `State`, `Size`, `Type`, `Layout`.

### `State`

The UI interaction state. Always a dropdown, never a boolean.

Values, in this order, including only the ones the component actually needs:

```
Enabled → Hovered → Focused → Pressed → Active → Dragged → Disabled
```

- **The default state is always named `Enabled`.** Rename whatever the vendor called it —
  `Default`, `Normal`, `Rest`, `Idle`, `Off`. `Default` is never a state value.
- `Hovered` / `Focused` / `Pressed` / `Disabled` are the common set.
- `Active` and `Dragged` only when the component genuinely has them.
- These values match the Styles-collection mode names and the State Layer's own state
  values, so the host's `State` property can be synced straight to the nested State Layer
  instance (Step 3).

### `Size`

Density / dimensional variant. Match the value names already used by adapted ZEUS 3.0
components rather than inventing a scale; if the file has no precedent yet, ask the
operator.

### `Type` and `Layout`

Both describe content composition, which is why the vendor usually ships one confused
property covering both:

| Property | Governs | Typical values |
|---|---|---|
| `Type` | What kind of content the component renders | `Text`, `Icon` |
| `Layout` | How that content is arranged | `Text only`, `Leading icon`, `Trailing icon`, `Both` |

Most components need **one of these, not both**. Use both only when they are genuinely
orthogonal (every `Type` works with every `Layout`). If it's ambiguous, propose one and
confirm with the operator.

> Inferred split — the source instruction described `Type` and `Layout` as "similar".
> Worth confirming against a real component before treating the value sets as fixed.

---

## 4. Instance swap properties

Every icon slot becomes an instance swap property.

| Slot arrangement | Property name(s) |
|---|---|
| A single icon slot | `Icon` |
| Icons on both sides, or a side-specific slot | `Leading Icon` and / or `Trailing Icon` |

- **Preferred value:** a basic system icon as the placeholder. Which specific icon it is
  usually doesn't matter — if it might, ask the operator.
- The swap target is always the **system icon component**, never a vendor vector pasted in
  with the base component (see Step 2c).
- Naming is `Leading` / `Trailing` — not `Left` / `Right`, not `Start` / `End`.
  (Note: variable *names* still abbreviate direction as `lft` / `rgt`; that convention
  applies to variables only, not to property names.)

---

## 5. Text properties

Every text layer becomes a text property, named after the **role** the text plays:

`Label`, `Title`, `Overline`, `Value`, `Placeholder`, `Caption`.

**Default value = the component's own name.** A button's `Label` defaults to `Button`; a
switch's `Label` defaults to `Switch`. The specific string doesn't matter — it just needs
to name the component so the placeholder is self-describing.

Text belonging to a nested system component is that component's property, not the host's —
AssistiveContent owns its own text (Section 6).

---

## 6. Nested instance properties

Whenever the component contains a nested system component, **expose the nested instance's
properties on the host** rather than rebuilding equivalents locally.

- **AssistiveContent** — always expose it. Its text, type, and state are its own
  properties, surfaced through the host.
- **Any component classified as nested / parent in Step 1–2** — expose the nested
  instance's properties as a matter of course.
- Do not create a host-level property that merely mirrors and forwards to a nested one.
  Exposure is the mechanism; duplication is the failure.

The one exception is a property the host must *drive* rather than pass through — the
State Layer's state, which is synced from the host's `State` property (Step 3c).

---

## 7. Anti-patterns

| Wrong | Why | Right |
|---|---|---|
| `variant2`, `Property 1`, `type=` left as shipped | Vendor scaffolding | Rename per this spec, or delete |
| A property with one value, kept "just in case" | Dead weight; pollutes the variant matrix | Delete it |
| `State` = `Default` / `Hover` | `Default` is never a state value; participle form required | `Enabled` / `Hovered` |
| `pressed` as a two-value variant | Two values means boolean | `IsPressed?` |
| `Right Icon` instance swap | Property names spell direction out as Leading/Trailing | `Trailing Icon` |
| An 8-value `Configuration` dropdown for three optional slots | Combinatorial explosion | Three `Has…?` booleans |
| A local `Helper Text` text property next to an AssistiveContent instance | Duplicates a nested component's property | Expose AssistiveContent's own property |
| Deciding a borderline boolean/variant split silently | The source rule is explicit: ask | Present both options, confirm |

---

## 8. Output

The final property table is what Step 5 consumes:

| Property | Type | Values | Feeds |
|---|---|---|---|
| `State` | Variant | Enabled, Hovered, Focused, Pressed, Disabled | Styles modes (if state changes more than background) |
| `Size` | Variant | … | Measures modes |
| `Layout` | Variant | Text only, Leading icon, Trailing icon | Measures groups |
| `HasLabel?` | Boolean | true / false | Measures group (`Label/`) |
| `Leading Icon` | Instance swap | system icon | — |
| `Label` | Text | "Button" | — |
