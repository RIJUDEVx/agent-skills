States that do not move: an emphasis that resizes its box, a hover that pushes the row, a focus ring that shifts the layout. The rule everywhere is the same — a state change paints, it never changes geometry.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## Emphasis must not resize anything

Making nav links bold on hover pushes everything next to them. Heavier weights are also wider. A weight change is a metric change: in a flex or inline row the wider glyphs take space from their siblings, so the row reflows, wraps, or nudges. Nothing about it looks like a bug; it looks like the page was never designed.

Three ways out, in order of preference.

**1. Do not change the weight.** Colour, opacity, an underline, or a small `scale` are enough to say "this one is hovered". These cost no layout.

```css
/* Good: the box never changes size */
.nav a {
  color: var(--fg-muted);
  transition: color 140ms ease-out;
}
.nav a:hover {
  color: var(--fg);
}
```

**2. Use the grade axis, if the family has one.** A grade axis darkens the stroke without touching the glyph's advance width, so the text looks bolder and stays exactly as wide. See [details-type.md](details-type.md) for which families ship `GRAD` and why the raw tag is correct here.

```css
/* Needs a family that ships GRAD (Roboto Flex and friends) */
.nav a:hover {
  font-variation-settings: "GRAD" 150;
}
```

**3. Reserve the width.** When bold really is the design, hold the space so the swap cannot move anything: keep the widest metrics in the box, always.

```css
/* The invisible bold copy holds the space the bold text will need */
.nav a {
  position: relative;
}
.nav a::after {
  content: attr(data-label);
  font-weight: 700;
  visibility: hidden;
  display: block;
  height: 0;
  pointer-events: none;
}
```

Do not reach for `min-width` in pixels to fake it; that breaks at the next font size and the next language.

## Weight is width

Any time the weight changes, the metrics change with it. The same rule applies to `font-stretch`, `font-style`, letter-spacing, and a different family.

- `font-weight: 700` is not "the same text, darker". It is a different set of glyphs.
- Synthetic bold (the browser faking a weight the family lacks) is wider still, and it doubles the paint.
- Interpolating between weights at runtime changes width per frame. Never animate `font-weight` on a live element.

Load the real weight whenever a state uses one, and verify the whole fallback stack; see [details-type.md](details-type.md) for `font-synthesis` and font loading.

## Animating a variable axis

`font-variation-settings` does not interpolate on its own. Register the number as a property and animate that.

```css
@property --grade {
  syntax: "<number>";
  inherits: false;
  initial-value: 0;
}

.nav a {
  font-variation-settings: "GRAD" var(--grade);
  transition: --grade 160ms ease-out;
}
.nav a:hover {
  --grade: 150;
}
```

Keep the transition short and on the exit path too, or the row feels sticky.

## States that do not move

- `:hover` and `:focus` change paint, not geometry. `transform`, `opacity`, `color`, `background`, `border-color`, `box-shadow`, `text-decoration-color` are safe. `font-weight`, `font-size`, `padding`, `border-width`, `width`, `height` are not.
- `:focus-visible` gets a ring that does not shift the layout. An `outline` (or `box-shadow`) does not affect layout; a `border` does. See [details-a11y.md](details-a11y.md) for the ring itself.
- Underline on hover: animate `text-decoration-color`, not the decoration itself. Colour is the only part of a real underline that animates reliably.
- Under `prefers-reduced-motion: reduce`, cut transforms and long transitions. Keep colour changes: they are not motion. See [details-motion.md](details-motion.md) and [details-a11y.md](details-a11y.md).

## Verify by measuring

- Measure the element in the layout at rest and on hover. Force the state in DevTools (`:hover` in the styles pane, or `.is-hovered`). Read the width again. Any difference is a failure, whatever the intent.
- Look at the row with the longest real content, in the longest supported language.
- Check the state at the sizes it ships at: a hover bug at 1440px can be invisible at 390px and the reverse.
- Load the real font. A fallback swaps the metrics and hides every bug above.

## Before you finish

| Mistake | Fix |
| --- | --- |
| Nav link grows on hover | Change colour or opacity, or raise `GRAD`, not `font-weight` |
| Bold text pushes its siblings | Reserve the width with an invisible bold copy; never a pixel `min-width` |
| Focus ring shifts the layout | `outline` or `box-shadow`, never `border` |
| Number column jitters as it updates | `font-variant-numeric: tabular-nums` |
| Axis animation does not interpolate | Register the number with `@property` and animate that |
| State animation drops frames | Paint-only properties; never `font-weight`, `font-size` or geometry on a live element |

## Checks

With a browser: measure the width at rest and in each state (hover, focus, active, selected) with the real font loaded — any difference fails. Repeat at each shipping size and with the longest supported content. Without one: confirm the CSS changes only paint properties, that any weight swap reserves its width, and that varying values use tabular figures. Report every check you could not run as `Not verified`.
