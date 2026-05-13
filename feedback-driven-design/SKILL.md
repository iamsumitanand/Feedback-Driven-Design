---
name: feedback-driven-design
description: >
  Iterative visual design workflow for components and pages. Use this skill whenever
  the user wants to design, prototype, or refine a UI component, page layout, or web
  interface BEFORE writing production code. Triggers include: "design a component",
  "let's mockup", "show me what it could look like", "design first then implement",
  "I want to see it before we build it", "feedback loop design", "iterate on the UI".
  Also trigger when the user says things like "I'm not sure about the layout" or
  "let's figure out the design first". This skill saves tokens and prevents
  expensive rewrites by locking in design decisions in a sandbox before any
  production code is written.

stack:
  framework: Next.js 15.1 (App Router)
  runtime: React 19
  styling: Tailwind CSS 3.4
  language: TypeScript
  sandbox-format: .tsx (Tailwind only — no inline styles, no CSS blocks)
---

# Feedback-Driven Design Skill

A token-efficient, iterative design workflow. Designs are validated visually and approved
by the user in a sandbox before a single line of production code is written.

---

## Philosophy

> **Design is cheap. Rewrites are expensive.**

The goal is to compress all design uncertainty into a fast, visual feedback loop. Each
iteration is a focused conversation between Claude and the user about what looks right —
not about implementation details. Production code is written *once*, after the design is
locked.

---

## Workflow Overview

```
PHASE 1 → Spec         [Layout Spec in Markdown]
PHASE 2 → Draft        [Component built in scratch/]
PHASE 3 → Feedback     [User reviews → Claude revises → repeat]
PHASE 4 → Freeze       [Design approved, tokens extracted]
PHASE 5 → Promote      [Styling moved to production]
```

---

## PHASE 1 — Layout Spec

**Goal**: Align on structure before pixels.

Brainstorm and draft a Markdown layout spec. This is a low-cost, high-value step that
prevents fundamental layout disputes from surfacing mid-iteration.

The spec must define:

- **Purpose** — What problem does this component/page solve? Who uses it?
- **Placement** — Ask the user: is this a page (`app/`) or a reusable component
  (`components/`)? Record the intended production path (e.g. `app/dashboard/page.tsx`
  or `components/ui/StatCard.tsx`). This determines Phase 5 structure.
- **Sections** — List every visible region (header, sidebar, content area, footer, modals)
- **Grid/Layout** — Bento grid, flex columns, sidebar+main, full-bleed, etc. Be explicit.
- **Navigation Flow** — How does a user move through or interact with this component?
- **Key States** — Empty state, loading state, error state, success state (only relevant ones)
- **Responsive Breakpoints** — Tailwind breakpoints only: `sm` (640px), `md` (768px),
  `lg` (1024px), `xl` (1280px). State which breakpoints matter and whether mobile-first.
- **Aesthetic Direction** — One sentence. E.g. "Soft editorial, lots of whitespace, muted
  pastels with a single bold accent". See `references/aesthetic-directions.md` for inspiration.
- **Content Model** — What real data will this render? Define realistic mock values now.

Save spec to: `scratch/[component_name].spec.md`

**Do not proceed to Phase 2 until the user has confirmed the spec or said "looks good, go".**

---

## PHASE 2 — Sandbox Draft

**Goal**: Build a visually complete, isolated mockup. No real data. No API calls.

### File location
```
scratch/[component_name].v1.tsx     ← first draft
scratch/[component_name].v2.tsx     ← second iteration, etc.
```

Always version the file. Never overwrite `v1` — each iteration creates a new file.
This gives the user a free "go back" option at any point.

### Build instructions

1. Read the approved spec (`scratch/[component_name].spec.md`).
2. Read the `frontend-design` skill (at `/mnt/skills/public/frontend-design/SKILL.md`) and
   apply all its aesthetic guidelines fully.
3. **Every sandbox file starts with these two lines — no exceptions:**
   ```tsx
   'use client'
   // sandbox mockup — hardcoded data, no API calls, no server logic
   ```
   This keeps the file valid in Next.js 15 App Router without any Server Component
   concerns. Server/Client boundaries are a production concern, not a design concern.

