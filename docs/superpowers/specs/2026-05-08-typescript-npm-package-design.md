# carouselWords — TypeScript npm Package Design

**Date:** 2026-05-08  
**Status:** Approved

---

## Goal

Implement the `carouselWords` ghost-library spec as two TypeScript files placed in a
sibling folder `/Users/achesohor/Desktop/carousel-words-ts/`. The result is runnable
as a TypeScript npm package (Vitest for tests, no build/dist step).

---

## File Structure

```
/Users/achesohor/Desktop/carousel-words-ts/
├── package.json          # name: carousel-words, vitest + typescript devDeps, jsdom env
├── tsconfig.json         # ESNext target, DOM lib, strict mode
├── carousel.ts           # full implementation (types + factory)
└── carousel.test.ts      # 82 vitest scenarios
```

No `dist/`, no bundler, no publish step. `package.json` exists only to wire up Vitest.

---

## carousel.ts Architecture

**Approach:** Functional closure with flat internal state (Option A).

### Exported types (top of file)

All four types required by the spec, exported at the module level:

```ts
export type Options = { ... }          // 21 fields with defaults
export type CarouselEventHandler = (api: CarouselAPI, event: string) => void
export type CarouselPlugin = (api: CarouselAPI) => { name: string; init(): void; destroy(): void; [key: string]: unknown }
export type CarouselAPI = { ... }      // 19 methods
```

### Defaults constant

A single `DEFAULT_OPTIONS` object covering all 21 options, used by `resolveOptions()`.

### Internal state shape (per instance)

```ts
type State = {
  destroyed: boolean
  selectedSnap: number
  previousSnap: number
  snaps: number[]                                        // computed snap positions
  listeners: Map<string, Set<CarouselEventHandler>>
  containerEl: HTMLElement
  slideEls: HTMLElement[]
  opts: Options                                          // resolved, post-breakpoints
}
```

### Internal helpers (inside the closure)

| Helper | Responsibility |
|---|---|
| `measure()` | Reads DOM once, computes `snaps[]` from slide offsets |
| `applyScroll(index, jump)` | Sets container translate or scrollLeft |
| `emit(event)` | Iterates `state.listeners` and calls each handler |
| `resolveOptions(raw)` | Merges defaults + active breakpoints |
| `guardDestroyed()` | Returns `true` if destroyed; used to short-circuit all public methods |

### CarouselAPI object

Returned directly from `createCarousel()`. All 19 methods as object literal properties.
Each method calls `guardDestroyed()` first. Plugin `init()` calls happen after the API
object is built, before the `init` event fires.

### Plugin lifecycle

```
createCarousel() called
  → carousel internal init (measure DOM, compute snaps)
  → build CarouselAPI object
  → for each plugin: call plugin factory with api → store result
  → for each plugin result: call result.init()
  → emit('init')
```

On `destroy()`:
```
  → emit('destroy')
  → for each plugin result (reverse registration order): call result.destroy()
  → set state.destroyed = true
  → remove all event listeners and observers
```

---

## carousel.test.ts Architecture

**Framework:** Vitest with `jsdom` environment (set in `package.json` vitest config).

### Shared helper

```ts
function makeCarousel(slideCount: number): HTMLElement
```

Builds the required 4-level DOM structure:
`.carousel > .carousel__viewport > .carousel__container > .carousel__slide*N`

This is the only test utility. All assertions use the public `CarouselAPI` directly.

### Scenario groups

| Group | Scenarios | Count |
|---|---|---|
| Init | factory call, error on null/non-element root, unknown options ignored | 1–10 |
| Scroll & state | scrollNext/Prev/To, snap clamping, loop wrapping, canScrollNext/Prev | 11–35 |
| Lifecycle | destroy no-ops, reInit preserves listeners, idempotent destroy | 36–50 |
| Events | ordering (select before settle), scroll fires during animation, reInit event | 51–65 |
| Plugins | init/destroy order, duplicate name deduplication | 66–75 |
| Edge cases | breakpoints, containScroll, RTL direction | 76–82 |

Each scenario maps to exactly one `it()` call. No shared mutable state between tests.

---

## Key Spec Invariants Encoded in the Design

- `guardDestroyed()` enforces the "graceful no-op after destroy" rule across all 19 methods.
- `measure()` is called once at init (and again on `reInit()`), never polled — satisfies "lazy DOM reads."
- `state` is a plain object scoped inside the factory closure — satisfies "no global state."
- `DEFAULT_OPTIONS` + `resolveOptions()` handles breakpoint merging without mutating the user-provided options object.
- `reInit()` calls `resolveOptions()` again, re-runs `measure()`, but does NOT re-register listeners — they are preserved in `state.listeners`.

---

## Constraints

- No `new` keyword in public API surface.
- No polling, no `setInterval`, no continuous DOM reads (except `ResizeObserver` when `resize: true`).
- `emit()` must not throw if a handler throws — wrap in try/catch and continue.
- TypeScript strict mode enabled; no `any` escapes.
