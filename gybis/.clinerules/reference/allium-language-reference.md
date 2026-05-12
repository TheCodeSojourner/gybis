# Allium v3 Lang Ref

---

## File Structure (Required Order)

```
-- allium: 3                  ← Version marker. MUST be first line.
module <name>                 ← Optional
use "<coordinate>" as <alias> ← Imports
given { <binding>: <Type> }   ← Module-scoped shared bindings

external entity declarations
value type declarations
contract declarations
enum declarations
entity/variant declarations
config block
default declarations
rule declarations
invariant declarations
actor declarations
surface declarations
deferred declarations
open question declarations
```

---

## Naming

| Kind                                                   | Convention          | Exs                              |
| ------------------------------------------------------ | ------------------- | -------------------------------- |
| Entities, Rules, Triggers, Actors, Surfaces, Contracts | `PascalCase`        | `Order`, `ShipOrder`, `Customer` |
| Fields, Config keys, Derived values, Enum values       | `snake_case`        | `shipped_at`, `max_retries`      |
| Collections (relationship fields)                      | Plural `PascalCase` | `Orders`, `LineItems`            |

---

## Entity

Domain concept with identity + optional lifecycle.

```allium
entity Order {
    status: pending | confirmed | shipped | delivered | cancelled
    customer: Customer
    total: Decimal

    notes: String?
    tracking_number: String when status = shipped | delivered
    shipped_at: Timestamp when status = shipped | delivered
    cancellation_reason: String? when status = cancelled

    transitions status {
        pending -> confirmed
        pending -> cancelled
        confirmed -> shipped
        confirmed -> cancelled
        shipped -> delivered
        terminal: delivered, cancelled
    }

    line_items: LineItem with order = this
    active_items: line_items where status != removed
    item_count: line_items.count
    tracking_url: tracking_base + tracking_number when status = shipped | delivered

    invariant NonNegativeTotal { this.total >= 0 }
}
```

## External Entity

Third-party/boundary concept — referenced but not owned.

```allium
external entity PaymentGateway {
    endpoint: URL
}
```

## Value Type

Immutable structured data, no identity, no lifecycle.

```allium
value Address {
    street: String
    city: String
    postcode: String
    country: Country
}
```

## Variant

Entity subtype extending base with extra fields.

```allium
variant SubscriptionOrder : Order {
    plan: SubscriptionPlan
    renewal_date: Date
}
```

## Enum

Fixed set of symbolic values.

```allium
enum Country { us | gb | de | fr }
enum Language { en | de | `zh-Hant-TW` | `de-CH-1996` }
```

---

## Rule

Behavior triggered by condition. Every rule MUST have `when:`, `requires:`, `ensures:`.

```allium
rule ShipOrder {
    -- Trigger forms (pick one):
    when: ShipOrder(order, tracking)              -- explicit event
    when: order: Order.status transitions_to shipped  -- field transition
    when: order: Order.status becomes confirmed    -- value change
    when: order: Order.created                     -- entity creation
    when: order: Order.expires_at <= now           -- temporal

    for item in order.line_items where item.status = pending:

    let fee = order.total * config.shipping_rate

    requires: order.status = confirmed
    requires: order.customer.emailVerified
    requires: tracking != null implies tracking.length > 0

    ensures: order.status = shipped
    ensures: order.trackingNumber = tracking
    ensures: order.shippedAt = now
    ensures: Email.created(to: order.customer.email, template: shipment_notice)

    if order.total >= config.freeShippingThreshold:
        ensures: order.shippingFee = 0

    traces: src/orders/ship.ts#shipOrder
    @guidance -- Use idempotency key on shipping provider call
}
```

---

## Module Invariant

Constraint across entire domain.

```allium
invariant NoOrphanedItems {
    for item in LineItems: item.order != null
}
```

---

## Config

Typed config params with defaults. Ref as `config.<key>`.

```allium
config {
    maxRetries: Integer = 3
    sessionTTL: Duration = 30.minutes
    failureThreshold: Decimal = 0.5
    freeShippingThreshold: Decimal = 50.00
    alertThreshold: Decimal = config.failureThreshold * 0.8
    apiTimeout: Duration = payments/config.timeout
}
```

---

## Default

Default field values on entity creation.

