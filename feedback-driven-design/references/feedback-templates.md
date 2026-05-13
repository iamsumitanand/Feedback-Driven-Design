# Feedback Templates

Use these during Phase 3 to elicit precise, actionable design feedback.
Show the appropriate template based on the complexity of the component being reviewed.

---

## Standard Template (default — use for most components)

```
──────────────────────────────────────────────────
🎨 DESIGN REVIEW  ·  v[N]
──────────────────────────────────────────────────
  ✅ What's working well?
  ❌ What feels off?
  🔧 Specific changes? (layout, color, type, spacing)
  📐 Anything missing from the spec?
  📱 Responsive feel right?

Or just say "approved" to lock this design. 🔒
──────────────────────────────────────────────────
```

---

## Deep Review Template (use after v1 on complex pages)

Send this when the component is large (full page, dashboard, multi-section layout)
and you want structured, section-by-section feedback to avoid vague responses.

```
──────────────────────────────────────────────────────────
🔍 DEEP DESIGN REVIEW  ·  v[N]
Please rate each section: ✅ Good / 🔧 Needs work / ❌ Redo

NAVIGATION / HEADER
  Rating: __
  Notes: __

HERO / PRIMARY CONTENT AREA
  Rating: __
  Notes: __

[SECTION 3 — customize from spec]
  Rating: __
  Notes: __

[SECTION 4 — customize from spec]
  Rating: __
  Notes: __

OVERALL FEEL
  Typography: __
  Color palette: __
  Spacing / breathing room: __
  Motion / animations: __

PRIORITY LIST (what to fix first):
  1. __
  2. __
  3. __

Or say "approved" to lock. 🔒
──────────────────────────────────────────────────────────
```

---

## Quick Pulse Template (use from v3 onwards when changes are minor)

When you're close and only doing small refinements, switch to this lighter template
to keep the loop fast.

```
──────────────────────
🎨 v[N] — Quick check

Any final tweaks, or are we good to ship?
[ ] One more thing: __
[ ] Approved 🔒
──────────────────────
```

---

## Direction Reset Template (use if user is going in circles)

If the user keeps changing their mind or you detect circular feedback (v4 looks like v2,
user prefers v3 but v3 has flaws), surface this explicitly:

```
────────────────────────────────────────────────────
🔄 We might be circling — let's reset direction.

Here's what I've noticed:
• v2 felt [description]
• v3 felt [description]
• v4 felt [description]

It seems like you're pulled between [direction A] and [direction B].

Which would you like to commit to?
  A — [direction A, e.g. "darker, more editorial, less color"]
  B — [direction B, e.g. "warmer, more playful, more whitespace"]
  C — Something else entirely: __
────────────────────────────────────────────────────
```

---

## State Review Template (use when multiple states were implemented)

After implementing loading/empty/error/success states, prompt specific feedback per state.

```
────────────────────────────────────────
🧪 STATE REVIEW  ·  v[N]

Toggle through the states using the buttons in the corner.

LOADING STATE
  Does it feel right? (duration, style): __

EMPTY STATE
  Is the messaging clear and encouraging?: __

ERROR STATE
  Does it feel appropriately serious but not alarming?: __

POPULATED / SUCCESS STATE
  This is the primary state — overall verdict: __

────────────────────────────────────────
```

---

## Responsive Review Template (use when mobile breakpoints are critical)

```
────────────────────────────────────────────────────
📱 RESPONSIVE REVIEW  ·  v[N]

Resize your browser window or use DevTools mobile view.

DESKTOP (1280px+)
  Layout: __
  Typography scale: __

TABLET (768px–1024px)
  Does the grid collapse cleanly?: __
  Any overflow/wrapping issues?: __

MOBILE (375px–480px)
  Is everything tap-target friendly?: __
  Any critical info hidden or cramped?: __

────────────────────────────────────────────────────
```

---

## Tips for Reading Feedback

### Translate vague feedback into specific changes

| User says | Likely means |
|---|---|
| "Feels heavy" | Reduce font weight, increase whitespace, lighter background |
| "Feels cheap" | Improve spacing, use fewer colors, better typography |
| "Feels cold" | Warmer color temperature, rounder shapes, softer shadows |
| "Feels busy" | Reduce elements, increase contrast ratio between hierarchy levels |
| "Doesn't feel premium" | Slow down animations, reduce colors to 2-3, tighten type |
| "Doesn't feel like us" | Ask: "What's an example of a product that does feel like you?" |
| "The spacing is off" | Usually means insufficient whitespace — add more padding/margin |
| "Something's missing" | Usually a strong visual anchor — hero, accent, or CTA is too weak |

### Red flags during feedback

- User approves after v1 of a complex page → push back gently, suggest one more iteration
- User is very specific about one tiny thing → check if the bigger hierarchy is actually right
- User's feedback contradicts the spec → update the spec explicitly, don't silently deviate
