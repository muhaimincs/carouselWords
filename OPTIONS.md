# carouselWords Options

**Version:** v1.0.0

Options are passed as the second argument to `createCarousel(root, options, plugins)`.
All options are optional. The full `Options` type is:

```typescript
type Options = {
  align: 'start' | 'center' | 'end' | number
  axis: 'x' | 'y'
  direction: 'ltr' | 'rtl'
  containScroll: 'trimSnaps' | 'keepSnaps' | false
  slidesToScroll: number | 'auto'
  dragFree: boolean
  dragThreshold: number
  draggable: boolean
  loop: boolean
  skipSnaps: boolean
  duration: number
  startSnap: number
  container: string | HTMLElement | null
  slides: string | HTMLElement[] | NodeListOf<HTMLElement> | null
  inViewThreshold: number | number[]
  inViewMargin: string
  breakpoints: { [mediaQuery: string]: Partial<Omit<Options, 'breakpoints'>> }
  active: boolean
  resize: boolean
  focus: boolean
  slideChanges: boolean
  ariaLabel: string
}
```

---

## Cross-Cutting Rules

1. **`breakpoints` inherits** — any option set under a `breakpoints` key overrides
   the root-level option when that media query matches. `breakpoints` cannot be nested
   inside another `breakpoints` value.

2. **Unknown options are silently ignored** — `createCarousel(root, { unknownOption: true })`
   must not throw. Unrecognized keys are discarded at init time.

---

## Layout

### align
- **Type**: `'start' | 'center' | 'end' | number`
- **Default**: `'center'`
- **Description**: Where the selected slide is positioned within the viewport.
  `'start'` aligns the slide's leading edge to the viewport's leading edge.
  `'center'` centers it. `'end'` aligns the trailing edge. A number (0–1) sets
  a proportional offset from the leading edge.
- **Constraints**: Numbers are clamped to the range [0, 1].
- **Interacts with**: `direction`, `axis`

### axis
- **Type**: `'x' | 'y'`
- **Default**: `'x'`
- **Description**: The scroll axis. `'x'` is horizontal; `'y'` is vertical.
  Changes the layout direction of the container and the direction drag is detected.
- **Constraints**: Values other than `'x'` or `'y'` are treated as `'x'`.
- **Interacts with**: `direction` (direction is ignored when axis is `'y'`), `align`

### direction
- **Type**: `'ltr' | 'rtl'`
- **Default**: `'ltr'`
- **Description**: Text direction. `'rtl'` reverses scroll so slides advance
  right-to-left.
- **Constraints**: Ignored when `axis: 'y'`.
- **Interacts with**: `axis`

### containScroll
- **Type**: `'trimSnaps' | 'keepSnaps' | false`
- **Default**: `'trimSnaps'`
- **Description**: Controls whether the carousel constrains scrolling to avoid
  whitespace at the edges. `'trimSnaps'` removes snap points that would cause empty
  space. `'keepSnaps'` retains all snap points. `false` disables containment.
- **Constraints**: When `loop: true`, containment is overridden to `false` at runtime regardless of this option's value.
- **Interacts with**: `loop` — containment is automatically disabled when `loop: true`.

### slidesToScroll
- **Type**: `number | 'auto'`
- **Default**: `1`
- **Description**: Number of slides to advance per `scrollNext()` or `scrollPrev()`
  call. `'auto'` advances by the number of fully visible slides.
- **Constraints**: Numbers must be positive integers. Values less than 1 default to 1.
- **Interacts with**: `align`, `containScroll`

---

## Interaction

### dragFree
- **Type**: `boolean`
- **Default**: `false`
- **Description**: When `true`, the carousel continues scrolling with momentum after
  a drag ends — no snapping. When `false`, it snaps to the nearest slide after release.
- **Constraints**: None.
- **Interacts with**: `skipSnaps`, `duration`, `draggable` — has no effect when `draggable: false`

### dragThreshold
- **Type**: `number`
- **Default**: `10`
- **Description**: Minimum pointer travel in pixels before a drag gesture is
  recognized. Lower values increase sensitivity.
- **Constraints**: Negative values default to `10`.
- **Interacts with**: `draggable`

### draggable
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Enables or disables drag-to-scroll. When `false`, all pointer
  events are ignored.
- **Constraints**: None.
- **Interacts with**: `dragThreshold`, `dragFree`

### loop
- **Type**: `boolean`
- **Default**: `false`
- **Description**: When `true`, the carousel wraps from the last slide to the first
  on `scrollNext()`, and from the first to the last on `scrollPrev()`. When `false`,
  scrolling stops at the boundaries.
- **Constraints**: None.
- **Interacts with**: `containScroll` — containment is disabled when `loop: true`.

