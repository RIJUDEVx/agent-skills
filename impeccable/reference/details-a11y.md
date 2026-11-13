Most accessibility is free if you use the platform. Reviewing means two walks: keyboard-only, where every flow completes without a mouse, then screen-reader, where every control announces a name, a role and its state. When unsure, take the platform default over a custom rebuild, and remove ARIA rather than add it.

These are exact values and recipes, loaded when the work reaches this topic. Direction comes from the playbook and [craft-floor.md](craft-floor.md); the brief, DESIGN.md, and the project's tokens and density override anything here. Use a value as written rather than a familiar-looking substitute. Web only: native platforms follow [ios.md](ios.md) or [android.md](android.md).

## The rules of ARIA

1. If a native HTML element with the semantics and behavior you need exists, use it instead of repurposing another element with ARIA.
2. Don't change native semantics unless you really have to.
3. Every interactive ARIA control must be keyboard-operable; a role is a promise of the full keyboard model, states and behavior.
4. Never put `role="presentation"` or `aria-hidden="true"` on a focusable element.
5. All interactive elements must have an accessible name.

No ARIA is better than bad ARIA. A screen reader trusts your roles, so a wrong one is worse than none.

## Button vs link vs div

| Element | Use for | Why |
| --- | --- | --- |
| `<a href>` | Navigation: anything that goes somewhere or changes the URL | Free Cmd/Ctrl/middle-click, right-click → copy link, Enter activation |
| `<button>` | Actions: submit, toggle, open, delete | Free focus, Enter *and* Space activation, form semantics |
| `<div onClick>` | Nothing | No role, no focus, no keyboard; screen readers see plain text |

```tsx
// Bad: invisible to keyboard and screen readers
<div onClick={openSettings}>Settings</div>

// Good: focus, Enter/Space activation and semantics for free
<button onClick={openSettings}>Settings</button>
```

If it looks clickable it must be clickable, and if it's clickable it must be a real interactive element. Rebuilding a link as a button, or the reverse, breaks user expectations. A "button" that navigates is a styled `<a>`.

Where a native element is truly impossible, the full polyfill is `role="button"` plus `tabindex="0"` plus Enter and Space handlers, which is why the native element is always less code.

## Landmarks and headings

- Expose one visible primary `<main>` landmark. `<header>`, `<nav>`, `<aside>`, `<footer>` map to landmarks screen-reader users jump between.
- Multiple landmarks of the same type need distinguishing labels: `<nav aria-label="Primary">`, `<nav aria-label="Breadcrumbs">`.
- Do not report the one-`<h1>` or the no-skipped-levels convention as a standalone WCAG failure without a concrete navigation or comprehension impact. Headings are structure, not styling; style a heading level with CSS instead of picking the tag by size.
- `<title>` matches the current context, most specific first: `Billing · Settings · Acme`.

## Accessible names

Name precedence: `aria-labelledby` > `aria-label` > native label (`<label>`, text content, `alt`) > `title` attribute.

- Prefer visible text or `aria-labelledby` over `aria-label`, which is invisible, drifts out of sync with the UI and is handled inconsistently by translation tools.
- Icon-only buttons always need a name: `<button aria-label="Close">` with the icon `aria-hidden="true"`.
- WCAG 2.5.3 Label in Name: a button showing "Send" with `aria-label="Submit message"` breaks voice control users who say "click Send".
- Accessible names must exist even when the design omits visible labels.

```tsx
// Good: name from visible text, icon hidden
<button>
  <TrashIcon aria-hidden="true" /> Delete
</button>

// Good: icon-only, explicit name
<button aria-label="Delete">
  <TrashIcon aria-hidden="true" />
</button>
```

Add `translate="no"` to brand names, code tokens and identifiers so auto-translation doesn't garble them.

## Common ARIA mistakes

| Mistake | Why it fails |
| --- | --- |
| `aria-label` on a plain `<div>` or `<span>` | Most screen readers ignore names on non-interactive, role-less elements |
| `<button role="button">` | Redundant role; adds noise, no benefit |
| `aria-hidden="true"` on or above a focusable element | Creates elements you can Tab to but that don't exist for screen readers |
| `aria-labelledby`/`aria-describedby` pointing at a missing ID | Silently produces no name or description |
| `role="menu"` on a nav list | `menu` promises app-style arrow-key behavior; site navigation is `<nav>` with a list |

## Disabled states

