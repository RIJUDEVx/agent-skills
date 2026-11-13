Grouping, shared edges, reading order, clearances, and layouts that survive resizing and translation. The numbers are starting points for interfaces with no density system; compact professional tools and project tokens stand where they still pass the stress tests. Resize it, translate it, mirror it for RTL.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## Group with space, not lines

Three tools create grouping, in order of preference:

1. **Negative space**, the default. Related items sit close, unrelated ones far apart.
2. **Background shapes**, a card or filled container, where a group must read as one unit such as a selectable row or a draggable card.
3. **Separator lines**, a last resort for dense data where space costs too much, such as tables and long settings lists.

The structural rule is that the gap between groups is at least 2× the gap within one. At `8px` inside a group, groups need `16px`+ between them, or the eye can't tell where one ends.

```css
/* Good: spacing alone communicates the grouping */
.field-group { display: flex; flex-direction: column; gap: 8px; }
.form { display: flex; flex-direction: column; gap: 24px; }

/* Bad: uniform spacing plus lines to compensate */
.form > * { margin-bottom: 12px; border-bottom: 1px solid var(--separator); }
```

```html
<!-- Good: Tailwind -->
<div class="space-y-6">
  <div class="space-y-2">…field group…</div>
  <div class="space-y-2">…field group…</div>
</div>
```

Where a separator is genuinely needed, keep it quiet: hairline width, low contrast, never combined with a large gap that already did the job.

## Keep controls distinct from content

Interactive elements need a visual signal: a background, a border, an underline, or placement in a consistent control zone such as a toolbar or footer row. A control styled identically to static text is invisible.

```html
<!-- Bad: action looks exactly like the description text next to it -->
<p class="text-zinc-600">Your trial ends soon. Upgrade now</p>

<!-- Good: the action reads as an action -->
<p class="text-zinc-600">Your trial ends soon.</p>
<button class="font-medium text-blue-600">Upgrade now</button>
```

The inverse holds too. A non-clickable badge shaped exactly like the buttons beside it collects dead clicks.

## Align to shared edges

Pick a small set of alignment edges and put everything on them, because the eye tracks straight edges to scan content.

- Every stray edge reads as noise even when nobody can name it: an icon 2px off the text edge, a card padded unlike its neighbor.
- Use one project spacing step to express hierarchy. `16px` is a useful default where no scale exists, and deeper nesting repeats the same step.
- Numbers in tables align to the trailing edge, text to the leading edge. Tabular figures are [details-type.md](details-type.md)'s.

```css
/* Good: one shared leading edge, one indent step */
.section { padding-inline: 24px; }
.section .child { margin-inline-start: 16px; }

/* Bad: three unrelated leading edges in one column */
.header { padding-inline-start: 20px; }
.list-item { padding-inline-start: 14px; }
.footer { padding-inline-start: 24px; }
```

## Logical properties, not physical

Express direction-dependent horizontal position as leading/trailing so the layout mirrors automatically under `dir="rtl"`:

| Physical (avoid) | Logical (use) |
| --- | --- |
| `margin-left` | `margin-inline-start` |
| `padding-right` | `padding-inline-end` |
| `left: 0` | `inset-inline-start: 0` |
| `text-align: left` | `text-align: start` |
| `border-right` | `border-inline-end` |

```html
<!-- Good: Tailwind logical utilities -->
<div class="ms-4 pe-6 text-start">…</div>

<!-- Bad: breaks in RTL -->
<div class="ml-4 pr-6 text-left">…</div>
```

Reserve physical properties for things that refer to physical screen sides whatever the language, such as positioning against a device notch or matching a gesture direction.

Where arrangement encodes progression, as in star ratings, step indicators and progress bars, the sequence mirrors in RTL and stars fill from the trailing side. Flexbox and grid with logical properties mirror automatically; hand-positioned elements do not. Digit order inside numbers never reverses, which with other bidi rules belongs to [details-type.md](details-type.md).

## Order by importance

Readers scan top-to-bottom and leading-to-trailing. Place content accordingly:

- The most important information sits near the top and the leading edge. The further down and trailing something sits, the less attention it gets.
- Give essential information room. Never bury the one number the user came for under rows of secondary detail. Push that into collapsed sections, tabs, or detail views.
- Within a row, identifying content leads and metadata and actions trail.

```html
<!-- Good: primary fact first, detail demoted -->
<div>
  <p class="text-2xl font-semibold">$4,320.00</p>
  <p class="text-sm text-zinc-500">Available balance</p>
</div>

<!-- Bad: the key fact is buried below the fold of the card -->
<div>
  <p class="text-sm">Account 4402 · Opened 2019 · Standard tier</p>
  <p class="text-sm">Last statement: June 30</p>
  <p class="text-sm">Balance: $4,320.00</p>
</div>
```

With logical properties, the same hierarchy mirrors correctly in RTL locales.

## Don't overload the entry point

The first screenful is a table of contents, not the whole book. If everything is prominent, nothing is:

- One primary action per view. [details-color.md](details-color.md) owns how color enforces it.
- Group secondary actions behind a menu once they exceed two or three.
- Prefer a short view that links deeper over a long view that shows everything at level one.

## Breathing room between targets

Controls placed too close get mis-tapped and read as one unit. Where the project has no density scale, start here:

| Between | Starting point |
| --- | --- |
| Adjacent bordered/filled controls (buttons, inputs) | `12px` |
| Around borderless controls (text buttons, icon buttons) | `24px` |
| Unrelated control groups | `24px`+ (2× the intra-group gap) |

Borderless controls need more clearance, because nothing marks where one target ends and the next begins. The space is the boundary. Compact professional tools may use less where hit areas stay distinct and never overlap. Preserve an established, usable density rather than expanding controls to match these values.

```html
<!-- Good: bordered buttons at 12px, icon buttons given room -->
<div class="flex gap-3">
  <button class="rounded-lg border px-4 py-2">Cancel</button>
  <button class="rounded-lg bg-blue-600 px-4 py-2 text-white">Save</button>
</div>

<!-- Bad: three borderless icon buttons packed at 4px -->
<div class="flex gap-1">
  <button><TrashIcon /></button>
  <button><ArchiveIcon /></button>
  <button><ShareIcon /></button>
</div>
```

WCAG target-size requirements, larger usability targets and pseudo-element expansion belong to [details-a11y.md](details-a11y.md); these clearances are in addition, so expanded hit areas never overlap.

## Inset buttons from the edges

In content layouts, buttons pressed against the viewport look like system chrome and clip against curved corners or gesture zones. Keep them inside the layout margins. Edge-to-edge actions stay valid where they are deliberately platform chrome and account for safe areas:

```css
/* Good: inset action bar */
.action-bar {
  padding-inline: 16px;
  padding-bottom: calc(16px + env(safe-area-inset-bottom));
}
.action-bar button { width: 100%; border-radius: 12px; }

/* Bad: button glued to all three edges */
.action-bar button {
  width: 100vw;
  border-radius: 0;
  position: fixed;
  bottom: 0;
}
```

Start near `16px` inline margin on mobile where the project has no layout token. The button can still span the full content width inside them.

## Progressive disclosure needs an affordance

Hiding complexity is good; hiding it without a cue is a trap. Every piece of off-screen or collapsed content needs a visible hint that it exists. Keep the product's established scroll indicator or disclosure pattern, and use the recipes below only where no cue exists:

- **Peeking items.** In a horizontal scroller or carousel, size items so the next one peeks `16–32px` past the container edge. A row of cards that ends exactly at the edge looks complete, and nobody scrolls it.
- **Disclosure controls.** Collapsed sections get a chevron or "Show more", labelled with what is hidden: "Show 12 more results", not "More".
- **Truncation cues.** Clamped text shows an ellipsis and a way to expand. Truncation mechanics are [details-type.md](details-type.md)'s.

In the peeking-scroller recipe, the container's padding creates the peek and snap points stay on the content edge.

