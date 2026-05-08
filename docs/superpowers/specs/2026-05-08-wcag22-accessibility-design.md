# carouselWords — WCAG 2.2 Accessibility Design

**Date:** 2026-05-08  
**Status:** Approved

---

## Goal

Add a WCAG 2.2 Level AA accessibility contract to the `carouselWords` spec suite.
Scope: library-owned surfaces only (auto-injected ARIA, keyboard navigation, cleanup).
Consumer responsibilities are documented explicitly, not enforced by the library.

---

## Approach

**Option A — Minimal ARIA injection** (selected).

The library auto-injects only what it can own. One new option (`ariaLabel`).
No autoPlay, no pause-on-hover, no focus-trap complexity. Consumer responsibilities
are documented in `ACCESSIBILITY.md` Section 4.

---

## New File: `ACCESSIBILITY.md`

Two sections:

**Section 1 — Library-owned injections** (auto-applied by `createCarousel()`):

| Surface | Injections |
|---|---|
| Root `.carousel` | `role="region"`, `aria-roledescription="carousel"`, `aria-label` |
| Viewport `.carousel__viewport` | `aria-live="polite"` |
| Each slide `.carousel__slide` | `role="group"`, `aria-roledescription="slide"`, `aria-label="N of M"` |
| Selected slide | `aria-current="true"` (removed from others on every `select`) |
| Root keydown | Arrow keys route to `scrollNext`/`scrollPrev` per `axis` option |

All injections are removed on `destroy()`.

**Section 2 — Consumer responsibilities** (documented, not enforced):
- Color contrast (WCAG 1.4.3, 1.4.11)
- Alt text on images (WCAG 1.1.1)
- Prev/next button labeling and 24×24px target size (WCAG 4.1.2, 2.5.8)
- Tab access to slide content (WCAG 2.1.1)
- Auto-advance pause mechanism if consumer adds autoPlay (WCAG 2.2.2)

---

## Changes to Existing Files

### `OPTIONS.md`
- `ariaLabel: string` added to `Options` type (22nd option)
- Default: `'Carousel'`. Empty string falls back to default.
- New "Accessibility" section at end of file.

### `SPEC.md`
- New Section 5 "Accessibility Contract" added before Versioning.
- States that breaking the a11y contract is a major version bump.

### `INSTALL.md`
- `ACCESSIBILITY.md` added to reading order at position 6 (before `tests.yaml`).
- Option count updated: 21 → 22.
- Test count updated: 82 → 92.
- Prompt template updated: includes `ACCESSIBILITY.md` in the reading list and
  adds the CSS approach clarifying question before implementation.

### `tests.yaml`
- Version bumped: 1.0.0 → 1.1.0.
- 10 new scenarios added (scenarios 83–92):
  - Root `role`, `aria-roledescription`, `aria-label` presence
  - Default and custom `ariaLabel` values
  - Empty `ariaLabel` fallback
  - Slide `role`, `aria-roledescription`, `aria-label` format
  - `aria-current` on selected slide only
  - ArrowRight/ArrowLeft keyboard navigation
  - All injected attributes absent after `destroy()`

---

## Versioning Impact

This is a **minor version** change to the spec (additive: new option, new file,
new test scenarios). No existing contracts are broken.

The accessibility contract itself is versioned at major level — removing or changing
an injected attribute in a future version requires a major bump.
