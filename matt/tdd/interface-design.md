# Interface Design for Testability

Good interfaces make testing natural by giving callers one stable place to exercise behavior.

## Principles

1. **Accept dependencies, don't create them**

   ```typescript
   // Testable
   async function processOrder(order, paymentGateway) {}

   // Harder to test
   async function processOrder(order) {
     const paymentGateway = new StripeGateway();
   }
   ```

2. **Return meaningful results**

   ```typescript
   // Testable
   function calculateDiscount(cart): DiscountResult {}

   // Harder to reason about
   function applyDiscount(cart): void {
     cart.total -= 10;
   }
   ```

3. **Keep the surface area small**

   - fewer entry points
   - fewer required params
   - fewer decisions pushed onto callers

4. **Hide workflow complexity inside the boundary**

   Good boundaries absorb:

   - validation
   - orchestration
   - retries
   - data mapping
   - side-effect sequencing
   - error translation

5. **Design around real callers**

   The best interface often matches one real workflow:

   - submit checkout
   - reset password
   - accept invite
   - save draft
   - publish post

## Anti-Pattern

Do not create a thin helper layer that merely forwards data between other helpers. If the caller still has to know the whole workflow, the interface is not doing enough.

## Web-App Heuristic

Useful interfaces in web apps often look like:

- a route or action helper that translates request input into domain input
- a service object that owns an end-to-end server workflow
- an adapter that hides a third-party system behind a focused API
- a browser-observable workflow whose behavior is best verified in `Playwright`

These skills generally do **not** treat React components or hooks as default unit-test targets. Prefer testing the server/workflow boundary in `Vitest`, or the user-visible browser behavior in `Playwright`.
