# carouselWords Plugin Interface

**Version:** v1.0.0

This file defines the contract for building a plugin. It does not define any
specific plugin implementations — those are out of scope for this spec.

The `CarouselAPI` type referenced throughout this file is defined in METHODS.md.

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

The full `CarouselAPI` — all 19 methods — is available to the plugin factory at
call time, before `init()` is invoked.

---

## Plugin Shape

A plugin factory receives the full `CarouselAPI` when registered. It must return
an object with three required properties:

- **`name`** (`string`) — a unique identifier for this plugin. Used to detect
  duplicate registrations. Must be a non-empty string.
- **`init()`** — called after the carousel's own init completes. The plugin
  should register any event listeners and set up its behavior here.
- **`destroy()`** — called when the carousel is destroyed, before the carousel's
  own `destroy` event fires. The plugin must remove all event listeners and clean
  up all resources it created.

Plugins may return additional properties (methods, state, configuration, etc.)
beyond the three required ones. The carousel does not restrict or inspect these.

---

## Registration Contract

Plugins are passed as the third argument to `createCarousel()`:

```typescript
createCarousel(root, options, [pluginA, pluginB])
```

**Initialization order:**
1. The carousel runs its own initialization.
2. Plugin `init()` functions are called in registration order (`pluginA.init()` first,
   then `pluginB.init()`).
3. The carousel fires the `init` event.

**Destruction order:**
1. The carousel fires the `destroy` event.
2. Plugin `destroy()` functions are called in **reverse** registration order
   (`pluginB.destroy()` first, then `pluginA.destroy()`).
3. The carousel tears down its own listeners and observers.

---

## Plugin Rules

1. **Must not mutate options** — a plugin must not modify the `options` object
   passed to `createCarousel()`. Options are read-only from the plugin's perspective.

2. **Must clean up in `destroy()`** — every event listener registered via
   `api.on()` inside `init()` must be removed via `api.off()` inside `destroy()`.
   Failing to clean up will cause memory leaks and stale handler calls.

3. **Duplicate `name` — last wins** — if two plugins share the same `name`, the
   second plugin replaces the first. Only the second plugin's `init()` is called.
   The first plugin is silently discarded. No error is thrown.

4. **May expose a public API** — a plugin may return additional properties alongside
   `name`, `init`, and `destroy`. Consumers access the plugin's public API through
   the object returned by the factory, not through the `CarouselAPI`.

---

## Example Plugin Structure

```typescript
function myPlugin(api: CarouselAPI) {
  function handleSelect(_api: CarouselAPI) {
    // react to slide change using _api.selectedScrollSnap()
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
