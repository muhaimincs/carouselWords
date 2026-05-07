# carouselWords Events

**Version:** v1.0.0

Events are subscribed to with `api.on(eventName, handler)` and unsubscribed with
`api.off(eventName, handler)`. See METHODS.md for the `on()`, `off()`, and `emit()`
method contracts.

All event handlers share the same signature:

```typescript
type CarouselEventHandler = (api: CarouselAPI, event: string) => void
```

The handler receives the full `CarouselAPI` as the first argument. Call any state
query method (e.g. `api.selectedScrollSnap()`) inside the handler to read current state.
`CarouselAPI` is defined in METHODS.md.

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
- **Order guarantee**: Fires before any interaction event. See Ordering Guarantee 1.
- **Notes**: Fires exactly once per carousel instance. Does not fire on `reInit()`.

### reInit
- **Fires when**: After `reInit()` completes and new state is applied.
- **Payload**: None. State queries inside the handler return post-reinit values.
- **Order guarantee**: Fires after all state changes from `reInit()` are applied.
  See Ordering Guarantee 4.
- **Notes**: Does not fire on the initial `createCarousel()` call (that fires `init`).

### destroy
- **Fires when**: When `destroy()` is called, before listeners are removed.
- **Payload**: None.
- **Order guarantee**: Fires once. After this event, no further events will fire for
  this instance.
- **Notes**: This is the last event a handler will ever receive for this instance.

---

## Interaction Events

### select
- **Fires when**: A new snap is selected. This happens immediately when
  `scrollNext()`, `scrollPrev()`, or `scrollTo()` is called and results in a
  different snap, or when a drag gesture commits to a new snap.
- **Payload**: None. Call `api.selectedScrollSnap()` inside the handler to get the
  new snap index.
- **Order guarantee**: Fires before `settle`. See Ordering Guarantee 2.
- **Notes**: Does not fire if `scrollNext()` is a no-op (e.g. already at the last
  slide with `loop: false`). Does not fire on `reInit()`.

### scroll
- **Fires when**: Continuously during any scroll animation or drag gesture, for
  every animation frame.
- **Payload**: None. Call `api.scrollProgress()` inside the handler for current
  scroll position.
- **Order guarantee**: Fires many times between `select` and `settle`. See Ordering
  Guarantee 3.
- **Notes**: Handlers must be lightweight. Avoid heavy DOM operations.

### settle
- **Fires when**: The scroll animation completes and the carousel comes to rest
  on a snap point.
- **Payload**: None. Call `api.selectedScrollSnap()` inside the handler.
- **Order guarantee**: Fires after `select` and after the last `scroll` frame.
  See Ordering Guarantee 2.
- **Notes**: Does not fire when `jump: true` is used on a scroll method (no
  animation occurs, so there is nothing to settle).

### slidesInView
- **Fires when**: The set of slides visible within the viewport changes.
- **Payload**: None. Call `api.slidesInView()` inside the handler for the current
  set of visible slide indices.
- **Order guarantee**: No guaranteed order relative to `select` or `settle`.
- **Notes**: Also fires on `init` with the initial set of visible slides.
  Visibility is determined by `inViewThreshold` and `inViewMargin` (see OPTIONS.md).

### slidesNotInView
- **Fires when**: The set of slides NOT visible within the viewport changes.
- **Payload**: None. Call `api.slidesNotInView()` inside the handler for the
  current set of hidden slide indices.
- **Order guarantee**: Fires at the same time as `slidesInView`.
- **Notes**: Also fires on `init`. The indices are the complement of `slidesInView()`.

### slideFocus
- **Fires when**: A slide element receives DOM focus (via keyboard navigation or
  programmatic focus). Only fires when the `focus` option is `true` (see OPTIONS.md).
- **Payload**: None. Call `api.selectedScrollSnap()` inside the handler to get
  the focused slide's snap index.
- **Order guarantee**: No guaranteed order relative to other events.
- **Notes**: Does not fire when `focus: false`.
