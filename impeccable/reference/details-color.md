A color system is a small set of ramps, named by role and verified against the backgrounds they actually render on. Most color bugs are system bugs: a value picked in isolation, a token borrowed because it looked right, a pair nobody measured. Report a contrast value only after measuring it, and compute a color rather than estimating it. [colorize.md](colorize.md) owns strategy and dosage.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## What a system needs

Most products need three kinds of ramp and nothing else:

| Ramp | How many | Notes |
| --- | --- | --- |
| Neutral | 1 | Carries 80–90% of the interface: backgrounds, borders, body text |
| Accent | 1 | The brand hue. Interactive and selected states |
| Status | 0–4 | `danger`, `warning`, `success`, `info`. Add one only when the product shows that state |

A second accent hue must also never sit adjacent to the first. Where it would, the accent ramp's own steps provide the range.

## Every step has a job

Each step maps to a role. Generate the steps the roles below call for and skip the rest.

| Role | Tailwind | Radix |
| --- | --- | --- |
| Page background | `50` | `1` |
| Subtle background | `50` | `2` |
| Component background | `100` | `3` |
| Component hover | `200` | `4` |
| Component active / selected | `200` | `5` |
| Subtle border | `200` | `6` |
| Border, separator | `300` | `7` |
| Strong border, focus ring | `400` | `8` |
| Solid fill | `500` | `9` |
| Solid fill hover | `600` | `10` |
| Low-contrast text | `700` | `11` |
| High-contrast text | `900` | `12` |

The two conventions differ in kind, not only in numbering:

- **Radix defines its 12 steps by role.** Step 9 is "the solid fill" in every ramp and appearance. The dark scale is a separate ramp reusing the same numbers, so `--accent-9` is the fill in both and component CSS never changes.
- **Tailwind defines its 11 steps by lightness.** `50` is light, `950` is dark. The mapping above therefore holds in light mode and inverts in dark, with the page background at `950` and high-contrast text at `50`. Components either swap step numbers per appearance or read a semantic token that swaps once.

Match whichever the project uses. For a new system prefer Radix's model, because a role-defined step survives a theme change that a lightness-defined step does not. On Tailwind, keep `50`–`950` and put the role mapping in the semantic tier.

Tailwind's 11 steps cover 12 roles, so some do double duty. Where the table repeats a step, the two roles are adjacent in practice and the collision is real. A design needing a subtle border and a component hover to be distinguishable needs a 12-step ramp.

## Neutrals

A pure gray ramp is a perfectly good default. It sits under any accent hue without competing and never needs revisiting when the brand color changes.

Tinting the neutral toward the accent hue is a stylistic option, not a correction. A trace of the accent, a few percent of its vividness, puts the greys in the same family as the accent rather than merely coexisting with it. Enough to measure, not enough to name. Neither choice reads as a mistake.

Warm neutrals, hue toward orange, read approachable and editorial; cool ones, toward blue, read technical and precise. Whichever you pick, including none, hold it across the whole ramp. A warm gray border on a cool gray background is visible even when neither color is nameable alone.

Neutrals carry the most roles, so they need the most steps. Never generate fewer neutral steps than accent steps.

## Status colors

Convention constrains status hues before taste does. Red reads as danger, amber as warning, green as success. See **Color across cultures** below.

Two rules govern them:

- **Keep every status hue distinct from the accent.** If the brand is red, the danger ramp cannot also be red. Move danger toward a deeper crimson and check the two side by side, or the destructive and primary actions are the same button.
- **Status ramps need fewer steps than the accent.** Most render four roles: a background, a border, a solid fill and text. Generate the full ramp only where the product styles status components across the whole range.

Status color is never the only signal of a state change; pair it with an icon or text. [details-a11y.md](details-a11y.md) owns that requirement.

## Auditing an existing palette

Before restructuring a system, inventory it. Most codebases hold several times more colors than the design has decisions.