4. **Tailwind only — zero inline styles, zero `<style>` blocks.**
   Every visual property must be a Tailwind utility class. For values outside the
   default scale, use arbitrary values: `w-[420px]`, `top-[72px]`, `bg-[#1a1a2e]`.
   This means every className travels to production without translation.

5. **Keep the component flat and lean.** One function component per file. No custom
   hooks, no `interface` declarations, no helper components in separate files. The goal
   is readable, patchable TSX — not clean architecture.

6. **Mock data pattern** — hardcode as a typed-loosely `const` above the component:
   ```tsx
   const MOCK = {
     user: { name: 'Priya Nair', role: 'Admin' },
     stats: [
       { label: 'Revenue', value: '₹2,84,500', delta: '+12%', up: true },
       { label: 'Orders',  value: '1,847',      delta: '+4%',  up: true },
     ],
     chart: [40, 65, 55, 80, 72, 90, 85],
   }
   ```
   No `interface`, no `type` — those are Phase 5. Use real-looking domain data, not
   placeholder text.

7. **State toggles** — implement all states from the spec using a single `useState`:
   ```tsx
   const [view, setView] = useState<'loaded' | 'loading' | 'empty' | 'error'>('loaded')
   ```
   Render toggle buttons in a fixed corner strip so the user can flip between states
   without reloading:
   ```tsx
   <div className="fixed bottom-4 right-4 flex gap-2 z-50">
     {(['loaded','loading','empty','error'] as const).map(s => (
       <button key={s} onClick={() => setView(s)}
         className={`px-3 py-1 rounded-full text-xs font-mono border
           ${view === s ? 'bg-black text-white' : 'bg-white text-black'}`}>
         {s}
       </button>
     ))}
   </div>
   ```

8. **Export DESIGN_TOKENS** as Tailwind class fragments — not hex values:
   ```ts
   export const DESIGN_TOKENS = {
     colors:  { primary: 'blue-600',  surface: 'zinc-900', accent: 'amber-500', text: 'zinc-100' },
     radii:   { card: 'rounded-2xl',  button: 'rounded-lg', badge: 'rounded-full' },
     spacing: { section: 'py-16',     gap: 'gap-6',         padding: 'p-6' },
     font:    { display: 'font-semibold tracking-tight', label: 'text-xs font-mono uppercase' },
   }
   ```
   These are real Tailwind classes. They slot directly into production without any
   conversion step.

9. **Version badge** — floating label in the bottom-left so the user always knows
   which version they're looking at:
   ```tsx
   <div className="fixed bottom-4 left-4 px-2 py-1 bg-black/60 text-white
                   text-xs font-mono rounded z-50">
     v{VERSION}
   </div>
   ```
   Set `const VERSION = 1` at the top of each file.

10. After saving, **open in browser** immediately. Don't wait for the user to ask.

### What good mock data looks like

| Bad | Good |
|---|---|
| `"Card Title"` | `"Q3 Revenue Analysis — APAC"` |
| `"User Name"` | `"Priya Nair"` |
| `"Description here"` | `"Shipped 3 days ahead of schedule"` |
| `42` | `"₹2,84,500"` |
| `className="text-blue-500"` | `className={cn(DESIGN_TOKENS.colors.accent, 'text-sm')}` |

### Tailwind class discipline

- Use **semantic token references** via `DESIGN_TOKENS` wherever possible — not raw
  color classes scattered through JSX. This makes surgical edits cheaper: one change
  in `DESIGN_TOKENS` propagates everywhere.
- Compose complex class strings with `cn()` (shadcn pattern) or template literals.
  Do **not** use `clsx` or `classnames` — assume only the standard Next.js + Tailwind
  setup is available in the sandbox.
- Responsive classes follow mobile-first: `className="flex-col md:flex-row"`
- Dark mode: if the design calls for it, use Tailwind's `dark:` prefix. Default
  to a single mode unless the spec explicitly mentions dark/light toggle.

---

