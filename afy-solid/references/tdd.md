# Test-Driven Development

## The Core Loop

```
RED → GREEN → REFACTOR → RED → ...
```

### RED Phase
Write a failing test that describes the behavior you want. The test should:
- Use domain language, not technical jargon
- Describe WHAT, not HOW
- Be a concrete example, not an abstract statement

```typescript
// BAD: Abstract
it('can add numbers', () => { ... });

// GOOD: Concrete example
it('when adding 2 + 3, returns 5', () => { ... });
```

**Go example:**
```go
// BAD: Abstract
t.Run("can create invoice", func(t *testing.T) { ... })

// GOOD: Concrete example
t.Run("when creating invoice with 2 products, returns total of both", func(t *testing.T) { ... })
```

**Vue example:**
```typescript
// BAD: Abstract
it('can add to cart', () => { ... })

// GOOD: Concrete example
it('when adding product with price 1000, cart total becomes 1000', () => { ... })
```

### GREEN Phase
Write the **simplest possible code** to make the test pass. Two strategies:

1. **Fake It** - Return a hardcoded value
   ```typescript
   add(a: number, b: number): number {
     return 5; // Simplest thing!
   }
   ```

2. **Obvious Implementation** - If you know the solution
   ```typescript
   add(a: number, b: number): number {
     return a + b;
   }
   ```

**Prefer Fake It** when learning or unsure. Let more tests drive the real implementation.

### REFACTOR Phase
This is where **design happens**. Look for:
- Duplication (but wait for Rule of Three)
- Long methods to extract
- Poor names to improve
- Complex conditions to simplify

## The Three Laws of TDD

1. **No production code** without a failing test
2. **No more test code** than sufficient to fail (compilation failures count)
3. **No more production code** than sufficient to pass the one failing test

## The Rule of Three

**Only extract duplication when you see it THREE times.**

Why? Wrong abstractions are worse than duplication. Wait for the pattern to emerge.

```typescript
// Duplication #1 - Leave it
// Duplication #2 - Note it, leave it
// Duplication #3 - NOW extract it
```

## Triangulation

Each new test "sculpts" the solution toward a general, robust implementation.

Think of **degrees of freedom** - like a car that needs forward/back, left/right, and rotation. Each test carves out one degree of freedom until the implementation handles all cases.

## Transformation Priority Premise

When going from RED to GREEN, prefer simpler transformations:

| Priority | Transformation |
|----------|----------------|
| 1 | {} → nil |
| 2 | nil → constant |
| 3 | constant → variable |
| 4 | unconditional → conditional |
| 5 | scalar → collection |
| 6 | statement → recursion |
| 7 | value → mutated value |

Higher priority = simpler. Avoid jumping to complex transformations too early.

## Arrange-Act-Assert

Structure every test:

```typescript
it('calculates total with discount', () => {
  // ARRANGE - Set up the world
  const order = new Order();
  order.addItem({ price: 100 });
  const discount = new PercentDiscount(10);

  // ACT - Execute the behavior
  const total = order.calculateTotal(discount);

  // ASSERT - Verify the outcome
  expect(total).toBe(90);
});
```

**Go — Table-driven tests:**
```go
func TestCreateInvoice(t *testing.T) {
    tests := []struct {
        name      string
        lines     []InvoiceLineInput
        wantTotal int64
        wantErr   bool
    }{
        {
            name:      "single product",
            lines:     []InvoiceLineInput{{ProductID: "p1", Quantity: 1}},
            wantTotal: 1000,
        },
        {
            name:      "multiple products",
            lines:     []InvoiceLineInput{{ProductID: "p1", Quantity: 2}, {ProductID: "p2", Quantity: 1}},
            wantTotal: 3000,
        },
        {
            name:    "empty lines",
            lines:   []InvoiceLineInput{},
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            svc := NewService(mockProducts, mockInvoices)
            invoice, err := svc.CreateInvoice(ctx, cmd)

            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.wantTotal, invoice.TotalCents)
        })
    }
}
```

**Vue — Composable tests:**
```typescript
import { describe, it, expect } from 'vitest'
import { usePaymentCalculation } from '@/composables/usePaymentCalculation'

describe('usePaymentCalculation', () => {
    it('when total is 1000 and paid 600, pending is 400', () => {
        const { pending } = usePaymentCalculation()
        pending.value = calculatePending(1000, 600)
        expect(pending.value).toBe(400)
    })
})
```

## Writing Tests Backwards

Sometimes it helps to write AAA in reverse:
1. Write the ASSERT first - what do you want to verify?
2. Write the ACT - what action produces that result?
3. Write the ARRANGE - what setup is needed?

## Test Naming Principles

- Use **behavior-driven names** with domain language
- Provide **concrete examples**, not abstract statements
- **One example per test** for easy debugging
- Avoid leaking implementation details

```typescript
// BAD: Technical, implementation-focused
it('should set the data property to 1', () => { ... });

// GOOD: Behavior-focused, domain language
it('should recognize "mom" as a palindrome', () => { ... });
```

## Classic vs Mockist TDD

**Classic (Detroit/Chicago) TDD:**
- Test with real dependencies
- Higher confidence, slower tests
- Best for: Pure functions, integration tests

**Mockist (London) TDD:**
- Mock external dependencies
- Faster tests, more isolated
- Best for: Classes with infrastructure dependencies

Start with Classic TDD to learn the technique. Add mocks when testing code with databases, APIs, etc.

## Common Mistakes

1. **Writing code before tests** - Violates the fundamental principle
2. **Writing too much test** - Just enough to fail
3. **Writing too much code** - Just enough to pass
4. **Skipping refactor** - This is where design lives
5. **Testing implementation** - Test behavior, not how it's done
6. **Abstract test names** - Use concrete examples
7. **Extracting too early** - Wait for Rule of Three
