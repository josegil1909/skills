# SOLID Principles

## Overview

SOLID helps structure software to be flexible, maintainable, and testable. These principles reduce coupling and increase cohesion.

## S - Single Responsibility Principle (SRP)

> "A class should have one, and only one, reason to change."

### Problem It Solves
God objects that do everything - hard to test, hard to change, hard to understand.

### How to Apply
Each class handles ONE responsibility. If you find yourself saying "and" when describing what a class does, split it.

```typescript
// BAD: Multiple responsibilities
class Order {
  calculateTotal(): number { ... }
  saveToDatabase(): void { ... }    // Persistence
  generateInvoice(): string { ... } // Presentation
}

// GOOD: Single responsibility each
class Order {
  private items: OrderItem[] = [];

  addItem(item: OrderItem): void { ... }
  calculateTotal(): number { ... }
}

class OrderRepository {
  save(order: Order): Promise<void> { ... }
}

class InvoiceGenerator {
  generate(order: Order): Invoice { ... }
}
```

### Go Example

```go
// BAD: One service handling documents, payments, fiscal hash, and reconciliation
package documents

type Service struct {
    docs, payments, products, settings ports.Repository
}

func (s *Service) SaveDocument(...)      // validation + hash + relations
func (s *Service) AddPayment(...)        // payments + conversion + IGTF
func (s *Service) CloseSale(...)         // full orchestration
func (s *Service) ReconcilePendingPayment(...) // bank reconciliation

// GOOD: Each service has one responsibility
package documents

type DocumentFiscalService struct {
    docs ports.DocumentRepository
}

func (s *DocumentFiscalService) ComputeHash(doc *domain.Document) string
func (s *DocumentFiscalService) ResolveVATRate(ctx context.Context, tenantID string) int64

type DocumentPaymentService struct {
    payments ports.PaymentRepository
}

func (s *DocumentPaymentService) AddPayment(ctx context.Context, p *domain.DocumentPayment) error
func (s *DocumentPaymentService) ReconcilePending(...) error

type DocumentService struct {
    fiscal  *DocumentFiscalService
    payment *DocumentPaymentService
}

func (s *DocumentService) SaveDocument(ctx context.Context, doc *domain.Document) error {
    // delegates to subservices
}
```

### Vue Example

```vue
<!-- BAD: Modal handling payment UI, numpad, split payments, change calculation -->
<script setup>
// 700+ lines handling everything
</script>

<!-- GOOD: Shell orchestrator with focused sub-components -->
<script setup>
import { usePaymentCalculation } from '@/composables/usePaymentCalculation'
import PosSplitPaymentList from './PosSplitPaymentList.vue'
import PosNumpadInput from './PosNumpadInput.vue'
</script>
```

### Detection Questions
- Does this class/module have multiple reasons to change?
- Can I describe it without using "and"?
- Would different stakeholders request changes to different parts?

---

## O - Open/Closed Principle (OCP)

> "Software entities should be open for extension but closed for modification."

### Problem It Solves
Having to modify existing, tested code every time requirements change. Risk of breaking working features.

### How to Apply
Design abstractions that allow new behavior through new classes, not edits to existing ones.

```typescript
// BAD: Must modify to add new shipping
class ShippingCalculator {
  calculate(type: string, value: number): number {
    if (type === 'standard') return value < 50 ? 5 : 0;
    if (type === 'express') return 15;
    // Must add more ifs for new types!
  }
}

// GOOD: Open for extension
interface ShippingMethod {
  calculateCost(orderValue: number): number;
}

class StandardShipping implements ShippingMethod {
  calculateCost(orderValue: number): number {
    return orderValue < 50 ? 5 : 0;
  }
}

class ExpressShipping implements ShippingMethod {
  calculateCost(orderValue: number): number {
    return 15;
  }
}

// Add new shipping by creating new class, not modifying existing
class SameDayShipping implements ShippingMethod {
  calculateCost(orderValue: number): number {
    return 25;
  }
}
```

### Go Example

```go
// BAD: Adding EUR requires editing existing function
func paymentAmountVESCents(currency string, amountCents, rateMicros int64) (int64, error) {
    switch strings.ToUpper(currency) {
    case "VES": return amountCents, nil
    case "USD": return (amountCents*rateMicros + 500_000) / 1_000_000, nil
    // Must add case "EUR" here!
    default: return 0, fmt.Errorf("invalid currency")
    }
}

// GOOD: Registry pattern - new currencies added without modifying existing code
type CurrencyConverter interface {
    ToVES(amountCents int64, rateMicros int64) (int64, error)
}

var converters = map[string]CurrencyConverter{
    "VES": VESConverter{},
    "USD": USDConverter{},
}

// New currency added in separate file, no changes to existing code
type EURConverter struct{}
func (c EURConverter) ToVES(amountCents int64, rateMicros int64) (int64, error) { ... }

func init() {
    converters["EUR"] = EURConverter{}
}
```