Native `disabled` supplies the platform's complete disabled behavior. It removes the control from the tab order, suppresses activation, applies `:disabled` and excludes form controls from submission. Use it when a native control is genuinely unavailable. `aria-disabled="true"` only announces the state, changing neither focusability, nor behavior, nor styling.

- Never disable submit buttons. Keep them enabled, validate on submit and focus the first error.
- A natively `disabled` control suppresses pointer events and leaves the tab order, so a tooltip on it never opens for keyboard or touch users and is unreliable for mouse users. Put the reason in visible text beside the control, or switch to `aria-disabled="true"`, which keeps it focusable and hoverable and can carry a tooltip.
- Use `aria-disabled="true"` where keeping a control discoverable in the tab order is intentional, or where a custom control cannot use native `disabled`.
- With `aria-disabled="true"`, block pointer and keyboard activation in the handler, prevent form submission where applicable, add explicit styling including forced-colors support and explain nearby why the action is unavailable.
- Never set both `disabled` and `aria-disabled` on the same element.
- Disabled controls are exempt from contrast minimums. Keep them legible anyway.

## Focus rings

Style `:focus-visible`, not bare `:focus`. The browser shows it for keyboard and assistive-tech focus and suppresses it for mouse clicks, where focus is already obvious. Never write `outline: none` or `focus:outline-none` without a visible replacement, which removes keyboard navigation for sighted keyboard users.

Prefer the browser's unmodified focus indicator, which adapts to platform and forced-color settings without the author predicting every background. Adding only `outline-offset` preserves it. A custom `outline: 2px solid` with no color renders `currentColor`, which is not automatically accessible, because the outline may cross colors unlike the text's own background. The preference order:

```css
/* Best: keep the browser ring, just give it breathing room */
:focus-visible {
  outline-offset: 2px;
}

/* Custom ring when the design requires one: use the project's verified token */
:focus-visible {
  outline: 2px solid var(--focus-ring);
  outline-offset: 2px;
}
```

```tsx
// Tailwind: use the project's focus token or established focus-ring utility
<button className="focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[var(--focus-ring)]">
  Save
</button>
```

A custom focus indicator must meet the applicable project or WCAG target for visible area and change of contrast. Inspect the whole perimeter against every adjacent color it crosses: component fills, page surfaces, images, gradients, hover and selected states. A token, brand color, or `currentColor` passes only when that rendered check does.

In `forced-colors: active` (Windows High Contrast), keep the default color adjustment or name a system color such as `Highlight`. `forced-color-adjust: none` freezes the authored color, so use it only where you have checked the control stays perceivable.

Group focus styles with `:focus-within` when a wrapper should light up while an inner input has focus (e.g. a search box with an icon inside the border).

## Skip link

Target `<main id="main">` and visually hide the link until focused:

```css
.skip-link {
  position: absolute;
  inset-inline-start: -999px;
}
.skip-link:focus {
  inset-inline-start: 16px;
  top: 16px;
}
```

```html
<body>
  <a class="skip-link" href="#main">Skip to content</a>
  <header>…</header>
  <main id="main">…</main>
</body>
```

Give in-page anchor targets `scroll-margin-top`, such as `80px` under a sticky header, so the target isn't hidden when jumped to.

## tabindex rules

- `tabindex="0"`: adds an element to the natural tab order. Only for custom interactive elements that aren't natively focusable.
- `tabindex="-1"`: focusable via JavaScript only (`el.focus()`). Use for headings you move focus to, modal containers and roving-tabindex members.
- Positive `tabindex`: never. It hijacks the tab order for the whole page. Fix the DOM order instead.

### Roving tabindex

Composite widgets, meaning tabs, menus, toolbars and radio groups, occupy one Tab stop. The active item has `tabindex="0"`, all others `tabindex="-1"`, and arrow keys move both focus and the `0`:

```tsx
<div role="tablist">
  {tabs.map((tab, i) => (
    <button
      role="tab"
      tabIndex={i === activeIndex ? 0 : -1}
      aria-selected={i === activeIndex}
      onKeyDown={handleArrowKeys} // ArrowLeft/ArrowRight move activeIndex, wrapping
    >
      {tab.label}
    </button>
  ))}
</div>
```

## Focus trapping and restoration

Modals must trap focus. Put `inert` on everything behind the dialog, which removes background content from the tab order and from assistive tech in one move:

