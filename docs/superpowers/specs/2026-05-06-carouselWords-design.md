# carouselWords — Design Spec

**Date:** 2026-05-06
**Version:** 1.0.0
**Status:** Approved

---

## What Is This

`carouselWords` is a ghost library — a specification package, not a code package. It contains everything an AI agent needs to implement a production-grade web carousel in Vanilla JS/TypeScript from scratch. No source code is distributed. The spec is the single source of truth.

Modeled after [whenwords](https://github.com/dbreunig/whenwords). Reference implementation surface: [Embla Carousel](https://github.com/davidjerleke/embla-carousel).

---

## Scope

- **Target**: Vanilla JS / TypeScript only. Framework adapters (React, Vue, Svelte) are out of scope.
- **Features**: Full production carousel — all options, methods, events, plugin interface.
- **Plugin built-ins**: Out of scope. The spec defines the plugin contract only; agents implement extensibility, not specific plugins.
- **Tests**: Scenario-style — prose descriptions of setup, action, and expected outcome. Not machine-executable YAML assertions.

---

## File Structure

```
carouselWords/
├── INSTALL.md       # Agent consumption guide + ready-to-paste prompt
├── SPEC.md          # Core concepts, HTML contract, init API, design principles
├── OPTIONS.md       # All 22 configuration options with schema
├── METHODS.md       # Full method API (15 methods) with schema
├── EVENTS.md        # Event system (9 events) with ordering guarantees
├── PLUGINS.md       # Plugin interface contract
└── tests.yaml       # ~80 scenario-based test cases
```

Each spec file has a single, isolated purpose. `INSTALL.md` is the orchestrator — it defines reading order and contains the agent prompt.

---

## File-by-File Design

### INSTALL.md

Three sections:
1. **What this is** — one paragraph on the ghost library concept
2. **File reading order** — `SPEC.md` → `OPTIONS.md` → `METHODS.md` → `EVENTS.md` → `PLUGINS.md` → `tests.yaml`
3. **Ready-to-paste prompt** — fenced code block. `[LANGUAGE]` is a user-fill placeholder (e.g. "TypeScript", "JavaScript"):

```
Read the following files in order: SPEC.md, OPTIONS.md, METHODS.md,
EVENTS.md, PLUGINS.md. Then read tests.yaml. Implement a carousel
library in [LANGUAGE] that satisfies all spec contracts and passes
all scenarios in tests.yaml. Output: one source file + one test file.
Do not output package config, build tools, or scaffolding.
```

---

### SPEC.md

Four sections:

1. **HTML contract** — required DOM structure (root element, viewport/container, slides). Specifies required classes or data attributes. Does not assume any CSS framework.

2. **Initialization API** — single factory function: `createCarousel(root: HTMLElement, options?: Options): CarouselAPI`. No `new`, no global singleton, no side effects outside the passed root. `CarouselAPI` is the return type defined in this file — it is the shape agents must expose.

3. **Design principles**:
   - No global state — each instance is fully isolated
   - Lazy DOM reads — reads DOM at init, not continuously
   - Deterministic behavior — same options + same DOM = same behavior
   - Graceful no-op — methods called on a destroyed carousel do nothing (no throw)
   - TypeScript types required — all public API surfaces have explicit types

4. **Versioning contract** — spec carries `v1.0.0`. Breaking changes to any file increment the major version.

---

### OPTIONS.md

Per-option schema:
```
### optionName
- **Type**: string | boolean | number | object
- **Default**: value
- **Description**: plain English
- **Constraints**: valid values, range limits, invalid combinations
- **Interacts with**: other options this affects or conflicts with
```

**22 options in four groups:**

| Group | Options |
|---|---|
| Layout | `align`, `axis`, `direction`, `containScroll`, `slidesToScroll` |
| Interaction | `dragFree`, `dragThreshold`, `draggable`, `loop`, `skipSnaps` |
| Timing | `duration`, `startSnap` |
| Advanced | `container`, `slides`, `inViewThreshold`, `inViewMargin`, `breakpoints`, `active`, `resize`, `focus`, `slideChanges` |

**Two cross-cutting rules at the top:**
1. `breakpoints` inherits — any option in a breakpoint key overrides the root option at that viewport width. `breakpoints` cannot be nested.
2. Unknown options are silently ignored — no throw on unrecognized keys.

---

### METHODS.md

Per-method schema:
```
### methodName()
- **Signature**: methodName(param: Type): ReturnType
- **Description**: what it does
- **Valid when**: required conditions (e.g. "must not be destroyed")
- **Returns**: what the return value represents
- **Error contract**: behavior on invalid input or wrong state
```

**15 methods in four groups:**

| Group | Methods |
|---|---|
| Scroll | `scrollNext()`, `scrollPrev()`, `scrollTo(index, jump?)` |
| State queries | `canScrollNext()`, `canScrollPrev()`, `selectedScrollSnap()`, `previousScrollSnap()`, `scrollSnapList()` |
| DOM access | `rootNode()`, `containerNode()`, `slideNodes()` |
| Lifecycle | `reInit(options?)`, `destroy()`, `on(event, handler)`, `off(event, handler)` |

**Three invariants at the top:**
1. All state queries are synchronous — reflect current state at call time.
2. `destroy()` is idempotent — calling it twice does nothing on the second call.
3. `reInit()` preserves event listeners — handlers registered with `on()` survive a `reInit()`.

---

### EVENTS.md

Per-event schema:
```
### eventName
- **Fires when**: exact trigger condition
- **Payload**: what the handler receives
- **Order guarantee**: relationship to other events
- **Notes**: edge cases (e.g. fires on reInit?)
```

**9 events in two groups:**

| Group | Events |
|---|---|
| Lifecycle | `init`, `reInit`, `destroy` |
| Interaction | `select`, `scroll`, `settle`, `slidesInView`, `slidesNotInView`, `slideFocus` |

**Four ordering guarantees at the top:**
1. `init` fires once, after carousel is fully ready, before any interaction event.
2. `select` fires before `settle` — `select` = target decided, `settle` = animation complete.
3. `scroll` fires continuously during drag/animation — handlers must be cheap.
4. `reInit` fires after new state is applied — methods queried inside handler return post-reinit values.

---

### PLUGINS.md

Plugin factory type:
```typescript
type CarouselPlugin = (carousel: CarouselAPI) => {
  name: string
  init: () => void
  destroy: () => void
}
```

**Three sections:**

1. **Plugin shape** — factory receives full carousel API. Returns `name`, `init()`, `destroy()`. No other lifecycle hooks — plugins use `carousel.on()` to react to events.

2. **Registration contract** — plugins passed as third argument: `createCarousel(root, options, [plugin1, plugin2])`. Carousel calls each plugin's `init()` in registration order after its own `init`. `destroy()` called in reverse order. `CarouselAPI` received by each plugin is the same object returned by `createCarousel()`, as defined in `SPEC.md`.

3. **Plugin rules**:
   - Must not mutate the carousel's options object
   - Must clean up all own event listeners in `destroy()`
   - Two plugins with same `name` — last one wins (no throw)
   - May expose additional public API by returning extra properties

---

### tests.yaml

**Structure:**
```yaml
version: 1.0.0

scenarios:
  - name: "Scrolls to next slide"
    setup:
      slides: 5
      options: { loop: false }
      startSnap: 0
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 1
      event_fired: select

  - name: "Does not scroll past last slide without loop"
    setup:
      slides: 3
      options: { loop: false }
      startSnap: 2
    action: "call scrollNext()"
    expect:
      selectedScrollSnap: 2
      event_fired: null
```

**Target: ~80 scenarios** covering:
- Options interactions (e.g. `loop` + `containScroll`)
- Method contracts (return values, error contracts)
- Event ordering (e.g. `select` before `settle`)
- Plugin lifecycle (init order, destroy order, cleanup)
- Edge cases (destroyed carousel, reInit, single slide)

---

## Roadmap (Ordered Deliverables)

Each step produces a file that can be reviewed and validated independently.

| # | Deliverable | Validates |
|---|---|---|
| 1 | `SPEC.md` | HTML contract, init API, design principles |
| 2 | `OPTIONS.md` | All 22 options with schema |
| 3 | `METHODS.md` | All 15 methods with schema |
| 4 | `EVENTS.md` | All 9 events with ordering guarantees |
| 5 | `PLUGINS.md` | Plugin interface contract |
| 6 | `tests.yaml` (core) | ~40 scenarios covering options + methods |
| 7 | `tests.yaml` (events + plugins) | ~40 more scenarios for events + plugin lifecycle |
| 8 | `INSTALL.md` | Agent prompt, reading order, end-to-end flow |

---

## What Is Explicitly Out of Scope

- Framework adapters (React, Vue, Svelte, Angular, Solid)
- Built-in plugin implementations (Autoplay, Fade, Accessibility, etc.)
- CSS, animation, or styling instructions
- Build tooling, bundler config, or npm publishing
- Any source code output from this package
