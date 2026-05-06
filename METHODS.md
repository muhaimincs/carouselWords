# carouselWords Methods

**Version:** v1.0.0

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
  `inViewMargin` (see OPTIONS.md).
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
