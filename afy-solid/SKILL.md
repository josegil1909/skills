---
name: afy-solid
description: Apply SOLID principles, clean architecture, and clean code to AFY ERP (Go + Vue/Pinia + Tauri).
---

# AFY SOLID — Professional Software Engineering

> Stack: Go (backend) + Vue 3 / Pinia / Tauri (frontend)
> Architecture: Clean / Hexagonal
> Base: Forked from `ramziddin/solid-skills`

You are operating as a senior software engineer for AFY ERP. Every line of code, design decision, and refactoring must embody professional craftsmanship.

## When This Skill Applies

**ALWAYS use this skill when:**
- Writing or modifying code in `server/` (Go) or `client/` (Vue)
- Refactoring existing modules
- Planning or designing architecture
- Reviewing code quality
- Debugging issues
- Creating tests

## Core Philosophy

> "Code that handles real money, fiscal data, and multi-tenancy cannot be a monolith."

The 70% of ERP value is in `internal/core/` and `internal/modules/`. Protecting that layer matters more than optimizing adapters.

---

## 1. ALWAYS Start with Tests (TDD)

```
RED    → Write a failing test
GREEN  → Minimum code to pass
REFACTOR → Clean up, apply SOLID
```

**The Three Laws of TDD:**
1. You cannot write production code unless it makes a failing test pass
2. You cannot write more test code than is sufficient to fail
3. You cannot write more production code than is sufficient to pass

**Design happens during REFACTORING, not during coding.**

**Go:** table-driven tests with `t.Run()` subtests.
**Vue:** Vitest for composables, Playwright for critical flows.

See: [references/tdd.md](references/tdd.md)

---

## 2. Apply SOLID Principles Rigorously

| Principle | Question to Ask |
|-----------|-----------------|
| **S**RP - Single Responsibility | "Does this have ONE reason to change?" |
| **O**CP - Open/Closed | "Can I extend without modifying?" |
| **L**SP - Liskov Substitution | "Can subtypes replace base types safely?" |
| **I**SP - Interface Segregation | "Are clients forced to depend on unused methods?" |
| **D**IP - Dependency Inversion | "Do high-level modules depend on abstractions?" |

See: [references/solid-principles.md](references/solid-principles.md)

### SRP — Size Limits

| Layer | Soft limit | Hard limit |
|-------|-----------|------------|
| Go service | 150 LOC | 200 LOC |
| HTTP handler | 30 LOC | 50 LOC |
| Vue SFC | 200 LOC | 300 LOC |
| Pinia store | 100 LOC | 150 LOC |
| Composable | 100 LOC | 150 LOC |

---

## 3. Write Clean, Human-Readable Code

**Naming (in order of priority):**
1. **Consistency** — same concept = same name everywhere
2. **Domain language** — `Invoice`, `Withholding`, `Entity` (not `data`, `info`, `manager`)
3. **Specificity** — `ResolveDocumentVATRateBps` (not `GetRate`)
4. **Brevity** — short but not cryptic
5. **Searchability** — unique, greppable

**Go conventions:**
- Public: `Save()`, `Validate()`
- Private: `resolveRefundMode()`, `computeFiscalHash()`
- Interfaces: `ProductRepository`, `SalesUseCase`
- Constructors: `NewService(...)`, `NewSQLProductRepository(...)`
- Errors: lowercase, no trailing punctuation

**Vue conventions:**
- Components: `PascalCase.vue` — `PosPaymentModal.vue`
- Composables: `useCamelCase.ts` — `usePaymentCalculation.ts`
- Stores: `useDomainStore.ts` — `usePosStore.ts`

**Structure:**
- **Go:** early returns, no `else` when possible, max 2 indentation levels
- **Vue:** `<script setup>` required, composables for reusable logic
- **Both:** Law of Demeter — one dot per line

See: [references/clean-code.md](references/clean-code.md)

---

## 4. Design with Responsibility in Mind

**Object Stereotypes:**
- **Information Holder** — domain entities (`domain.Invoice`, `domain.Product`)
- **Service Provider** — use cases (`sales.Service`, `documents.Service`)
- **Coordinator** — orchestrates multiple services
- **Interfacer** — HTTP handlers, repository adapters

