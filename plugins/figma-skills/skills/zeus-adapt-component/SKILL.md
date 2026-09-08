---
name: zeus-adapt-component
description: "Orchestrate the full pipeline that turns a purchased/pasted base component in the ZEUS 3.0 Components Figma file into a properly tokenized, state-layered, foundation-backed ZEUS 3.0 component. Classification, final validation, and the end-of-run review are performed directly by the orchestrating session itself; every mutating step (resolve nested components, state layer, normalize properties, create variables, assign foundation tokens, apply/clean up) is delegated to a Figma-executor subagent, with independent steps run in parallel and dependent ones gated on the right join points. Surfaces every decision gate or unexpected situation to the operator instead of guessing. Use whenever the user pastes a Figma link to a component in the ZEUS 3.0 Components file (fileKey 2qBaAweE1afdX1UCMxtyId) and asks to \"adapt\", \"set up\", \"run the full component workflow\", \"onboard this component\", \"/zeus-adapt-component\", or similar — for the WHOLE pipeline, not a single step. Available in every project/session (user-level skill)."
---

# ZEUS 3.0 — Adapt Component (Orchestrator)

> Turns a pasted base component into a properly tokenized, state-layered,
> foundation-backed ZEUS 3.0 component by running a set of steps — some gated
> sequentially, some fanned out in parallel. Most steps are executed by a
> dedicated subagent; Classify, Validate, and Review are performed directly by
> the orchestrator itself.

## Your role: orchestrator — mostly, not always, an executor

**This session does not call `use_figma`, `get_design_context`, or any other
Figma tool directly for the mutating steps (2, 3, 4a, 4b, 5, 6, 7a).** Those are
always delegated to a subagent. **Steps 1 (Classify) and 7b (Validate) are the
exceptions — the orchestrator performs both itself, with no subagent.** Both are
read-only inspection/reasoning passes whose result the orchestrator has to act
on the moment it lands: Step 1's output decides the entire Batch A dispatch,
and Step 7b's result is what gets presented in Step 8. Routing either through a
subagent would add a round-trip and a trust boundary for information the
orchestrator immediately needs to act on itself, for a step whose tool-call
volume is low enough that isolating it doesn't protect much. Step 8 (Review)
is also done directly — pure synthesis, no Figma calls at all.

For the delegated steps, the orchestrator's job is to:

1. Prepare a self-contained prompt for each step and hand it to the **Agent**
   tool — for steps that can run in parallel (see `## Execution Graph`), issue
   **all of that batch's `Agent` calls in a single message** so they run
   concurrently; for a step that depends on prior output, issue it alone and
   wait.
2. Wait for each batch to fully return before starting anything gated on it
   (always `run_in_background: false` for these calls, since the next action —
   either the next batch or a join check — depends on the result).
3. Fold every result into a running **workflow state** object.
4. Decide, from Step 1's output, which of the conditionally-run steps actually
   need to happen at all (see `## Conditional Steps` below) — and skip spawning
   a subagent for a step whose precondition isn't met, recording the skip in
   the workflow state instead.
5. Surface any decision gate *or anything unexpected* a subagent flags to the
   operator (via plain question or `AskUserQuestion`), get an answer, and pass
   it into the next relevant subagent's prompt.
6. Present each step's output to the operator per `## Operator Communication
   Rules` below, then move to the next step or batch.

Never do a *delegated* step's work yourself "to save time" — for Steps 2 through
7a, every Figma read/write happens inside an isolated subagent's context, and
the orchestrator's job there is purely sequencing/parallelizing, state-carrying,
gating, and operator communication. Steps 1, 7b, and 8 are the deliberate
exceptions described above, not a loophole to extend to anything else.

### Universal rule: never guess

**Any time a subagent — or the orchestrator itself, during Step 1 or 7b — hits
something its reference doc doesn't clearly cover, is ambiguous, or looks
unexpected — not just at the decision gates explicitly listed per step — it
must stop and ask rather than picking an answer.** For a delegated step, that
means the subagent returns `needs_operator_decision` instead of guessing. For
Step 1 or 7b, since the orchestrator is doing the work itself with no subagent
in between, it asks the operator directly (e.g. via `AskUserQuestion`) rather
than routing the question through a JSON field first. Either way, the answer
always comes from the operator, never from the orchestrator's own judgment
call. This applies for the whole pipeline, every step, every batch.

### Subagent contract (put this in every delegated step's prompt — i.e. every step except 1, 7b, and 8)

Every subagent prompt must be self-contained — the subagent has no memory of this
conversation — and must include:

- **The Figma file key** (`2qBaAweE1afdX1UCMxtyId` for ZEUS 3.0 Components, unless
  the operator's link points elsewhere) and the **node ID** of the component.
- **The workflow state so far** — everything carried forward from prior steps
  (classification, nesting map, property table, variable IDs, etc.), inlined as
  JSON. The subagent cannot see earlier steps' subagents' context, and cannot
  see sibling subagents running in the same parallel batch.
- **An instruction to load `figma:figma-use` before any `use_figma` call** — this
  is mandatory per that skill and is not optional.
- **A pointer to the specific reference doc(s) this step needs**, as an absolute
  path under `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\`
  (see the table in `## Reference Documents` below) — tell the subagent to
  `Read` it before acting, since the doc contains the exact rules for that step.