1. **Collect every literal.** Grep for hex, `rgb(`, `hsl(`, `oklch(` and the project's utility-class prefixes. Include SVG `fill`/`stroke`, chart configs and email templates. Colors hide outside stylesheets.
2. **Sort by perceived lightness within each hue family.** Duplicates surface immediately as near-identical neighbors.
3. **Collapse near-duplicates.** Two colors closer than about one ramp step are one color that drifted. Keep the one used most and retire the others. Never average them.
4. **Assign each survivor a role** from the table above. A color matching no role is a missing token or a mistake. Decide which, and say so in the finding.
5. **Count what is left.** More than one ramp per role above means the palette outgrew its structure, not that the product needs more color.

Report the inventory before changing anything. Consolidating a palette changes rendered output on screens nobody asked you to touch, so it stays a proposal until the user accepts it.

## Start from the brand color

A brand color arrives as one value, usually a hex. Two decisions come before any ramp exists:

**Which step does it occupy?** A brand color meant for buttons and links belongs on the solid-fill step: `500` in Tailwind, `9` in Radix. Then `bg-brand-500` renders the actual brand color, not an approximation.

**Is it pinned or snapped?** Pin a contractually fixed brand color. It stays exact, the ramp builds outward from it and that one step spaces slightly unevenly. Otherwise snap it onto the ramp so every step spaces evenly. That looks better almost always, and nobody notices without a swatch held to the screen.

A brand color that fails contrast behind white text is still the brand color, just not the solid-fill step. Put it where it lands and use a darker step for interactive fills. Never quietly darken the brand.

## What a correct ramp looks like

Properties of the finished ramp, checkable against any output in any notation:

- **Steps are evenly spaced in perceived lightness.** Not in the number your format calls "lightness". HSL's is not perceptual, and evenly spaced HSL values bunch at one end.
- **Hue is constant end to end.** Every step is recognisably the same color. A wandering hue reads as two colors blended and will not sit correctly against a neutral built on a different hue.
- **Vividness peaks in the middle and falls off at both ends.** The lightest and darkest steps are nearly neutral; the middle carries the color. Holding full vividness into the extremes gives a `50` that glows and a `950` like ink spilled on the brand.
- **Steps are denser at the light end.** Light backgrounds need finer distinctions than dark ones. Keep `50` to `200` close together and `800` to `950` further apart. Even spacing across the whole range makes the pale end unusable, because `50` and `100` stop reading as two surfaces.
- **No two adjacent steps are indistinguishable.** If `200` and `300` look identical on a calibrated screen, the ramp has more steps than decisions. Drop one.
- **Both ends stop short of pure black and white.** A ramp that reaches them loses its identity exactly where the page background lives.

## Use a color library

Never compute these by hand or by eye. `culori`, `colorjs.io` and `chroma.js` all convert between notations, measure perceived lightness and interpolate perceptually. Read the brand color in whatever format it arrives, do the math in a perceptual space and emit the project's notation:

```js
import { formatHex, interpolate, samples } from 'culori'

// Perceptual interpolation, hex in and hex out.
const ramp = interpolate(['#eff6ff', '#3b82f6', '#172554'], 'lab')
const steps = samples(11).map((t) => formatHex(ramp(t)))
```

The output format is the project's choice. For a ramp the interpolation space is not, because the steps have to land evenly in perceived lightness and sRGB interpolation produces muddy mid-steps. Decorative gradients are the opposite case, where the space is a deliberate look.

```css
:root {
  --brand-50: #eff6ff;
  --brand-100: #dbeafe;
  --brand-200: #bfdbfe;
  --brand-300: #93c5fd;
  --brand-400: #60a5fa;
  --brand-500: #3b82f6;
  --brand-600: #2563eb;
  --brand-700: #1d4ed8;
  --brand-800: #1e40af;
  --brand-900: #1e3a8a;
  --brand-950: #172554;
}
```

## Several hues at once

With an accent plus status ramps, the ramps must agree step for step. `danger-500` and `brand-500` should read as equally bright and vivid, or a red button looks heavier than a blue one at the same step.

- **Match perceived lightness exactly.** Same step, same brightness, across every hue.
- **Match vividness relatively, not absolutely.** Hues do not share a maximum vividness. A saturated yellow and a saturated blue are not equally far from gray, and no format makes them so. Set each ramp to the same *proportion* of what its own hue reaches. Copying a saturation number across hues leaves one washed out.

