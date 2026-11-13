---
name: design-engineering
description: "Frontend craft rules for details that break under real use: hover and focus states that move layout, typography that shifts, variable-font axes, motion that costs frames. Use when building or reviewing UI where a state change, a font, or an animation can push neighbouring elements around — nav links that grow on hover, buttons that resize, tables whose numbers jitter, hover effects that reflow. Also use when the user asks about font weight on hover, variable fonts, font-variation-settings, GRAD or grade axes, or layout shift from a style change."
---

# Design engineering

The rules that only show up after the design is built: a hover that moves the nav, a
weight that is wider than the one it replaced, a transition that drops frames, a
synthetic bold that renders twice.

Use this when the work touches states, typography, or motion. For direction, hierarchy,
colour and the wider craft floor, the `impeccable` skill has the deeper references.

## The rule that catches the most: emphasis must not resize anything

**Making nav links bold on hover pushes everything next to them. Heavier weights are
also wider.**

A weight change is a metric change. In a flex or inline row the wider glyphs take space
from the siblings, so the row reflows, wraps, or nudges. Nothing about it looks like a
bug: it looks like the page was never designed.

Three ways out, in order of preference:

**1. Do not change the weight.** Colour, opacity, an underline, or a small `scale` are
enough to say "this one is hovered". These cost no layout.

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

**2. Use the grade axis, if the family has one.** A grade axis darkens the stroke
without touching the glyph's advance width, so the text looks bolder and stays exactly
as wide.

```css
/* Needs a family that ships GRAD (Roboto Flex and friends) */
.nav a:hover {
  font-variation-settings: "GRAD" 150;
}
```

**3. Reserve the width.** When bold really is the design, hold the space so the swap
cannot move anything: keep the widest metrics in the box, always.

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

Do not reach for `min-width` in pixels to fake it; that breaks at the next font size and
the next language.

## Weight is width

Any time the weight changes, the metrics change with it. The same rule applies to
`font-stretch`, `font-style`, letter-spacing, and a different family.

- `font-weight: 700` is not "the same text, darker". It is a different set of glyphs.
- Synthetic bold (the browser faking a weight the family lacks) is wider still, and it
  doubles the paint.
- Interpolating between weights at runtime changes width per frame. Never animate
  `font-weight` on a live element.

## Properties over axis tags

`font-weight` keeps working when a non-variable fallback renders.
`font-variation-settings` silently does nothing on a fallback that has no such axis.
Use the property for the axes that have one, and the raw tag only for custom axes.

```css
/* Good */
.heading {
  font-weight: 650;
  font-optical-sizing: auto;
}

/* Good: custom axis, no property of its own */
.heading-grade {
  font-variation-settings: "GRAD" 80;
}

/* Bad: weight via raw tag breaks on fallback fonts */
.heading {
  font-variation-settings: "wght" 650;
}
```

Set `font-synthesis: none` only after verifying the real family is loaded for every
weight and style the design uses. Otherwise the browser fakes the missing ones, and the
fakes are wider than the real thing.

## Animating a variable axis

`font-variation-settings` does not interpolate on its own. Register the number as a
property and animate that.

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

## Numbers that stay put

Digits change width between fonts and between weights, so a table of numbers jitters as
it updates.

```css
.data {
  font-variant-numeric: tabular-nums;
}
```

Use it in tables, counters, timers, prices, and any list where one value updates in
place.

## States that do not move

- `:hover` and `:focus` change paint, not geometry. `transform`, `opacity`, `color`,
  `background`, `border-color`, `box-shadow`, `text-decoration-color` are safe.
  `font-weight`, `font-size`, `padding`, `border-width`, `width`, `height` are not.
- `:focus-visible` gets a ring that does not shift the layout. An `outline` does not
  affect layout; a `border` does.
- Underline on hover: animate `text-decoration-color`, not the decoration itself. Colour
  is the only part of a real underline that animates reliably.
- Under `prefers-reduced-motion: reduce`, cut transforms and long transitions. Keep
  colour changes: they are not motion.

## Verify, do not assume

- Measure the element in the layout at rest and on hover. Any width difference is a
  failure, whatever the intent.
- Look at the row with the longest real content, in the longest supported language.
- Check the state at the sizes it ships at: a hover bug at 1440px can be invisible at
  390px and the reverse.
- Load the real font. A fallback swaps the metrics and hides every bug above.

## Reference

- [references/typography.md](references/typography.md): the axes, what each one does to
  the metrics, and the fallback behaviour.
- `impeccable`, reference `details-type.md`: type scale, wrapping, truncation, features.
- `impeccable`, reference `details-motion.md`: transitions, press feedback, entrances.