- **The universal never-guess rule** above, plus the specific decision gates
  listed for that step, all returned as a `needs_operator_decision` field with
  the question and options instead of picking one.
- **An explicit instruction to return a structured JSON result** (not prose) as
  its final message: the "Output to carry forward" fields for that step, plus
  `createdNodeIds` / `mutatedNodeIds` / `createdVariableIds` / `errors` /
  `needs_operator_decision` as applicable.
- **Full tool access** — use a subagent type with unrestricted tools (e.g.
  `general-purpose` or `claude`) since the step needs `use_figma`,
  `get_design_context`, `get_metadata`, `get_screenshot`, `search_design_system`,
  `get_libraries`, `get_variable_defs`, `list_file_components_for_code_connect`,
  and `Read`.

If a subagent's result is missing required fields, contains an error, or carries
a `needs_operator_decision`, do not proceed to the next step/batch — report it
to the operator and get an answer (retry, adjust, or abort), per
`## Operator Communication Rules`.

---

## Prerequisites

The operator has already copied the base component from the source Figma file
into the ZEUS 3.0 Components file (`2qBaAweE1afdX1UCMxtyId`) and provides a Figma
link (URL with `fileKey` and `node-id`) pointing to it.

## Execution Graph

Steps are grouped into solo steps and two parallel batches, joined at two gates.
Dispatch a batch's `Agent` calls together in one message; do not start the next
gated step until every call in the batch has returned.

```
START
  │
  ▼
[1. Classify]  ── solo, always runs, DIRECT (no subagent) ──────────
  │  produces: isInteractive, hasNestedComponents,
  │            hasNormalizableProperties, + full classification
  ▼
┌─────────────────────── BATCH A (parallel) ───────────────────────┐
│  [2. Resolve Nested Components]   — only if hasNestedComponents   │
│  [3. State Layer]                 — only if isInteractive         │
│  [4a. Normalize Properties]       — only if hasNormalizableProps  │
│       (structure/typing pass; does NOT expose nested properties)  │
└─────────────────────────────────┬──────────────────────────────── ┘
                                   ▼
                    ── JOIN A: wait for whichever of 2/3/4a ran ──
                                   │
                                   ▼
        [4b. Expose Nested Instance Properties]  — solo, sequential
             runs only if Step 2 ran AND found something to expose
             (AssistiveContent, same-family/foreign nested instances).
             Needs Step 2's *finished* resolved-nesting map as input —
             this is exactly the piece that can't be parallelized.
                                   │
                                   ▼
        [5. Create Variable Collections]  — solo, ALWAYS runs
             (every component needs variables, even a trivial one —
             this step has no skip condition)
                                   │
                                   ▼
┌─────────────────────── BATCH B (parallel) ───────────────────────┐
│  [6. Assign Foundation Tokens]        — always runs                │
│  [7a. Apply Variables & Clean Up]     — always runs                │
│       (bind layers to variables + strip non-system styling;        │
│        does NOT run the validation checklist — that's after both)  │
└─────────────────────────────────┬──────────────────────────────── ┘
                                   ▼
                    ── JOIN B: wait for BOTH 6 and 7a ──
                                   │
                                   ▼
        [7b. Validate]  — solo, DIRECT (no subagent), only after Join B
             needs both 6's token bindings and 7a's layer bindings +
             cleanup to check the full checklist correctly — the
             orchestrator re-inspects Figma itself (get_metadata /
             get_variable_defs / get_screenshot) rather than trusting
             6's and 7a's self-reported JSON
                                   │
                                   ▼
        [8. Review]  — done directly by the orchestrator, no subagent
END
```

