# carouselWords Accessibility

**Version:** v1.0.0

This file defines the WCAG 2.2 Level AA accessibility contract for `carouselWords`.
It is part of the spec suite. An agent implementing this spec must satisfy every
requirement in this file.

---

## 1. Library-Owned Injections

`createCarousel()` automatically injects the following ARIA attributes at
initialization time. No consumer configuration is required beyond setting the
`ariaLabel` option.

### Root element (`.carousel`)

| Attribute | Value |
|---|---|
| `role` | `"region"` |
| `aria-roledescription` | `"carousel"` |
| `aria-label` | Value of the `ariaLabel` option (default: `"Carousel"`) |

### Viewport element (`.carousel__viewport`)

| Attribute | Value |
|---|---|
| `aria-live` | `"polite"` |

### Each slide element (`.carousel__slide`)

| Attribute | Value |
|---|---|
| `role` | `"group"` |
| `aria-roledescription` | `"slide"` |
| `aria-label` | `"N of M"` where N is the 1-based slide index and M is the total slide count |

### Selected slide

| Attribute | Value |
|---|---|
| `aria-current` | `"true"` on the currently selected slide; attribute absent on all others |

`aria-current` is updated on every `select` event to reflect the new selected slide.

---

## 2. Keyboard Navigation

When the root element or any slide element has keyboard focus, the following
keys are handled:

| Key | `axis: 'x'` | `axis: 'y'` |
|---|---|---|
| `ArrowRight` | `scrollNext()` | — |
| `ArrowLeft` | `scrollPrev()` | — |
| `ArrowDown` | — | `scrollNext()` |
| `ArrowUp` | — | `scrollPrev()` |

- The keydown event is handled on the root element using event delegation.
- `event.preventDefault()` is called for handled keys to prevent page scroll.
- When `active: false` or after `destroy()`, key handlers do nothing.

---

## 3. Cleanup on Destroy

All injected ARIA attributes are removed from all elements when `destroy()` is
called. The keyboard event listener is removed from the root element.

Attributes removed:
- Root: `role`, `aria-roledescription`, `aria-label`
- Viewport: `aria-live`
- Each slide: `role`, `aria-roledescription`, `aria-label`, `aria-current`

---

## 4. Consumer Responsibilities

The following WCAG 2.2 requirements fall outside the library boundary. The
consumer is responsible for satisfying them.

### Color contrast (WCAG 1.4.3, 1.4.11)
Text and UI components inside slides must meet contrast ratios. The library
applies no colors.

### Alt text on images (WCAG 1.1.1)
Any `<img>` elements inside slides must have a descriptive `alt` attribute. The
library does not inspect or modify slide content.

### Prev/next button labeling (WCAG 4.1.2)
If the consumer renders prev/next buttons, each must have an accessible name
(e.g. `aria-label="Previous slide"`). The library does not render control buttons.

### Prev/next button target size (WCAG 2.5.8)
Control buttons must meet the 24×24 CSS pixel minimum target size. The library
does not render or size control buttons.

### Slide content keyboard access (WCAG 2.1.1)
If slides contain interactive elements (links, buttons, inputs), the consumer
must ensure they are reachable via Tab. The library does not manage tab order
inside slide content.

### Auto-advancing behavior (WCAG 2.2.2)
If the consumer adds auto-advancing logic (e.g. via a plugin or external timer),
it must provide a mechanism to pause, stop, or hide the motion. The library does
not implement auto-advancing.