Yellows and cyans are the usual casualties, peaking much lower than reds and blues. Copy the numbers across and the warning color looks weak beside the danger one.

## Dark mode

A dark palette is not the light one reversed. Reversal is the starting point, not the output.

Swap the semantic roles first, then tune the values:

```css
:root {
  --color-bg: var(--brand-50);
  --color-text: var(--brand-950);
}

.dark {
  --color-bg: var(--brand-950);
  --color-text: var(--brand-50);
}
```

Three things almost always need hand-tuning after the swap:

- **Vividness comes down.** A color that reads as confident on white reads as neon on near-black. Dark appearances need the accent a step or two less vivid.
- **The dark end needs more separation.** Steps distinguishable as pale backgrounds collapse into each other as dark surfaces.
- **Contrast does not survive the mirror.** A pair passing in light mode can fail reversed, because contrast is not symmetric. Recheck every foreground against its real background in both appearances.

### Choosing the switching mechanism

Pick one and use it throughout:

- **`prefers-color-scheme` alone** is correct with no theme toggle. Nothing to persist, nothing to hydrate.
- **A `.dark` class** is required as soon as users can override the system setting. The media query then sets only the initial value.
- **`light-dark()`** collapses both values into one declaration, the least code when the project also sets `color-scheme`. It reads that property rather than a class, so a class-based toggle must set `color-scheme` too.

```css
:root {
  color-scheme: light dark;
  --color-bg: light-dark(#ffffff, #172554);
}
```

Mixing mechanisms is the common failure. A media query setting some tokens and a class setting others gives a half-themed interface the moment a user overrides their system preference.

## Two tiers

**Primitives** name a value. They are the ramp, named by hue and step: `--blue-500`, `--neutral-200`. A primitive describes what the color *is*, so it never changes meaning between themes and is never applied directly in a component.

**Semantics** name a job. They point at a primitive and take the name of the role they fill: `--color-text-secondary`, `--color-border-subtle`. Components only ever reference this tier.

```css
:root {
  /* Tier 1: primitives, named by appearance. Never used directly. */
  --blue-500: #3b82f6;
  --neutral-200: #e5e7eb;
  --neutral-700: #374151;

  /* Tier 2: semantics, named by role. This is what components use. */
  --color-accent-solid: var(--blue-500);
  --color-border: var(--neutral-200);
  --color-text-secondary: var(--neutral-700);
}
```

The tiering is what makes theming possible. Dark mode, a white-label theme and an increased-contrast variant all repoint the semantic tier, leaving the primitives and every component untouched. A codebase applying `--blue-500` directly in components has no theming seam. Adding one later means auditing every usage to work out which meant "the accent" and which just wanted blue.

Add a third, component-level tier (`--color-button-danger-bg`) only where a component genuinely and intentionally diverges from the system. One component token is a documented exception; twenty mean the semantic tier is missing roles.

## The role inventory

A system is complete when every role below has a token. Build against this list rather than adding tokens as components demand them, or the palette ends up shaped like whichever screen came first.

| Group | Roles |
| --- | --- |
| Surfaces | page background, surface, raised (menus, popovers), sunken (inputs, wells), overlay scrim |
| Text | primary, secondary, disabled, inverse, on-accent |
| Borders | subtle, default, strong, focus ring, separator |
| Accent | subtle background, border, solid, solid hover, text |
| Status | per status shipped: subtle background, border, solid, text |

Separator and border are separate roles even when they share a value today. A separator divides content; a border encloses a control. They diverge the first time someone restyles inputs, and a system that conflated them gets untangled at that moment.

## Naming grammar

Use one shape and never deviate: `--color-{role}-{variant}-{state}`.

```css
--color-bg-surface
--color-text-secondary
--color-border-strong
--color-accent-solid-hover
```

Pick one word per concept and use only that word. Consistency matters more than the vocabulary. A reader who has seen `--color-text-primary` must be able to guess `--color-text-disabled` without looking:

| Concept | Pick one | Never mix in |
| --- | --- | --- |
| Foreground | `text` | `fg`, `foreground`, `content`, `ink` |
| Background | `bg` | `background`, `surface` as a synonym, `fill` |
| Edge | `border` | `stroke`, `outline`, `line` |
| Brand color | `accent` | `primary`, `brand`, `theme` used interchangeably |