### Why 4a/4b are split, and why validation moved

Step 4 (Normalize Component Properties) has one sub-part —"expose nested
instance properties"— that structurally depends on Step 2's *finished* resolved
nesting map (you can't expose a nested instance's properties on the host until
you know which ZEUS component that nested instance actually resolved to). The
rest of Step 4 (delete dead properties, retype, rename, the boolean-vs-variant
decision) only needs Step 1's property list and runs independently. So Step 4
splits: **4a** joins Batch A (parallel with Step 2 and Step 3), **4b** is a short
solo step right after Join A that only does the nested-exposure part, using
whatever Step 2 (if it ran) produced.

Validation (`references/validation-checklist.md`) was previously folded into
Step 7. It's pulled out into its own **7b**, gated on Join B, because several of
its checks span both halves of the old Step 7 batch — e.g. "border-color
bindings use only `outline`/`subtle-outline` in every mode" depends on both
Step 6's token aliasing and Step 7a's cleanup being done; checking it while
Step 6 might still be mid-flight in the other parallel branch would be checking
against incomplete state.

**Steps 1 and 7b are performed directly by the orchestrator, not delegated.**
Both are read-only inspection/reasoning passes rather than mutation-heavy work,
and both produce output the orchestrator has to act on the instant it's ready —
Step 1's booleans decide Batch A's dispatch, Step 7b's pass/fail is what Step 8
presents. Delegating either would mean waiting on a subagent, parsing its JSON,
and trusting its self-report for exactly the information the orchestrator needs
to reason about next — with none of the upside subagent-isolation gives the
mutating steps, since neither one generates the kind of tool-call volume worth
keeping out of the orchestrator's context. For Step 7b specifically, doing it
directly also means the orchestrator independently re-inspects the actual Figma
state (`get_metadata`, `get_variable_defs`, `get_screenshot`) rather than taking
Step 6's and 7a's own reports of their work at face value — a real check, not a
rubber stamp.

---

## Conditional Steps

Determined entirely from **Step 1's** output — the orchestrator decides which
subagents in Batch A to actually spawn *before* dispatching that batch:

| Step | Runs only if | Skipped when | On skip |
|---|---|---|---|
| 2. Resolve Nested Components | Step 1's nested-component inventory is non-empty | No nested instances or detached look-alikes found at all | Record `nestingResolved: false`, empty resolved-nesting map, no foreign exclusions. Step 4b then has nothing to expose and is skipped too. |
| 3. State Layer | Step 1 classified the component as interactive | Non-interactive (badges, avatars, static banners, dividers, etc.) | Record `stateLayerAdded: false`. |
| 4a. Normalize Properties | Step 1 found a non-trivial property list to normalize | The component has effectively no properties worth restructuring (e.g. zero or one already-clean property) | Record `propertiesNormalized: false`, carry Step 1's property list forward unchanged into Step 5. |
| 4b. Expose Nested Properties | Step 2 ran **and** its resolved-nesting map contains at least one instance whose properties should be exposed | Step 2 was skipped, or ran but found nothing needing exposure | No-op, nothing to carry forward beyond Step 4a's table. |
| 5. Create Variable Collections | **Always** | Never — every adapted component needs variables | — |
| 6. Assign Foundation Tokens | **Always** | Never | — |
| 7a. Apply Variables & Clean Up | **Always** | Never | — |
| 7b. Validate | **Always**, after Join B | Never | — |

If it's genuinely unclear during Step 1 whether any of these apply (e.g. it's
not clear whether the property list is "trivial enough" to skip 4a) — per the
universal never-guess rule, the orchestrator asks the operator directly (Step 1
has no subagent to route the question through), not a judgment call to make
silently.

---

## Step 1 — Classify the Component *(performed directly by the orchestrator, no subagent)*

**Goal:** Determine what the component is, how it behaves, and what it needs —
including the three booleans that gate everything downstream.

**Do this yourself, using your own tool calls** (not a subagent):
1. Use `get_design_context` (and `get_metadata` / `get_screenshot` as follow-ups
   if the first pass leaves anything unclear) to inspect structure, layers,
   variants, properties.