## PHASE 3 — Feedback Loop

**Goal**: Iterate fast. Stay in the sandbox. Don't touch production.

### After opening the browser

Show the user this prompt block (copy-paste ready):

```
──────────────────────────────────────────────────
🎨 DESIGN REVIEW  ·  v[N]
──────────────────────────────────────────────────
Tell me what you think! You can use any format, or
answer these questions:

  ✅ What's working well?
  ❌ What feels off?
  🔧 Specific changes (layout, color, type, spacing)?
  📐 Anything missing from the spec?
  📱 Does the responsive behavior feel right?

Or just say "approved" to lock this design. 🔒
──────────────────────────────────────────────────
```

### Processing feedback

When the user responds with feedback:

1. **Parse the feedback** into three buckets:
   - 🔴 **Critical** — Must fix before next show (broken layout, wrong hierarchy)
   - 🟡 **Refine** — Improve in next iteration (color, spacing, type)
   - 🟢 **Noted** — Nice-to-have, do if easy (micro-interactions, subtle details)

2. **Acknowledge** — Summarize what you heard in 2–3 bullet points before coding.
   This prevents silent misinterpretation.

3. **Classify the edit scope** — before writing a single line of code, decide which
   strategy applies (see Surgical Editing below).

4. **Show a Diff Summary** before opening the browser — a short list of what changed:
   ```
   ✦ Changes in v[N+1]:
   • Increased card padding from 16px → 24px  [surgical]
   • Replaced purple accent with #E8631A (warm amber)  [surgical]
   • Added skeleton loader for empty state  [surgical]
   • Reduced font size on metadata from 14px → 12px  [surgical]
   ```

5. Save the new version: `scratch/[component_name].v[N+1].tsx`

6. Open in browser.

7. Repeat from top of Phase 3.

---

### Surgical Editing — the core iteration rule

> **Never regenerate the whole file when a targeted edit will do.**
> Rewriting 400 lines to change a padding value wastes tokens, risks introducing
> regressions, and makes diffs impossible to read.

#### Edit Strategy Decision Tree

```
Is the feedback about layout structure?
  YES → Is it a total restructure (grid → sidebar, bento → list)?
          YES → Full rewrite (rare, only for fundamental layout changes)
          NO  → Surgical: move/reorder JSX blocks, update grid config
  NO  → Surgical edit. Always.
```

#### What counts as surgical vs. full rewrite

| Change type | Strategy | Example |
|---|---|---|
| Color value | Surgical | `blue-600` → `violet-600` in DESIGN_TOKENS |
| Font size / weight | Surgical | `text-sm` → `text-xs` |
| Spacing / padding | Surgical | `p-4` → `p-6` |
| Tailwind arbitrary value | Surgical | `w-[340px]` → `w-[420px]` |
| Adding a new state | Surgical | Add `'skeleton'` to state union + render block |
| Swapping a component | Surgical | Replace `<BarChart>` with `<LineChart>` |
| Reordering sections | Surgical | Move JSX block up/down |
| Changing grid columns | Surgical | `grid-cols-2` → `grid-cols-3` |
| New section added | Surgical | Insert new JSX block at correct position |
| Total layout paradigm change | Full rewrite | Grid → vertical stack |
| Aesthetic direction reversal | Full rewrite | Dark luxury → light editorial |

#### How to apply surgical edits

Copy the previous version file first, then use `str_replace` to patch only the
changed regions. Never use `create_file` on an iteration — only on v1.

```bash
# Step 1: Copy previous version as new version
cp scratch/[name].v2.tsx scratch/[name].v3.tsx

# Step 2: Apply each change as a targeted str_replace
# Change 1: update accent color in DESIGN_TOKENS
# Change 2: update card padding
# Change 3: add new state block
# ... each change is one str_replace call
```

Each `str_replace` call should be the **minimum contiguous block** needed to make
the change. If 3 lines change, replace those 3 lines — not the whole function.

#### Annotate surgical changes in the file

Add a comment above each changed block so the version history is readable:

```tsx
// v3: increased card padding per feedback (was p-4)
className="p-6 rounded-2xl bg-zinc-900"

// v3: swapped accent from blue-600 → violet-600
primary: 'violet-600',
```

Remove these annotations when the design is frozen in Phase 4 to keep production
code clean.

#### When feedback touches the same region as a previous edit

Read the current file before editing — do not rely on memory of what's in the file.
Use `view` to read the exact lines, then `str_replace` the precise block.

```
view scratch/[name].v2.tsx   ← always read before editing
```

This prevents accidental double-application or stale context edits.

### Loop rules

- **Never touch production files** during Phase 3.
- **Always version** — never overwrite a previous `.vN.tsx` file.
- **Keep DESIGN_TOKENS updated** in each version file.
- **If the spec needs to change**, update `scratch/[component_name].spec.md` and call it out
  explicitly: "I'm updating the spec to reflect this change — is that OK?"
- **If the user is going in circles**, gently surface the last two versions side by side
  and ask: "v2 felt more X, v4 feels more Y — which direction do you want to commit to?"

---

## PHASE 4 — Design Freeze

When the user says "approved" / "looks good" / "ship it" / "let's go":

1. **Rename** the approved file: `scratch/[component_name].APPROVED.tsx`
2. **Print the final DESIGN_TOKENS block** clearly in the conversation.
3. **Generate a one-page Design Summary** (`scratch/[component_name].design-summary.md`):
   - Approved version number
   - Final aesthetic direction
   - DESIGN_TOKENS
   - List of all states implemented
   - Responsive behaviour notes
   - Any deferred items (things the user said "nice to have later")
4. Confirm with user: "Design locked. Ready to promote to production?"

---

## PHASE 5 — Production Promotion

**Goal**: Translate the approved sandbox component into production Next.js code. This is
a mechanical step — no design decisions, no layout changes, no aesthetic deviations.

### Pre-promotion check

Read the spec to confirm:
- Is this a **page** (`app/`) or a **reusable component** (`components/`)?
- Does it need to be a Server Component or Client Component in production?
  - If it has `onClick`, `useState`, `useEffect`, or any browser API → `'use client'`
  - If it only renders data passed as props with no interactivity → Server Component
    (remove `'use client'` directive)

### File output — Reusable Component

```
components/
  [ComponentName]/
    [ComponentName].tsx        ← production component
    [ComponentName].types.ts   ← TypeScript interfaces extracted from MOCK
    index.ts                   ← barrel export
```

`[ComponentName].tsx`:
- Replace `const MOCK = {...}` with a proper typed props interface from `.types.ts`
- Replace all hardcoded values with `props.fieldName`
- Keep every Tailwind className exactly as-is from the approved sandbox
- Add `'use client'` only if the component has interactivity
- Keep loading/empty/error states — they're now driven by props not `useState`
- Strip all `// vN:` annotations and the version badge

`[ComponentName].types.ts`:
```ts
export interface [ComponentName]Props {
  // derived directly from the shape of MOCK in the sandbox
}
```

### File output — Page

```
app/
  [route]/
    page.tsx      ← Server Component shell, fetches data, passes to component
    loading.tsx   ← loading state (maps to the sandbox 'loading' state)
    error.tsx     ← error boundary (maps to the sandbox 'error' state)
```

`page.tsx` — Server Component:
```tsx
// No 'use client' — this is a Server Component
import { [ComponentName] } from '@/components/[ComponentName]'

export default async function [PageName]Page() {
  const data = await fetch(...)  // real data fetching here
  return <[ComponentName] {...data} />
}
```

The Client Component parts (interactivity, state) stay in `components/` with
`'use client'`. The page shell stays a Server Component.

### Token file

```
lib/
  design-tokens/
    [component_name].tokens.ts   ← DESIGN_TOKENS exported verbatim from sandbox
```

These tokens are reference documentation — they aren't imported at runtime (Tailwind
classes are strings). They serve as a single source of truth for future edits.

### Promotion rules

- **Zero visual changes.** The sandbox is the source of truth. If something looks wrong
  in production, fix the data flow — not the classNames.