Reserve `primary` for exactly one meaning. `--color-text-primary` for body text beside `--color-primary` for the brand is the most common naming collision there is, and it makes every `primary` token ambiguous until you open its definition. Use `accent` for the brand and let `primary` mean "the most prominent of its group".

## Anti-patterns

| Name | Problem | Instead |
| --- | --- | --- |
| `--color-blue-button` | Appearance at the semantic tier; lies the moment the brand changes | `--color-accent-solid` |
| `--color-sidebar-gray` | Named for where it was used first; the second usage makes it nonsense | `--color-bg-surface` |
| `--color-light-gray` | Lies in dark mode, where it is the dark one | `--neutral-200` as a primitive |
| `--color-text-2` | Numbered semantics carry no meaning; nobody can guess what `3` would be | `--color-text-secondary` |
| `--color-gray-hover` | Mixes a hue with a state and belongs to no tier | `--color-bg-surface-hover` |
| `--blue-500` used in a component | Skips the semantic tier and removes the theming seam | Point a semantic token at it |

Every one of them is a case of **Use a token only in its role**.

## In Tailwind projects

Tailwind v4 generates utilities from `@theme`, so names declared there become the API. Declare primitives and semantics in the same block; the `--color-*` namespace is what produces `bg-*`, `text-*` and `border-*`:

```css
@theme {
  /* Primitives */
  --color-brand-50: #eff6ff;
  --color-brand-500: #3b82f6;
  --color-brand-900: #1e3a8a;

  /* Semantics: what templates should use */
  --color-accent-solid: var(--color-brand-500);
  --color-text-secondary: var(--color-neutral-700);
}
```

That yields `bg-accent-solid` and `text-secondary` alongside `bg-brand-500`. Both are reachable, so the discipline is a convention rather than a constraint. Templates use the semantic utilities, and a raw `bg-brand-500` in a component is the thing to flag.

Opacity modifiers work on either tier, as in `bg-accent-solid/50`. But a color carrying alpha cannot be contrast-checked against a static background, because what it renders depends on what sits behind it. Use solid tokens for anything with text on it.

## One color, one meaning

Users read a near-miss in hue as a slightly different shade, not as a different color.

```css
/* Bad: the accent means both "link" and "decorative heading" */
a { color: #3b82f6; }
.section-title { color: #4f8ef7; }

/* Good: interactive elements own the accent; headings stay neutral */
a { color: var(--color-accent-text); }
.section-title { color: var(--color-text-primary); }
```

The rule runs both ways. A color must not be *absent* where its meaning occurs. If the accent means interactive, an interactive element rendered neutral is just as misleading.

Color is never the only carrier of meaning. Pair it with an icon, a label, or a shape. [details-a11y.md](details-a11y.md) owns that requirement.

## Use tokens in their role

Apply a semantic token only for the role it names. `--color-text-secondary` is muted foreground text. Use it as a background and every future theme change that assumes the role breaks, because a value that happened to work as both stops working as both.

```css
/* Bad: separator token repurposed as a text color because it looked right */
.caption { color: var(--color-border); }

/* Bad: text token repurposed as a background */
.tag { background: var(--color-text-secondary); }
```

The role inventory above is the list of roles a system needs.

## One colored action per view

Preserve an established component hierarchy that communicates emphasis another way. Do not recolor controls merely to impose this recipe.

```html
<!-- Good: one filled primary action, neutral secondaries -->
<button class="bg-accent-solid text-white">Save</button>
<button class="text-neutral-700">Cancel</button>

<!-- Bad: every action colored, so nothing is primary -->
<button class="bg-accent-solid text-white">Save</button>
<button class="bg-accent-solid text-white">Duplicate</button>
<button class="bg-accent-solid text-white">Export</button>
```

Selected states may use the accent on the glyph and label. An active tab or a checked segment is state, not emphasis.

## Gradients

**The interpolation space is a look, not a correctness setting.** Three are worth knowing, and the difference between them is most visible in the middle of the gradient:

```css
/* sRGB: the default and the classic. Midpoint darkens and mutes. */
background: linear-gradient(#3b82f6, #ec4899);

/* oklab: even brightness across the transition. The best default. */
background: linear-gradient(in oklab, #3b82f6, #ec4899);

/* oklch: travels around the hue wheel, staying vivid throughout. */
background: linear-gradient(in oklch, #3b82f6, #ec4899);
```

`oklab` and sRGB are **rectangular**, interpolating in a straight line through the color space. `oklch` is **polar**, interpolating the hue angle, so it arcs around the wheel through every hue between the stops. That is why it stays saturated and why it can produce hues nobody asked for. A blue-to-pink gradient routes through purple, which is either the look or a surprise.

**The gray dead zone is a rectangular-space problem.** Two hues on opposite sides of the wheel sit either side of the neutral axis. A straight line between them passes near gray, and the middle goes lifeless. Either switch to a polar space, which routes around the axis, or add a third stop between the two and keep the space you have.

With a polar space you also control which way it goes around:

```css
/* The short way round, usually what you want */
background: linear-gradient(in oklch shorter hue, #3b82f6, #ec4899);

/* The long way, sweeps most of the spectrum */
background: linear-gradient(in oklch longer hue, #3b82f6, #ec4899);
```

**Banding shows up on large areas.** A gradient spanning a hero with little contrast between its stops steps visibly on 8-bit displays. Widen the contrast, shrink the area, or overlay a subtle noise texture.

**Keep text off gradients where you can.** Contrast varies continuously across one, so a single measurement does not describe it. Where text must sit on a gradient, measure the worst region rather than the average, or put a scrim behind it.

## Color across cultures

Color meaning is not universal. Where a color is load-bearing in finance, status, or alerts, verify the meaning holds in every locale you ship to.

| Color | Common Western reading | Elsewhere |
| --- | --- | --- |
| Red | Danger, loss, errors | Luck, prosperity; **gains** in Chinese financial UIs |
| Green | Success, gains, go | Losses in Chinese financial UIs |
| White | Purity, cleanliness | Mourning in parts of East Asia |
| Gold | Premium, luxury | Religious significance in some regions |

The classic case is stock tickers, which show gains in green for English locales and red for Chinese ones. Where the product ships to such markets, make gain and loss per-locale tokens rather than hardcoded values.

## Light, dark and increased contrast

Every custom color needs a light and a dark variant, derived per below. Beyond that, users who enable increased contrast expect visibly stronger differentiation:

```css
:root {
  --color-accent-solid: #3b82f6;
}

@media (prefers-color-scheme: dark) {
  :root { --color-accent-solid: #60a5fa; }
}

@media (prefers-contrast: more) {
  :root { --color-accent-solid: #1d4ed8; }
}
```

The increased-contrast variant widens the foreground/background gap by at least 15 points of perceived lightness over the default. Re-verify against APCA's preferred thresholds, Lc 90 body and Lc 75 non-body. Widening the gap without remeasuring is not fixing it.

## Choosing a notation

| Notation | Good for | Weakness |
| --- | --- | --- |
| Hex | Universal support, compact, what design tools hand you | Opaque. No channel is readable or editable by hand |
| `rgb()` | Same reach as hex, readable alpha | Channels do not correspond to anything a designer thinks about |
| `hsl()` | Channels look like design controls | Its lightness is not perceptual and its hue drifts; a ramp built by varying lightness bunches at one end and shifts hue |
| `oklch()` | Perceptually uniform lightness, stable hue, predictable ramps | Baseline 2023, so very old browser matrices need a fallback |

**Match whatever the project already uses.** Notation is not a defect: a project on hex is not doing it wrong.

**For a genuinely new color system, `oklch()` is the best default.** Even lightness steps stay even, and a fixed hue stays fixed.

```css
oklch(L C H)          /* lightness 0–1, chroma 0–~0.4, hue 0–360 */
oklch(L C H / alpha)  /* alpha uses a slash, never a comma */
```

## Converting

Convert when the user asks, when an agreed migration is in scope, or when the project is standardizing on a notation and this value is the straggler. Never convert an isolated value in a project that deliberately uses something else and never because this file happened to load.

