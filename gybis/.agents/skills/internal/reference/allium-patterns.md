# Allium v3 Patterns

Common patterns for idiomatic Allium v3 specs. See `allium-language-reference.md` for full syntax.

---

## Pattern 1: Lifecycle Entity + Full Transition Graph

Entity with lifecycle status field, state-dependent fields, complete transition graph. Most common pattern.

```allium
-- allium: 3

entity Order {
    status: pending | confirmed | shipped | delivered | cancelled
    customer: Customer
    total: Decimal

    confirmed_at: Timestamp when status = confirmed | shipped | delivered
    tracking_number: String when status = shipped | delivered
    shipped_at: Timestamp when status = shipped | delivered
    cancelled_at: Timestamp when status = cancelled
    cancellation_reason: String? when status = cancelled

    transitions status {
        pending -> confirmed
        pending -> cancelled
        confirmed -> shipped
        confirmed -> cancelled
        shipped -> delivered
        terminal: delivered, cancelled
    }

    invariant NonNegativeTotal { this.total >= 0 }
}

rule ConfirmOrder {
    when: ConfirmOrder(order)
    requires: order.status = pending
    requires: order.total > 0
    ensures: order.status = confirmed
    ensures: order.confirmedAt = now
    traces: src/orders/confirm.ts#confirmOrder
}

rule CancelOrder {
    when: CustomerCancels(order, reason)
    requires: order.status in {pending, confirmed}
    ensures: order.status = cancelled
    ensures: order.cancelledAt = now
    ensures: order.cancellationReason = reason
}
```

---

## Pattern 2: State-Dependent Fields (When-Clause Obligations)

Field `when`-qualified → rule transitioning IN must set, rule transitioning OUT must clear.

```allium
entity Ticket {
    status: open | in_progress | resolved | closed
    assignee: Agent? when status = in_progress | resolved
    resolved_at: Timestamp when status = resolved | closed
    resolution_note: String when status = resolved | closed

    transitions status {
        open -> in_progress
        in_progress -> resolved
        in_progress -> open        -- re-open
        resolved -> closed
        resolved -> in_progress   -- re-open after review
        terminal: closed
    }
}

rule AssignTicket {
    when: AssignTicket(ticket, agent)
    requires: ticket.status = open
    ensures: ticket.status = in_progress
    ensures: ticket.assignee = agent         -- MUST set on enter in_progress
}

rule ReOpenTicket {
    when: ReOpenTicket(ticket)
    requires: ticket.status = in_progress
    ensures: ticket.status = open
    ensures: ticket.assignee = null          -- MUST clear on exit in_progress
}
```

---

## Pattern 3: Actor + Surface Boundary Contract

Define who sees what + actions they can take. `@guarantee` for behavioral promises visible to actor.

```allium
actor Customer { identified_by: User where user.accountType = customer }
actor SupportAgent { identified_by: User where user.role = support }

surface CustomerTicketView {
    facing customer: Customer
    context ticket: Ticket where ticket.customer = viewer
    let viewer = customer

    exposes:
        ticket.status
        ticket.resolvedAt when ticket.status = resolved | closed
        ticket.resolutionNote when ticket.status = resolved | closed

    provides:
        CustomerCancels(customer, ticket) when ticket.status = open | in_progress

    @guarantee NoInternalDataLeak
        -- Customer sees only exposed fields; assignee + internal notes never exposed

    @guidance -- Cache ticket status 10s; invalidate on status change
}

surface AgentTicketView {
    facing agent: SupportAgent
    context ticket: Ticket

    exposes:
        ticket.status
        ticket.assignee
        ticket.resolutionNote when ticket.status = resolved | closed

    provides:
        AssignTicket(agent, ticket) when ticket.status = open
        ReOpenTicket(ticket) when ticket.status = resolved
}
```

---

## Pattern 4: External Dependency Contract

Declare external systems as contracts — obligations explicit.

```allium
external entity EmailProvider {
    endpoint: URL
}

contract EmailService {
    send: (to: Email, template: EmailTemplate, context: Map<String, String>) -> MessageId
    @invariant DeliveryTracking
        -- Every sent msg has unique MessageId for delivery tracking
}

contract PaymentGateway {
    charge: (amount: Decimal, currency: Currency, method: PaymentMethod) -> ChargeReceipt
    refund: (charge_id: ChargeId, amount: Decimal) -> RefundReceipt
    @invariant Idempotency
        -- Same charge + same idempotency key → same ChargeReceipt
}
```