```tsx
// On open
document.getElementById("app-content").inert = true;
const dialog = dialogRef.current;
(dialog.querySelector("[autofocus]") ??
  dialog.querySelector("button, [href], input, select, textarea"))?.focus();

// On close
document.getElementById("app-content").inert = false;
triggerRef.current?.focus(); // always return focus to the element that opened it
```

Prefer native `<dialog>` with `showModal()`, which gives you the trap, the `inert` background and Escape handling for free. A custom overlay that can't use it needs `role="dialog"`, `aria-modal="true"` and an accessible name via `aria-labelledby`. Either way:

- On open, focus the first focusable element. For destructive confirmations, focus the least destructive action instead.
- On close, return focus to the trigger, or to the nearest logical container if the trigger is gone.
- Add `overscroll-behavior: contain` on the dialog so scrolling inside never scrolls the page behind it.

## Keyboard patterns (ARIA APG)

Native elements come with these behaviors; custom widgets must implement them. A role is a promise. Give something `role="tab"` and users expect the full tab keyboard model.

| Widget | Keys |
| --- | --- |
| Dialog | Tab/Shift+Tab cycle inside (wrap at ends); Escape closes |
| Tabs | Arrow keys move between tabs (wrapping); Tab exits to the panel; Home/End jump to first/last |
| Menu button | Enter/Space/ArrowDown opens and focuses first item; ArrowUp opens and focuses last; arrows navigate; Escape closes and refocuses the button |
| Disclosure / accordion | Header is a `<button aria-expanded>`; Enter and Space toggle |
| Combobox | ArrowDown opens/moves into the list; Enter accepts; Escape closes and returns to the input; typing filters |
| Listbox / radio group | Arrow keys move selection; one Tab stop for the whole group |

Universal rules:

- Escape dismisses whatever opened last: tooltip, then menu, then dialog.
- Arrow keys, not Tab, move within a composite widget; Tab moves between widgets.
- Tabs choose activation mode: automatic (panel switches on arrow focus) when panels render instantly, manual (Enter/Space to activate) when switching is expensive.
- Enter submits the focused input's form. In `<textarea>`, Enter inserts a newline and ⌘/Ctrl+Enter submits.

## SPA route changes

Client-side navigation doesn't reset focus or announce anything. On route change, update `document.title` to match the new context, then move focus to the new view's `<h1>` (given `tabindex="-1"`) or to `<main>`. Restore scroll position on back and forward navigation, and scroll to top on forward navigation.

## Labels

Every control needs a programmatic label: `<label for>` pointing at the input's `id`, or a wrapping `<label>`. A placeholder is never a label. It disappears the moment the user types and usually fails contrast.

```html
<!-- Good: explicit association -->
<label for="email">Email</label>
<input id="email" type="email" autocomplete="email" />

<!-- Good: wrapping label, so label and control share one hit target -->
<label>
  <input type="checkbox" /> Send me updates
</label>
```

Label and control share one hit target, so clicking "Send me updates" toggles the checkbox with no dead zone between them. Mark required fields with native `required` plus a visible indicator explained once per form ("* required").

A placeholder used *in addition to* a label shows an example of the expected format: `placeholder="name@company.com"`.

## Error messaging

The complete pattern:

```html
<label for="email">Email</label>
<input
  id="email"
  type="email"
  autocomplete="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>
<p id="email-error">Enter a valid email address.</p>
```

- `aria-invalid="true"` on the failing field, removed once fixed.
- `aria-describedby` links the field to its inline error so screen readers announce it with the field.
- Errors render inline beside their fields, with an icon or text. Never a red border alone, which is a color-only cue.
- On submit, focus the first invalid field.
- Allow incomplete submission so validation can surface. Never disable submit until valid (see below).
- Accept free text and validate after. Never block typing or filter characters as the user types. Trim values before validating, because autocomplete and text expansion add trailing spaces.

## Autocomplete and input types

`autocomplete` with a meaningful `name` fills forms in one tap and is a WCAG requirement (1.3.5) for fields about the user. The common tokens:

| Field | `autocomplete` |
| --- | --- |
| Name | `name` (or `given-name` / `family-name`) |
| Email | `email` |
| Phone | `tel` |
| Address | `street-address`, `address-line1`, `postal-code`, `country` |
| Card | `cc-number`, `cc-exp`, `cc-csc`, `cc-name` |
| Login | `username`, `current-password` |
| Signup / reset | `new-password` |
| 2FA code | `one-time-code` |

Prefix with a section where relevant: `autocomplete="shipping street-address"`.

