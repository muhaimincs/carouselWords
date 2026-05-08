# carouselWords

A ghost library for building production-grade web carousels with AI.

`carouselWords` ships no code. It ships a specification. You hand the spec
to your AI agent; the agent writes the carousel. Every line of the result
is locally generated, auditable, and free of external dependencies.

Inspired by [whenWords](https://github.com/Rich-Harris/whenwords).

---

## The idea

Most carousel libraries ship kilobytes of JavaScript you vendor into your
bundle and trust blindly. `carouselWords` inverts that: the specification is
the library. You own the implementation.

---

## How to use it

**1. Give the files to your agent — in this order**

1. `SPEC.md` — core concepts, HTML contract, `createCarousel()` API
2. `OPTIONS.md` — 22 configuration options
3. `METHODS.md` — 19 public methods
4. `EVENTS.md` — 9 events with ordering guarantees
5. `PLUGINS.md` — plugin interface contract
6. `ACCESSIBILITY.md` — WCAG 2.2 Level AA contract
7. `tests.yaml` — 93 test scenarios

**2. Paste this prompt** (replace `[LANGUAGE]` with `TypeScript` or `JavaScript`)

```
Read the following files in order: SPEC.md, OPTIONS.md, METHODS.md, EVENTS.md,
PLUGINS.md, ACCESSIBILITY.md. Then read tests.yaml.

Before implementing, ask: should structural styles (overflow: hidden, flex layout)
be applied as inline styles or via a companion stylesheet the consumer imports?

Implement a carousel library in [LANGUAGE] that:
- Satisfies every contract described in the spec files
- Satisfies the WCAG 2.2 Level AA accessibility contract in ACCESSIBILITY.md
- Passes every scenario in tests.yaml
- Exports createCarousel() as the sole public entry point
- If using TypeScript: also export types for Options, CarouselAPI, CarouselPlugin,
  and CarouselEventHandler

Output exactly two files:
1. carousel.[ext] — the full library implementation
2. carousel.test.[ext] — a test file that verifies every scenario in tests.yaml

Do not output package.json, tsconfig.json, build scripts, or any other scaffolding.
```

**3. Run the tests. Iterate on failures.**

```
The following scenarios failed:

[paste test output here]

Fix the implementation to make them pass. Do not change the test file.
```

Repeat until all 93 scenarios pass.

---

## What the spec covers

| File | Contents |
|---|---|
| `SPEC.md` | HTML contract, `createCarousel()` factory, design principles |
| `OPTIONS.md` | 22 options: alignment, axis, drag, loop, breakpoints, and more |
| `METHODS.md` | 19 methods: scroll, state queries, DOM access, lifecycle |
| `EVENTS.md` | 9 events with strict ordering guarantees |
| `PLUGINS.md` | Plugin factory interface and registration rules |
| `ACCESSIBILITY.md` | WCAG 2.2 Level AA: ARIA injections, keyboard nav, cleanup |
| `tests.yaml` | 93 scenario-based test cases |

---

## Design principles

- **No global state** — two carousels on the same page never share state.
- **No polling** — DOM is measured once at init. ResizeObserver handles resize.
- **Deterministic** — same options + same DOM always produces the same result.
- **Graceful no-ops** — any method called after `destroy()` does nothing, throws nothing.
- **WCAG 2.2 Level AA** — accessibility is a versioned contract. Breaking it bumps the major version.

---

## Quick API reference

```typescript
const carousel = createCarousel(root, options?, plugins?)

carousel.scrollNext()
carousel.scrollPrev()
carousel.scrollTo(index)
carousel.selectedScrollSnap()   // → number
carousel.slidesInView()         // → number[]
carousel.on('select', handler)
carousel.reInit({ loop: true })
carousel.destroy()
```

---

## Versioning

This spec is at **v1.0.0**. Breaking changes (removed options, changed signatures)
increment the major version. Additive changes increment the minor version. Patch
versions are not used for spec changes.

---

## License

MIT
# carouselWords
