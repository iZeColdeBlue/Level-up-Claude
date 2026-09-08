---
name: figma-skill-creator
description: >
  Create, review, and iterate on custom skills for the Figma Design Agent. Use this skill whenever
  someone wants to write a new Figma Agent skill, improve an existing one, turn a workflow into a
  reusable slash command, or review a draft skill for quality. Trigger on phrases like: "create a
  Figma skill", "write a skill for the agent", "turn this into a Figma skill", "review my Figma
  skill", "make a slash command for", "skill for the Figma agent", "/figma-skill", or when
  someone describes a repeatable design workflow they want to automate in Figma. Also trigger when
  someone says "agent skill", "design agent instruction", or references the Agent Skills spec in
  a Figma context. This skill produces single Markdown files that follow the Agent Skills
  specification and work with the Figma Design Agent and Figma Make.
---
 
# Figma Skill Creator
 
Create high-quality custom skills for the Figma Design Agent. This skill guides you through
scoping, drafting, and iterating on Markdown-based instruction files that the Figma agent
executes as slash commands.
 
Before writing anything, read [references/figma-skill-patterns.md](references/figma-skill-patterns.md)
to understand what makes Figma Agent skills effective and how they differ from Claude skills.
 
## What You're Producing
 
A Figma Agent skill is a **single Markdown file** — no scripts, no references directory, no
assets folder. Figma's custom skill upload accepts one `.md` file only. Everything the agent
needs must fit in that file.
 