When conversion is in scope, change the values and nothing else:

- Leave CSS keywords alone: `currentColor`, `inherit`, `transparent`, `initial`, `unset`.
- Leave gradient functions alone. Convert the color stops inside them; do not touch the interpolation method.
- Leave colors in third-party configs that expect a specific format.
- Preserve comments and formatting.

```css
/* Before */
color: #3b82f6;
border: 1px solid rgba(0, 0, 0, 0.1);

/* After */
color: oklch(0.623 0.188 259.815);
border: 1px solid oklch(0 0 0 / 0.1);
```

Bulk conversion is a migration, not cleanup. It shifts every rendered color by a rounding margin and touches files nobody asked about, so it has to be the task rather than a side effect.

## Gamut

Every sRGB color exists in Display P3, but not the reverse. P3 covers roughly 50% more colors, which matters only for the most saturated values. A color at 60% of maximum vividness looks the same on both.

A color more vivid than its display can render gets clipped, and clipping is not graceful. It flattens neighbouring steps into one rendered color, so the top of a ramp can lose its distinctions on an sRGB screen. Maximum vividness varies by hue and lightness. Cyans top out far lower than reds and purples, so a clipping ramp clips at some steps and not others.

The fix is to reduce vividness while holding hue and lightness. Generate ramps against sRGB unless the product is display-restricted, and add P3 as an enhancement:

```css
.accent {
  background: #3b82f6;
}

@media (color-gamut: p3) {
  .accent {
    background: oklch(0.62 0.24 259);
  }
}
```

Order matters. The sRGB value comes first so every display gets something, and the P3 rule overrides only where it will render. A P3 color with no fallback is a `HIGH` finding; it does not degrade, it fails.

For browser matrices predating `oklch()` support, the same layering works with `@supports`:

```css
.accent {
  background: #3b82f6;
}

@supports (color: oklch(0 0 0)) {
  .accent {
    background: oklch(0.62 0.19 259);
  }
}
```

Check the project's actual browser matrix before adding this. On a modern baseline it is dead weight.

## Modern CSS worth knowing

- **`color-mix()`** derives one color from another, as in `color-mix(in oklab, var(--color-accent-solid) 15%, white)` for a tinted background. Useful for states, but keep generated values out of the token layer, since a mixed color cannot be inspected in a design tool.
- **Relative color syntax** adjusts one channel of an existing color. `oklch(from var(--color-accent-solid) calc(l - 0.1) c h)` darkens the accent by hand. Powerful and easy to overuse, since a token defined by three chained derivations is unreadable.
- **`light-dark()`** puts both appearances in one declaration.

All three compute at render time, so their output cannot be contrast-checked statically. Measure the rendered result.

## APCA thresholds (recommended)

APCA (Accessible Perceptual Contrast Algorithm) models perceived contrast more accurately than WCAG 2 and is the better default for design decisions. Lc (Lightness Contrast) measures perceived contrast between foreground and background. These levels simplify APCA's full font-size and weight lookup table:

| Content type | Minimum | Preferred |
| --- | --- | --- |
| Body text (columns or blocks of text) | Lc 75 | Lc 90 |
| Non-body text (labels, headlines) | Lc 60 | Lc 75 |
| Large text (≥36px) | Lc 45 | Lc 60 |
| UI components | Lc 30 | n/a |

Lc 30 is also APCA's minimum for disabled and placeholder text. The floor for a non-text element to be discernible at all is Lc 15.

Lc is signed: positive means dark text on a light background, negative means light text on a dark background. Compare the absolute value against the threshold.

## WCAG 2 thresholds (for legal compliance)

WCAG 2 is still required for formal WCAG 2.x conformance claims. Its luminance ratio is both too strict and too lenient depending on the pair, but it has the legal standing.

| Content type | AA | AAA |
| --- | --- | --- |
| Normal text (<24px / <18.5px bold) | 4.5:1 | 7:1 |
| Large text (≥24px / ≥18.5px bold) | 3:1 | 4.5:1 |
| UI components and graphical objects | 3:1 | n/a |

WCAG defines large text in points: 18pt ≈ `24px`, 14pt bold ≈ `18.5px`.

