# carouselWords Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create the complete `carouselWords` ghost library — 7 specification files an AI agent reads to implement a production web carousel in Vanilla JS/TypeScript.

**Architecture:** Modular spec package with one file per concern. `INSTALL.md` orchestrates reading order and carries the agent prompt. `SPEC.md` establishes the shared vocabulary. `tests.yaml` is standalone and references behaviors described across all spec files.

**Tech Stack:** Markdown (spec files), YAML (test scenarios), TypeScript type notation inline in spec prose — no executable code is produced by this package.

---

## File Map

| File | Creates | Responsibility |
|---|---|---|
| `SPEC.md` | New | HTML contract, `createCarousel()` API, design principles, versioning |
| `OPTIONS.md` | New | 21 configuration options with full schema |
| `METHODS.md` | New | 19 public methods with signature, behavior, error contract |
| `EVENTS.md` | New | 9 events with trigger conditions, payload, ordering guarantees |
| `PLUGINS.md` | New | Plugin factory type, registration contract, plugin rules |
| `tests.yaml` | New | 82 scenario-based test cases |
| `INSTALL.md` | New | Agent guide, reading order, ready-to-paste prompt |

> **Note on method count:** The approved design listed 15 methods. This plan adds 4 that are required for internal consistency: `slidesInView()` and `slidesNotInView()` (needed because events with those names exist), `scrollProgress()` (needed by plugin authors for progress UI), and `emit()` (needed for plugin testing). Total: 19.

All files live at the root of `carouselWords/`.

---

### Task 0: Repository Setup

**Files:**
- Create: `carouselWords/` directory structure

- [ ] **Step 1: Initialize git repository**

```bash
cd /Users/achesohor/Desktop/carouselWords
git init
```

Expected output: `Initialized empty Git repository in .../carouselWords/.git/`

