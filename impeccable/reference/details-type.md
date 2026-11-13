How text renders, wraps, and behaves in mixed-direction content. Bad wrapping, widows and truncation only show up at real content lengths, so read the rendered page rather than scanning the code. [typeset.md](typeset.md) owns measure, hierarchy, and role decisions.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## Choosing within a family

Applying or reviewing typography never requires a new typeface. Use the product's type system unless the task asks for a type change; [new-work.md](new-work.md) owns choosing a new one.

- Weight and size define hierarchy, and overusing them hurts readability fast.
- Pair for contrast, not similarity. A serif headline over a sans body reads as a deliberate split; two near-identical sans-serifs read as a mistake.
- Thin weights are display-only. Below `18px` stay at weight `400`+, because Ultralight, Thin and Light (`100`–`300`) strokes disappear at text sizes and on low-DPI screens. Reserve them for `28px`+ display text, and check even there that they hold against the background.
- "Display" in a font's name does not make it a display font. SF Pro and Heldane ship a `Display` variant for large sizes and a `Text` variant for small ones. Use the variant matching the size you are setting.

## Formats

| Format | Notes |
| --- | --- |
| `.woff2` | Brotli compression, broadly supported. Use this on the web. |
| `.woff` | Older compression. Fallback only for very old browsers. |
| `.ttf` / `.otf` | Raw formats, no web compression, larger files. Desktop only unless there is no other option. |

## Type scale

A small set of sizes used across a product, deviated from as little as possible. Hard-coding sizes with no system behind them breaks down at scale.

```css
:root {
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.5rem;
  --text-2xl: 2rem;
}
```

Pick an existing scale or define one. Tailwind's (`text-xs` through `text-9xl`, each class pairing a size with a matching line height) is a solid ready-made choice.

Solo, the default names work fine given clear rules for where each size is used. On a team, name sizes semantically. `text-sm` tells you the size but not the use, where `text-body-sm` carries both.

A role-based scale pairs each size with its line-height and weight, making a role one decision instead of three. A starting point for a product interface:

| Role | Size | Line-height | Weight |
| --- | --- | --- | --- |
| Display | `2.25rem` (36px) | `1.1` | `600` |
| Title | `1.5rem` (24px) | `1.2` | `600` |
| Heading | `1.125rem` (18px) | `1.3` | `600` |
| Body | `1rem` (16px) | `1.5` | `400` |
| Caption | `0.8125rem` (13px) | `1.4` | `400` |

Emphasis within a role is one weight step up (`400` → `500`), not a size change.

## Heading hierarchy

Assign each heading level to a descending step of the scale, so hierarchy comes from the scale instead of one-off sizes:

```css
h1 { font-size: var(--text-2xl); }
h2 { font-size: var(--text-xl); }
h3 { font-size: var(--text-lg); }
```

In Tailwind the same mapping is utility classes per level (`text-2xl`, `text-xl`, `text-lg`), centralized in a component or `@layer base` rather than repeated inline.

When reviewing, compare the computed size of headings within each semantic section. A child rendering more prominently than its parent breaks the hierarchy. Deep levels may share a size where the scale runs out of comfortable steps, as long as weight or letter-spacing keeps them distinct. A heading is never smaller than body text unless it is deliberately a label-style overline.

Heading semantics and outline quality belong to [details-a11y.md](details-a11y.md). Pick the element from the document structure, then use this file to make that structure visually legible. Never pick a heading element for its browser-default size.

## Kerning and letter-spacing

- **Kerning** adjusts specific pairs such as `AV` or `Ye`. It is built into the font and applied automatically. Switch it off only deliberately, with `font-kerning: none`.
- **`letter-spacing`** adds the same space between every character.

```css
/* Good */
.display-heading {
  letter-spacing: -0.02em;
}

.uppercase-label {
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
```

## Line-height

| Text | Value |
| --- | --- |
| Headings | ~`1.1` |
| Body copy | `1.5`–`1.6` |

Tailwind's `leading-snug`, `leading-normal` and `leading-relaxed` are sensible defaults that rarely need overriding. A tightly-leaded paragraph is harder to read than a taller row is to fit.

```css
/* Bad: card description at heading leading */
.card-description { line-height: 1.1; }

/* Good: it wraps to 3 lines, so it reads as body text */
.card-description { line-height: 1.4; }
```

## Text trimming with text-box

