# feedback-driven-design

> **Design is cheap. Rewrites are expensive.**

A Claude Code skill for iterative, feedback-driven UI design. Lock in how something looks *before* writing production code — through a structured spec, sandbox, and feedback loop — then promote the approved design with zero visual drift.

Built for **Next.js 15.1 · React 19 · Tailwind CSS 3.4 · TypeScript**.

---

## The problem this solves

The typical Claude Code UI workflow goes like this:

1. Ask Claude to build a component
2. Hate how it looks
3. Ask for changes
4. Watch Claude regenerate 400 lines to fix a padding value
5. Repeat until you give up or it's good enough

Every iteration rewrites the whole file. Token cost scales with file size, not change size. And you're making design decisions while reading code, not looking at pixels.

This skill flips the order. Design decisions happen visually, in a sandbox, before any production code exists. By the time Claude writes the real component, the design is already approved.

---

## How it works

```
PHASE 1 → Spec        Markdown layout spec, confirmed before any code
PHASE 2 → Draft       scratch/component.v1.tsx, opened in browser immediately
PHASE 3 → Loop        Review → Parse feedback → Surgical edit → Repeat
PHASE 4 → Freeze      Design approved, tokens extracted, summary written
PHASE 5 → Promote     Production component written once, correctly
```

### Phase 1 — Layout Spec

Claude drafts a Markdown spec covering: purpose, page/component placement, grid structure, aesthetic direction, key states, responsive breakpoints, and a realistic content model. You sign off before a single line of code is written.

### Phase 2 — Sandbox Draft

Claude builds the component in `scratch/component.v1.tsx`:

- `'use client'` at the top — no Server Component complexity during design
- **Tailwind only** — zero inline styles, zero `<style>` blocks
- Hardcoded realistic mock data (not Lorem Ipsum)
- All states (loading, empty, error, populated) togglable via a fixed button strip
- `DESIGN_TOKENS` exported as Tailwind class fragments
- Version badge in the corner
- Browser opened immediately

### Phase 3 — Feedback Loop

You look at it. You give feedback. Claude:

1. Parses it into 🔴 Critical / 🟡 Refine / 🟢 Noted
2. Acknowledges what it heard before touching any code
3. Runs `cp v1.tsx v2.tsx`, then **`str_replace` only the changed lines** — never a full rewrite for a minor change
4. Shows a diff summary before opening the browser
5. Repeat

Token cost scales with the **size of the change**, not the size of the file.

### Phase 4 — Design Freeze

You say "approved". Claude renames the file to `.APPROVED.tsx`, prints the final `DESIGN_TOKENS`, and writes a design summary capturing everything decided during iteration — including deferred items.

### Phase 5 — Production Promotion

Claude writes the production component **once**, correctly:

- Real TypeScript props interface derived from the mock data shape
- Tailwind classNames copied **verbatim** from the approved sandbox — no translation, no drift
- Server vs Client Component boundary decided based on interactivity
- Pages get `page.tsx` + `loading.tsx` + `error.tsx`
- Reusable components get a barrel export and a types file
- Design tokens saved to `lib/design-tokens/`

---

## Installation

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Unzip the skill into it
unzip feedback-driven-design.skill -d ~/.claude/skills/
```

That's it. The skill is now available in any Claude Code session.

### What gets installed

```
~/.claude/skills/
  feedback-driven-design/
    SKILL.md                          ← main skill file Claude reads
    references/
      aesthetic-directions.md        ← 9 named aesthetic archetypes for Phase 1
      feedback-templates.md          ← structured review prompts for Phase 3
```

---

## Triggering the skill

Claude picks up this skill automatically when you say things like:

- `"Design a dashboard component"`
- `"Let's mockup the settings page before we build it"`
- `"Show me what it could look like first"`
- `"I want to see it before we implement"`
- `"Design first, then we'll build"`

---

## Surgical editing — the core rule

> Never regenerate the whole file when a targeted edit will do.

From Phase 2 onwards, every iteration follows this pattern:

```bash
# Copy previous version as new version (never overwrite)
cp scratch/component.v2.tsx scratch/component.v3.tsx

# Apply each change as a separate str_replace
# One logical change = one str_replace call
```

Each changed line gets an inline annotation:

```tsx
// v3: increased card padding per feedback (was p-4)
className="p-6 rounded-2xl bg-zinc-900"
```

Annotations are stripped when the design is frozen.

---

## DESIGN_TOKENS

Every sandbox file exports tokens as Tailwind class fragments — not hex values:

```ts
export const DESIGN_TOKENS = {
  colors:  { primary: 'violet-600', surface: 'zinc-900', accent: 'amber-500' },
  radii:   { card: 'rounded-2xl', button: 'rounded-lg' },
  spacing: { section: 'py-16', gap: 'gap-6', padding: 'p-6' },
  font:    { display: 'font-semibold tracking-tight', label: 'text-xs font-mono uppercase' },
}
```

These are real Tailwind classes. One change in the token object propagates everywhere it's referenced — and they travel to production without any conversion.

---

## Stack

| | |
|---|---|
| Framework | Next.js 15.1 (App Router) |
| Runtime | React 19 |
| Styling | Tailwind CSS 3.4 |
| Language | TypeScript |
| Sandbox format | `.tsx` — same as production |

Because the sandbox format matches production exactly, the "promote to production" step is mechanical. No rewriting inline styles as classNames. No guessing what hex value maps to which Tailwind class. The design just moves.

---

## File structure during a session

```
scratch/
  dashboard.spec.MD          ← Phase 1 layout spec
  dashboard.v1.tsx           ← initial draft
  dashboard.v2.tsx           ← after first round of feedback
  dashboard.v3.tsx           ← after the second round
  dashboard.APPROVED.tsx     ← frozen design
  dashboard.design-summary.md

src/
  components/
    Dashboard/
      Dashboard.tsx          ← production component (Phase 5)
      Dashboard.types.ts     ← TypeScript interfaces
      index.ts               ← barrel export
  lib/
    design-tokens/
      dashboard.tokens.ts    ← DESIGN_TOKENS reference
```

---

## References included

**`aesthetic-directions.md`** — 9 named aesthetic archetypes to help commit to a direction in Phase 1: Minimal/Editorial, Bento/Dashboard, Warm/Human, Brutalist, Luxury, Futuristic, Playful, Retro, Organic. Prevents "modern and clean" from becoming the brief.

**`feedback-templates.md`** — Structured review prompts for Phase 3, including a standard template, deep review (section by section), quick pulse (for late-stage minor tweaks), direction reset (for when iteration is going in circles), state review, and responsive review. Also includes a translation table for vague feedback like "feels heavy" or "feels cheap."

---

