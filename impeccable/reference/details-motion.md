Micro-interaction recipes: interruptible transitions, press feedback, staged entrances and exits, icon swaps, and compositing. [animate.md](animate.md) decides whether something moves; this file sets the values once it does. Every duration, curve, scale and blur is a specific value: `cubic-bezier(0.2, 0, 0, 1)` is not `cubic-bezier(0.4, 0, 0.2, 1)`, and `0.96` is not `0.95`.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## Interruptible animations

Users change intent mid-interaction. Animations that cannot be interrupted make the interface feel broken.

### CSS transitions vs. keyframes

| | CSS Transitions | CSS Keyframe Animations |
| --- | --- | --- |
| **Behavior** | Interpolate toward latest state | Run on a fixed timeline |
| **Interruptible** | Yes, retargets mid-animation | No, restarts from beginning |
| **Use for** | Interactive state changes (hover, toggle, open/close) | Staged sequences that run once (enter animations, loading) |
| **Duration** | Fixed; retargets the value mid-flight, not the timeline | Fixed timeline, restarts from the beginning |

```css
/* Good: interruptible transition for a toggle */
.drawer {
  transform: translateX(-100%);
  transition: transform 200ms ease-out;
}
.drawer.open {
  transform: translateX(0);
}

/* Clicking again mid-animation smoothly reverses, no jank */
```

```css
/* Bad: keyframe animation for interactive element */
.drawer.open {
  animation: slideIn 200ms ease-out forwards;
}

/* Closing mid-animation snaps or restarts, feels broken */
```

Prefer CSS transitions for interactive elements. Reserve keyframes for one-shot sequences.

## Scale on press

A subtle scale-down on click gives buttons tactile feedback. Always `scale(0.96)`, never below `0.95`, which feels exaggerated. Use CSS transitions so a release mid-press returns smoothly.

Not every button needs it. Add a `static` prop that disables the scale where the motion would distract.

### CSS example

```css
.button {
  transition-property: scale;
  transition-duration: 150ms;
  transition-timing-function: ease-out;
}

.button:active {
  scale: 0.96;
}
```

### Tailwind example

```tsx
<button className="transition-transform duration-150 ease-out active:scale-[0.96]">
  Click me
</button>
```

### Motion example

```tsx
<motion.button whileTap={{ scale: 0.96 }}>
  Click me
</motion.button>
```

### Static prop pattern

Extract the scale class into a variable and apply it conditionally on a `static` prop:

```tsx
const tapScale = "active:not-disabled:scale-[0.96]";

function Button({ static: isStatic, className, children, ...props }) {
  return (
    <button
      className={cn(
        "transition-transform duration-150 ease-out",
        !isStatic && tapScale,
        className,
      )}
      {...props}
    >
      {children}
    </button>
  );
}

// Usage
<Button>Click me</Button>           {/* scales on press */}
<Button static>Submit</Button>       {/* no scale */}
```

## Skip animation on page load

Use `initial={false}` on `AnimatePresence` to stop enter animations firing on first render. An element already in its default state animates on later state changes, not on page load.

### When it works

```tsx
// Good: icon doesn't animate in on mount, only on state change
<AnimatePresence initial={false} mode="popLayout">
  <motion.span
    key={isActive ? "active" : "inactive"}
    initial={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
    animate={{ opacity: 1, scale: 1, filter: "blur(0px)" }}
    exit={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
  >
    <Icon />
  </motion.span>
</AnimatePresence>
```

Works well for icon swaps, toggles, tabs and segmented controls, anything with a default state on page load.

### When it breaks

Never use `initial={false}` where the component relies on its `initial` prop for a first-time enter animation, such as a staggered page hero or a loading state. Removing it skips the entire entrance.

```tsx
// Bad: initial={false} would skip the staggered page enter entirely
<AnimatePresence initial={false}>
  <motion.div initial="hidden" animate="visible" variants={...}>
    ...
  </motion.div>
</AnimatePresence>
```

Verify the component still looks right on a full page refresh before applying this.

## Suppress transitions on theme switch

Flipping the theme changes `color`, `background-color`, `border-color` and `box-shadow` on nearly every element at once. Everything carrying a transition on those properties animates together, so the switch reads as a slow smear rather than an instant change. Disable transitions for the swap and restore them right after.