When a project must claim WCAG conformance, WCAG is the gate and APCA is the tiebreaker for anything above it.

## Fixing a failing pair (on request)

**Change lightness first.** It is the channel contrast responds to. Hue and saturation move the measured value far less, so fixing contrast by changing hue is wasted effort.

Move the foreground away from the background in perceived lightness, holding hue and saturation, then remeasure. Keeping hue fixed is what stops a contrast fix becoming a palette change.

```css
/* Failing: text too close to its background in lightness (Lc ≈ 50) */
color: #7d93b0;
background: #eef2f7;

/* Fixed: darker text, same hue (Lc ≈ 90) */
color: #2b3a4f;
background: #eef2f7;
```

Two constraints on the fix:

- **Mid-lightness backgrounds cap what is achievable.** On a background near 75% perceived lightness, even pure black text reaches only about Lc 60. Body text needs a background near one extreme, so a mid-range background is the thing that has to change.
- **Pushing lightness can push the color out of gamut.** Reduce saturation as needed to keep it renderable.

Always remeasure after changing a value. Do not assume a fix landed.

## Quick approximations

Useful for a first pass and approximations. Verify by measuring before reporting a result.

For body text targeting |Lc| ≥ 75:

- **Light background (above ~90% perceived lightness):** foreground below ~35%.
- **Dark background (below ~25% perceived lightness):** foreground above ~90%.

The gap is asymmetric because APCA is polarity-aware. Mirrored pairs do not score identically, which is why a pair passing in light mode can fail in dark.

**Light or dark background?** The crossover is around 73% perceived lightness. Above it use dark text; at or below it light text scores higher. That is higher than intuition suggests. Between roughly 60% and 73% the background already looks light, yet white text still measures meaningfully better than black.

## What to check

- **Every pair, in every appearance.** A pair passing in light mode can fail in dark. The palettes are not mirror images.
- **Translucent surfaces.** A color on a `backdrop-filter` header or an overlay shifts with whatever scrolls behind it. Test against the lightest and darkest content it can sit over, or make the surface opaque enough that the shift cannot break the pair.
- **Computed colors.** `color-mix()`, relative color syntax and opacity modifiers resolve at render time; measure the rendered result, not the declaration.
- **Text over images.** There is no single background color. Measure the worst region, or guarantee one with a scrim.

## Before you finish

| Mistake | Fix |
| --- | --- |
| A raw value where the project has a token | Reuse or add the role token, in the project's notation |
| An isolated `oklch()` value dropped into a hex codebase | Keep the established notation unless a migration is in scope |
| A primitive like `--blue-500` used directly in a component | Point a semantic token at it |
| Token named for its appearance (`--color-blue-button`) or first use (`--color-sidebar-gray`) | Name it for its role: `--color-accent-solid`, `--color-bg-surface` |
| `--color-primary` meaning the brand and `--color-text-primary` meaning body text | Reserve `accent` for the brand; let `primary` mean "most prominent of its group" |
| Semantic token used outside its role (separator as text) | Add a token for the missing role; never borrow by value |
| Ramp built by varying HSL lightness | Rebuild against perceived lightness with a constant hue |
| Ramp spaced evenly across the full range | Tighten the light end until `50` and `100` read as two surfaces |
| Same saturation number reused across hues | Match the proportion of each hue's own maximum, not the raw value |
| Status hue that collides with the accent hue | Move it until destructive and primary read apart side by side |
| Dark mode made by mechanically reversing the light palette | Reverse as a starting point, then reduce vividness, widen the dark end and recheck every pair |
| `prefers-color-scheme` setting some tokens and a `.dark` class setting others | Pick one switching mechanism and use it throughout |
| Contrast fixed by changing hue | Change lightness, the channel contrast responds to |
| P3 color with no sRGB fallback | Declare the sRGB value first, then override inside `@media (color-gamut: p3)` |

## Checks

Without a browser: token values, the gamut of every declared color, both theme blocks present and contrast computed from the declared token pair. With one: the background actually rendered behind the text, including opacity and any image beneath it, measured in both light and dark. A failing pair is reported, not repainted. Report every check you could not run as `Not verified`.