2. Determine functional category (Action / Container / Navigation / Information /
   Input).
3. Determine scaling class (A / B / C) — `Read` and follow
   `references/component-scaling-classification.md`.
4. Determine interactivity (interactive / non-interactive) → `isInteractive`.
5. Inventory nested components — every instance and detached look-alike, with its
   role in the layout → `hasNestedComponents` (non-empty inventory). **Do not
   resolve or swap anything yet — Step 2 does that.**
6. Assess whether the property list is non-trivial enough to need normalization
   → `hasNormalizableProperties`. If genuinely unclear, ask the operator
   directly rather than guessing.
7. Identify base-control vs. composed-wrapper vs. standalone, and whether the
   family partner already exists in the file.

**Output to carry forward into every later prompt:** component name, functional
category, scaling class, `isInteractive`, `hasNestedComponents`,
`hasNormalizableProperties`, variant/property list, visual-element list,
nested-component inventory, family position.

**Once you have this:** present the classification summary (including the three
gating booleans) to the operator per Operator Communication Rules, determine
which of Step 2 / Step 3 / Step 4a to spawn, then dispatch Batch A.

---

## Batch A (parallel) — Steps 2, 3, 4a

Dispatch every applicable one of these as `Agent` calls **in a single message**.
Each gets Step 1's full output; none of them see each other's results (they run
concurrently, in isolated contexts) — anything one needs from another belongs in
a later join step, not in these prompts.

### Step 2 — Resolve Nested Components *(only if `hasNestedComponents`)*

**Goal:** Every nested component must be an instance of the existing ZEUS 3.0
component, never a copy of vendor internals.

**Subagent prompt must include:** the Figma link, Step 1's nesting inventory +
family position, and instructions to:
1. Triage each nested-inventory entry: already-resolved / candidate for swap /
   not a nested component.
2. Classify each candidate as same-family or foreign — `Read`
   `references/nested-component-map.md` for always-use components, family
   registry, and vendor→ZEUS name mappings.
3. Search the design system (`search_design_system` against
   `2qBaAweE1afdX1UCMxtyId`, plus `get_libraries` /
   `list_file_components_for_code_connect`) before concluding anything needs to
   be created. Match on function first, name second.
4. Apply the per-candidate decision table below.
5. Swap and verify matched instances; preserve host layout; delete leftover
   vendor layers the swap now renders.
6. Record ownership boundaries: foreign-owned properties (hands off) vs.
   same-family shared values.

**Per-candidate decision table:**

| Situation | Action |
|---|---|
| Same-family control, already present in the file | Nest the existing base control; collection handling goes to Step 5. |
| ZEUS equivalent exists and covers the usage | Swap the instance to the ZEUS component. |
| ZEUS equivalent exists but missing a needed variant/property | `needs_operator_decision` — extend first, never fork. |
| Match plausible but not certain | `needs_operator_decision` — present candidates + reasoning. |
| No ZEUS equivalent exists at all | `needs_operator_decision` — ask whether to run this same orchestrator on the nested component first, or park the host. |

**Two things the subagent must never do:** create a new component to fill a slot
that already exists; duplicate a ZEUS component into the host to modify locally.

**Output to carry forward:** resolved nesting map (host layer → component used →
variant/property mapping), foreign-owned properties (excluded from Step 5),
same-family split, unresolved nesting flagged with reason, whether anything
needs exposing in Step 4b.

### Step 3 — State Layer Setup *(only if `isInteractive`)*

**Goal:** Add the existing, published State Layer component instance for hover,
pressed, focus, and disabled visual feedback. Never create a new State Layer;
always instance the existing one (node `388:824`, file `2qBaAweE1afdX1UCMxtyId`).

**Depends only on Step 1's output** (host background/context, corner radius) —
not on Step 2 — which is why it can run in the same parallel batch.

**Subagent prompt must include:** the Figma link, Step 1's output, and
instructions to `Read` `references/add-state-layer.md` in full, then:
1. Determine shape (Default / Pill / Checkbox — border-radius ≥ half height ⇒
   Pill) and color (Primary default; Black if host bg is primary-blue or
   neutral/gray per the base component's intent; Error if host has an error
   state; AI only if operator confirms; White only if operator says so —
   `needs_operator_decision` if ambiguous).
