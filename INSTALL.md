# Installing carouselWords

`carouselWords` is a ghost library. No package is installed. No code is distributed.
You give the specification files to your AI agent and it implements the library for you.
Every line of the resulting code is locally generated, auditable, and free of external
dependencies. The spec is the single source of truth.

---

## How It Works

1. Point your AI agent at the spec files listed below.
2. Paste the prompt template into your agent session, replacing `[LANGUAGE]` with your
   target language (e.g. `TypeScript`, `JavaScript`).
3. The agent implements the carousel library and a test file.
4. Run the tests. If any scenarios fail, paste the failures back and tell the agent
   to iterate until all 93 scenarios pass.

---

## File Reading Order

Give your agent the files in this order:

1. `SPEC.md` — core concepts, HTML contract, `createCarousel()` factory API, design principles
2. `OPTIONS.md` — all 22 configuration options with types, defaults, and constraints
3. `METHODS.md` — all 19 public methods with signatures, behavior, and error contracts
4. `EVENTS.md` — all 9 events with trigger conditions and ordering guarantees
5. `PLUGINS.md` — plugin interface contract and registration rules
6. `ACCESSIBILITY.md` — WCAG 2.2 Level AA contract: injected ARIA attributes, keyboard bindings, consumer responsibilities
7. `tests.yaml` — 93 scenario-based test cases

---

## Ready-to-Paste Prompt

Copy this block and paste it into your agent session. Replace `[LANGUAGE]` with your
target (e.g. `TypeScript`, `JavaScript`):

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
- If using TypeScript: also export types for Options, CarouselAPI, CarouselPlugin, and CarouselEventHandler

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

Repeat until all 93 scenarios pass.
