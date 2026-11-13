# Typography: axes, metrics, fallbacks

The detail behind the rule in `SKILL.md`: a weight change is a metric change, and the
only way to get emphasis without moving the layout is an axis that does not touch the
advance width.

## The axes

A variable font exposes one or more axes. Each has a four-letter tag, and a font
supports only the axes its designer built in.

| Axis | Tag | Controls | Changes width? |
|---|---|---|---|
| Weight | `wght` | Stroke thickness | Yes |
| Optical size | `opsz` | Detail and spacing tuned to the size | Slightly |
| Width | `wdth` | Glyph width | Yes |
| Slant | `slnt` | Slant angle | Slightly |
| Grade | `GRAD` | Stroke darkness at the same weight and width | No |

Inter's variable file exposes only `wght` and `opsz`. Roboto Flex exposes `GRAD` among
others. Check the family's documentation before you use a tag: an unsupported tag is
ignored, silently, with no error.

`GRAD` is the one that matters for interaction states. It raises or lowers the stroke
contrast at a fixed weight, so the text reads as bolder while keeping the exact same
advance widths. Nothing reflows.

```css
/* Same box, same metrics, more contrast */
.nav a:hover {
  font-variation-settings: "GRAD" 150;
}
```

The tag is case-sensitive in practice: `"GRAD"`, `"wght"`, `"opsz"`.

## Properties first

Use the CSS property when one exists for the axis. Properties degrade gracefully; raw
tags do not.

```css
font-weight: 650;            /* maps to wght on a variable font, falls back cleanly */
font-optical-sizing: auto;   /* maps to opsz */
font-stretch: 95%;           /* maps to wdth */
font-style: oblique 12deg;   /* maps to slnt */
font-variation-settings: "GRAD" 150; /* no property exists for GRAD */
```

The catch: `font-variation-settings` overrides the property for the axes it names, and
it does not work at all on a font that is not variable. `font-weight: 650` still works
on a static font (rounded to the nearest available weight, or synthesised).

Keep the two apart: use `font-weight` for weight, and add `font-variation-settings`
only for the custom axis.

## Synthetic weight

When a family has no weight at the requested value, the browser fakes it by smearing
the outline. The result is:

- wider than the real weight, so it shifts the layout,
- painted twice, so it costs a fill,
- slightly soft at small sizes.

```css
/* Load every weight the design uses first, then this is safe */
.brand-wordmark {
  font-synthesis: none;
}
```

`font-synthesis: none` disables weight, style, small-cap, superscript and subscript
synthesis together. If only one mode is unwanted, use the longhand
(`font-synthesis-weight`, `font-synthesis-style`).

For body and interface text, keep synthesis enabled unless the font setup is verified
to supply every requested form.

## Optical size

`opsz` tunes a font for the size it is rendered at: tighter spacing, finer details,
smaller counters. A display cut set for 40px looks clumsy at 14px.

```css
body {
  font-optical-sizing: auto;
}
```

It changes the metrics slightly, so treat it like any other size-sensitive change when
the text sits in a row that must not move.

## Tabular figures

Digits are usually proportional: `1` is narrower than `0`. A number that changes in
place, or a column of numbers, will jitter unless the figures are tabular.

```css
.data {
  font-variant-numeric: tabular-nums;
}
```

The property form is the right one. `font-feature-settings: "tnum"` works on static
fonts too, but the property is what you should reach for.

## Measuring a state change

The check that catches the bug this skill is about:

1. In DevTools, select the element and read its width at rest.
2. Force the state (`:hover` in the styles pane, or the `.is-hovered` class).
3. Read the width again.

Any difference is a layout bug, whatever the design intended. Fix it by changing the
weight to a grade axis, by reserving the width, or by not changing the weight at all.