`next-themes` ships this as `disableTransitionOnChange`, so use that where the project has it. Otherwise inject a stylesheet that turns off every transition, force a reflow so the new colors commit while it still applies, then drop it on the next frame. Do this around the theme flip in the toggle's event handler; a listener for the OS-level change belongs in `useMountEffect`, per the `no-use-effect` skill:

```ts
const style = document.createElement("style");
style.append(document.createTextNode("*,*::before,*::after{transition:none !important}"));
document.head.append(style);

applyTheme(next); // the project's own theme flip
const _flushReflow = document.body.offsetHeight;

requestAnimationFrame(() => {
  requestAnimationFrame(() => style.remove());
});
```

`document.body.offsetHeight` is read for its side effect, forcing a synchronous style flush so the new theme resolves while the override is still in the document and no transition starts. The nested `requestAnimationFrame` removes the override only after that paint, restoring transitions before the next interaction.


## Motion restraint

Motion is a budget, not a garnish. Three rules decide whether an animation belongs at all:

- **Give high-frequency interactions instant feedback instead of animation.** Every keystroke, every list-row hover, every tab switch in a work tool. Reserve expressive motion for infrequent moments (first load of a view, success states, empty states); high-frequency interactions get instant feedback or the subtlest possible transition (`opacity`/`background-color` at ≤150ms).
- **Motion is never the only feedback channel.** Every state change an animation communicates stays visible when it doesn't run: a color change, an icon swap, a label. Users with reduced motion enabled and anyone who blinked still need to see what happened.
- **Brief and precise beats prominent.** Where a shorter, smaller animation says the same thing, use it. When in doubt cut the duration, not the clarity.

```css
/* Good: high-frequency hover gets a minimal transition */
.row:hover {
  background-color: var(--surface-hover);
  transition: background-color 100ms ease-out;
}

/* Bad: every hover replays a full entrance */
.row:hover .row-icon {
  animation: bounceIn 500ms;
}
```

## Enter animations: split and stagger

Use this for infrequent staged entrances where sequence communicates hierarchy: the first load of a page hero, a success state, an empty state. Break a large container into semantic chunks and animate each one. Stagger only those moments, keeping routine interactions such as row hovers, keystrokes, and repeated tab changes immediate. A staged entrance is the surface's one authored moment when it has earned one, not a default for every section ([craft-floor.md](craft-floor.md)).

### Step by step

1. **Split** into logical groups (title, description, buttons)
2. **Stagger** with ~100ms delay between groups
3. **For titles**, consider splitting into individual words with ~80ms stagger
4. **Combine** `opacity`, `blur` and `translateY` for the enter effect

### Code example

```tsx
// Motion (Framer Motion): staggered enter
function PageHeader() {
  return (
    <motion.div
      initial="hidden"
      animate="visible"
      variants={{
        visible: { transition: { staggerChildren: 0.1 } },
      }}
    >
      <motion.h1
        variants={{
          hidden: { opacity: 0, y: 12, filter: "blur(4px)" },
          visible: { opacity: 1, y: 0, filter: "blur(0px)" },
        }}
      >
        Welcome
      </motion.h1>

      <motion.p
        variants={{
          hidden: { opacity: 0, y: 12, filter: "blur(4px)" },
          visible: { opacity: 1, y: 0, filter: "blur(0px)" },
        }}
      >
        A description of the page.
      </motion.p>

      <motion.div
        variants={{
          hidden: { opacity: 0, y: 12, filter: "blur(4px)" },
          visible: { opacity: 1, y: 0, filter: "blur(0px)" },
        }}
      >
        <Button>Get started</Button>
      </motion.div>
    </motion.div>
  );
}
```

### CSS-only stagger

```css
/* `backwards` holds the from-state during the delay; the default stays visible if animation fails */
.stagger-item {
  animation: fadeInUp 400ms ease-out backwards;
}

.stagger-item:nth-child(1) { animation-delay: 0ms; }
.stagger-item:nth-child(2) { animation-delay: 100ms; }
.stagger-item:nth-child(3) { animation-delay: 200ms; }

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(12px);
    filter: blur(4px);
  }
}
```

## Exit animations

Exits are softer and less attention-grabbing than enters. The user's focus is moving to the next thing, so do not fight for it.