Fonts reserve space above and below the letters, which is why text sits slightly too low in buttons and badges. `text-box` trims it. Two parts: which edges to trim (`trim-both`, `trim-start`, `trim-end`) and where:

| Keyword | Trims at |
| --- | --- |
| `cap` | The cap height (top) |
| `alphabetic` | The baseline (bottom) |
| `text` | The font's own text edge, keeping room for descenders |

```css
/* trim top and bottom */
.badge {
  text-box: trim-both cap alphabetic;
}

/* trim only the top */
.heading {
  text-box: trim-start cap;
}

/* trim only the bottom */
.label {
  text-box: trim-end alphabetic;
}
```

Supported in Chromium (133+) and Safari (18.2+), not yet Firefox. Treat it as progressive enhancement, where unsupported browsers keep the default leading.

## Alignment

`text-align` controls where each line starts and ends. `justify` stretches spaces until both edges line up, which works in specific editorial layouts and nowhere else in an interface.

## Wrapping

| Property | Use |
| --- | --- |
| `text-wrap: balance` | Distributes text evenly across multiple lines |
| `text-wrap: pretty` | Avoids leaving a single short word on the final line |
| `overflow-wrap: break-word` | Lets long words, links and IDs break before escaping the container |
| `white-space: nowrap` | Keeps labels and badges on one line where a break looks broken |

[typeset.md](typeset.md) sets where `balance` and `pretty` apply. Browsers ignore `balance` past a few lines, so it does nothing on a paragraph.

## Truncation

- Single line: `text-overflow: ellipsis`, which needs `overflow: hidden` and `white-space: nowrap`.
- Multiple lines: `line-clamp` allows any number of lines before the ellipsis.

Truncation hides content. Where the missing text matters, make the full value available in a tooltip or an expanded view.

## Case

`text-transform` changes how case appears without changing the underlying text. Write copy naturally and control presentation with CSS, so a redesign never means rewriting copy.

## Smart punctuation

Keyboard characters are not always the best characters:

| Instead of | Use |
| --- | --- |
| Straight quotes `"..."` | Curly quotes that curve around the text (keep straight quotes in code) |
| Hyphen in ranges | En dash: `2010–2020` |
| Two hyphens for an aside | Em dash character |
| Three periods `...` | The single ellipsis character `…` |
| Regular space in `16 px` | `&nbsp;` so the value never breaks apart |
| Uncontrolled word breaks | `&shy;` to mark where a word may break |

## Internationalization

Two refinements for mixed-direction text:

- **Long paragraphs align by their own language.** A one- or two-line snippet follows the surrounding UI's direction. A paragraph of three or more lines aligns to its own script instead, so an English paragraph stays start-aligned LTR even inside an RTL interface. `text-align: start` with the correct `lang`/`dir` on the paragraph element handles this.
- **Never reverse digits.** Numbers keep their order in every direction, so a phone number or "541" reads identically in RTL. Browsers handle this through the Unicode bidi algorithm. Never fight it with manual reordering, and wrap mixed number and text values in `<bdi>` where adjacent RTL text disturbs them.

## Tabular numbers on changing values

Digits have different widths by default, so timers, counters and prices shift the layout as they update. Apply `font-variant-numeric: tabular-nums` to any value that changes.

## Language and bidi behavior

Set `lang` so browsers and assistive technology pick the right pronunciation, quotes and hyphenation. Set `dir` at the document or at the content boundary where direction changes. Preserve digit order, and use `<bdi>` to isolate a mixed-direction value. Spatial mirroring and logical CSS properties belong to [details-layout.md](details-layout.md).

## Keep useful text selectable

Keep text selectable by default. `::selection` can carry brand into the reading experience, as long as the selected combination stays legible.

`user-select: none` belongs on a draggable or gesture-driven surface where accidental selection interferes. Never across the interface and never because a button label can be highlighted.

## Underlines

Default underline position is browser-determined, sometimes too close, sometimes cutting through descenders, sometimes too thin. Pull position and thickness from the font's own metrics:

```css
a {
  text-underline-position: from-font;
  text-decoration-thickness: from-font;
}
```

A dotted underline on an abbreviation:

```css
abbr {
  text-decoration: underline dotted;
}
```

Or tune manually:

```css
a {
  text-decoration-thickness: 1px;
  text-underline-offset: 3px;
  text-decoration-skip-ink: auto;
  text-decoration-color: var(--color-gray-1000);
  transition: text-decoration-color 200ms ease-out;
}

a:hover {
  text-decoration-color: var(--color-gray-1200);
}
```

