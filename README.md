# feedback-driven-design

> **Design is cheap. Rewrites are expensive.**

A Claude Code skill for iterative, feedback-driven UI design. Lock in how something
looks *before* writing production code — through a structured spec, sandbox, and
feedback loop — then promote the approved design into your codebase.

**Works with any stack.** The sandbox is plain HTML. The promote phase adapts to
whatever you're building with.

---

## The problem this solves

The typical Claude Code UI workflow:

1. Ask Claude to build a component
2. Hate how it looks
3. Ask for changes
4. Watch Claude regenerate 400 lines to fix a padding value
5. Repeat until you give up or it's good enough

Every iteration rewrites the whole file. Token cost scales with file size, not change
size. And you're making design decisions while reading code, not looking at pixels.

This skill flips the order. Design decisions happen visually, in a sandbox, before
any production code exists. By the time Claude writes the real component, the design
is already approved and you've seen it in a browser.

---

## How it works

```
PHASE 1 → Spec        Markdown layout spec, confirmed before any code
PHASE 2 → Draft       scratch/component.v1.html, opened in browser immediately
PHASE 3 → Loop        Review → Parse feedback → Surgical edit → Repeat
PHASE 4 → Freeze      Design approved, tokens extracted, summary written
PHASE 5 → Promote     Production code written once, correctly, for your stack
```

### Phase 1 — Layout Spec

Claude drafts a Markdown spec covering: purpose, sections, grid/layout structure,
aesthetic direction, key states (loading, empty, error, success), responsive
breakpoints, and a realistic content model. You confirm before a single line of
code is written.

Crucially: Claude also asks whether this is a **page** or a **reusable component**,
and what the intended production path is. This determines Phase 5 structure and
prevents surprises at the end.

### Phase 2 — Sandbox Draft

Claude builds the component in `scratch/component.v1.html` — a single self-contained
file with everything inline:

- Hardcoded realistic mock data (not Lorem Ipsum — real names, real numbers, real dates)
- All states from the spec, togglable via a fixed button strip in the corner
- `DESIGN_TOKENS` const at the top capturing all design decisions
- Version badge so you always know which iteration you're looking at
- Browser opened immediately — no build step, no server, just open the file

### Phase 3 — Feedback Loop

You look at it. You give feedback. Claude:

1. **Parses** it into 🔴 Critical / 🟡 Refine / 🟢 Noted
2. **Acknowledges** what it heard in 2–3 bullet points before touching any code
3. **Classifies** the edit — surgical (almost always) or full rewrite (only for
   total layout changes)
4. Runs `cp v1.html v2.html`, then **`str_replace` only the changed lines**
5. Shows a diff summary of what changed before opening the browser
6. Opens the browser

Repeat until you say "approved".

Token cost scales with the **size of the change**, not the size of the file.

### Phase 4 — Design Freeze

You say "approved". Claude:

- Renames the file to `scratch/component.APPROVED.html`
- Prints the final `DESIGN_TOKENS` clearly in the conversation
- Writes `scratch/component.design-summary.md` — approved version, aesthetic
  direction, tokens, states implemented, deferred items

### Phase 5 — Production Promotion

Claude writes production code **once**, from the frozen design. What gets written
depends on your stack — Claude adapts. The core rules are always the same:

- Zero visual changes from the approved sandbox. The design is frozen.
- Real data interfaces replacing hardcoded mock values
- All states implemented properly
- Styling values carried over from `DESIGN_TOKENS`
- Promotion is a mechanical translation, not a creative step

---

## Installation

```bash
mkdir -p ~/.claude/skills
unzip feedback-driven-design.skill -d ~/.claude/skills/
```

Done. The skill is available in any Claude Code session.

### What gets installed

```
~/.claude/skills/
  feedback-driven-design/
    SKILL.md                        ← main skill Claude reads
    references/
      aesthetic-directions.md      ← 9 named aesthetic archetypes for Phase 1
      feedback-templates.md        ← structured review prompts for Phase 3
    variants/
      nextjs-tailwind.md           ← drop-in upgrade for Next.js 15 + Tailwind 3.4
```

---

## Triggering the skill

Claude picks this up automatically when you say things like:

- `"Design a dashboard component"`
- `"Let's mockup the settings page before we build it"`
- `"Show me what it could look like first"`
- `"I want to see the design before we implement"`
- `"Design first, then we'll build"`
- `"I'm not sure about the layout — can we figure it out visually?"`

---

## Surgical editing — the core rule

> Never regenerate the whole file when a targeted edit will do.