```css
.scroller {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-inline: 24px;
  scroll-padding-inline: 24px;
  scroll-snap-type: x mandatory;
}
.scroller > * {
  flex: 0 0 calc(100% - 48px - 24px); /* container minus margins minus peek */
  scroll-snap-align: start;
}
```

```html
<!-- Tailwind: the 80% width keeps the next card's leading 16-32px visible -->
<div class="flex gap-3 overflow-x-auto px-6 [scroll-padding-inline:1.5rem] snap-x snap-mandatory">
  <div class="w-[80%] shrink-0 snap-start">…</div>
  <div class="w-[80%] shrink-0 snap-start">…</div>
</div>
```

## Content bleeds, controls float

The two layers behave differently at the edges:

- **Content layer**: backgrounds, hero media and scrollable lists extend to the viewport edges.
- **Control layer**: text and controls stay inside the layout margins and safe areas, floating above the content.

```css
/* Good: full-bleed media inside a constrained article */
.article {
  display: grid;
  grid-template-columns: 1fr min(65ch, calc(100% - 48px)) 1fr;
}
.article > * { grid-column: 2; }
.article > .full-bleed { grid-column: 1 / -1; }
```

Sticky headers and floating action buttons account for safe areas:

```css
.fab {
  position: fixed;
  inset-inline-end: calc(16px + env(safe-area-inset-right));
  bottom: calc(16px + env(safe-area-inset-bottom));
}
```

## Hold structure until it breaks

Breakpoints belong to the content, not the device catalog:

- Break where the layout actually stops fitting, not at `768px` because a preset says so. That is where the sidebar squeezes content below its minimum measure, or the card grid drops below a usable column width.
- Collapse late. A layout keeping its expanded structure as long as it genuinely fits stays stable and familiar. Premature collapsing throws away space users paid for.
- Prefer **container queries** for components. A card adapts to the column it is in, not to the viewport.

```css
/* Good: component adapts to its container */
.card-list { container-type: inline-size; }
@container (max-width: 400px) {
  .card { grid-template-columns: 1fr; }
}

/* Bad: viewport media query breaks the card inside a narrow sidebar */
@media (max-width: 768px) {
  .card { grid-template-columns: 1fr; }
}
```

Test the smallest and largest supported sizes first, since those break first, then the sizes between.

## Plan for growth and clipping

Layouts fail in two directions. Content grows, and viewports shrink.

**String expansion varies by language and by source-string length.** Never rely on one universal percentage.

- No fixed widths sized to English labels. Use `max-width` plus wrapping.
- No fixed heights on text containers. Use `min-height` where a floor is needed.
- Buttons size themselves from their label (`padding-inline`), never a hardcoded width.
- Test with pseudo-localization or a long-string locale before shipping.

```css
/* Good: label defines the size */
.button { padding-inline: 16px; white-space: nowrap; }

/* Bad: German will overflow or truncate */
.button { width: 96px; overflow: hidden; }
```

**Clipping.** Never park a critical action where it can be cut off: the bottom edge of a resizable pane, below the fold of a fixed-height modal, behind an expanding keyboard. Keep primary actions in stable chrome, a sticky footer with safe-area padding or the top of the view. Where a modal's content scrolls, its action row does not.

## Before you finish

| Mistake | Fix |
| --- | --- |
| `margin-left` / `padding-right` in a localizable layout | `margin-inline-start` / `padding-inline-end` |
| Content-layout button touches the viewport edge | Inset within the project margins; keep intentional platform chrome |
| Breakpoints at 768/1024 because they're the defaults | Break where the content actually stops fitting |
| Fixed-width text container sized to one language | `max-width` and wrapping; test pseudo-localization |
| Primary action at the clip-prone bottom of a pane | Sticky positioning or stable chrome with safe-area padding |

## Checks

Without a browser: logical properties in place of physical ones, container and media queries against the supported viewport list and DOM order against the intended reading order. With one: every supported width, 200% zoom and the RTL mirror. Report every check you could not run as `Not verified`.