Correct `type` and `inputmode` pick the right mobile keyboard:

| Input | Use |
| --- | --- |
| Email, URL, phone | `type="email"`, `type="url"`, `type="tel"` |
| OTP / PIN / card number | `type="text" inputmode="numeric"` (keeps text semantics, no spinner) |
| Money, decimals | `type="text" inputmode="decimal"` |
| True numeric quantity | `type="number"` |

Disable spellcheck on emails, codes and usernames: `spellcheck="false"`.

## Never fight the user's tools

- Stay compatible with password managers and 2FA autofill: real `<form>`, correct `autocomplete`, no fake inputs.

## Submit behavior

- Keep submit enabled until the request starts, then disable it and show a spinner *beside the original label*. "Save" with a spinner, not a bare spinner. The label is what tells assistive tech which button is busy.
- Announce results. Success goes through a polite live region. On failure, focus the first invalid field, which is itself the announcement. Reserve `role="alert"` for form-level errors not tied to a field.
- Warn on unsaved changes before navigation, and never lose typed input to a re-render. Hydration must preserve focus and value.
- Enter submits from any focused input; in `<textarea>`, ⌘/Ctrl+Enter submits.

## Target sizes

Separate the conformance baseline from larger usability targets:

| Standard | Minimum |
| --- | --- |
| WCAG 2.5.8 (AA) | 24×24px, the hard floor |
| WCAG 2.5.5 (AAA) | 44×44px |
| Apple HIG | 44×44pt |
| Material Design | 48×48dp |

WCAG 2.5.8 Level AA requires a 24×24 CSS-pixel target or one of its exceptions. Treat 44px as the recommended touch target for primary controls and 40px as a useful desktop target where density permits. Smaller controls are not automatic failures. Check the spacing, equivalent-control, inline, user-agent and essential exceptions before reporting one.

Under the spacing exception, an undersized target passes when a 24px circle centered on its bounding box intersects no other target and no other undersized target's circle. In the simple case, 20px targets need a 4px gap.

The visible element can stay small; the hit area is what must be big. Anything that looks clickable must be clickable across its whole visual extent, with no dead zones. A checkbox and its label share one hit target.

## Expanding the hit area

Where the visible element is smaller, say a 20×20 checkbox, extend the hit area with a pseudo-element. Put it on the wrapping `<label>` or `<button>`, never on the `<input>`, because replaced elements don't render `::before`/`::after` reliably.

### CSS example

```css
/* Small checkbox with expanded 44px hit area, on the wrapping label */
.checkbox-label {
  position: relative;
  width: 20px;
  height: 20px;
}

.checkbox-label::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%; /* physical centering: direction-independent */
  transform: translate(-50%, -50%);
  width: 44px;
  height: 44px;
}
```

### Tailwind example

```tsx
<button className="relative size-5 after:absolute after:top-1/2 after:left-1/2 after:size-11 after:-translate-1/2">
  <CheckIcon />
</button>
```

### Layout alternative

Where the element can afford real box size, skip the pseudo-element and let the box be the target. That hands the browser real geometry for scrolling and gestures:

```css
.icon-button {
  min-width: 44px;
  min-height: 44px;
  display: inline-grid;
  place-items: center;
}
```

## Collision rule

Where the extended hit area overlaps another interactive element, shrink the pseudo-element to the largest size that does not collide. Two interactive elements never have overlapping hit areas.

## Decorative layers

A decorative layer painted over interactive content absorbs every pointer event its box covers: a gradient scrim, a glow, a blurred sheen, a full-bleed `::after`. The control underneath looks live and does nothing, and no hit-area sizing fixes it.

Give each one `pointer-events: none` (Tailwind `pointer-events-none`) so events reach the control below, plus `aria-hidden="true"` to keep it out of the accessibility tree:

```css
.card-glow {
  position: absolute;
  inset: 0;
  pointer-events: none;
}
```

Keep pointer events on any layer the user is meant to hit: a modal scrim that dismisses on click is a control, not decoration.

## Touch behavior

- Add `touch-action: manipulation` to interactive elements to remove the double-tap-to-zoom delay on mobile.
- Set `touch-action: none` on a surface implementing its own pan, zoom, or drag gestures, so the browser stops claiming them for scrolling and pinch-zoom. Scope it to that surface; at page level it removes scrolling.
- Set `-webkit-tap-highlight-color` to match the design instead of the default gray flash.
- Put hover-only styling behind `@media (hover: hover)`. On touch, `:hover` latches after a tap and holds until the user taps elsewhere, so it reads as a stuck selected state. Tailwind 4's `hover:` variant already compiles under this query.
- Prefer generous targets and clear affordances over finicky interactions such as tiny drag handles and precise hover zones.

