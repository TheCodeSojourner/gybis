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

λ(naming, PascalCase, entities ∨ rules ∨ triggers ∨ actors ∨ surfaces ∨ contracts)
λ(naming, snake_case, fields ∨ config_keys ∨ derived_values ∨ enum_values)
λ(naming, plural_PascalCase, collections_relationship_fields)

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

λ(field_mod, required, field: Type)
λ(field_mod, optional, field: Type?)
λ(field_mod, state_gated_required, field: Type when status = x)
λ(field_mod, multi_state_gated, field: Type when status = x | y)
λ(field_mod, state_gated_optional, field: Type? when status = x)

λ(when_clause_obligations,
  transitioning_INTO(when_set) → MUST_set_field(in ensures:)
  transitioning_OUT_OF(when_set) → MUST_clear_field(in ensures:)
  requires_reading(when_qualified_field) → MUST_guard_on_state_first
)

---

## Collection Types

λ(collection, Set<T>, unordered_default)
λ(collection, List<T>, explicit_ordering_insert_order_matters)
λ(collection, Sequence<T>, inferred_ordered_relationships)

λ(collection_methods, .count ∨ .any() ∨ .all() ∨ .first ∨ .last ∨ .unique ∨ .add() ∨ .remove())
λ(collection_warning, .first/.last require List<T> ∨ Sequence<T> — Set<T> = warning(future_error))
λ(collection_unique, .unique → Set<T>)

Domain-specific ops — free-standing syntax only:

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

λ(transition_rule, non_terminal → MUST_have(≥1_outbound_edge))
λ(transition_rule, ensures_status_X → MUST_correspond_to_declared_edge)
λ(transition_rule, declared_edge → MUST_have(≥1_witnessing_rule_ensures))
λ(transition_rule, enum_not_in_graph → flagged_unreachable)
λ(transition_rule, terminal_states → ¬outbound_edges ∧ ¬rules_transition_away)

---

## Annotations

λ(annotation, @guidance, rule ∨ surface ∨ top_level → non_normative_impl_advice ∨ excluded_from_semantics)
λ(annotation, @invariant, contract ∨ surface → prose_assertion ∨ checked_by(allium_analyse))
λ(annotation, @guarantee, surface → prose_guarantee_to_facing_actor)