- **One component per file.** If the sandbox had sub-sections that grew large, extract
  them into `components/[ComponentName]/partials/` but keep classNames identical.
- **No new dependencies.** Don't introduce libraries during promotion that weren't
  planned in the spec. If a chart library is needed, that was a Phase 1 decision.
- After writing all files, confirm: "Production component written at `[path]`. Sandbox
  files in `scratch/` are safe to delete whenever you're ready."

---

## Anti-Patterns to Avoid

| Anti-pattern | Why bad | What to do instead |
|---|---|---|
| Skipping the spec | Fundamental layout disputes mid-iteration waste loops | Always write and confirm the spec |
| Overwriting `vN` files | Loses ability to go back | Always increment version |
| Touching production in Phase 3 | Mixes design exploration with real code | Stay in `scratch/` until approved |
| Generic mock data | Makes it hard to evaluate real hierarchy | Use realistic, domain-appropriate content |
| Silent design changes | User can't track what changed | Always show diff summary |
| Implementing during design | Premature coupling of structure and logic | Design tokens only; logic in Phase 5 |
| Full rewrite on minor feedback | Wastes tokens, risks regressions, loses diff clarity | Use `cp + str_replace` surgical edits |
| Editing from memory | Stale context causes wrong replacements | Always `view` the file before `str_replace` |
| One giant `str_replace` for many changes | Hard to debug if one change breaks layout | One `str_replace` call per logical change |
| Inline styles in sandbox | Breaks the zero-translation rule — won't travel to production | Tailwind classes only, arbitrary values for custom sizes |
| Adding 'use server' or async components | Adds Next.js complexity that blocks visual iteration | All sandbox components are `'use client'` flat functions |
| Changing classNames during promotion | Promotion is mechanical — visual changes belong in Phase 3 | Copy classNames verbatim; fix data flow not styles |

---

## Quick Reference — Phase Checklist

**Phase 1 — Spec**
- [ ] Purpose defined
- [ ] Sections listed
- [ ] Grid/layout named
- [ ] Aesthetic direction committed
- [ ] Mock content model written
- [ ] User confirmed spec

**Phase 2 — Draft**
- [ ] File saved as `scratch/[name].v1.tsx`
- [ ] Starts with `'use client'` directive
- [ ] Tailwind only — zero inline styles, zero `<style>` blocks
- [ ] DESIGN_TOKENS exported with Tailwind class fragments
- [ ] Realistic mock data in `const MOCK = {...}` above component
- [ ] All spec states present, togglable via fixed button strip
- [ ] `const VERSION = 1` set, badge visible bottom-left
- [ ] Browser opened

**Phase 3 — Loop**
- [ ] Feedback parsed into buckets
- [ ] Acknowledged before coding
- [ ] Edit scope classified (surgical vs. full rewrite)
- [ ] File read with `view` before any `str_replace`
- [ ] New version created with `cp` (never `create_file` on iterations)
- [ ] Each change applied as a separate `str_replace`
- [ ] Change annotated with `// vN:` comment inline
- [ ] Diff summary shown before browser open
- [ ] Browser opened

**Phase 4 — Freeze**
- [ ] File renamed to `.APPROVED.tsx`
- [ ] DESIGN_TOKENS printed
- [ ] Design summary written

**Phase 5 — Promote**
- [ ] Server vs Client boundary decided per component
- [ ] `[ComponentName].types.ts` written from MOCK shape
- [ ] Production component written — classNames copied verbatim from sandbox
- [ ] `'use client'` present only if component has interactivity
- [ ] All states driven by props (not useState) or split into page.tsx / loading.tsx / error.tsx
- [ ] Token file written to `lib/design-tokens/`
- [ ] Barrel export (`index.ts`) created
- [ ] Zero visual changes from approved sandbox

---

## References

- `references/aesthetic-directions.md` — Palette of aesthetic archetypes for Phase 1
- `references/feedback-templates.md` — More structured feedback prompts for complex UIs

Read the `frontend-design` skill at `/mnt/skills/public/frontend-design/SKILL.md` during
Phase 2 before writing any code.