---

## Pattern 5: Multi-Module Spec (Use Imports)

Split large domain into modules, reference across them.

```allium
-- allium: 3
-- File: specs/billing/subscription.allium

use "auth/users" as users
use "billing/payment-methods" as payments

entity Subscription {
    customer: users.User
    plan: SubscriptionPlan
    status: active | paused | cancelled
    payment_method: payments.PaymentMethod

    nextBillingDate: Date when status = active
    cancelledAt: Timestamp when status = cancelled

    transitions status {
        active -> paused
        active -> cancelled
        paused -> active
        paused -> cancelled
        terminal: cancelled
    }
}
```

---

## Pattern 6: Config-Driven Constraints

Externalize thresholds/limits into `config` — no rule logic changes needed.

```allium
config {
    maxLoginAttempts: Integer = 5
    lockoutDuration: Duration = 15.minutes
    sessionTTL: Duration = 8.hours
    freeShippingThreshold: Decimal = 50.00
    failureWindow: Duration = 5.minutes
    warningThreshold: Integer = config.maxLoginAttempts - 1
}

rule LockAccount {
    when: account: Account.failedAttempts becomes config.maxLoginAttempts
    requires: account.status = active
    ensures: account.status = locked
    ensures: account.lockedUntil = now + config.lockoutDuration
}
```

---

## Pattern 7: Derived Values + Projections

Compute values from relationships — no redundant storage.

```allium
entity ShoppingCart {
    customer: Customer
    status: active | checked_out | abandoned

    items: CartItem with cart = this
    active_items: items where status = active
    removed_items: items where status = removed

    item_count: active_items.count
    subtotal: active_items.sum(item => item.price * item.quantity)
    has_items: active_items.any()

    discount_amount: subtotal * config.memberDiscountRate when customer.isMember
}
```

---

## Pattern 8: Temporal Triggers

Rules firing on time — scheduled behaviors, expiry, timeouts.

```allium
entity Session {
    user: User
    status: active | expired
    createdAt: Timestamp
    expiresAt: Timestamp

    transitions status {
        active -> expired
        terminal: expired
    }
}

rule ExpireSession {
    when: session: Session.expiresAt <= now
    requires: session.status = active
    ensures: session.status = expired
    @guidance -- Run via scheduled job every minute; do not rely on lazy expiry
}

entity Interview {
    status: scheduled | in_progress | completed | expired
    scheduledAt: Timestamp
    duration: Duration
    endsAt: scheduledAt + duration

    transitions status {
        scheduled -> in_progress
        in_progress -> completed
        scheduled -> expired           -- missed
        terminal: completed, expired
    }
}

rule InterviewExpires {
    when: interview: Interview.endsAt <= now
    requires: interview.status in {scheduled, in_progress}
    ensures: interview.status = expired
}
```

---

## Pattern 9: Black Box Functions (Free-Standing)

Domain ops opaque to spec — free-standing call syntax, not dot-methods.

```allium
rule RegisterUser {
    when: RegisterUser(email, password)
    requires: isValidEmail(email)
    requires: meetsPasswordPolicy(password)
    ensures: User.created(
        email: email,
        passwordHash: hash(password),   -- free-standing: hash(x) not x.hash()
        status: pendingVerification
    )
}

rule VerifyPassword {
    when: LoginAttempt(user, password)
    requires: user.status = active
    requires: verify(password, user.passwordHash)   -- free-standing predicate
    ensures: Session.created(user: user, expiresAt: now + config.sessionTTL)
}

rule RankCandidates {
    when: RankCandidates(jobPosting)
    requires: jobPosting.status = acceptingApplications
    let ranked = rank(jobPosting.applicants, by: score)   -- free-standing, not .rank()
    ensures: jobPosting.rankedApplicants = ranked
}
```

---

## Pattern 10: Entity-Level + Module-Level Invariants

Encode properties that hold regardless of which rule fires.

```allium
entity Account {
    balance: Decimal
    status: active | suspended | closed

    -- Entity-level: checked by allium analyse for every rule touching entity
    invariant NonNegativeBalance { this.balance >= 0 }
    invariant ClosedAccountsHaveNoBalance { this.status = closed implies this.balance = 0 }
}

-- Module-level: checked across all entities
invariant AllOrdersHaveCustomers {
    for order in Orders: order.customer != null
}

invariant NoOrphanedLineItems {
    for item in LineItems: item.order != null
}