### Vue Example

```typescript
// BAD: Adding new payment method requires editing existing component
function isElectronicMethod(method: string) {
    switch(method) {
        case 'card': return true
        case 'transfer': return true
        // Must add more cases!
    }
}

// GOOD: Payment method registry
const methodClassifiers: Record<string, PaymentClassifier> = {
    card: new ElectronicClassifier(),
    transfer: new ElectronicClassifier(),
    cash: new CashClassifier(),
}

// New method added by registering, not editing
methodClassifiers['crypto'] = new CryptoClassifier()
```

### Architectural Insight
OCP at architecture level means: **design your codebase so new features are added by adding code, not changing existing code.**

---

## L - Liskov Substitution Principle (LSP)

> "Subtypes must be substitutable for their base types without altering program correctness."

### Problem It Solves
Subclasses that break expectations, requiring type-checking and special cases.

### How to Apply
Subclasses must honor the contract of the parent. If the parent returns positive numbers, subclasses cannot return negatives.

```typescript
// BAD: Violates parent's contract
class DiscountPolicy {
  getDiscount(value: number): number {
    return 0; // Non-negative expected
  }
}

class WeirdDiscount extends DiscountPolicy {
  getDiscount(value: number): number {
    return -5; // Increases cost! Breaks expectations
  }
}

// GOOD: Enforces contract
class DiscountPolicy {
  constructor(private discount: number) {
    if (discount < 0) throw new Error("Discount must be non-negative");
  }

  getDiscount(): number {
    return this.discount;
  }
}
```

### Go Example

```go
// In-memory and SQL repositories must honor the same contract
// If tests pass with memory repo but fail with SQL, there's an LSP violation

type ProductRepository interface {
    GetByID(ctx context.Context, tenantID, productID string) (*domain.Product, error)
    Save(ctx context.Context, product *domain.Product) error
}

// Both implementations must:
// - Return domain.ErrNotFound when product doesn't exist
// - Validate tenant isolation
// - Return nil product + error on failure, never nil + nil
```

### Nil-Safe LSP in AFY

```go
// Optional audit repository degrades gracefully - documented behavior
type Service struct {
    products ports.ProductRepository
    invoices ports.InvoiceRepository
    audits   ports.AuditRepository  // optional, nil-safe
}

func (s *Service) CreateInvoice(...) {
    // ...
    if s.audits != nil {
        _ = s.audits.Save(ctx, event)  // silently skips if nil
    }
}
```

### Key Insight
This is why you can swap `InMemoryUserRepo` for `PostgresUserRepo` - they both honor the `UserRepo` interface contract.

---

## I - Interface Segregation Principle (ISP)

> "Clients should not be forced to depend on methods they do not use."

### Problem It Solves
Fat interfaces that force partial implementations, empty methods, or throws.

### How to Apply
Split large interfaces into smaller, cohesive ones. Clients depend only on what they need.

```typescript
// BAD: Fat interface
interface WarehouseDevice {
  printLabel(orderId: string): void;
  scanBarcode(): string;
  packageItem(orderId: string): void;
}

class BasicPrinter implements WarehouseDevice {
  printLabel(orderId: string): void { /* works */ }
  scanBarcode(): string { throw new Error("Not supported"); } // Forced!
  packageItem(orderId: string): void { throw new Error("Not supported"); }
}

// GOOD: Segregated interfaces
interface LabelPrinter {
  printLabel(orderId: string): void;
}

interface BarcodeScanner {
  scanBarcode(): string;
}

interface ItemPackager {
  packageItem(orderId: string): void;
}

class BasicPrinter implements LabelPrinter {
  printLabel(orderId: string): void { /* only what it does */ }
}
```

### Go Example