2. Instance the State Layer, position as bottom-most content layer (above
   background fill, below all content), absolute-position stretched to fill
   (Left+Right+Top+Bottom), match corner radius to host, set color/shape/state
   properties, sync `State` to the host's state mechanism via `setProperties()`.
3. Turn `clipsContent` off on every host variant that can show Focused — the
   ring extends outside the frame and gets silently clipped otherwise.
4. Never modify the State Layer's own variables (`Statelayer.Styles` /
   `Statelayer.Measures`).
5. Verify wiring with `getMainComponentAsync()` on each created instance, not by
   inferring from property values.

**Note for the subagent to flag, not fix:** if the base component already has
hand-rolled hover/focus feedback (background-color swaps per state, drop-shadow
focus rings), removing those hardcoded values is Step 7a's job, not Step 3's —
**except** a hand-rolled focus shadow that would double up with the State
Layer's own focus stroke should be removed now, since leaving both produces a
visibly broken double ring. Flag any hardcoded hover background left in place so
Step 7a picks it up.

**Output to carry forward:** state layer added (yes/no), shape, color(s) used
(may be more than one across variants), any hand-rolled treatment removed or
flagged for Step 7a.

### Step 4a — Normalize Component Properties (structure only) *(only if `hasNormalizableProperties`)*

**Goal:** Replace the vendor's Figma component properties with ZEUS 3.0
convention — delete dead ones, retype survivors, rename. **Does not** expose
nested-instance properties — that's 4b, after Step 2 has finished.

**Subagent prompt must include:** the Figma link, Step 1's property list, and
instructions to `Read` `references/component-property-spec.md` in full and
follow it exactly:
1. Inventory and verify every property — count only values that render a real
   visual difference.
2. Delete single-value properties.
3. Assign type per the spec's table (delete / boolean / variant / instance swap
   / text). `State` is **always** a variant, default renamed `Enabled`.
4. Apply the boolean-cluster-vs-variant-dropdown rule for independent optional
   slots — `needs_operator_decision` for anything near the line (headers, input
   fields, list rows); never guess.

**Explicitly out of scope for 4a:** exposing nested-instance properties. Leave a
placeholder note in the output for which slots *will* need exposure once Step
2's result is available, if that's visually obvious, but do not attempt the
exposure itself.

**Output to carry forward:** property table so far (name, type, values),
properties deleted and why, any operator-confirmed structure from the gate.

---

## Join A

Wait for every subagent dispatched in Batch A to return. Resolve any
`needs_operator_decision` from any of them before continuing — get the answer
from the operator and fold it into workflow state (a follow-up subagent call may
be needed to apply that answer before Join A is considered closed).

Then determine whether Step 4b is needed: only if Step 2 ran and its resolved
nesting map contains something to expose (AssistiveContent, any same-family or
foreign nested instance). If Step 2 didn't run or found nothing to expose, skip
straight to Step 5 with Step 4a's property table (or Step 1's raw list, if 4a
also didn't run) as-is.

---

## Step 4b — Expose Nested Instance Properties *(solo, only if triggered by Join A)*

**Goal:** For AssistiveContent and every component resolved as nested in Step 2,
expose the nested instance's properties on the host rather than rebuilding
equivalents.

**Subagent prompt must include:** the Figma link, Step 2's finished resolved
nesting map, Step 4a's property table (or Step 1's raw property list if 4a
didn't run), and instructions to:
1. For each resolved nested instance, expose its relevant properties on the
   host component.
2. Never create a host-level property that only mirrors and forwards to a
   nested one — exposure is the mechanism, duplication is the failure.
3. The one exception: the State Layer's `State` property is *driven* from the
   host's own `State` (already wired in Step 3), not exposed the other way.

**Output to carry forward:** final property table (Step 4a's table plus exposed
nested properties) — this is what Step 5 consumes.

---

## Step 5 — Create Variable Collections *(solo, always runs)*

**Goal:** Create `{Component}.Styles` and `{Component}.Measures` with correct
variables, modes, groups, naming, and scopes. This step has no skip condition —
every adapted component needs variables, even a minimal one.