Color is the only part of a real underline that animates reliably. Unless the only thing animating is the color, build the underline as a separate element rather than using `text-decoration`.

## Selection

- `::target-text` styles the phrase a shared link scrolls to.
- The Custom Highlight API styles ranges you pick yourself, like search matches, without extra markup.

## Forms and editable text

- `::placeholder` styles the hint in an empty field.
- `caret-color` colors the blinking insertion bar. Color is about as far as caret styling goes; a fully custom caret is hard to build and rarely worth it.

### iOS input zoom

This is an accessibility feature: `16px` is the web default, and Safari treats smaller as too hard to read while typing.

The two fixes differ in what they do to the design, not in correctness.

**Size up on mobile.** The input renders at `16px` on small screens and drops to the design size from the `sm` breakpoint up. Nothing to compensate, but the mobile input no longer matches the desktop one.

```tsx
<input className="text-base sm:text-sm" type="email" />
```

**Scale the text down.** Keep `font-size` at `16px` so Safari never zooms, then render at the intended size with a transform. The design survives at every viewport, at the cost of two compensating calcs. Widen the element by the inverse of the scale so it still fills its container once shrunk, and divide `line-height` by the same factor so the intended leading survives. `origin-left` pins the text to the start edge, `origin-right` under RTL. Above the breakpoint, drop the transform and set the real size.

```tsx
// 13px rendered from a 16px font-size: 13 / 16 = 0.8125
<div className="flex h-10 items-center rounded-[10px] bg-gray-300 px-2.5">
  <input
    className="h-full w-[calc(100%/0.8125)] origin-left scale-[0.8125] bg-transparent text-base leading-[calc(1.125/0.8125)] outline-none sm:w-full sm:scale-100 sm:text-[13px]"
    type="email"
  />
</div>
```

The transform shrinks the whole box, not only the glyphs, so let a wrapper draw the field's surface and keep the input transparent. A background, border, or ring on the scaled element shrinks with the text and misses the intended hit area.

## Decorative text

| Property | Effect |
| --- | --- |
| `::first-letter` | Drop cap, widely supported |
| `::first-line` | Styles only the first line |
| `initial-letter` | Sizes the drop cap; limited support, no Firefox yet |
| `background-clip: text` | Clips a background to the letter shapes; gradient text stays on the [craft-floor.md](craft-floor.md) refuse list |
| `-webkit-text-stroke` | Outlines the letters; works across modern browsers despite the prefix |
| `text-shadow` | Like `box-shadow` but follows the character shapes |

A text stroke drawing lines inside the letters is the font. The stroke traces every contour, and variable fonts usually keep overlapping shapes unmerged. Static fonts do not have this issue.

## Sizes

Typography must survive the reader changing it: zoom, a larger browser font size, an overridden line height or letter spacing.

| Text | Size |
| --- | --- |
| Long-form body starting point | Around `16px`, verified in the actual typeface and measure |
| Inputs and menus starting point | Around `14px` |
| Captions | `13px` |
| Floor | Rarely below `12px` |

## Font smoothing

Tailwind's `antialiased` sets both properties:

```css
html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

```tsx
<html lang="en">
  <body class="font-sans antialiased">
    <main>{children}</main>
  </body>
</html>
```

## Static vs variable

- **Static font:** one weight and one style per file. Regular, medium and bold is three files.
- **Variable font:** a whole range in one file. Any value in it works, such as `font-weight: 589`.

A variable font is not automatically better. At one or two weights, static files can be smaller. At several weights, optical sizes, or custom axes, a variable font usually wins.

## Load intended weights and styles

Use a weight or style the active family does not provide and the browser may synthesize it, so load the faces the design uses. `none` disables weight, style, small-cap, superscript and subscript synthesis together and can erase distinctions when the real face is unavailable. Verify the whole fallback stack and every emphasis state before setting it.

```css
.brand-wordmark {
  /* Safe only after this isolated treatment is verified */
  font-synthesis: none;
}
```

For body and interface text, keep synthesis enabled unless a verified font setup supplies every requested form. If only one mode is unwanted, use the specific longhand (`font-synthesis-weight`, `font-synthesis-style` and related properties) instead of the blanket shorthand.

## Axes

Variable-font controls, each with a four-letter tag. A font supports only the axes its designer included.

| Axis | Tag | Controls |
| --- | --- | --- |
| Weight | `wght` | Stroke thickness (like `font-weight`) |
| Optical size | `opsz` | Details and spacing tuned for the display size |
| Width | `wdth` | Glyph width |
| Slant | `slnt` | Slant angle |
| Custom | e.g. `GRAD` (Roboto Flex) | Whatever the designer built |

Inter's variable file exposes only `wght` and `opsz`.

Optical sizes predate variable fonts, and many families still ship them as separate files. Heldane Text is sturdier and more spaced for reading sizes, Heldane Display finer for large ones.

**Weight is width.** A heavier weight is a wider set of glyphs, so swapping the weight on hover pushes the neighbours along a nav row. When the family ships a grade axis (`GRAD`), raise the grade instead: it darkens the stroke at the same weight and the same width, so nothing reflows. See the `design-engineering` skill for the pattern and for the fallback when the family has no grade axis.

## Properties over axis tags

When a property exists, use it. `font-weight` keeps working when a non-variable fallback renders, where `font-variation-settings` silently does nothing. Save raw tags for custom axes with no property of their own:

```css
/* Good: common axes use the properties */
.heading {
  font-weight: 650;
  font-optical-sizing: auto;
}