**Ask for every file:**
1. "What pattern is this?"
2. "Is it doing too much?"

See: [references/object-design.md](references/object-design.md)

---

## 5. Manage Complexity Ruthlessly

**Essential complexity** = inherent to the problem domain
**Accidental complexity** = introduced by our solutions

**Detect complexity through:**
- Change amplification (small change = many files)
- Cognitive load (hard to understand)
- Unknown unknowns (surprises in behavior)

**Fight complexity with:**
- YAGNI — Don't build what you don't need NOW
- KISS — Simplest solution that works
- DRY — But only after Rule of Three

See: [references/complexity.md](references/complexity.md)

---

## 6. Architect for Change

**Vertical Slicing:**
- `modules/sales/` → sales feature
- `modules/purchases/` → purchases feature
- `modules/inventory/` → inventory feature

**Horizontal Decoupling:**
```
Infrastructure → Application → Domain
      ↑               ↑            ↑
    outer          middle         inner
```

**Dependency Rule:** Source code dependencies point **inward** toward high-level policies. Never reverse.

**Prohibited imports:**
```go
// Domain importing infrastructure
package domain
import "internal/adapters/db_libsql"  // NEVER

// Modules importing HTTP framework
package sales
import "github.com/gofiber/fiber/v3"  // NEVER
```

See: [references/architecture.md](references/architecture.md)

---

## 7. Code Smell Detection

**Critical smells — stop and refactor:**

| Smell | Limit | Solution |
|-------|-------|----------|
| God file | > 1000 LOC | Split by domain |
| God service | > 200 LOC | Extract subservices |
| God store | > 150 LOC | Split into focused stores |
| Type switches | `if/switch` on types | Strategy pattern |
| Fat interface | > 10 methods | Segregate by role |

See: [references/code-smells.md](references/code-smells.md)

---

## 8. Design Patterns Awareness

**Creational:** Factory, Builder
**Structural:** Adapter, Repository
**Behavioral:** Strategy, Command

**Warning:** Don't force patterns. Let them emerge from refactoring.

See: [references/design-patterns.md](references/design-patterns.md)

---

## 9. Testing Strategy

**Test Types (inner to outer):**
1. **Unit Tests** — single function, fast, isolated
2. **Integration Tests** — multiple components
3. **E2E Tests** — full system, user perspective

**Arrange-Act-Assert:**
```go
// Arrange
s := NewService(mockProducts, mockInvoices)

// Act
invoice, err := s.CreateInvoice(ctx, cmd)

// Assert
assert.NoError(t, err)
assert.Equal(t, expectedTotal, invoice.TotalCents)
```

See: [references/testing.md](references/testing.md)

---

## Checklists

### Pre-Code
1. [ ] Understand the requirement
2. [ ] What test will I write first?
3. [ ] Is this the simplest solution?
4. [ ] Will this file exceed its limit?
5. [ ] Real problem or hypothetical? (YAGNI)

### During Code
1. [ ] Simplest thing that could work?
2. [ ] Single responsibility?
3. [ ] Depending on abstractions?
4. [ ] Clear names?
5. [ ] Duplication to extract? (Rule of Three)
6. [ ] Adding `if` for a new type? → Consider Strategy

### Post-Code
1. [ ] All tests pass? (`go test ./...`, `bun run test`)
2. [ ] Dead code removed?
3. [ ] Complex conditions simplified?
4. [ ] Names still accurate?
5. [ ] Would a junior understand this in 6 months?
6. [ ] File within size limit?

---

## Red Flags — Stop and Rethink

- Writing code without a test
- File exceeding layer limit
- Adding `if` for a new type/method/currency
- Handler with > 50 LOC
- Service with > 3 repository dependencies
- Injecting full `Dependencies` into handler that needs 2 interfaces
- Creating abstractions before 3rd duplication
- Adding features "just in case"

---

## Remember

> "A little bit of duplication is 10x better than the wrong abstraction."

> "Focus on WHAT needs to happen, not HOW it needs to happen."

> "Design principles become second nature through practice."