### Subtle exit (recommended)

```tsx
// Small fixed translateY: indicates direction without drama
<motion.div
  exit={{
    opacity: 0,
    y: -12,
    filter: "blur(4px)",
    transition: { duration: 0.15, ease: "easeOut" },
  }}
>
  {content}
</motion.div>
```

### Full exit (when context matters)

```tsx
// Slide fully out: use when spatial context is important
// (e.g., a card returning to a list, a drawer closing)
<motion.div
  exit={{
    opacity: 0,
    x: "-100%",
    transition: { duration: 0.2, ease: "easeOut" },
  }}
>
  {content}
</motion.div>
```

### Good vs. bad

```css
/* Good: subtle exit */
.item-exit {
  opacity: 0;
  transform: translateY(-12px);
  transition: opacity 150ms ease-out, transform 150ms ease-out;
}

/* Bad: dramatic exit that steals focus */
.item-exit {
  opacity: 0;
  transform: translateY(-100%) scale(0.5);
  transition: all 400ms ease-out;
}

/* Sometimes correct: remove immediately when motion adds no context */
.item-exit {
  display: none;
}
```

**Key points:**
- Use a small fixed `translateY`, say `-12px`, rather than the full container height
- Keep some directional movement to indicate where the element went
- Exit duration should be shorter than enter duration (150ms vs 300ms)
- Use a subtle exit when it preserves spatial context. Remove immediately when motion adds no information, the interaction repeats frequently, or reduced motion is requested.

## Contextual icon animations

When icons appear or disappear contextually, on hover or a state change, animate them with `opacity`, `scale` and `blur` rather than toggling visibility.

### Motion example

This uses the `motion` package. Where the project has `framer-motion`, import the same APIs from `"framer-motion"`. Never mix an installed package with the other's import path.

```tsx
import { AnimatePresence, motion } from "motion/react";

function IconButton({ isActive, icon: Icon }) {
  return (
    <button>
      <AnimatePresence mode="popLayout">
        <motion.span
          key={isActive ? "active" : "inactive"}
          initial={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
          animate={{ opacity: 1, scale: 1, filter: "blur(0px)" }}
          exit={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
          transition={{ type: "spring", duration: 0.3, bounce: 0 }}
        >
          <Icon />
        </motion.span>
      </AnimatePresence>
    </button>
  );
}
```

### CSS transition approach (no Motion)

Without Motion or Framer Motion, keep both icons in the DOM and cross-fade with CSS transitions. Neither unmounts, so enter and exit both animate smoothly.

One icon is absolutely positioned on top of the other. Toggling state cross-fades them, the entering icon scaling up from `0.25` while the exiting one scales down to `0.25`, both with opacity and blur.

```tsx
function IconButton({ isActive, ActiveIcon, InactiveIcon }) {
  return (
    <button>
      <div className="relative">
        <div
          className={cn(
            "absolute inset-0 flex items-center justify-center",
            "transition-[opacity,filter,scale] duration-300",
            "ease-[cubic-bezier(0.2,0,0,1)]",
            isActive
              ? "scale-100 opacity-100 blur-0"
              : "scale-[0.25] opacity-0 blur-[4px]"
          )}
        >
          <ActiveIcon />
        </div>
        <div
          className={cn(
            "transition-[opacity,filter,scale] duration-300",
            "ease-[cubic-bezier(0.2,0,0,1)]",
            isActive
              ? "scale-[0.25] opacity-0 blur-[4px]"
              : "scale-100 opacity-100 blur-0"
          )}
        >
          <InactiveIcon />
        </div>
      </div>
    </button>
  );
}
```

The non-absolute icon, `InactiveIcon`, defines the layout size. The absolute one, `ActiveIcon`, overlays it without affecting flow.

### Choosing between Motion and CSS

| | Motion (Framer Motion) | CSS transitions (both icons in DOM) |
| --- | --- | --- |
| **Enter animation** | Yes | Yes |
| **Exit animation** | Yes (via `AnimatePresence`) | Yes (cross-fade, icon never unmounts) |
| **Spring physics** | Yes | No, use `cubic-bezier(0.2, 0, 0, 1)` as approximation |
| **When to use** | Project already uses `motion` or `framer-motion` | No motion dependency, or keeping bundle small |

