Radius, optical alignment, shadows, image outlines, and icons: the small details that compound into polish. Review with the interface slowed down: what feels off at 10% speed is what is subtly wrong at full speed.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## Concentric border radius

When nesting rounded elements, the outer radius must equal the inner radius plus the padding between them:

```
outerRadius = innerRadius + padding
```

The rule matters most when nested surfaces sit close together. Past `24px` of padding, treat the layers as separate surfaces and choose each radius independently rather than forcing concentric math.

### Example

```css
/* Good: concentric radii */
.card {
  border-radius: 20px; /* 12 + 8 */
  padding: 8px;
}
.card-inner {
  border-radius: 12px;
}

/* Bad: same radius on both */
.card {
  border-radius: 12px;
  padding: 8px;
}
.card-inner {
  border-radius: 12px;
}
```

### Tailwind example

```tsx
// Good: outer radius accounts for padding
<div className="rounded-2xl p-2">       {/* 16px radius, 8px padding */}
  <div className="rounded-lg">          {/* 8px radius = 16 - 8 ✓ */}
    ...
  </div>
</div>

// Bad: same radius on both
<div className="rounded-xl p-2">
  <div className="rounded-xl">          {/* same radius, looks off */}
    ...
  </div>
</div>
```

Mismatched radii on closely nested surfaces are a common source of visual tension. Calculate concentrically where the layers share a visible, even inset. Keep an established component token where they are independent or the padding is deliberately asymmetric.

## Optical alignment

When geometric centering looks off, align optically instead.

### Buttons with text + icon

Where an icon makes symmetric padding look unbalanced, use slightly less on the icon side. A starting point:
`icon-side padding = text-side padding - 2px`.

```css
/* Good: less padding on icon side */
.button-with-icon {
  padding-inline-start: 16px;
  padding-inline-end: 14px; /* trailing icon side = text side - 2px */
}

/* Bad: equal padding looks like icon is pushed too far right */
.button-with-icon {
  padding-inline: 16px;
}
```

```tsx
// Tailwind
<button className="ps-4 pe-3.5 flex items-center gap-2">
  <span>Continue</span>
  <ArrowRightIcon />
</button>
```

### Play button triangles

Play icons are triangular and their geometric center is not their visual center. Shift slightly right:

```css
/* Good: optically centered */
.play-button svg {
  transform: translateX(2px); /* physical correction to the glyph itself */
}

/* Bad: geometrically centered but looks off */
.play-button svg {
  /* no adjustment */
}
```

### Asymmetric icons (stars, arrows, carets)

Some icons carry uneven visual weight. The best fix is adjusting the SVG directly, so the component needs no extra margin or padding.

```tsx
// Best: fix in the SVG itself
// Adjust the viewBox or path to visually center the icon

// Fallback: adjust with margin
<span className="translate-x-px">
  <StarIcon />
</span>
```

## Shadows instead of borders

Where **buttons, cards and containers** use a border for depth or elevation, replace it with a subtle `box-shadow`. The neutral ring below is a border substitute, not the colored zero-offset halo [craft-floor.md](craft-floor.md) refuses. Shadows use transparency, so they adapt to any background where solid borders do not. That matters most over images or multiple background colors, which a fixed border color was never designed for.

**Never apply this to dividers**, meaning `border-b`, `border-t` and side borders, or any border whose purpose is layout separation rather than depth. Those stay borders.

### Shadow as border (light mode)

Three layers. The first acts as a 1px border ring, the second adds subtle lift, the third ambient depth:

```css
:root {
  --shadow-border:
    0px 0px 0px 1px oklch(0 0 0 / 0.06),
    0px 1px 2px -1px oklch(0 0 0 / 0.06),
    0px 2px 4px 0px oklch(0 0 0 / 0.04);
  --shadow-border-hover:
    0px 0px 0px 1px oklch(0 0 0 / 0.08),
    0px 1px 2px -1px oklch(0 0 0 / 0.08),
    0px 2px 4px 0px oklch(0 0 0 / 0.06);
}
```

### Shadow as border (dark mode)

In dark mode, simplify to one white ring, since layered depth shadows are invisible on dark backgrounds:

```css
/* Dark mode: adapt to whatever setup the project uses
   (prefers-color-scheme, class, data attribute, etc.) */
--shadow-border: 0 0 0 1px oklch(1 0 0 / 0.08);
--shadow-border-hover: 0 0 0 1px oklch(1 0 0 / 0.13);
```

### Usage with hover transition

Apply the variable and add `transition-[box-shadow]` for a smooth hover:

```css
.card {
  box-shadow: var(--shadow-border);
  transition-property: box-shadow;
  transition-duration: 150ms;
  transition-timing-function: ease-out;
}

.card:hover {
  box-shadow: var(--shadow-border-hover);
}
```

### When to use shadows vs. borders

| Use shadows | Use borders |
| --- | --- |
| Cards, containers with depth | Dividers between list items |
| Buttons with bordered styles | Table cell boundaries |
| Elevated elements (dropdowns, modals) | Form input outlines (for accessibility) |
| Elements on varied backgrounds | Hairline separators in dense UI |
| Hover/focus states for lift effect | |

## Image outlines

Add a `1px` outline at low opacity to images for consistent depth, especially where other elements use borders or shadows.

### Color rules (non-negotiable)

