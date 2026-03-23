# When to Mock

Mock at **system boundaries** only:

- Third-party APIs and SDKs
- HTTP boundaries, preferably with `MSW`
- Time, randomness, and process environment
- File system or OS interactions when needed
- Databases only when no realistic local test setup exists

Prefer not to mock:

- Your own services, modules, or helpers
- Internal collaborators inside the same workflow
- Code you control and can run directly in tests

## Boundary Rule

Mock where ownership stops.

If you own both sides of the behavior, prefer a real test path. If the dependency is external, unstable, slow, expensive, or hard to reproduce locally, mock or substitute it at the boundary.

For HTTP calls, prefer `MSW` over mocking `fetch` directly. Stub the network boundary, keep your client code real.

## Prefer Substitutes Over Mocks

When possible, use local stand-ins instead of mocks:

- test database over mocked repository
- in-memory adapter over mocked internal service
- fake queue or mailer over per-call expectations
- `MSW` handlers over `fetch` mocks

This keeps tests behavior-focused and less coupled to wiring.

## Designing for Mockability

At external boundaries, design interfaces that are easy to replace.

**1. Use dependency injection**

```typescript
// Easy to replace in tests
async function processPayment(order, paymentGateway) {
  return paymentGateway.charge(order.total);
}

// Hard to replace in tests
async function processPayment(order) {
  const gateway = new StripeGateway(process.env.STRIPE_KEY);
  return gateway.charge(order.total);
}
```

**2. Prefer task-specific interfaces over generic transport wrappers**

```typescript
// GOOD: Specific operations, clear behavior
const billingApi = {
  createInvoice: (input) => client.post("/invoices", input),
  refundCharge: (chargeId) => client.post(`/charges/${chargeId}/refund`),
};

// BAD: One generic wrapper, behavior hidden in callers
const billingApi = {
  request: (path, options) => client.request(path, options),
};
```

Task-specific boundaries make tests simpler:

- each fake returns one specific shape
- setup reflects real use cases
- fewer conditionals in test doubles
- easier to see what behavior is under test

## Browser Testing Stance

Prefer real browser tests with `Playwright` for DOM behavior and user interactions. Avoid `jsdom` or other simulated browser environments unless there is a strong, explicit reason to use them.

## Web-App Heuristic

In web apps, do not mock across boundaries you are explicitly trying to define. If the test target is `checkoutService.submitOrder`, stub Stripe with `MSW` or inject a focused fake client if needed, but do not mock `validateCart`, `priceCart`, or `saveOrder` if those are part of the workflow you own.