**Subagent prompt must include:** the Figma link, the final property table
(from 4b, or 4a, or Step 1 — whichever actually ran last), Step 2's
foreign-owned-properties list and same-family split (empty if Step 2 didn't
run), Step 3's output (whether a State Layer was added — affects whether Styles
needs UI-state modes at all), and instructions to `Read`
`references/component-variable-collection-spec.md` (primary spec) and
`references/naming-convention.md`, then follow Section 7's execution order
exactly:
1. Determine whether this component gets its own collection pair at all (see
   the spec's family-position table — composed wrappers of an existing base
   control extend that family's collection with a new group instead).
2. Decide what variables to create (Styles vs. Measures checklists), excluding
   anything owned by a foreign nested component and never duplicating
   state-layer variables.
3. Decide mode structure (Measures modes = scaling only; Styles modes = UI
   states **only if** the State Layer can't cover it).
4. Decide group structure (Measures groups = structural/content variants and the
   composed-component split; Styles groups = semantic/color variants).
5. Name per `{component}-{element}-{property}-{modifier}`.
6. Set explicit, non-default scopes per the spec's Section 4 table.
7. Create everything via `use_figma`, returning every created variable ID and
   collection ID.

**Output to carry forward:** collection IDs, variable IDs by name, mode names,
group names.

---

## Batch B (parallel) — Steps 6 and 7a

Both depend only on Step 5's output and touch different targets (Step 6 writes
variable→foundation-token aliases; Step 7a writes layer→variable bindings), so
dispatch both `Agent` calls in a single message once Step 5 has returned.

### Step 6 — Assign Foundation Tokens

**Goal:** Alias every component variable created in Step 5 to the matching
ZEUS 3.0 Foundations token.

**Subagent prompt must include:** the Figma link, Step 5's variable list, and
instructions to `Read` `references/foundation-token-map.md` in full, then:
1. For each variable, find the matching foundation token by **intent**, not by
   coincidental value match.
2. Apply as an alias binding via `use_figma`.
3. If no fitting token exists and no close-enough token covers the same intent,
   use a hardcoded value (a **snowflake**) rather than misusing a wrong-category
   token — a wrong-category binding is worse than a snowflake.
4. Collect every snowflake with its variable name, value, and reason — these
   must be flagged to the operator, never silently introduced (not a blocking
   gate — snowflakes are expected — but always reported).

**Output to carry forward:** token bindings applied (count), list of snowflakes
with variable/value/reason.

### Step 7a — Apply Variables & Clean Up

**Goal:** Bind component variables to the actual layers, and strip anything that
doesn't belong in the ZEUS 3.0 system. **Does not** run the validation
checklist — that's 7b, after Join B.

**Subagent prompt must include:** the Figma link, Step 5's variable list, any
hand-rolled treatment flagged by Step 3, and instructions to `Read`
`references/cleanup-rules.md` in full and apply every rule (whole-component
opacity for disabled, shadow-as-focus-ring, border-color token restriction,
fixed-size-token misuse, assistive-text-as-local-layer, leftover single-value
properties), then:
1. Bind every layer/element to its correct variable (fill, stroke, corner
   radius, padding, etc.).
2. Strip non-system styling per the cleanup rules.
3. **When uncertain whether to remove something, STOP and return
   `needs_operator_decision`** — never silently remove a visual property the
   subagent is unsure about.

**Output to carry forward:** bindings applied, cleanup actions taken, any items
left pending operator decision.

---

## Join B

Wait for both Step 6 and Step 7a to return. Resolve any `needs_operator_decision`
from either before continuing.

---

## Step 7b — Validate *(performed directly by the orchestrator, no subagent — solo, only after Join B)*

**Goal:** Run the full validation checklist now that both the token bindings
(Step 6) and the layer bindings/cleanup (Step 7a) are complete — several checks
need both halves finished to be meaningful.

**Do this yourself, using your own tool calls** (not a subagent) — `Read`
`references/validation-checklist.md`, then for each check, re-inspect the
actual Figma state (`get_metadata`, `get_variable_defs`, `get_screenshot` as
needed) rather than trusting Step 6's and 7a's self-reported JSON at face value,
and report pass/fail per check. Do not silently "fix and re-check" — if
something fails, that's the result; a fix is a follow-up, operator-directed
action decided in Step 8, not something to resolve here on your own.

**Output to carry forward:** validation checklist results (pass/fail per item),
based on your own direct inspection.

---

## Step 8 — Review & Feedback

**Goal:** Present the completed component for operator approval and collect
feedback. This step is **this session's job**, not a subagent's — it's where all
carried-forward state gets synthesized for the human.

**Actions (done by the orchestrator directly, no subagent needed unless
corrections require Figma writes):**
1. Summarize everything from Steps 1–7b: classification, which conditional steps
   ran vs. were skipped and why, state layer (added/skipped, variant), final
   property table, variables created (per collection: count, modes, groups),
   foundation tokens applied + snowflakes, cleanup actions, validation checklist
   results.
2. Ask the operator to review in Figma and confirm: correct across all
   variants/states? Variable values correct? Anything missing?
3. If corrections are needed, spin up a **new, narrowly-scoped subagent** for
   just that fix — do not re-run the whole pipeline.

### Self-Improvement

After the operator confirms or gives feedback, evaluate whether anything from
this run should update the reference docs under
`C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\` — a new
vendor-name → ZEUS-name mapping, a missing foundation token worth recording, a
cleanup rule that should become explicit, a classification edge case, or a
gating condition (e.g. what counts as "trivial enough to skip 4a") that turned
out to need sharper criteria. If yes, propose the specific doc edit to the
operator; with their approval, edit that reference file directly (it's a local
file, no subagent needed for a doc edit).

---

## Reference Documents

All paths are absolute under this skill's own folder — always usable regardless
of which project/session invoked this skill.

| Reference | Used in | Description |
|---|---|---|
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\component-scaling-classification.md` | Step 1 *(read directly by the orchestrator)* | Class A/B/C scaling decision framework |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\nested-component-map.md` | Step 2 | Always-use system components, family pairs, vendor→ZEUS name mappings |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\add-state-layer.md` | Step 3 | State Layer insertion spec |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\component-property-spec.md` | Step 4a / 4b | Component property types, naming, boolean-vs-variant gate |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\component-variable-collection-spec.md` | Step 5 | Full spec for Styles/Measures collections |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\naming-convention.md` | Step 5 | `{component}-{element}-{property}-{modifier}` pattern |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\foundation-token-map.md` | Step 6 | Component property → foundation token mapping |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\cleanup-rules.md` | Step 7a | Explicit list of base-component patterns to strip |
| `C:\Users\jakobs\.claude\skills\zeus-adapt-component\references\validation-checklist.md` | Step 7b *(read directly by the orchestrator)* | Quality gate run after Join B, before presenting results |

---

## Operator Communication Rules

- **Always present Step 1's classification before proceeding**, including which
  of Steps 2/3/4a will run and which will be skipped and why.
- **Never let a subagent create a component that already exists in the
  system.** Step 2: search first, swap second, stop and ask if no match found.
- **Never create a second variable collection for a composed component.** One
  family, one collection pair, split by groups (Step 5).
- **Any assistive/help text means an AssistiveContent instance** — never a
  local text row (Step 2).
- **Never let a subagent guess a boolean-vs-variant split** on a complex
  component — relay both structures to the operator and confirm (Step 4a).
- **Single-value properties are always deleted**, never kept for later (Step 4a).
- **Always relay "uncertain, should I remove this?" to the operator** rather
  than deciding on the orchestrator's own judgment (Step 7a) — and this applies
  to *any* unexpected situation in *any* step, per the universal never-guess
  rule, not just the ones named here. This includes Steps 1 and 7b, which the
  orchestrator runs itself — being the one doing the work doesn't mean it gets
  to decide ambiguous calls alone; it still asks.
- **Always present the full Step 8 summary at the end**, including the 7b
  validation checklist pass/fail and which conditional steps ran.
- **Flag missing foundation tokens (snowflakes) immediately** — don't silently
  substitute (Step 6).
- **If a subagent's step fails, returns an error, or produces unexpected
  results — or the orchestrator's own Step 1 or Step 7b work turns up
  something unexpected:** stop, report to the operator, and ask how to proceed
  rather than continuing the pipeline on bad data. Do not paper over a failed
  step by having a later step "figure it out."
- **Step 7b re-inspects Figma itself rather than trusting Step 6's and 7a's
  self-reported JSON.** A subagent's own report of what it did is a starting
  point, not proof — the point of a direct, independent validation pass is to
  actually check.
- **State only flows forward through this session, never sideways between
  subagents in the same batch.** Each subagent gets exactly the state this
  session explicitly includes in its prompt — never assume a subagent can see a
  prior subagent's transcript, or a sibling's result from the same parallel
  batch.