- **Light mode**: pure black, `oklch(0 0 0 / 0.1)`.
- **Dark mode**: pure white, `oklch(1 0 0 / 0.1)`.
- Never a near-black or near-white from the project palette, such as slate-900, zinc-900, `#0a0a0a`, `#111827`, or `#f5f5f7`. Tinted outlines pick up the surrounding surface color and read as dirt on the image edge.
- Never match the outline to the project's accent or ink color. The outline is a neutral separator, not a themed element.

### Light mode

```css
img {
  outline: 1px solid oklch(0 0 0 / 0.1);
  outline-offset: -1px; /* draw the ring just inside the image edge */
}
```

### Dark mode

```css
img {
  outline: 1px solid oklch(1 0 0 / 0.1);
  outline-offset: -1px;
}
```

### Tailwind with dark mode

```tsx
<img
  className="outline outline-1 -outline-offset-1 outline-black/10 dark:outline-white/10"
  src={src}
  alt={alt}
/>
```

Use `outline-black/10` and `outline-white/10` specifically, not `outline-slate-*`, `outline-zinc-*`, `outline-neutral-*`, or any tinted scale.

**Why outline instead of border?** `outline` never affects layout, adding no width or height at any offset, and `outline-offset: -1px` draws the ring just inside the image edge so it hugs the corner radius.

## Match icon stroke to text weight

A hairline icon beside semibold text reads as broken; a heavy icon beside regular text shouts.

| Adjacent text | Icon stroke width (24px grid) |
| --- | --- |
| Regular (400), 14–16px | `1.5px` |
| Medium/Semibold (500–600) | `2px` |
| Bold (700), or emphasized standalone | `2.5px` |

```html
<!-- Good: stroke tuned to the label weight -->
<button class="flex items-center gap-2 font-semibold">
  <PlusIcon stroke-width="2" class="size-4" />
  New project
</button>

<!-- Bad: default 1.5px stroke against a bold label -->
<button class="flex items-center gap-2 font-bold">
  <PlusIcon stroke-width="1.5" class="size-4" />
  New project
</button>
```

Two related consistency rules:

- **One optical strategy per surface.** Never mix icon libraries with incompatible stroke conventions on one toolbar. Where the library supports stroke variants, match them to adjacent text as above; otherwise keep the set's native stroke and use size or color for emphasis.
- **Size icons relative to the text's cap height**, typically `1em`–`1.25em` when inline with text, so the pair scales together.

## One SVG, recolored per state

Never ship separate assets for default, hover, selected and disabled states. Use one SVG drawn with `currentColor` and let CSS state drive the color:

```html
<!-- Good: one asset, states are CSS -->
<svg fill="none" stroke="currentColor" stroke-width="2">…</svg>
```

```css
.icon-button { color: oklch(0.552 0.016 285.938); }
.icon-button:hover { color: oklch(0.21 0.006 285.885); }
.icon-button[aria-pressed="true"] { color: oklch(0.623 0.188 259.815); }
.icon-button:disabled { opacity: 0.4; }
```

```html
<!-- Tailwind -->
<button class="text-zinc-500 hover:text-zinc-900 aria-pressed:text-blue-600 disabled:opacity-40">
  <BookmarkIcon />
</button>
```

Hardcoded fills inside the SVG, such as `fill="#666"`, break this. Strip them to `currentColor` when importing icons.

## Outline default, fill active

Where an icon set offers outline and filled variants, use them as a state pair, never interchangeably:

| Variant | Use for |
| --- | --- |
| Outline | Default state: toolbars, list rows, inline with text |
| Fill | Selected/active state: the active tab, a toggled bookmark, a liked heart |

```tsx
// Good: variant communicates state
<TabIcon variant={isActive ? "solid" : "outline"} />

// Bad: filled icons everywhere, so the active tab has no state signal
<TabIcon variant="solid" />
```

The swap between variants is a contextual icon animation. Use the exact cross-fade values in [details-motion.md](details-motion.md).

## Design at render size

An icon that looks great at 48px collapses into mush at 16px. Thin interior lines, tight counters and fine texture all blur or alias when small.

- Test every icon at the smallest size it will render, often `16px`. It must stay recognizable there.
- Prefer simplified glyphs for small contexts over scaling down detailed artwork.
- Keep icons on the pixel grid at their render size. A 16px icon drawn on a 24px grid with fractional scaling renders soft, so use the set's native grid sizes (`16`, `20`, `24`) rather than arbitrary scales.
- Always SVG, never raster, so the same asset stays crisp at every density.

## Icons in RTL

Under `dir="rtl"`, flip icons whose meaning is tied to reading direction, and leave the rest alone:

| Flip | Don't flip |
| --- | --- |
| Back/forward arrows, chevrons in navigation | Logos and brand marks |
| Text-block glyphs (alignment, lists, indent) | Checkmarks |
| Speaker/volume waves (emanate in reading direction) | Physical objects: clocks, cups, pencils |
| "Send" style directional glyphs | Media playback (play/rewind refer to tape direction, convention keeps them LTR) |

```css
/* Good: mirror only direction-dependent icons */
[dir="rtl"] .icon-directional {
  scale: -1 1;
}
```

```html
<!-- Tailwind -->
<ChevronRightIcon class="icon-directional rtl:-scale-x-100" />
```

Analyze composite icons part by part. A badge or slash overlay may keep its position even when the base glyph flips. Accessible names for icon-only buttons belong to [details-a11y.md](details-a11y.md).
