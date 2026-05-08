# carouselWords Specification

**Version:** v1.0.0

`carouselWords` is a ghost library. This file, together with OPTIONS.md, METHODS.md,
EVENTS.md, PLUGINS.md, and tests.yaml, is everything an AI agent needs to implement a
production web carousel in Vanilla JS/TypeScript. No source code is distributed.
The spec is the single source of truth.

---

## 1. HTML Contract

A carousel requires four nested DOM elements. The agent must produce and manage
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
- The container lays out slides in a single row (or column when `axis: 'y'`; see OPTIONS.md).
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
- `options` — configuration object. All keys are optional. Unrecognized keys are silently ignored. `Options` is defined in OPTIONS.md.
- `plugins` — array of plugin factories. Optional, defaults to `[]`. `CarouselPlugin` is defined in PLUGINS.md.

Returns a `CarouselAPI` object. `CarouselAPI` is defined in METHODS.md.

**Invariants:**
- No `new` keyword. `createCarousel` is a plain factory function.
- No global state. Each call returns a fully isolated instance.
- No side effects outside `root`. The function must not modify `document`, `window`,
  or any element outside `root`.

---

## 3. Design Principles

- **No global state** — two carousels on the same page must not share any state.
- **Lazy DOM reads** — measure the DOM at initialization time. Do not poll continuously.
  Exception: when the `resize` option is `true` (see OPTIONS.md), a ResizeObserver watches the root element and calls `reInit()` on resize.
- **Deterministic behavior** — identical options plus identical DOM always produces
  identical initial state.
- **Graceful no-op** — any `CarouselAPI` method called after `destroy()` does nothing
  and returns `undefined`. No exceptions are thrown.
- **TypeScript types required** — all public surfaces (`Options`, `CarouselAPI`,
  `CarouselPlugin`, `CarouselEventHandler`) must have explicit TypeScript type definitions
  exported from the library entry point. Each type is defined in its corresponding spec file.

---

## 5. Accessibility Contract

WCAG 2.2 Level AA compliance for library-owned surfaces is a versioned contract.
Breaking it (removing an ARIA injection, changing a keyboard binding) is a **major
version** bump.

The full accessibility specification — injected attributes, keyboard bindings,
cleanup rules, and consumer responsibilities — is defined in `ACCESSIBILITY.md`.
Agents must read `ACCESSIBILITY.md` as part of the spec suite.

---

## 4. Versioning Contract

This spec is at `v1.0.0`. Each spec file carries this version in its header.

- **Major version** increments on breaking changes (removed option, changed method signature).
- **Minor version** increments on additive changes (new option, new event).
- Patch versions are not used for spec changes.

An agent implementing `v1.0.0` of any spec file is not expected to handle `v2.0.0`
behavior without re-reading the updated spec.