### skipSnaps
- **Type**: `boolean`
- **Default**: `false`
- **Description**: When `true`, a fast drag can skip over snap points and land
  further than one slide away. When `false`, each drag advances by exactly one snap.
- **Constraints**: Only meaningful when `dragFree: false`.
- **Interacts with**: `dragFree`

---

## Timing

### duration
- **Type**: `number`
- **Default**: `25`
- **Description**: Controls scroll animation speed. This is an abstract unit, not
  milliseconds. Higher values produce slower animations. The actual time varies with
  the distance scrolled.
- **Constraints**: Values ≤ 0 default to `25`.
- **Interacts with**: `dragFree`

### startSnap
- **Type**: `number`
- **Default**: `0`
- **Description**: Zero-based index of the slide the carousel starts on.
- **Constraints**: Values below 0 are clamped to 0. Values above the last valid
  index are clamped to the last valid index.
- **Interacts with**: `loop` (affects whether index wraps), `slidesToScroll` (affects valid snap indices), `breakpoints` (a breakpoint may change the slide count, re-triggering clamping)

---

## Advanced

### container
- **Type**: `string | HTMLElement | null`
- **Default**: `null`
- **Description**: Selector string or element reference to use as the slides
  container, instead of the first child of the viewport. Useful when the container
  is not a direct child of the root.
- **Constraints**: String values must be valid CSS selectors. If the selector
  matches nothing, falls back to the first child of the root.
- **Interacts with**: `slides`

### slides
- **Type**: `string | HTMLElement[] | NodeListOf<HTMLElement> | null`
- **Default**: `null`
- **Description**: Selector or element list to use as the slides, instead of the
  direct children of the container.
- **Constraints**: String values must be valid CSS selectors scoped within the
  container.
- **Interacts with**: `container`

### inViewThreshold
- **Type**: `number | number[]`
- **Default**: `0`
- **Description**: Intersection threshold for determining which slides are "in view".
  `0` means any pixel is visible; `1` means the entire slide must be visible.
  An array provides per-slide thresholds.
- **Constraints**: Values must be between 0 and 1 inclusive.
- **Interacts with**: `inViewMargin`, `slidesInView` event (see EVENTS.md)

### inViewMargin
- **Type**: `string`
- **Default**: `'0px'`
- **Description**: CSS margin shorthand applied to the viewport when computing
  in-view state (e.g. `'0px 10px'`). Positive values extend the effective viewport;
  negative values shrink it.
- **Constraints**: Must be a valid CSS margin string.
- **Interacts with**: `inViewThreshold`

### breakpoints
- **Type**: `{ [mediaQuery: string]: Partial<Omit<Options, 'breakpoints'>> }`
- **Default**: `{}`
- **Description**: Media-query-keyed option overrides. When a media query matches,
  its options override the root-level options. Example:
  `{ '(max-width: 768px)': { slidesToScroll: 1, loop: false } }`.
- **Constraints**: Keys must be valid CSS media query strings. Values must not
  include a `breakpoints` key (no nesting).
- **Interacts with**: All options.

### active
- **Type**: `boolean`
- **Default**: `true`
- **Description**: When `false`, the carousel initializes but does not attach any
  event listeners or apply any interactive behavior. Interactive behavior is suspended.
  All API methods are no-ops.
- **Constraints**: None.
- **Interacts with**: All behavior options — all are disabled when `active: false`.

### resize
- **Type**: `boolean`
- **Default**: `true`
- **Description**: When `true`, the carousel automatically calls `reInit()` when
  the root element is resized (via ResizeObserver). When `false`, layout changes
  from resizing are ignored.
- **Constraints**: None.
- **Interacts with**: `breakpoints`, `active` — ResizeObserver is not attached when `active: false`

### focus
- **Type**: `boolean`
- **Default**: `true`
- **Description**: When `true`, the carousel listens for focus events on slides and
  fires the `slideFocus` event. When `false`, focus events are not tracked.
- **Constraints**: None.
- **Interacts with**: `slideFocus` event (see EVENTS.md)

### slideChanges
- **Type**: `boolean`
- **Default**: `true`
- **Description**: When `true`, the carousel uses a MutationObserver on the container
  and calls `reInit()` automatically when slides are added or removed. When `false`,
  DOM mutations in the container are ignored.
- **Constraints**: None.
- **Interacts with**: `reInit` event (see EVENTS.md)

---

## Accessibility

### ariaLabel
- **Type**: `string`
- **Default**: `'Carousel'`
- **Description**: The accessible name for the carousel region. Applied as `aria-label`
  on the root element. Used by screen readers to identify the carousel landmark.
  See `ACCESSIBILITY.md` for the full list of ARIA attributes the library injects.
- **Constraints**: Empty string is treated as `'Carousel'` (falls back to default).
- **Interacts with**: WCAG 2.2 accessibility contract (see ACCESSIBILITY.md)