```go
// BAD: Monolithic interface (~20 methods)
type AuthUseCase interface {
    Login, Register, VerifyToken, RevokeToken, RefreshToken,
    RegisterByActor, ListUsersByActor, DeactivateUserByActor,
    ResetPasswordByActor, ChangeOwnPasswordWithCredentials,
    ListAuditByActor, ListImpersonationEventsByActor,
    InvalidateAllSessions, ImpersonateByActor, TenantAccessByActor,
    RotateSigningKey, ListSessionsByActor, InvalidateSessionByActor,
    InvalidateAllUserSessionsByActor
}

// GOOD: Segregated by role/operation
type AuthBasicUseCase interface {
    Login(ctx context.Context, cmd LoginCommand) (string, error)
    Register(ctx context.Context, cmd RegisterUserCommand) (*domain.User, error)
    VerifyToken(ctx context.Context, token string) (*domain.AuthClaims, error)
}

type AuthAdminUseCase interface {
    RegisterByActor(ctx context.Context, actor *AuthClaims, cmd RegisterUserCommand) (*domain.User, error)
    ListUsersByActor(ctx context.Context, actor *AuthClaims, tenantID string, limit, offset int) ([]domain.User, error)
}

type AuthSessionUseCase interface {
    ListSessionsByActor(ctx context.Context, actor *AuthClaims, tenantID string, limit, offset int) ([]domain.UserSession, error)
    InvalidateSessionByActor(ctx context.Context, actor *AuthClaims, tenantID, sessionID string) error
}
```

### Vue Example

```typescript
// BAD: Store with unrelated state
const usePosStore = defineStore('pos', () => {
    const cart = ref([])
    const checkoutStep = ref(1)
    const tickets = ref([])
    const customer = ref(null)
    const payments = ref([])
    // ...
})

// GOOD: Segregated stores
const useCartStore = defineStore('cart', () => { ... })
const useCheckoutStore = defineStore('checkout', () => { ... })
const useTicketStore = defineStore('ticket', () => { ... })
const useCustomerStore = defineStore('customer', () => { ... })
```

### Detection
If you see `throw new Error("Not implemented")` or empty method bodies, the interface is too fat.

---

## D - Dependency Inversion Principle (DIP)

> "High-level modules should not depend on low-level modules. Both should depend on abstractions."

### Problem It Solves
Tight coupling to specific implementations (databases, APIs, frameworks). Hard to test, hard to swap.

### How to Apply
Depend on interfaces, inject implementations.

```typescript
// BAD: Direct dependency on concrete class
class OrderService {
  private emailService = new SendGridEmailService(); // Locked in!

  confirmOrder(email: string): void {
    this.emailService.send(email, "Order confirmed");
  }
}

// GOOD: Depend on abstraction
interface EmailService {
  send(to: string, message: string): void;
}

class OrderService {
  constructor(private emailService: EmailService) {}

  confirmOrder(email: string): void {
    this.emailService.send(email, "Order confirmed");
  }
}

// Now can inject any implementation
new OrderService(new SendGridEmailService());
new OrderService(new SESEmailService());
new OrderService(new MockEmailService()); // For tests!
```

### Go Example

```go
// BAD: Direct dependency on concrete SQL repository
package sales

import "github.com/afy-erp/server/internal/adapters/db_libsql"  // PROHIBITED

type Service struct {
    products *db_libsql.SQLProductRepository  // concrete!
}

// GOOD: Depend on abstraction (ports)
package sales

import "github.com/afy-erp/server/internal/core/ports"

type Service struct {
    products ports.ProductRepository
    invoices ports.InvoiceRepository
    audits   ports.AuditRepository  // optional
}

func NewService(products ports.ProductRepository, invoices ports.InvoiceRepository, audits ...ports.AuditRepository) *Service {
    s := &Service{products: products, invoices: invoices}
    if len(audits) > 0 {
        s.audits = audits[0]
    }
    return s
}
```

### Vue Example

```typescript
// BAD: API layer importing stores
// src/lib/api/http.ts
import { usePosStore } from '@/stores/usePosStore'  // PROHIBITED

// GOOD: Components depend on stores and API
// src/components/pos/PosPaymentModal.vue
import { usePosStore } from '@/stores/usePosStore'     // OK
import { createPayment } from '@/lib/api/payments'      // OK
```

### The Dependency Rule
Source code dependencies should point **inward** toward high-level policies (domain logic), never toward low-level details (infrastructure).

```
Infrastructure → Application → Domain
      ↑              ↑            ↑
    (outer)       (middle)     (inner)

Dependencies flow: outer → inner
Never: inner → outer
```

---

## Applying SOLID at Architecture Level

These principles scale beyond classes:

| Principle | Architecture Application |
|-----------|--------------------------|
| SRP | Each bounded context has one responsibility |
| OCP | New features = new modules, not edits to existing |
| LSP | Microservices with same contract are substitutable |
| ISP | Thin interfaces between services |
| DIP | High-level business logic doesn't know about databases/frameworks |

---

## Quick Reference

| Principle | One-Liner | Red Flag |
|-----------|-----------|----------|
| SRP | One reason to change | "This class handles X and Y and Z" |
| OCP | Add, don't modify | `if/else` chains for types |
| LSP | Subtypes are substitutable | Type-checking in calling code |
| ISP | Small, focused interfaces | Empty method implementations |
| DIP | Depend on abstractions | `new ConcreteClass()` in business logic |