The file follows the [Agent Skills specification](https://agentskills.io/specification):
- YAML frontmatter with `name` and `description` (required)
- Markdown body with the instructions
The `name` becomes the `/slash-command`. The `description` helps the agent decide when the skill
is relevant. The body teaches the agent what to do.
 
## Workflow
 
### 1. Capture Intent
 
Start by understanding what the skill should do. If the conversation already contains a workflow
the user wants to capture, extract the steps from it. Otherwise, ask:
 
1. **What task does this skill automate?** (e.g., "review designs against our brand guidelines,"
   "create a settings page layout," "prep a design crit")
2. **What's the trigger?** What would someone type to invoke this? This becomes the `/name`.
3. **What conventions or rules should the agent follow?** Component names, spacing values,
   color tokens, naming patterns, accessibility standards.
4. **What should the agent avoid?** Anti-patterns, common mistakes, things that look right
   but aren't.
5. **Does it need connector context?** Should the skill reference external tools via MCP
   connectors (Notion, Slack, Jira, etc.)?
### 2. Scope the Skill
 
Good Figma skills are narrow and specific. One skill, one job.
 
**Right-sized:** `/form-layout`, `/accessibility-check`, `/design-crit-prep`, `/brand-audit`
**Too broad:** `/design-everything`, `/follow-all-guidelines`, `/build-any-page`
 
If the user's workflow spans multiple distinct tasks, split it into separate skills that can
reference each other. A `/new-feature` skill can invoke `/form-layout` for input sections and
`/accessibility-check` for compliance review.
 
Target **under 200 lines** for the final `.md` file. Skills that exceed this tend to produce
inconsistent results because the agent loses focus. If the content naturally exceeds 200 lines,
that's a signal to split into multiple skills.
 
### 3. Draft the Skill
 
Use this structure as a starting template, then adapt to the specific task:
 
```markdown
---
name: skill-name
description: One-sentence description of what the skill does and when to use it.
---
 
# Skill Title
 
> Brief purpose statement — what this skill helps the agent do.
 
## Context
What the agent needs to understand about the design system, conventions, or
domain before acting.
 
## Steps
Ordered instructions the agent follows. Be specific about component names,
token values, and layout rules.
 
## Component Rules
Which published components to use and how (variants, props, when to use each).
 
## Conventions
Spacing, grid, typography, color token, and naming rules.
 
## Don'ts
Explicit anti-patterns. Concrete examples of what to avoid and why.
```
 
Not every section is needed for every skill. A review/audit skill won't have "Steps" in the
same way a generative skill does — it might have "What to Check" and "How to Report Findings"
instead. Adapt the structure to the task.
 
### 4. Apply Quality Checks
 
Before presenting the skill, verify it against these criteria:
 
**Specificity over generality.** Every instruction should be concrete enough that two different
agents would produce similar output. "Use appropriate spacing" fails this test. "16px between
form fields, 32px between sections" passes it.
 
**Name real components and tokens.** Don't say "use the primary button." Say "use
`Button/Primary/Medium`" or whatever the actual component path is in the team's library. If you
don't know the exact names, ask — the skill is only as good as its references to real assets.
 
**Explain the why for non-obvious rules.** "Never use placeholder text as the only label"
is clear but stronger as "Never use placeholder text as the only label — placeholders disappear
on focus, making the field unidentifiable for users and accessibility tools." The agent follows
rules better when it understands the reasoning.
 
**Include do/don't pairs for judgment calls.** When a rule involves subjective judgment,
show both sides:
- ✅ DO: "Use radio buttons for 2-3 mutually exclusive options"
- ❌ DON'T: "Use a dropdown for fewer than 4 options"
**Test the slash command name.** The `name` field becomes `/name` in Figma. It should be:
- Lowercase, hyphenated, max 64 characters
- Action-oriented and memorable: `/brand-audit`, `/form-layout`, `/crit-prep`
- Not generic: avoid `/check`, `/review`, `/build`
**Verify the description triggers correctly.** The description should include specific
keywords the agent uses to match skills to prompts. Include both what the skill does AND
when to use it.
 
### 5. Present and Iterate
 
Present the draft skill as a downloadable `.md` file. Explain:
- How to upload it in Figma (Skills → Add skill → Upload a file)
- How to invoke it (`/skill-name` in the chat)
- What to watch for in the first few runs
- How to publish it to the team once validated
After the user tests it in Figma, iterate based on what the agent got wrong. Common fixes:
- Agent ignored a rule → make it more prominent, add a "Don't" entry
- Agent used wrong components → double-check component names match the library exactly
- Output was inconsistent → reduce scope, remove ambiguous instructions
- Agent hallucinated steps → remove vague language, be more prescriptive
## Intact-Specific Guidance
 
When creating skills for the Intact Platform or Intact team workflows, apply these defaults
unless the user specifies otherwise:
 
- **Accessibility standard:** EN 301 549 as primary reference, WCAG 2.1 AA as secondary
- **Design system:** Reference Intact Platform components by their actual Figma library names
- **Color tokens:** Use the Intact token system (primary green `#2AAC65`, etc.) — never
  hardcode hex values when a token exists
- **Font:** Calibri for documents, Montserrat for presentations — check which context applies
- **Enterprise B2B context:** Intact Platform serves the TIC industry (testing, inspection,
  certification). Dense list views, compliance-oriented workflows, and power-user patterns
  are the norm. Consumer UX assumptions often don't transfer — flag when a pattern needs
  requalification for this context.
- **Connectors:** Intact uses Confluence (`intact-systems.atlassian.net`) and Jira. Skills
  that reference external documentation should note connector requirements.
## Review an Existing Skill
 
When reviewing rather than creating, check the draft against these common failure modes:
 
1. **Too vague:** Instructions the agent can interpret multiple ways
2. **Too long:** Over 200 lines — the agent loses focus
3. **Wrong component names:** References that don't match the actual Figma library
4. **Missing anti-patterns:** Rules without "Don'ts" produce inconsistent results
5. **No accessibility guidance:** Skills that generate UI should specify a11y standards
6. **Scope creep:** A single skill trying to do too many things
7. **Missing description keywords:** The description doesn't include the right trigger words
Provide specific, actionable feedback — not "this could be improved" but "line 47 says 'use
appropriate spacing' — replace with the actual spacing value from your design system."