```allium
default Order newOrder = {
    status: pending
    total: 0
}
```

---

## Contract

Reusable obligation for external systems. Ref from surfaces.

```allium
contract PaymentProcessor {
    charge: (amount: Decimal, method: PaymentMethod) -> Receipt
    refund: (receipt: Receipt) -> RefundConfirmation
    @invariant Idempotency
        -- Same charge + same idempotency key → same receipt
}
```

---

## Surface

Boundary contract between system + named actor. Declares visibility + actions.

```allium
surface CustomerOrderView {
    facing customer: Customer
    context order: Order where order.customer = viewer
    let viewer = customer

    exposes:
        order.status
        order.trackingNumber when order.status = shipped | delivered

    provides:
        CustomerCancelsOrder(customer, order) when order.status = pending

    contracts:
        demands PaymentProcessor
        fulfils NotificationService

    @guarantee DataMinimization
        -- Only fields in exposes accessible; no internal fields leak

    related:
        CustomerOrderHistory(customer) when order.status = delivered

    timeout:
        OrderExpires when order.expires_at <= now

    @guidance -- Cache 30s; never return stale cancellation state
}
```

---

## Actor

Named participant identified by predicate on entity.

```allium
actor AdminUser { identified_by: User where user.role = admin }
actor TenantMember { identified_by: User where user.tenantId != null }
```

---

## Given

Shared entity bindings visible to all rules/surfaces in module.

```allium
given { tenant: Tenant }
```

---

## Use

Import from another spec module.

```allium
use "payments/billing" as billing
use "auth/users" as users
-- Ref: users.User, billing.Receipt
```

---

## Deferred

Placeholder for future work.

```allium
deferred Order.refundPolicy
deferred PaymentProcessor.disputeHandling
```

---

## Open Question

Unresolved design decision inline in spec.

```allium
open question "Should cancelled order reopen within 24h?"
open question "In-flight shipments on account suspend?"
```

---

## Field Modifier Ref

| Syntax                             | Meaning                            |
| ---------------------------------- | ---------------------------------- |
| `field: Type`                      | Required, always present           |
| `field: Type?`                     | Optional                           |
| `field: Type when status = x`      | Required when status is `x`        |
| `field: Type when status = x \| y` | Required when status is `x` or `y` |
| `field: Type? when status = x`     | Optional + state-gated             |

**When-clause obligations** (enforced by `allium check`):
- Rule transitioning INTO `when` set → MUST set field in `ensures:`
- Rule transitioning OUT of `when` set → MUST clear field in `ensures:`
- `requires:` reading `when`-qualified field → MUST guard on state first

---

## Collection Types

| Type          | Ordering  | Use When                          |
| ------------- | --------- | --------------------------------- |
| `Set<T>`      | Unordered | Default; order doesn't matter     |
| `List<T>`     | Explicit  | Insertion order matters           |
| `Sequence<T>` | Inferred  | Produced by ordered relationships |

**Built-in dot-methods:** `.count` `.any()` `.all()` `.first` `.last` `.unique` `.add()` `.remove()`

> `.first`/`.last` require `List<T>` or `Sequence<T>` — `Set<T>` = warning (error in future).
> `.unique` → `Set<T>`.

**Domain-specific ops** — free-standing syntax only:

```allium
-- CORRECT:
filter(events, e => e.occurredAt > cutoff)
hash(password)
verify(password, storedHash)
rank(candidates, by: score)

-- WRONG:
events.filter(e => ...)
password.hash()
```

---

## Transition Graph Rules (enforced by `allium check`)

- Non-terminal state → MUST have ≥1 outbound edge
- `ensures: entity.status = X` → MUST correspond to declared edge
- Every declared edge → MUST be witnessed by ≥1 rule `ensures:`
- Enum values not in graph → flagged unreachable
- `terminal:` states: no outbound edges; rules cannot transition away

---

## Annotations

| Annotation        | Context                  | Meaning                                            |
| ----------------- | ------------------------ | -------------------------------------------------- |
| `@guidance`       | Rule, Surface, top-level | Non-normative impl advice; excluded from semantics |
| `@invariant Name` | Contract, Surface        | Prose assertion; checked by `allium analyse`       |
| `@guarantee Name` | Surface                  | Prose guarantee to facing actor                    |