Check the project's `package.json`. Import from `"motion/react"` when `motion` is installed, or `"framer-motion"` when that is. Where both exist, follow the imports the component or its nearest peers already use. Where neither is present, use the CSS cross-fade and never add a dependency just for icon transitions.

### When to animate icons

| Animate | Don't animate |
| --- | --- |
| Icons that appear on hover (action buttons) | Static navigation icons |
| State change icons (play → pause, like → liked) | Decorative icons |
| Icons in contextual toolbars | Icons that are always visible |
| Loading/success state indicators | Icon labels (text next to icon) |

Use exactly these values for contextual icon animations. Do not deviate:
- `scale`: `0.25` → `1` (never use `0.5` or `0.6`)
- `opacity`: `0` → `1`
- `filter`: `"blur(4px)"` → `"blur(0px)"`
- `transition`: `{ type: "spring", duration: 0.3, bounce: 0 }`; **bounce must always be `0`**, never `0.1` or any other value

## Transition only what changes

Never use `transition: all` or Tailwind's `transition-all`. Always name the exact properties that change. Tailwind's bare `transition` maps to a curated list of colors, opacity, shadow and transforms rather than `all`, and naming what changes is still better.

### Why

- `transition: all` forces the browser to watch every property for changes
- Causes unexpected transitions on properties you didn't intend to animate (colors, padding, shadows)
- Prevents browser optimizations

### CSS example

```css
/* Good: only transition what changes */
.button {
  transition-property: scale, background-color;
  transition-duration: 150ms;
  transition-timing-function: ease-out;
}

/* Bad: transition everything */
.button {
  transition: all 150ms ease-out;
}
```

### Tailwind

```tsx
// Good: explicit properties
<button className="transition-[scale,background-color] duration-150 ease-out">

// Bad: transition all
<button className="transition-all duration-150 ease-out">
```

### Tailwind `transition-transform` note

`transition-transform` in Tailwind maps to `transition-property: transform, translate, scale, rotate`, covering every transform-related property rather than only `transform`. Use it when animating transforms alone. For several non-transform properties, use the bracket syntax `transition-[scale,opacity,filter]`.

## Use `will-change` sparingly

`will-change` hints the browser to pre-promote an element to its own GPU compositing layer. Without it the browser promotes only when the animation starts, and that one-time promotion can cause a micro-stutter on the first frame.

It helps most for `scale`, `rotation` and movement through `transform`. For other properties it does little, because the browser cannot composite them on the GPU anyway.

### Rules

```css
/* Good: specific property that benefits from GPU compositing */
.animated-card {
  will-change: transform;
}

/* Good: multiple compositor-friendly properties */
.animated-card {
  will-change: transform, opacity;
}

/* Bad: never use will-change: all */
.animated-card {
  will-change: all;
}

/* Bad: properties that can't be GPU-composited anyway */
.animated-card {
  will-change: background-color, padding;
}
```

### Useful properties

| Property | GPU-compositable | Worth using `will-change` |
| --- | --- | --- |
| `transform` | Yes | Yes |
| `opacity` | Yes | Yes |
| `filter` (blur, brightness) | Yes | Yes |
| `clip-path` | Newer Chromium only | Rarely; not reliable cross-browser |
| `top`, `left`, `width`, `height` | No | No |
| `background`, `border`, `color` | No | No |

### When to skip

Modern browsers optimize well on their own. Add `will-change` only when you see first-frame stutter, which Safari benefits from most. Never add it preemptively to every animated element, since each extra compositing layer costs memory.

## Before you finish

| Mistake | Fix |
| --- | --- |
| Icons look off-center | Nudge optically with padding, or fix the SVG |
| Jarring staged entrance or exit | Stagger infrequent entrances; keep exits subtle |
| Theme toggle crossfades the whole page | Disable transitions for the swap, force a reflow, restore on the next frame |
| `transition: all` on elements | Specify exact properties |
| First-frame animation stutter | Add `will-change: transform` (sparingly) |
| Hairline icon beside bold text | Match the stroke width to the text weight |

## Checks

Without a browser: every state the component defines, meaning hover, focus, active, loading and empty, plus motion durations and easings read from the code. With one: walk each state, and replay motion at 10% speed in the browser's Animations panel. Report every check you could not run as `Not verified`.
