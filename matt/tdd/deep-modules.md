# Useful Boundaries

Adapted from "A Philosophy of Software Design":

**Useful boundary** = small interface + meaningful hidden complexity

```
┌─────────────────────┐
│   Small Interface   │  <- Few entry points, simple inputs
├─────────────────────┤
│                     │
│ Workflow complexity │  <- Validation, orchestration, retries,
│ and business rules  │     mapping, side effects, error handling
│                     │
└─────────────────────┘
```

**Shallow helper layer** = large interface + little hidden value (avoid)

```
┌─────────────────────────────────┐
│      Large Interface            │  <- Many methods, many params,
│                                 │     exposes too many decisions
├─────────────────────────────────┤
│ Thin Implementation             │  <- Mostly passes through
└─────────────────────────────────┘
```

When designing interfaces, ask:

- Can I give callers one stable entry point for this workflow?
- Can I simplify what the caller must know?
- Can I hide orchestration, validation, retries, mapping, and side effects inside?
- Am I creating a real boundary callers touch, instead of another thin helper layer?

In web apps, useful boundaries often appear as:

- a service object that executes a full workflow
- a route or action helper that owns request-to-domain translation
- an adapter that hides a third-party integration behind one small API
- an end-to-end browser workflow whose value is best verified in `Playwright`
