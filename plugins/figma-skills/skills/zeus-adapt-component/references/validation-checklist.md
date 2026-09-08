# Validation Checklist
### Auto-runs in Step 7, before results are presented to the operator

> **Status: rebuilt from the specs.** The original was an 11-check gate; these checks are
> derived from the current reference documents rather than recovered verbatim. Sound, but
> worth a review pass.

Run every check. Report pass/fail per check. Do not present a component as complete with a
failing check — report the failure instead.

## Structure

1. **Two collections, correctly named** — `{Component}.Styles` and `{Component}.Measures`.
   Colors and dimensions are never mixed in one collection.
2. **No duplicate collection for a composed component** — a base control and its wrapper
   share one collection pair, split by groups. No second pair exists for the wrapper.
3. **Default mode renamed** — no collection contains a mode called `Mode 1`.

## Naming

4. **Every variable matches** `{component}-{element}-{property}-{modifier}`.
5. **Directional abbreviations** are `rgt` / `lft` / `top` / `btm` — never spelled out.
6. **No variant context in any name** — no `-hover`, `-disabled`, `-checked`, `-ai`,
   `-condensed` suffixes. That context lives in modes and groups.

## Scopes

7. **No variable is left on `ALL_SCOPES`.** Every scope is set explicitly, per the mapping
   table in `component-variable-collection-spec.md` Section 4.

## Nesting

8. **No state-layer variables exist locally** — no `{component}-state-layer-*` in the
   component's own collections.
9. **No foreign-owned properties are shadowed** — the component has no variables for
   properties owned by AssistiveContent, the State Layer, or any other foreign nested
   component.
10. **Assistive text is an AssistiveContent instance**, not a local text layer with its own
    spacing and color.
11. **No duplicated components** — every nested component resolves to an existing system
    component, not a local copy or fork.

## Cleanup

12. **No whole-component opacity** used for disabled state.
13. **No shadow/elevation effects** standing in for a focus ring.
14. **`clipsContent` is off** on every variant that can show focus.
15. **Border-color bindings use only** `outline` or `subtle-outline`, in every mode
    including Disabled.
16. **No hardcoded colors or spacing** remain on layers — everything is a variable binding,
    except confirmed snowflakes.

## Snowflakes

17. **Every snowflake is reported** to the operator by name, value, and reason. A snowflake
    is acceptable; an unreported one is not.

## Properties

18. **No single-value properties remain** on the component.
19. **Every two-value property is a boolean**, formatted `IsPressed?` / `HasIcon?` —
    PascalCase, `Is`/`Has` prefix, `?` suffix.
20. **`State` is a variant, not a boolean**, and its default value is named `Enabled` —
    never `Default`, `Normal`, or `Rest`.
21. **State values match the Styles mode names** — `Hovered` / `Focused` / `Pressed` /
    `Disabled`, participle form, in canonical order.
22. **Every icon slot is an instance swap property** named `Icon`, `Leading Icon`, or
    `Trailing Icon` — never `Left`/`Right`.
23. **Every text layer is a text property** named by role, defaulting to the component's
    own name.
24. **Nested instance properties are exposed**, not mirrored by local host properties.
25. **No vendor property names survive** — nothing called `variant2`, `Property 1`, or
    similar.