/* Good: custom axis with no property of its own */
.heading-grade {
  font-variation-settings: "GRAD" 80;
}

/* Bad: weight via raw tag breaks on fallback fonts */
.heading {
  font-variation-settings: "wght" 650;
}
```

## OpenType features

OpenType is the standard behind almost every modern font. Features are extra built-in options and, unlike axes, work the same on static and variable fonts. A font ships only the features its designer included.

| Tag | Feature |
| --- | --- |
| `tnum` | Tabular numbers: every digit the same width |
| `zero` | Slashed zero: `0` distinct from `O` |
| `liga` | Ligatures: joins pairs like "fi" into one shape |
| `ss01`–`ss20` | Stylistic sets (numbered slots) |
| `cv01`–`cv99` | Character variants (numbered slots) |

Same rule as axes. Prefer the `font-variant-*` properties and reserve `font-feature-settings` for tags with no property:

```css
/* Good: common features use the properties */
.price {
  font-variant-numeric: tabular-nums;
}

/* Good: slashed zero via the property too */
.id {
  font-variant-numeric: slashed-zero;
}

/* Good: niche feature with no property of its own */
.logo {
  font-feature-settings: "ss01" 1;
}
```

Tabular numbers matter for changing values. Without them each digit has a different width and the layout shifts as values update.

## Small caps, superscripts, subscripts

- **Small capitals:** uppercase letters drawn at a smaller size. Enable real ones with `font-variant-caps`.
- **Superscripts** sit above the normal line, the 2 in x², and **subscripts** below it, as in H₂O. Enable proper glyphs with `font-variant-position`.

Both require the font to include the glyphs.

## Stylistic sets and character variants

`ss01` = stylistic set, slot 01. `cv11` = character variant, slot 11. What each slot does differs font to font, which is why they are numbered, not named. Check the font's docs. In Inter, `ss01` switches to open digits and `cv11` swaps in a single-story `a`.

## Before you finish

| Mistake | Fix |
| --- | --- |
| Synthesized face differs from the design | Load the real face; disable only the verified synthesis mode |
| Child heading visually overpowers its parent | Map that section's hierarchy to descending scale steps |
| Heading element picked for its default size | Choose semantics first, then set the size in CSS |
| Orphan on the last line of a paragraph | `text-wrap: pretty` |
| Lopsided two-line heading | `text-wrap: balance` |
| Justified text in an interface | `text-align: start`; reserve justify for specific editorial layouts |
| Underline cuts through descenders | `text-decoration-skip-ink: auto`, `from-font` metrics |
| Mixed-direction value renders in the wrong order | Correct `lang`/`dir`; isolate the value with `<bdi>` |
| Selection disabled across application chrome | Restore it; suppress only where it conflicts with a drag or gesture |
| Extra-info hint with no visual cue | Dotted underline via `text-decoration-style: dotted` |
| Thin/Light weight on `14px` UI text | Weight `400`+ below `18px`; thin weights are display-only |
| `leading-none` on a three-line card description | At least `1.4` on any text that wraps to 3+ lines |

## Checks

Without a browser: computed size and weight for each heading level, checked descending; declared line-height and measure; truncation rules against realistic string lengths. With one: resize the viewport to catch wrapping, widows and truncation at real content lengths. Report every check you could not run as `Not verified`.