- [ ] **Step 2: Verify directory is empty except for docs/**

```bash
ls -la
```

Expected: only `.git/` and `docs/` present. No other files yet.

- [ ] **Step 3: Initial commit**

```bash
git add docs/
git commit -m "chore: initialize carouselWords repo with design spec"
```

---

### Task 1: SPEC.md

**Files:**
- Create: `SPEC.md`

- [ ] **Step 1: Create SPEC.md**

```markdown
# carouselWords Specification

**Version:** 1.0.0

`carouselWords` is a ghost library. This file, together with OPTIONS.md, METHODS.md,
EVENTS.md, PLUGINS.md, and tests.yaml, is everything an AI agent needs to implement a
production web carousel in Vanilla JS/TypeScript. No source code is distributed.
The spec is the single source of truth.

---

## 1. HTML Contract

A carousel requires three nested DOM elements. The agent must produce and manage
this structure. No CSS framework is assumed.

```html
<div class="carousel">               <!-- root: passed to createCarousel() -->
  <div class="carousel__viewport">   <!-- viewport: clips overflow -->
    <div class="carousel__container"><!-- container: holds all slides -->
      <div class="carousel__slide">Slide 1</div>
      <div class="carousel__slide">Slide 2</div>
      <div class="carousel__slide">Slide 3</div>
    </div>
  </div>
</div>
```

**Required classes:**
- Root element: `.carousel`
- Viewport element: `.carousel__viewport`
- Container element: `.carousel__container`
- Each slide: `.carousel__slide`

**Layout rules:**
- The viewport must have `overflow: hidden`.
- The container lays out slides in a single row (or column when `axis: 'y'`).
- Slides must not wrap to a second row or column.

---

## 2. Initialization API

```typescript
function createCarousel(
  root: HTMLElement,
  options?: Partial<Options>,
  plugins?: CarouselPlugin[]
): CarouselAPI
```

- `root` — the `.carousel` element. Required. Throws `TypeError` if null or not an HTMLElement.
- `options` — configuration object. All keys are optional. Unrecognized keys are silently ignored.
- `plugins` — array of plugin factories. Optional, defaults to `[]`.

Returns a `CarouselAPI` object. See METHODS.md for the full shape of `CarouselAPI`.

**Invariants:**
- No `new` keyword. `createCarousel` is a plain factory function.
- No global state. Each call returns a fully isolated instance.
- No side effects outside `root`. The function must not modify `document`, `window`,
  or any element outside `root`.

---

## 3. Design Principles

- **No global state** — two carousels on the same page must not share any state.
- **Lazy DOM reads** — measure the DOM at `init` time. Do not poll continuously.
  Exception: `resize: true` uses a ResizeObserver.
- **Deterministic behavior** — identical options plus identical DOM always produces
  identical initial state.
- **Graceful no-op** — any `CarouselAPI` method called after `destroy()` does nothing
  and returns `undefined`. No exceptions are thrown.
- **TypeScript types required** — all public surfaces (Options, CarouselAPI,
  CarouselPlugin, event handlers) must have explicit TypeScript type definitions
  exported from the library entry point.

---

## 4. Versioning Contract

This spec is at `v1.0.0`. Each spec file carries this version in its header.

- **Major version** increments on breaking changes (removed option, changed method signature).
- **Minor version** increments on additive changes (new option, new event).
- Patch versions are not used for spec changes.

An agent implementing `v1.0.0` of any spec file is not expected to handle `v2.0.0`
behavior without re-reading the updated spec.
```

- [ ] **Step 2: Verify SPEC.md**

Check each item:
- [ ] HTML structure with all 4 required elements and correct class names
- [ ] `createCarousel()` signature with all 3 params typed correctly
- [ ] `TypeError` specified for null root
- [ ] All 3 factory invariants (no new, no global state, no outside side effects)
- [ ] All 5 design principles present
- [ ] Versioning contract with major/minor rules
- [ ] No mention of `EmblaCarousel` anywhere in the file

- [ ] **Step 3: Commit**

```bash
git add SPEC.md
git commit -m "spec: add SPEC.md — HTML contract, init API, design principles"
```

---

### Task 2: OPTIONS.md

**Files:**
- Create: `OPTIONS.md`

- [ ] **Step 1: Create OPTIONS.md**

```markdown
# carouselWords Options

**Version:** 1.0.0

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
}
```

---

## Cross-Cutting Rules

1. **`breakpoints` inherits** — any option set under a `breakpoints` key overrides
   the root-level option when that media query matches. `breakpoints` cannot be nested
   inside another `breakpoints` value.

2. **Unknown options are silently ignored** — `createCarousel(root, { foo: 'bar' })`
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
- **Constraints**: None.
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
- **Interacts with**: `skipSnaps`, `duration`

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
- **Interacts with**: None.

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
- **Interacts with**: `inViewMargin`, `slidesInView` event

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
  event listeners or apply any interactive behavior. The DOM is not modified.
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
- **Interacts with**: `breakpoints`

### focus
- **Type**: `boolean`
- **Default**: `true`
- **Description**: When `true`, the carousel listens for focus events on slides and
  fires the `slideFocus` event. When `false`, focus events are not tracked.
- **Constraints**: None.
- **Interacts with**: `slideFocus` event

### slideChanges
- **Type**: `boolean`
- **Default**: `true`
- **Description**: When `true`, the carousel uses a MutationObserver on the container
  and calls `reInit()` automatically when slides are added or removed. When `false`,
  DOM mutations in the container are ignored.
- **Constraints**: None.
- **Interacts with**: `reInit` event
```

- [ ] **Step 2: Verify OPTIONS.md**

Check each item:
- [ ] `Options` TypeScript type at the top with all 21 properties
- [ ] Cross-cutting rules present before the groups
- [ ] Layout group: `align`, `axis`, `direction`, `containScroll`, `slidesToScroll` (5)
- [ ] Interaction group: `dragFree`, `dragThreshold`, `draggable`, `loop`, `skipSnaps` (5)
- [ ] Timing group: `duration`, `startSnap` (2)
- [ ] Advanced group: `container`, `slides`, `inViewThreshold`, `inViewMargin`, `breakpoints`, `active`, `resize`, `focus`, `slideChanges` (9)
- [ ] Total: 21 options
- [ ] Each option has: Type, Default, Description, Constraints, Interacts with
- [ ] `loop` + `containScroll` interaction documented in both entries

- [ ] **Step 3: Commit**

```bash
git add OPTIONS.md
git commit -m "spec: add OPTIONS.md — 21 configuration options with full schema"
```

---

### Task 3: METHODS.md

**Files:**
- Create: `METHODS.md`

> **Note:** The design doc listed 15 methods. This task implements 19. The 4 additions are required for internal consistency: `slidesInView()` and `slidesNotInView()` pair with events of the same name; `scrollProgress()` enables progress indicator plugins; `emit()` enables plugin testing. All 4 are additive and do not conflict with anything in the approved design.

- [ ] **Step 1: Create METHODS.md**

```markdown
# carouselWords Methods

**Version:** 1.0.0

`createCarousel()` returns a `CarouselAPI` object with the following methods.

```typescript
type CarouselAPI = {
  // Scroll
  scrollNext(jump?: boolean): void
  scrollPrev(jump?: boolean): void
  scrollTo(index: number, jump?: boolean): void
  // State queries
  canScrollNext(): boolean
  canScrollPrev(): boolean
  selectedScrollSnap(): number
  previousScrollSnap(): number
  scrollSnapList(): number[]
  scrollProgress(): number
  slidesInView(): number[]
  slidesNotInView(): number[]
  // DOM access
  rootNode(): HTMLElement
  containerNode(): HTMLElement
  slideNodes(): HTMLElement[]
  // Lifecycle
  reInit(options?: Partial<Options>): void
  destroy(): void
  on(event: string, handler: CarouselEventHandler): CarouselAPI
  off(event: string, handler: CarouselEventHandler): CarouselAPI
  emit(event: string): CarouselAPI
}

type CarouselEventHandler = (api: CarouselAPI, event: string) => void
```

---

## Invariants

1. **All state queries are synchronous** — they reflect the current state at the
   exact moment they are called. They are never async and never return Promises.

2. **`destroy()` is idempotent** — calling it a second time does nothing and does
   not throw.

3. **`reInit()` preserves event listeners** — handlers registered with `on()` before
   a `reInit()` call remain registered after. They do not need to be re-added.

---

## Scroll

### scrollNext()
- **Signature**: `scrollNext(jump?: boolean): void`
- **Description**: Scrolls to the next snap point. When `loop: true`, wraps from
  the last snap to the first. When `loop: false` and already at the last snap,
  does nothing.
- **Valid when**: Carousel is not destroyed. If destroyed, does nothing.
- **Returns**: `void`
- **Error contract**: No throw under any input. Out-of-bounds or no-op calls are silent.
- **`jump` param**: When `true`, teleports to the target snap with no animation.

### scrollPrev()
- **Signature**: `scrollPrev(jump?: boolean): void`
- **Description**: Scrolls to the previous snap point. When `loop: true`, wraps from
  the first snap to the last. When `loop: false` and already at the first snap,
  does nothing.
- **Valid when**: Carousel is not destroyed. If destroyed, does nothing.
- **Returns**: `void`
- **Error contract**: No throw under any input.
- **`jump` param**: When `true`, teleports to the target snap with no animation.

### scrollTo()
- **Signature**: `scrollTo(index: number, jump?: boolean): void`
- **Description**: Scrolls to the snap at the given zero-based index.
- **Valid when**: Carousel is not destroyed.
- **Returns**: `void`
- **Error contract**: Negative indices and indices beyond the last snap are no-ops.
  No throw.
- **`jump` param**: When `true`, teleports to the target snap with no animation.

---

## State Queries

### canScrollNext()
- **Signature**: `canScrollNext(): boolean`
- **Description**: Returns `true` if there is a next snap to scroll to. Always
  `true` when `loop: true` (unless there is only one slide). Returns `false` when
  already at the last snap and `loop: false`.
- **Valid when**: Any state. Returns `false` after `destroy()`.
- **Returns**: `boolean`
- **Error contract**: No throw.

### canScrollPrev()
- **Signature**: `canScrollPrev(): boolean`
- **Description**: Returns `true` if there is a previous snap to scroll to. Always
  `true` when `loop: true` (unless there is only one slide). Returns `false` when
  at the first snap and `loop: false`.
- **Valid when**: Any state. Returns `false` after `destroy()`.
- **Returns**: `boolean`
- **Error contract**: No throw.

### selectedScrollSnap()
- **Signature**: `selectedScrollSnap(): number`
- **Description**: Returns the zero-based index of the currently selected snap.
  This is the snap the carousel is on or is animating toward.
- **Valid when**: Any state. Returns `0` after `destroy()`.
- **Returns**: `number`
- **Error contract**: No throw.

### previousScrollSnap()
- **Signature**: `previousScrollSnap(): number`
- **Description**: Returns the zero-based index of the snap that was selected before
  the most recent `select` event. On init (before any navigation), returns `0`.
- **Valid when**: Any state. Returns `0` after `destroy()`.
- **Returns**: `number`
- **Error contract**: No throw.

### scrollSnapList()
- **Signature**: `scrollSnapList(): number[]`
- **Description**: Returns an array of all snap point scroll positions, each
  expressed as a value between 0 and 1 representing progress along the scroll axis.
  The array length equals the number of snaps.
- **Valid when**: Any state. Returns `[]` after `destroy()`.
- **Returns**: `number[]`
- **Error contract**: No throw.

### scrollProgress()
- **Signature**: `scrollProgress(): number`
- **Description**: Returns the current scroll position as a value between 0 and 1,
  where 0 is the start and 1 is the end. Useful for progress bar plugins.
- **Valid when**: Any state. Returns `0` after `destroy()`.
- **Returns**: `number`
- **Error contract**: No throw.

### slidesInView()
- **Signature**: `slidesInView(): number[]`
- **Description**: Returns the zero-based indices of all slides currently visible
  within the viewport. Visibility is determined by `inViewThreshold` and
  `inViewMargin`.
- **Valid when**: Any state. Returns `[]` after `destroy()`.
- **Returns**: `number[]`
- **Error contract**: No throw.

### slidesNotInView()
- **Signature**: `slidesNotInView(): number[]`
- **Description**: Returns the zero-based indices of all slides not currently visible.
  The complement of `slidesInView()`.
- **Valid when**: Any state. Returns `[]` after `destroy()`.
- **Returns**: `number[]`
- **Error contract**: No throw.

---

## DOM Access

### rootNode()
- **Signature**: `rootNode(): HTMLElement`
- **Description**: Returns the root element that was passed to `createCarousel()`.
- **Valid when**: Any state, including after `destroy()`.
- **Returns**: `HTMLElement`
- **Error contract**: No throw.

### containerNode()
- **Signature**: `containerNode(): HTMLElement`
- **Description**: Returns the `.carousel__container` element.
- **Valid when**: Any state, including after `destroy()`.
- **Returns**: `HTMLElement`
- **Error contract**: No throw.

### slideNodes()
- **Signature**: `slideNodes(): HTMLElement[]`
- **Description**: Returns an array of all `.carousel__slide` elements in DOM order.
- **Valid when**: Any state. Returns the slide elements as they existed at last
  init/reInit, even after `destroy()`.
- **Returns**: `HTMLElement[]`
- **Error contract**: No throw.

---

## Lifecycle

### reInit()
- **Signature**: `reInit(options?: Partial<Options>): void`
- **Description**: Re-initializes the carousel. If `options` are provided, they are
  merged into the current options (not replaced). The carousel re-reads the DOM,
  re-applies layout, and fires the `reInit` event. Existing event listeners are
  preserved.
- **Valid when**: Carousel is not destroyed.
- **Returns**: `void`
- **Error contract**: No throw. Calling on a destroyed carousel does nothing.

### destroy()
- **Signature**: `destroy(): void`
- **Description**: Tears down the carousel. Removes all event listeners, disconnects
  all observers, and fires the `destroy` event. All subsequent API calls become
  no-ops. Does not remove or modify the DOM elements.
- **Valid when**: Any state.
- **Returns**: `void`
- **Error contract**: Idempotent. Second call does nothing. No throw.

### on()
- **Signature**: `on(event: string, handler: CarouselEventHandler): CarouselAPI`
- **Description**: Registers a handler for the given event name. The same handler
  can be registered multiple times and will fire multiple times per event.
- **Valid when**: Any state. If called after `destroy()`, does nothing.
- **Returns**: `CarouselAPI` (for method chaining)
- **Error contract**: No throw. Unrecognized event names are accepted silently.

### off()
- **Signature**: `off(event: string, handler: CarouselEventHandler): CarouselAPI`
- **Description**: Removes a previously registered handler. If the handler was not
  registered, does nothing.
- **Valid when**: Any state.
- **Returns**: `CarouselAPI` (for method chaining)
- **Error contract**: No throw.

### emit()
- **Signature**: `emit(event: string): CarouselAPI`
- **Description**: Manually fires an event, calling all registered handlers for
  that event name. Primarily useful in plugin development and testing.
- **Valid when**: Any state. If called after `destroy()`, does nothing.
- **Returns**: `CarouselAPI` (for method chaining)
- **Error contract**: No throw. Unrecognized event names fire with no handlers called.
```

- [ ] **Step 2: Verify METHODS.md**

Check each item:
- [ ] `CarouselAPI` TypeScript type at top with all 19 methods
- [ ] `CarouselEventHandler` type defined
- [ ] All 3 invariants present at top
- [ ] Scroll group: `scrollNext`, `scrollPrev`, `scrollTo` (3)
- [ ] State queries group: `canScrollNext`, `canScrollPrev`, `selectedScrollSnap`, `previousScrollSnap`, `scrollSnapList`, `scrollProgress`, `slidesInView`, `slidesNotInView` (8)
- [ ] DOM access group: `rootNode`, `containerNode`, `slideNodes` (3)
- [ ] Lifecycle group: `reInit`, `destroy`, `on`, `off`, `emit` (5)
- [ ] Total: 19 methods
- [ ] Every method has: Signature, Description, Valid when, Returns, Error contract
- [ ] `destroy()` explicitly documented as idempotent in its own entry
- [ ] `on()` and `off()` both return `CarouselAPI` for chaining

- [ ] **Step 3: Commit**

```bash
git add METHODS.md
git commit -m "spec: add METHODS.md — 19 public methods with full schema"
```

---

### Task 4: EVENTS.md

**Files:**
- Create: `EVENTS.md`

- [ ] **Step 1: Create EVENTS.md**

```markdown
# carouselWords Events

**Version:** 1.0.0

Events are subscribed to with `api.on(eventName, handler)` and unsubscribed with
`api.off(eventName, handler)`.

All event handlers receive the same signature:

```typescript
type CarouselEventHandler = (api: CarouselAPI, event: string) => void
```

The handler receives the full `CarouselAPI` as the first argument. Call any state
query method (e.g. `api.selectedScrollSnap()`) inside the handler to read current state.

---

## Ordering Guarantees

These are the rules agents most commonly get wrong. Implement these exactly.

1. **`init` fires once**, synchronously after `createCarousel()` completes. It fires
   before any interaction-triggered event can fire.

2. **`select` fires before `settle`** — `select` fires when the target snap is
   decided (immediately on `scrollNext()`, `scrollPrev()`, or `scrollTo()` call).
   `settle` fires when the scroll animation completes.

3. **`scroll` fires continuously** during any drag or programmatic scroll animation.
   Handlers registered for `scroll` must be cheap — they may fire dozens of times
   per second.

4. **`reInit` fires after new state is applied** — any state query called inside
   a `reInit` handler returns post-reinit values, not pre-reinit values.

---

## Lifecycle Events

### init
- **Fires when**: Immediately after `createCarousel()` completes and the carousel
  is fully ready.
- **Payload**: None. Call `api.selectedScrollSnap()` etc. inside the handler.
- **Order guarantee**: Fires before any interaction event.
- **Notes**: Fires exactly once per carousel instance. Does not fire on `reInit()`.

### reInit
- **Fires when**: After `reInit()` completes and new state is applied.
- **Payload**: None.
- **Order guarantee**: Fires after all state changes from `reInit()` are applied.
- **Notes**: Does not fire on the initial `createCarousel()` call.

### destroy
- **Fires when**: When `destroy()` is called, before listeners are removed.
- **Payload**: None.
- **Order guarantee**: Fires once. After this event, no further events fire.
- **Notes**: This is the last event a handler will ever receive for this instance.

---

## Interaction Events

### select
- **Fires when**: A new snap is selected. This happens immediately when
  `scrollNext()`, `scrollPrev()`, or `scrollTo()` is called and results in a
  different snap, or when a drag gesture commits to a new snap.
- **Payload**: None. Call `api.selectedScrollSnap()` inside the handler.
- **Order guarantee**: Fires before `settle`.
- **Notes**: Does not fire if `scrollNext()` is a no-op (e.g. already at last
  slide with `loop: false`).

### scroll
- **Fires when**: Continuously during any scroll animation or drag gesture, for
  every animation frame.
- **Payload**: None. Call `api.scrollProgress()` inside the handler for position.
- **Order guarantee**: Fires many times between `select` and `settle`.
- **Notes**: Handlers must be lightweight. Avoid heavy DOM operations.

### settle
- **Fires when**: The scroll animation completes and the carousel comes to rest
  on a snap point.
- **Payload**: None. Call `api.selectedScrollSnap()` inside the handler.
- **Order guarantee**: Fires after `select` and after the last `scroll` frame.
- **Notes**: Does not fire when `jump: true` is used (no animation occurs).

### slidesInView
- **Fires when**: The set of slides visible within the viewport changes.
- **Payload**: None. Call `api.slidesInView()` inside the handler for indices.
- **Order guarantee**: No guaranteed order relative to `select` or `settle`.
- **Notes**: Also fires on `init` with the initial set of visible slides.
  Visibility is determined by `inViewThreshold` and `inViewMargin`.

### slidesNotInView
- **Fires when**: The set of slides NOT visible within the viewport changes.
- **Payload**: None. Call `api.slidesNotInView()` inside the handler for indices.
- **Order guarantee**: Fires at the same time as `slidesInView`.
- **Notes**: Also fires on `init`. The indices returned are the complement of
  `slidesInView()`.

### slideFocus
- **Fires when**: A slide element receives DOM focus (via keyboard or programmatic
  focus). Only fires when `focus: true`.
- **Payload**: None. Call `api.selectedScrollSnap()` inside the handler to get
  the focused slide's snap index.
- **Order guarantee**: No guaranteed order relative to other events.
- **Notes**: Does not fire when `focus: false`.
```

- [ ] **Step 2: Verify EVENTS.md**

Check each item:
- [ ] `CarouselEventHandler` type shown at top
- [ ] All 4 ordering guarantees present
- [ ] Lifecycle group: `init`, `reInit`, `destroy` (3)
- [ ] Interaction group: `select`, `scroll`, `settle`, `slidesInView`, `slidesNotInView`, `slideFocus` (6)
- [ ] Total: 9 events
- [ ] Each event has: Fires when, Payload, Order guarantee, Notes
- [ ] `init` explicitly says it does NOT fire on `reInit()`
- [ ] `select` explicitly says it does NOT fire for no-op scrolls
- [ ] `settle` explicitly says it does NOT fire when `jump: true`
- [ ] `slidesInView` and `slidesNotInView` note they fire on `init`

- [ ] **Step 3: Commit**

```bash
git add EVENTS.md
git commit -m "spec: add EVENTS.md — 9 events with ordering guarantees"
```

---

### Task 5: PLUGINS.md

**Files:**
- Create: `PLUGINS.md`

- [ ] **Step 1: Create PLUGINS.md**

```markdown
# carouselWords Plugin Interface

**Version:** 1.0.0

This file defines the contract for building a plugin. It does not define any
specific plugin implementations — those are out of scope for this spec.

---

## Plugin Type

A plugin is a factory function that receives the carousel API and returns a plugin
object:

```typescript
type CarouselPlugin = (api: CarouselAPI) => {
  name: string
  init: () => void
  destroy: () => void
  [key: string]: unknown  // plugins may expose additional public properties
}
```

The `CarouselAPI` type is defined in METHODS.md. The full API — all 19 methods —
is available to the plugin factory at call time.

---

## Plugin Shape

A plugin factory receives the full `CarouselAPI` when registered. It must return
an object with three required properties:

- **`name`** (`string`) — a unique identifier for this plugin. Used to detect
  duplicate registrations.
- **`init()`** — called after the carousel's own `init` completes. The plugin
  should register any event listeners and set up its behavior here.
- **`destroy()`** — called when the carousel is destroyed, before the carousel's
  own `destroy` event fires. The plugin must remove all event listeners and clean
  up all resources it created.

Plugins may return additional properties (methods, state, etc.) beyond the three
required ones. The carousel does not restrict or inspect these.

---

## Registration Contract

Plugins are passed as the third argument to `createCarousel()`:

```typescript
createCarousel(root, options, [pluginA, pluginB])
```

**Initialization order:**
1. The carousel runs its own init.
2. Plugin `init()` functions are called in registration order (`pluginA` first,
   then `pluginB`).
3. The carousel fires the `init` event.

**Destruction order:**
1. The carousel fires the `destroy` event.
2. Plugin `destroy()` functions are called in **reverse** registration order
   (`pluginB` first, then `pluginA`).
3. The carousel tears down its own listeners.

---

## Plugin Rules

1. **Must not mutate options** — a plugin must not modify the `options` object
   passed to `createCarousel()`. Options are owned by the carousel.

2. **Must clean up in `destroy()`** — every event listener registered via
   `api.on()` inside `init()` must be removed via `api.off()` inside `destroy()`.
   Failing to clean up causes memory leaks.

3. **Duplicate `name` — last wins** — if two plugins share the same `name`, the
   second plugin's `init()` is called; the first plugin's `init()` is not. No
   error is thrown.

4. **May expose a public API** — a plugin may return additional properties (methods,
   getters, configuration) alongside `name`, `init`, and `destroy`. Consumers access
   the plugin's public API through the object returned by the factory, not through
   the `CarouselAPI`.

---

## Example Plugin Structure

```typescript
function myPlugin(api: CarouselAPI) {
  function handleSelect(api: CarouselAPI) {
    // react to slide change
  }

  return {
    name: 'myPlugin',
    init() {
      api.on('select', handleSelect)
    },
    destroy() {
      api.off('select', handleSelect)
    },
  }
}

// Usage:
const carousel = createCarousel(root, {}, [myPlugin])
```
```

- [ ] **Step 2: Verify PLUGINS.md**

Check each item:
- [ ] `CarouselPlugin` TypeScript type with `name`, `init`, `destroy`, and index signature
- [ ] Reference to `CarouselAPI` being defined in METHODS.md
- [ ] Plugin shape section explains all 3 required properties
- [ ] Registration contract covers init order (registration order) and destroy order (reverse)
- [ ] All 4 plugin rules present
- [ ] Example plugin structure at the end using `on()` and `off()`
- [ ] No specific plugin implementations (Autoplay, Fade, etc.) — those are out of scope

- [ ] **Step 3: Commit**

```bash
git add PLUGINS.md
git commit -m "spec: add PLUGINS.md — plugin interface contract and registration rules"
```

---

### Task 6: tests.yaml (Scenarios 1–42)

**Files:**
- Create: `tests.yaml`

- [ ] **Step 1: Create tests.yaml with first 42 scenarios**

```yaml
version: 1.0.0

# Scenario format:
# name: unique readable description
# setup: initial state (slides count, options, start position)
# action: what to do (method call, event trigger)
# expect: state after action and/or event fired

scenarios:

  # --- Initialization ---

  - name: "Initializes with minimum required markup"
    setup:
      slides: 3
      options: {}
    action: "call createCarousel(root)"
    expect:
      no_error: true
      selectedScrollSnap: 0

  - name: "All options are optional — no options argument"
    setup:
      slides: 3
    action: "call createCarousel(root) with no options argument"
    expect:
      no_error: true
      selectedScrollSnap: 0

  - name: "Unknown option is silently ignored"
    setup:
      slides: 3
      options: { unknownOption: true }
    action: "call createCarousel(root, { unknownOption: true })"
    expect:
      no_error: true

  - name: "Null root throws TypeError"
    setup:
      slides: 3
    action: "call createCarousel(null)"
    expect:
      throws: TypeError

  - name: "Two carousels on same page are isolated"
    setup:
      carousels: 2
      slides_each: 3
      options_each: { startSnap: 0 }
    action: "call scrollNext() on carousel 1"
    expect:
      carousel_1_selectedScrollSnap: 1
      carousel_2_selectedScrollSnap: 0

  # --- scrollNext ---

  - name: "scrollNext advances from first slide"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 1
      event_fired: select

  - name: "scrollNext at last slide with loop:false is no-op"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 4
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 4
      event_fired: null

  - name: "scrollNext at last slide with loop:true wraps to first"
    setup:
      slides: 5
      options: { loop: true }
      startSnap: 4
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 0
      event_fired: select

  - name: "scrollNext with jump:true changes snap without animation"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call scrollNext(true)"
    expect:
      selectedScrollSnap: 1
      settle_fired: false

  # --- scrollPrev ---

  - name: "scrollPrev retreats from last slide"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 4
    action: "call scrollPrev()"
    expect:
      selectedScrollSnap: 3
      event_fired: select

  - name: "scrollPrev at first slide with loop:false is no-op"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call scrollPrev()"
    expect:
      selectedScrollSnap: 0
      event_fired: null

  - name: "scrollPrev at first slide with loop:true wraps to last"
    setup:
      slides: 5
      options: { loop: true }
      startSnap: 0
    action: "call scrollPrev()"
    expect:
      selectedScrollSnap: 4
      event_fired: select

  - name: "scrollPrev with jump:true changes snap without animation"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 4
    action: "call scrollPrev(true)"
    expect:
      selectedScrollSnap: 3
      settle_fired: false

  # --- scrollTo ---

  - name: "scrollTo(2) navigates directly to slide 2"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "call scrollTo(2)"
    expect:
      selectedScrollSnap: 2
      event_fired: select

  - name: "scrollTo(0) from middle returns to first"
    setup:
      slides: 5
      options: {}
      startSnap: 3
    action: "call scrollTo(0)"
    expect:
      selectedScrollSnap: 0
      event_fired: select

  - name: "scrollTo with negative index is no-op"
    setup:
      slides: 5
      options: {}
      startSnap: 2
    action: "call scrollTo(-1)"
    expect:
      selectedScrollSnap: 2
      event_fired: null

  - name: "scrollTo with out-of-range index is no-op"
    setup:
      slides: 3
      options: {}
      startSnap: 1
    action: "call scrollTo(10)"
    expect:
      selectedScrollSnap: 1
      event_fired: null

  - name: "scrollTo with jump:true skips animation"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "call scrollTo(3, true)"
    expect:
      selectedScrollSnap: 3
      settle_fired: false

  # --- canScrollNext / canScrollPrev ---

  - name: "canScrollNext returns true when not at last slide"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call canScrollNext()"
    expect:
      result: true

  - name: "canScrollNext returns false at last slide with loop:false"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 4
    action: "call canScrollNext()"
    expect:
      result: false

  - name: "canScrollNext returns true at last slide with loop:true"
    setup:
      slides: 5
      options: { loop: true }
      startSnap: 4
    action: "call canScrollNext()"
    expect:
      result: true

  - name: "canScrollNext returns false with one slide and loop:false"
    setup:
      slides: 1
      options: { loop: false }
    action: "call canScrollNext()"
    expect:
      result: false

  - name: "canScrollPrev returns false at first slide with loop:false"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call canScrollPrev()"
    expect:
      result: false

  - name: "canScrollPrev returns true at first slide with loop:true"
    setup:
      slides: 5
      options: { loop: true }
      startSnap: 0
    action: "call canScrollPrev()"
    expect:
      result: true

  - name: "canScrollPrev returns true when not at first slide"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 2
    action: "call canScrollPrev()"
    expect:
      result: true

  - name: "canScrollPrev returns false with one slide and loop:false"
    setup:
      slides: 1
      options: { loop: false }
    action: "call canScrollPrev()"
    expect:
      result: false

  # --- selectedScrollSnap / previousScrollSnap / scrollSnapList ---

  - name: "selectedScrollSnap returns startSnap on init"
    setup:
      slides: 5
      options: { startSnap: 2 }
    action: "call selectedScrollSnap() immediately after init"
    expect:
      result: 2

  - name: "selectedScrollSnap returns new snap after scrollNext"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "call scrollNext(), then call selectedScrollSnap()"
    expect:
      result: 1

  - name: "previousScrollSnap returns 0 before any navigation"
    setup:
      slides: 5
      options: {}
    action: "call previousScrollSnap() immediately after init"
    expect:
      result: 0

  - name: "previousScrollSnap returns prior snap after scrollNext"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "call scrollNext(), then call previousScrollSnap()"
    expect:
      result: 0

  - name: "scrollSnapList length equals slide count with default options"
    setup:
      slides: 4
      options: {}
    action: "call scrollSnapList()"
    expect:
      result_length: 4

  # --- DOM access ---

  - name: "rootNode returns the element passed to createCarousel"
    setup:
      slides: 3
    action: "call rootNode()"
    expect:
      result: "the root HTMLElement passed to createCarousel"

  - name: "containerNode returns the .carousel__container element"
    setup:
      slides: 3
    action: "call containerNode()"
    expect:
      result: "the .carousel__container HTMLElement"

  - name: "slideNodes returns all .carousel__slide elements"
    setup:
      slides: 3
    action: "call slideNodes()"
    expect:
      result_length: 3
      result_type: "array of HTMLElement"

  # --- reInit ---

  - name: "reInit re-reads DOM and reapplies layout"
    setup:
      slides: 3
      options: { loop: false }
      startSnap: 0
    action: "call reInit()"
    expect:
      no_error: true
      selectedScrollSnap: 0

  - name: "reInit with new options merges options"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 4
    action: "call reInit({ loop: true }), then call canScrollNext()"
    expect:
      result: true

  - name: "reInit preserves existing event listeners"
    setup:
      slides: 3
      options: {}
    action: "register a 'select' handler, call reInit(), then call scrollNext()"
    expect:
      handler_called: true

  - name: "reInit fires the reInit event"
    setup:
      slides: 3
      options: {}
    action: "register a 'reInit' handler, call reInit()"
    expect:
      handler_called: true

  # --- destroy ---

  - name: "After destroy, scrollNext is a no-op"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "call destroy(), then call scrollNext()"
    expect:
      no_error: true
      selectedScrollSnap: 0

  - name: "destroy is idempotent — calling twice does not throw"
    setup:
      slides: 3
      options: {}
    action: "call destroy() twice"
    expect:
      no_error: true

  - name: "After destroy, canScrollNext returns false"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call destroy(), then call canScrollNext()"
    expect:
      result: false

  - name: "destroy fires the destroy event"
    setup:
      slides: 3
      options: {}
    action: "register a 'destroy' handler, call destroy()"
    expect:
      handler_called: true
```

- [ ] **Step 2: Verify scenarios 1–42**

Check each item:
- [ ] 42 scenarios present with no duplicated names
- [ ] Every scenario has: `name`, `setup`, `action`, `expect`
- [ ] Initialization: 5 scenarios covering no-args, unknown options, null root, isolation
- [ ] scrollNext: 4 scenarios (advance, no-op at end, loop wrap, jump)
- [ ] scrollPrev: 4 scenarios (retreat, no-op at start, loop wrap, jump)
- [ ] scrollTo: 5 scenarios (direct nav, return, negative, out-of-range, jump)
- [ ] canScrollNext / canScrollPrev: 8 scenarios
- [ ] State queries: 5 scenarios
- [ ] DOM access: 3 scenarios
- [ ] reInit: 4 scenarios
- [ ] destroy: 4 scenarios

- [ ] **Step 3: Commit**

```bash
git add tests.yaml
git commit -m "spec: add tests.yaml — scenarios 1-42 (init, scroll, state, lifecycle)"
```

---

### Task 7: tests.yaml (Scenarios 43–82)

**Files:**
- Modify: `tests.yaml` (append to `scenarios:` list)

- [ ] **Step 1: Append scenarios 43–82 to tests.yaml**

```yaml
  # --- Options: startSnap ---

  - name: "startSnap:2 initializes at slide 2"
    setup:
      slides: 5
      options: { startSnap: 2 }
    action: "call selectedScrollSnap() after init"
    expect:
      result: 2

  - name: "startSnap out of range is clamped to last valid index"
    setup:
      slides: 3
      options: { startSnap: 10 }
    action: "call selectedScrollSnap() after init"
    expect:
      result: 2

  - name: "startSnap negative is clamped to 0"
    setup:
      slides: 3
      options: { startSnap: -1 }
    action: "call selectedScrollSnap() after init"
    expect:
      result: 0

  # --- Options: loop ---

  - name: "loop:true — scrollNext wraps last to first"
    setup:
      slides: 3
      options: { loop: true }
      startSnap: 2
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 0

  - name: "loop:false — scrollNext stops at last"
    setup:
      slides: 3
      options: { loop: false }
      startSnap: 2
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 2

  # --- Options: slidesToScroll ---

  - name: "slidesToScroll:2 advances two slides per scrollNext"
    setup:
      slides: 6
      options: { slidesToScroll: 2 }
      startSnap: 0
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 2

  - name: "slidesToScroll:2 near end does not overshoot"
    setup:
      slides: 5
      options: { slidesToScroll: 2, loop: false }
      startSnap: 3
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 4

  # --- Options: active ---

  - name: "active:false — scrollNext does nothing"
    setup:
      slides: 5
      options: { active: false }
      startSnap: 0
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 0
      event_fired: null

  - name: "active:false — carousel does not fire init event"
    setup:
      slides: 3
      options: { active: false }
    action: "register 'init' handler, call createCarousel()"
    expect:
      handler_called: false

  # --- Options: dragFree ---

  - name: "dragFree:false — carousel snaps to nearest slide after drag"
    setup:
      slides: 5
      options: { dragFree: false }
    action: "simulate drag gesture that ends between slides"
    expect:
      carousel_is_on_a_snap_point: true

  - name: "dragFree:true — carousel does not snap after drag"
    setup:
      slides: 5
      options: { dragFree: true }
    action: "simulate drag gesture that ends between slides"
    expect:
      carousel_is_on_a_snap_point: false

  # --- Options: breakpoints ---

  - name: "breakpoints: matching query overrides root option"
    setup:
      slides: 5
      options:
        slidesToScroll: 1
        breakpoints:
          "(min-width: 0px)": { slidesToScroll: 2 }
    action: "call scrollNext() (breakpoint matches at any viewport width)"
    expect:
      selectedScrollSnap: 2

  - name: "breakpoints: non-matching query leaves root option unchanged"
    setup:
      slides: 5
      options:
        slidesToScroll: 1
        breakpoints:
          "(min-width: 99999px)": { slidesToScroll: 3 }
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 1

  # --- Events: init ---

  - name: "init event fires once after createCarousel"
    setup:
      slides: 3
      options: {}
    action: "register 'init' handler, call createCarousel()"
    expect:
      handler_call_count: 1

  - name: "init event does not fire on reInit"
    setup:
      slides: 3
      options: {}
    action: "register 'init' handler, call createCarousel(), then call reInit()"
    expect:
      handler_call_count: 1

  - name: "init event fires before any select event"
    setup:
      slides: 5
      options: { startSnap: 0 }
    action: "register handlers for both 'init' and 'select', call createCarousel()"
    expect:
      first_event_fired: init

  # --- Events: select and settle ---

  - name: "select event fires when scrollNext changes slide"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "register 'select' handler, call scrollNext()"
    expect:
      handler_called: true

  - name: "select fires before settle"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "register handlers for both 'select' and 'settle', call scrollNext()"
    expect:
      event_order: [select, settle]

  - name: "select does not fire for no-op scrollNext"
    setup:
      slides: 3
      options: { loop: false }
      startSnap: 2
    action: "register 'select' handler, call scrollNext()"
    expect:
      handler_called: false

  - name: "selectedScrollSnap inside select handler returns new snap"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "in 'select' handler, read api.selectedScrollSnap(); call scrollNext()"
    expect:
      handler_result: 1

  - name: "settle does not fire when jump:true"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "register 'settle' handler, call scrollNext(true)"
    expect:
      handler_called: false

  # --- Events: scroll ---

  - name: "scroll event fires during programmatic animation"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "register 'scroll' handler, call scrollTo(4)"
    expect:
      handler_call_count: ">1"

  - name: "scroll does not fire after carousel settles"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: >
      register 'scroll' handler, call scrollNext(), wait for settle,
      then call scrollProgress() — scroll count should stop increasing
    expect:
      scroll_fires_after_settle: false

  # --- Events: reInit and destroy ---

  - name: "reInit event fires after reInit() is called"
    setup:
      slides: 3
      options: {}
    action: "register 'reInit' handler, call reInit()"
    expect:
      handler_called: true

  - name: "reInit event: state queries inside handler return post-reinit values"
    setup:
      slides: 5
      options: { startSnap: 0 }
    action: >
      register 'reInit' handler that reads selectedScrollSnap(),
      call reInit({ startSnap: 3 })
    expect:
      handler_result: 3

  - name: "destroy event fires when destroy() is called"
    setup:
      slides: 3
      options: {}
    action: "register 'destroy' handler, call destroy()"
    expect:
      handler_called: true

  # --- Events: slidesInView / slidesNotInView ---

  - name: "slidesInView event fires on init"
    setup:
      slides: 5
      options: {}
    action: "register 'slidesInView' handler, call createCarousel()"
    expect:
      handler_called: true

  - name: "slidesInView — api.slidesInView() inside handler returns visible indices"
    setup:
      slides: 5
      options: {}
    action: "in 'slidesInView' handler, call api.slidesInView()"
    expect:
      result_type: "non-empty array of numbers"

  - name: "slidesNotInView — api.slidesNotInView() is complement of slidesInView"
    setup:
      slides: 5
      options: {}
    action: "after init, compare api.slidesInView() and api.slidesNotInView()"
    expect:
      combined_length: 5
      no_overlap: true

  # --- Events: slideFocus ---

  - name: "slideFocus event fires when a slide receives focus and focus:true"
    setup:
      slides: 3
      options: { focus: true }
    action: "register 'slideFocus' handler, trigger focus on second slide element"
    expect:
      handler_called: true

  - name: "slideFocus event does not fire when focus:false"
    setup:
      slides: 3
      options: { focus: false }
    action: "register 'slideFocus' handler, trigger focus on second slide element"
    expect:
      handler_called: false

  # --- Plugin lifecycle ---

  - name: "Plugin init() is called after createCarousel"
    setup:
      slides: 3
      options: {}
      plugins: [pluginA]
    action: "call createCarousel() with pluginA"
    expect:
      pluginA_init_called: true

  - name: "Plugin destroy() is called when carousel is destroyed"
    setup:
      slides: 3
      options: {}
      plugins: [pluginA]
    action: "call createCarousel(), then call destroy()"
    expect:
      pluginA_destroy_called: true

  - name: "Two plugins — init called in registration order"
    setup:
      slides: 3
      options: {}
      plugins: [pluginA, pluginB]
    action: "call createCarousel()"
    expect:
      init_order: [pluginA, pluginB]

  - name: "Two plugins — destroy called in reverse registration order"
    setup:
      slides: 3
      options: {}
      plugins: [pluginA, pluginB]
    action: "call createCarousel(), then call destroy()"
    expect:
      destroy_order: [pluginB, pluginA]

  - name: "Plugin receives full CarouselAPI in factory"
    setup:
      slides: 3
      options: {}
    action: "create plugin that reads api.selectedScrollSnap() in factory, register it"
    expect:
      no_error: true
      factory_receives_carousel_api: true

  - name: "Plugin with duplicate name — second init() is called"
    setup:
      slides: 3
      options: {}
      plugins: [pluginA_v1, pluginA_v2]
    action: "register two plugins with name 'pluginA', call createCarousel()"
    expect:
      pluginA_v1_init_called: false
      pluginA_v2_init_called: true

  - name: "Plugin can register event listeners in init() and receive them"
    setup:
      slides: 5
      options: {}
      plugins: [listenerPlugin]
    action: "call createCarousel(), then call scrollNext()"
    expect:
      plugin_select_handler_called: true

  - name: "Plugin removes its listeners in destroy()"
    setup:
      slides: 5
      options: {}
      plugins: [listenerPlugin]
    action: "call createCarousel(), call destroy(), then verify no stale listeners remain"
    expect:
      no_error: true
      listeners_removed: true

  - name: "Plugin can call api.scrollNext() inside init()"
    setup:
      slides: 5
      options: {}
      startSnap: 0
    action: "create plugin whose init() calls api.scrollNext(), register it"
    expect:
      no_error: true
      selectedScrollSnap: 1
```

- [ ] **Step 2: Verify scenarios 43–82**

Check each item:
- [ ] 40 additional scenarios appended, total = 82
- [ ] No duplicate `name` values across the full file
- [ ] Options: startSnap (3), loop (2), slidesToScroll (2), active (2), dragFree (2), breakpoints (2)
- [ ] Events: init (3), select/settle (5), scroll (2), reInit/destroy (3), slidesInView/slidesNotInView (3), slideFocus (2)
- [ ] Plugin lifecycle: 9 scenarios covering init order, destroy order, duplicate name, API access, listener cleanup

- [ ] **Step 3: Commit**

```bash
git add tests.yaml
git commit -m "spec: complete tests.yaml — 82 scenarios total"
```

---

### Task 8: INSTALL.md

**Files:**
- Create: `INSTALL.md`

- [ ] **Step 1: Create INSTALL.md**

```markdown
# Installing carouselWords

`carouselWords` is a ghost library. No package is installed. No code is distributed.
You give the specification files to your AI agent and it implements the library for you.
Every line of the resulting code is locally generated, auditable, and free of external
dependencies. The spec is the single source of truth.

---

## How It Works

1. Point your AI agent at the spec files listed below.
2. Paste the prompt template into your agent session.
3. The agent implements the carousel library and a test file.
4. Run the tests. If any scenarios fail, paste the failures back and tell the agent
   to iterate.

---

## File Reading Order

Give your agent the files in this order:

1. `SPEC.md` — core concepts, HTML contract, factory API, design principles
2. `OPTIONS.md` — all 21 configuration options
3. `METHODS.md` — all 19 public methods
4. `EVENTS.md` — all 9 events and ordering guarantees
5. `PLUGINS.md` — plugin interface contract
6. `tests.yaml` — 82 scenario-based test cases

---

## Ready-to-Paste Prompt

Copy this block and paste it into your agent session. Replace `[LANGUAGE]` with your
target (e.g. `TypeScript`, `JavaScript`).

```
Read the following files in order: SPEC.md, OPTIONS.md, METHODS.md, EVENTS.md,
PLUGINS.md. Then read tests.yaml.

Implement a carousel library in [LANGUAGE] that:
- Satisfies every contract described in the spec files
- Passes every scenario in tests.yaml
- Exports createCarousel() as the sole public entry point
- Exports TypeScript types for Options, CarouselAPI, CarouselPlugin, and CarouselEventHandler

Output exactly two files:
1. carousel.[ext] — the full library implementation
2. carousel.test.[ext] — a test file that verifies every scenario in tests.yaml

Do not output package.json, tsconfig.json, build scripts, or any other scaffolding.
```

---

## Iterating on Failures

If any scenarios fail after the first pass, paste the test output back to your agent:

```
The following scenarios failed:

[paste test output here]

Fix the implementation to make them pass. Do not change the test file.
```

Repeat until all 82 scenarios pass.
```

- [ ] **Step 2: Verify INSTALL.md**

Check each item:
- [ ] Ghost library concept explained in one paragraph, no jargon
- [ ] File reading order lists all 6 files in the correct sequence
- [ ] Prompt template is in a fenced code block, self-contained and copy-pasteable
- [ ] `[LANGUAGE]` is clearly labeled as a placeholder to replace
- [ ] Prompt requests both an implementation file and a test file
- [ ] Prompt explicitly excludes scaffolding (package.json, tsconfig, etc.)
- [ ] Iteration section included with failure-paste pattern

- [ ] **Step 3: Final commit**

```bash
git add INSTALL.md
git commit -m "spec: add INSTALL.md — agent guide, reading order, prompt template"
```

---

## Self-Review Checklist

After all 8 tasks are complete, verify the package as a whole:

- [ ] `git log --oneline` shows 9 commits (1 init + 8 tasks)
- [ ] Root directory contains exactly: `INSTALL.md`, `SPEC.md`, `OPTIONS.md`, `METHODS.md`, `EVENTS.md`, `PLUGINS.md`, `tests.yaml`, `docs/`
- [ ] `SPEC.md` version matches `OPTIONS.md` version matches all other spec file versions (all `1.0.0`)
- [ ] `createCarousel` is the only factory name used across all files (no `EmblaCarousel`)
- [ ] `CarouselAPI` type is defined in `METHODS.md` and referenced consistently in `PLUGINS.md` and `EVENTS.md`
- [ ] `CarouselEventHandler` type is defined in `METHODS.md` and referenced in `EVENTS.md`
- [ ] All 82 scenario names in `tests.yaml` are unique
- [ ] `tests.yaml` contains all 4 required fields per scenario: `name`, `setup`, `action`, `expect`
