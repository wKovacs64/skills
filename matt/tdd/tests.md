# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real caller-facing boundaries, not mocks of internal parts.

```typescript
// GOOD: Tests observable behavior through a real boundary
test("user can complete checkout with a valid cart", async () => {
  const result = await checkoutService.submitOrder({
    cart: validCart,
    paymentMethod: validPaymentMethod,
  });

  expect(result.status).toBe("confirmed");
});
```

```typescript
// GOOD: Tests an API boundary in Vitest
test("POST /api/orders creates an order", async () => {
  const response = await app.request("/api/orders", {
    method: "POST",
    body: JSON.stringify({ productId: "abc", quantity: 2 }),
    headers: { "content-type": "application/json" },
  });

  expect(response.status).toBe(201);
  await expect(response.json()).resolves.toMatchObject({
    order: { productId: "abc", quantity: 2 },
  });
});
```

```typescript
// GOOD: Tests browser behavior in Playwright
test("customer can complete checkout", async ({ page }) => {
  await page.goto("/checkout");
  await page.getByLabel("Email").fill("a@b.com");
  await page.getByRole("button", { name: "Submit order" }).click();

  await expect(page.getByText("Order confirmed")).toBeVisible();
});
```

Characteristics:

- Tests behavior users or callers care about
- Uses a public API or real workflow boundary only
- Survives internal refactors
- Describes WHAT, not HOW
- Keeps setup aligned with realistic usage

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

```typescript
// BAD: Tests internal wiring
test("checkout calls paymentService.process", async () => {
  const paymentService = { process: vi.fn() };

  await submitOrder(validCart, paymentService);

  expect(paymentService.process).toHaveBeenCalledTimes(1);
});
```

Red flags:

- Mocking internal collaborators you own
- Testing private methods or helper functions instead of the real boundary
- Asserting on call counts or call order when behavior is what matters
- Test breaks when refactoring without behavior change
- Test name describes HOW, not WHAT
- Verifying through a side channel instead of the interface

```typescript
// BAD: Bypasses the boundary to verify
test("createUser saves a row", async () => {
  const user = await createUser({ name: "Alice" });
  const row = await db.query("select * from users where id = ?", [user.id]);

  expect(row).toBeDefined();
});

// GOOD: Verifies through the interface a caller would use
test("created users can be retrieved", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);

  expect(retrieved?.name).toBe("Alice");
});
```

```tsx
// BAD: Tests component implementation details
test("CheckoutForm sets internal state on change", () => {
  const setState = vi.fn();
  vi.spyOn(React, "useState").mockReturnValue(["", setState]);
  // ...render and type
  expect(setState).toHaveBeenCalled();
});
```

```typescript
// BAD: Tests hook internals instead of user-visible behavior
test("useCart stores items in reducer state", () => {
  const result = useCartForTest();
  result.addItem({ id: "a", price: 10 });
  expect(result._state.itemMap["a"].qty).toBe(1);
});
```

## Web-App Heuristic

In web apps, good tests usually exercise one of these boundaries:

- a route, action, or request handler
- a server-side workflow or service object
- a browser workflow exercised in `Playwright`
- a third-party integration adapter

If a test needs 3 mocks and 4 imported helpers to prove one user-visible behavior, the boundary is probably too low-level.

Preferred stack for these skills:

- `Vitest` for complex or important logic and server-side workflow boundaries
- `Playwright` for real browser interactions
- `MSW` for HTTP stubbing when needed
- avoid `jsdom`-style fake browser tests unless there is a very strong reason