## prefers-reduced-motion

Make motion opt-in. Wrap animations in `@media (prefers-reduced-motion: no-preference)` so users who asked for reduced motion get the static version by default, rather than you chasing every animation with an override.

```css
/* Good: motion is opt-in */
.card {
  /* static styles */
}
@media (prefers-reduced-motion: no-preference) {
  .card {
    transition: transform 200ms ease-out;
  }
}
```

```tsx
// Tailwind: motion-safe / motion-reduce variants
<div className="motion-safe:transition-transform motion-safe:hover:-translate-y-1" />
```

Where opt-in isn't feasible, give each animation its own reduced alternative. [audit.md](audit.md) flags a global duration kill, because it destroys useful feedback along with the movement.

### What to disable vs reduce

Reduced motion means reduced, not eliminated. It targets vestibular triggers, not feedback.

| Disable entirely | Replace | Keep |
| --- | --- | --- |
| Parallax scrolling | Slide/scale/zoom transitions → opacity crossfade | Loading spinners and progress |
| Autoplaying video, GIFs, looping decoration | Smooth scrolling → instant jump | Instant state changes (hover color, focus ring) |
| Spinning, large-scale movement across the screen | Auto-rotating carousels → start paused | Brief functional feedback (button press) |

Animations are interruptible and driven by user input. Nothing autoplays or refuses to stop, and under reduced motion carousels start paused.

## Autoplay and timed UI

Motion the user didn't ask for and UI acting on its own schedule:

- **No autoplaying media without visible controls** (WCAG 2.2.2). Anything moving, blinking, or updating on its own for more than 5 seconds needs a visible pause or stop control, muted looping hero videos included.
- **Prefer explicit dismissal over timers.** Auto-dismissal suits low-stakes confirmations and nothing else. A toast carrying an action, an error, or information the user may need stays until dismissed. Where one must time out, 5 seconds is the floor, and hovering or focusing it pauses the timer.
- **Never put critical information only in a timed element.** A vanished toast with the only link to an undo action is data loss on a schedule.

## Zoom and reflow

- **200% zoom** (WCAG 1.4.4). All content and functionality survives text scaled to 200%, and the viewport leaves the reader able to zoom.
- **Reflow at 320px** (WCAG 1.4.10). At 400% zoom on a 1280px viewport, equivalent to a 320px one, the page must work with vertical scrolling alone. Genuinely 2D content is the exception: tables, maps and code blocks scroll inside their own container.

Fixed heights are what break under zoom. Use `min-height` on anything containing text and let containers grow.

### rem vs px

Respect how the codebase is set up. Where the project sizes in `px`, or on an established Tailwind scale, stay consistent and never introduce mixed units into someone else's system. Where you do have the choice, in new code or a codebase already on `rem`, `rem` respects the user's base font size and `px` ignores it:

| Use `rem` | Use `px` |
| --- | --- |
| `font-size` | Borders and hairlines |
| `max-width` of text containers | Focus outline width and offset |
| Media-query breakpoints (`@media (min-width: 48rem)`) | `box-shadow` details |
| Spacing that should scale with text | Fixed-size decorations |

Breakpoints are where the choice matters most. At a larger base font size an `em` or `rem` query switches to the mobile layout when the text needs it, and a `px` query does not.

## Visually hidden content