From Phase 2 onwards, every iteration follows this pattern:

```bash
# Copy previous version as new version — never overwrite
cp scratch/component.v2.html scratch/component.v3.html

# Apply each change as a separate str_replace
# One logical change = one str_replace call
```

**Decision tree:**

```
Is the change structural (total layout paradigm)?
  YES → Full rewrite (rare)
  NO  → Surgical. Always.
```

Examples of surgical changes: color values, font sizes, spacing, adding a state,
swapping a chart type, reordering sections, changing grid columns.

Each changed block gets an inline comment:

```html
<!-- v3: increased card padding per feedback (was 16px) -->
<div style="padding: 24px;">
```

Comments are stripped when the design is frozen in Phase 4.

---

## DESIGN_TOKENS

Every sandbox file exports a `DESIGN_TOKENS` const at the top capturing all design
decisions as a single source of truth:

```js
const DESIGN_TOKENS = {
  colors:  { primary: '#7C3AED', surface: '#18181B', accent: '#F59E0B', text: '#F4F4F5' },
  radii:   { card: '16px', button: '8px', badge: '999px' },
  spacing: { section: '64px', gap: '24px', padding: '24px' },
  font:    { display: '600 28px/1.2 system-ui', label: '500 11px/1 system-ui' },
}
```

One change here propagates everywhere it's referenced. In Phase 5, these values
travel directly into your production styling — CSS variables, Tailwind config,
design system tokens, or whatever your stack uses.

---

## What good mock data looks like

The sandbox uses real-looking domain data, not placeholders. This matters because
you can't evaluate visual hierarchy with fake content.

| Instead of | Use |
|---|---|
| "Card Title" | "Q3 Revenue Analysis — APAC" |
| "User Name" | "Priya Nair" |
| "Description here" | "Shipped 3 days ahead of schedule" |
| 42 | $284,500 |
| "Tag" | "In Review" |

---

## File structure during a session

```
scratch/
  dashboard.spec.md           ← Phase 1 layout spec
  dashboard.v1.html           ← initial draft
  dashboard.v2.html           ← after first round of feedback
  dashboard.v3.html           ← after second round
  dashboard.APPROVED.html     ← frozen design
  dashboard.design-summary.md ← tokens, states, deferred items

your-project/
  [wherever your stack puts components — Claude adapts in Phase 5]
```

---

## References included

**`aesthetic-directions.md`** — 9 named aesthetic archetypes to help commit to a
clear direction in Phase 1 instead of settling for "modern and clean":

- Minimal / Editorial
- Bento / Dashboard
- Warm / Human
- Brutalist / Raw
- Luxury / Refined
- Futuristic / Sci-Fi
- Playful / Toy-like
- Retro / Nostalgic
- Organic / Natural

Each includes font pairings, color direction, motion character, and what it's good for.

**`feedback-templates.md`** — Structured review prompts Claude uses during Phase 3:

- **Standard** — default for most components
- **Deep review** — section-by-section rating for complex pages
- **Quick pulse** — lightweight check for late-stage minor tweaks
- **Direction reset** — surfaces circular feedback and forces a commitment
- **State review** — per-state feedback (loading, empty, error, success)
- **Responsive review** — breakpoint-by-breakpoint check

Also includes a translation table for vague feedback:

| You say | Claude hears |
|---|---|
| "Feels heavy" | Reduce weight, increase whitespace, lighter background |
| "Feels cheap" | Improve spacing, fewer colors, better typography |
| "Feels cold" | Warmer color temperature, rounder shapes, softer shadows |
| "Doesn't feel premium" | Slow animations, reduce palette to 2–3, tighten type |

---

## Stack-specific variant

If you're on **Next.js 15 + React 19 + Tailwind CSS 3.4**, a drop-in variant is
included at `variants/nextjs-tailwind.md`. It replaces the HTML sandbox with `.tsx`
files using Tailwind utility classes, so every `className` travels to production
without any translation. See the variant file for full details.

---

## Anti-patterns this skill prevents

| Anti-pattern | What happens instead |
|---|---|
| Full rewrite for minor feedback | `cp + str_replace` surgical edits only |
| Editing from memory | `view` file before every `str_replace` |
| Design decisions while reading code | All decisions made in a browser, visually |
| Production code before design is settled | Phase 5 only runs after explicit approval |
| Generic placeholder content | Realistic mock data enforced from Phase 2 |
| Silent design changes | Diff summary shown before every browser open |
| Going in circles on direction | Direction reset prompt surfaces the conflict |

---