The canonical `.sr-only` pattern hides content visually while keeping it in the accessibility tree:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
```

Use `1px` boxes, not `0`, because some screen readers skip zero-sized elements. `white-space: nowrap` stops words being read as one run-together string. Never `display: none` or `visibility: hidden`, which remove the content from assistive tech entirely.

Tailwind ships this as `sr-only`. Skip links add a focus variant that un-hides it, `focus:not-sr-only` or an override on `:focus`.

Use it for context sighted users get visually: `<span class="sr-only">Opens in new tab</span>`, table captions, or an icon-only control's label where `aria-label` isn't an option.

## Choosing how to announce a change

Work down this list and stop at the first match:

1. **Focus moves there anyway**, as with an opened modal or the first invalid field. Nothing extra needed; the focus move is the announcement.
2. **Tied to a specific control**, such as a field error or character count: `aria-describedby` on the control, announced with the field.
3. **Non-urgent, not tied to a control**, such as a toast, "Saved", a result count, or a loading state: a polite live region, `role="status"`.
4. **Urgent and not tied to a control**, such as a form-level failure or session expiry: `role="alert"`.

## Live regions

Live regions announce content that changes without a page load: toasts, validation, search-result counts, loading states.

| Mechanism | Politeness | Use for |
| --- | --- | --- |
| `role="status"` (= `aria-live="polite"` + `aria-atomic="true"`) | Waits for a pause | Toasts, "Saved", result counts, loading updates |
| `role="alert"` (= `aria-live="assertive"` + `aria-atomic="true"`) | Interrupts immediately | Errors and urgent problems only |

Rules for reliable announcements:

- For repeated polite updates, keep a stable empty region in the DOM before changing its text. Inserting a new polite region with its content is announced inconsistently.
- Dynamically inserted `role="alert"` content is usually announced, but behavior varies. Use it only for urgent errors not tied to a control, and test the target browser and screen-reader combinations.
- Default to polite. Overusing `assertive` is the most common live-region mistake, because it interrupts whatever the user was reading.
- Keep messages short and self-contained. `aria-atomic="true"` re-reads the whole region on change.
- Never move focus to a toast. Announce it and leave focus where the user is working. Give toasts a generous timeout or a dismiss button, and never put the only path to an action inside an auto-dismissing one.

```tsx
// Region rendered from the start, message injected later
<div role="status" className="sr-only">
  {statusMessage}
</div>
```

For loading states: set `aria-busy="true"` on the updating region, announce "Loading…" politely, then announce the outcome ("Loaded, 12 results").

## aria-hidden

`aria-hidden="true"` removes an element and its whole subtree from assistive tech. Use it for decorative icons and content duplicated for visual effect. Never put it on or above a focusable element, which creates stops you can Tab to that do not exist for a screen reader. Hiding something interactive means removing it from the tab order too.

## Alt text

Choose by purpose, not by what the image looks like:

| Purpose | Alt | Example |
| --- | --- | --- |
| Decorative, or redundant with adjacent text | `alt=""` (empty, but present) | Logo next to the company name in text |
| Informative | Describe the meaning it adds | `alt="Ticket QR code"` |
| Functional (image is the link/button) | Describe the action or destination | Search icon → `alt="Search"`, not `alt="magnifying glass"` |
| Image of text | The exact text (better: use real text) | `alt="50% off everything"` |
| Complex (chart, diagram) | Short summary in `alt`, full data as a table or text nearby | `alt="Revenue by quarter, described below"` |

A missing `alt` is worse than an empty one, because screen readers fall back to reading the file name.

## SVG

- Decorative SVG: `aria-hidden="true"` and `focusable="false"`, the latter for legacy Edge and IE tabbing. No title needed.
- Meaningful inline SVG: `role="img"` plus `aria-label="…"`, or a `<title>` as the first child referenced by `aria-labelledby`.
- Simple cases: `<img src="icon.svg" alt="…">` is the most reliable delivery.

```tsx
// Decorative icon inside a labeled button
<button aria-label="Close">
  <svg aria-hidden="true" focusable="false">…</svg>
</button>

// Standalone meaningful icon
<svg role="img" aria-label="Verified account">…</svg>
```

## Video and audio

Prerecorded video needs captions; provide transcripts for audio. Never autoplay with sound, and always render controls.

## Before you finish

| Mistake | Fix |
| --- | --- |
| Custom focus color assumed to work everywhere | Verify it against every adjacent color and in forced-colors mode |
| Repeated polite update inconsistently announced | Keep a stable empty status region and update its text |
| `assertive` live region for a routine toast | Use `polite`; reserve `assertive` for errors |
| `aria-hidden="true"` on a focusable element | Remove it or make the element non-focusable |
| Submit disabled until the form is valid | Keep it enabled; validate on submit and focus the first error |
| Hover treatment stuck after a tap on touch | Gate hover styling with `@media (hover: hover)` |
| Tooltip on a natively `disabled` control | Text beside it, or `aria-disabled` so it stays focusable |

## Checks

Without a browser: accessible names on every interactive element, keyboard handlers on non-native controls, focus styles, `prefers-reduced-motion` guards and form labels bound to their inputs. With one: tab the flow in order, read computed names and roles from the accessibility tree, confirm a visible focus indicator at every stop and run an automated audit. Report every check you could not run as `Not verified`.
