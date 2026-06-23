# Allium construct registry

Single source of truth for how each Allium construct is recognised, framed for product managers and developers, synthesized into implementation, and tested. Loaded by all `gybis-spec-*` skills (and `gybis-arch-*` where applicable). Skills override only where their dispatch needs deviate. Each entry references `allium-language-reference.md` sections and numbered validation rules.

Columns: **Cues** (source patterns → construct, for `distill`) | **PM frame** (one-sentence plain English, for `describe`) | **Dev frame** (technical framing + validation semantics, for `explain`) | **Synthesis** (code-gen expectation, for `propagate`) | **Obligation categories** (from `internal/allium-plan`, for `propagate` and `weed`) | **Validation rules** (numbers in language-ref §Validation rules) | **Gotchas** (drift, mistakes, breakage modes, for `check`/`tend`/`weed`)

---

## Type definitions

### entity (internal)

- **Cues**: record-with-identity, mutable lifecycle, primary key
- **PM frame**: A first-class thing the system tracks over time (an order, a user, a candidacy)
- **Dev frame**: Entity with identity, lifecycle and rules; participates as a relationship target
- **Synthesis**: Record/class/table with identity column; fields, relationships, projections, derived values map to columns, foreign keys, queries, computed properties
- **Obligation categories**: `entity_fields`, `entity_optional`
- **Validation rules**: 1, 2, 11, 12
- **Gotchas**: Do not instantiate a base entity that carries a sum-type discriminator (rule 19)

### external entity

- **Cues**: third-party resource, system-external concept referenced but not owned; or an abstract type the consuming spec substitutes (type placeholder, dependency inversion)
- **PM frame**: A concept this domain relies on but does not manage; another system or the host product owns it. When declared empty, it acts as a replaceable slot a consumer fills with their own concrete thing
- **Dev frame**: Structure-only declaration with no lifecycle; the checker warns on reference; an `external entity {}` with no fields acts as a type placeholder a consumer fills with a concrete entity
- **Synthesis**: Interface / abstract type / DI port; do not generate persistence or lifecycle handlers; for type placeholders, expect the consumer to supply the concrete implementation
- **Obligation categories**: (none — boundary type)
- **Validation rules**: 1
- **Gotchas**: Distill from a library should prefer an empty `external entity` placeholder over inventing fields; consumer maps its own entity into the placeholder slot

### value type

- **Cues**: immutable record, no identity, structural equality, frozen dataclass, value object, plain struct
- **PM frame**: A piece of structured data with no separate identity (an address, a time range, a money amount)
- **Dev frame**: Embedded; compared by value not reference; no lifecycle; cannot be the target of a relationship's identity navigation
- **Synthesis**: Immutable value object / record / struct with structural equality and a hash derived from all fields
- **Obligation categories**: `entity_fields` (for shape)
- **Validation rules**: 1, 2, 12
- **Gotchas**: Never confuse with entity — value types must not carry identity or lifecycle

### sum type (variant) and discriminator

- **Cues**: sealed class, tagged union, ADT with constructors, Rust enum with data, OCaml variant, polymorphic record with `kind` tag
- **PM frame**: One concept that comes in a fixed set of kinds, each with its own details (notifications can be mentions, replies or shares)
- **Dev frame**: Base entity declares a discriminator field whose type is a pipe-separated list of variant names; each `variant X : BaseEntity` extends with its own fields; discriminator is set automatically on creation; `.created` on the base fires for any variant
- **Synthesis**: Discriminated union / sealed type; exhaustive matching at every dispatch site; variant constructors named after variant, not base
- **Obligation categories**: `sum_type_variant`, `variant_exhaustive`, `type_guard_required`
- **Validation rules**: 15–21
- **Gotchas**: Create via the variant name (`MentionNotification.created(...)`), never the base; accessing variant-specific fields without a `requires:` or `if` guard is rule 18 error; discriminator field name is author-chosen (no reserved name)

### type guard

- **Cues**: `if x is T:` / `match x with T(...)` style narrowing in source; explicit kind check before accessing kind-specific data
- **Dev frame**: A `requires` clause narrowing to a single variant guards the whole rule; an `if discriminator = Variant:` branch narrows within `ensures`
- **Synthesis**: Pattern-match / `instanceof` narrowing wrapping access to variant-specific fields; outside guards, those fields are unreachable
- **Obligation categories**: `type_guard_required`
- **Validation rules**: 18
- **Gotchas**: Combined `if` over multiple discriminators is not supported; nest guards instead

### optional field (`T?`)

- **Cues**: nullable field unrelated to any lifecycle state (nickname, optional note, optional reviewer)
- **PM frame**: This information may simply be absent for some instances. Distinct from a `when`-clause field, which ties presence to a particular stage
- **Dev frame**: Distinct from `when`-clause presence; `T?` is genuine optionality independent of state; `field = null` and `field != null` are presence checks (rule 12); arithmetic with null produces null; comparisons with null are false; the two forms may compose (`T? when status = ...` — exists in those states but may still be null)
- **Synthesis**: `Optional<T>` / nullable column / `T | null`; null-propagating arithmetic and short-circuiting comparisons
- **Obligation categories**: `entity_optional`
- **Validation rules**: 12
- **Gotchas**: Temporal triggers on optional fields (`when: u: User.next_digest_at <= now`) do not fire when the field is null

### state-dependent field (`when` clause on field)

- **Cues**: field that only carries a value in certain lifecycle states; e.g. `tracking_number` set after status reaches `shipped`
- **PM frame**: This information only appears once the thing has reached a particular state
- **Dev frame**: `field: T when status = a | b` ties presence to a named status field that must carry a `transitions` block; rules entering the when-set must set the field, rules leaving must clear it, rules within carry no obligation; accessing the field without a `requires` guard narrowing to a qualifying state is an error
- **Synthesis**: Persisted column that is null outside the when-set; entry/exit handlers that set/clear; runtime or static access guards
- **Obligation categories**: `when_set`, `when_clear`, `when_access_guard`, `derived_when_inferred`
- **Validation rules**: 7f–7m
- **Gotchas**: Adding a `when` clause to an existing field is a tooling-impact change — all rules touching the entity gain entry/exit obligations and may stop checking; convergent transitions (two rules reaching the same state) must both set the field

### transition graph

- **Cues**: enum status field with explicit allowed pairs and terminal states; runtime check rejecting illegal transitions
- **PM frame**: This thing moves through a defined sequence of states and stops once it reaches a final state
- **Dev frame**: `transitions field { from -> to ... terminal: a, b }` is authoritative when declared; rules producing edges outside the graph are validation errors; every non-terminal state must have at least one outbound edge; every declared edge must be witnessed by a rule; the graph is opt-in
- **Synthesis**: State machine where edges are guards and terminal states reject further transitions at runtime or compile time
- **Obligation categories**: `transition_edge`, `transition_rejected`, `transition_terminal`
- **Validation rules**: 7a–7e
- **Gotchas**: Adding a `transitions` block to an existing entity is a tooling-impact change — currently valid rules may become 7a errors; absence of outbound edges does not imply terminal — terminal must be declared explicitly; graphs are opt-in, never auto-synthesised from rule inference; `transition_edge` and `transition_rejected` pair per `source_construct` — emit a single combined guard test

### inline enum / named enum

- **Cues**: small fixed set of string values; type alias over a literal union
- **Dev frame**: Inline `status: pending | active` is anonymous; named `enum Recommendation { ... }` is reusable and required when values are compared across fields or entities
- **Synthesis**: Sealed enum type / string union; named enums become a shared type
- **Validation rules**: 14
- **Gotchas**: Two inline enum fields cannot be compared with each other even on the same entity (rule 14) — extract a named enum to share values

### backtick-quoted enum literal

- **Cues**: enum value referencing an external standard with hyphens, dots, mixed case, or leading digits (`` `de-CH-1996` ``, `` `no-cache` ``)
- **Dev frame**: Byte-exact UTF-8 comparison; not normalised; distinct from the snake_case form (`de_ch_1996` ≠ `` `de-CH-1996` ``); permitted in enum declarations and literal comparisons only — never as an identifier
- **Synthesis**: Preserve the canonical external-standard form byte-for-byte; do not lowercase, normalise dashes, or convert to snake_case
- **Gotchas**: Code that normalises canonical forms is a divergence even when the spec passes locally

### relationship

- **Cues**: foreign key, one-to-one or one-to-many association
- **PM frame**: How two concepts are connected
- **Dev frame**: `child: Type with field = this` declares the relationship; singular name → at most one related entity (equivalent to `T?`), plural name → zero-or-more collection; relationships currently produce `Set` (ordered-relationship syntax pending an ALP)
- **Synthesis**: Foreign-key column + repository/query method; singular returns optional, plural returns collection
- **Validation rules**: 3
- **Gotchas**: Relationship declarations must use `with` and reference `this`; using `where` here is rule 3 error; multiple matches on a singular relationship is a spec error

### projection

- **Cues**: filtered view of a relationship; computed subset
- **Dev frame**: `name: source where predicate [-> field]`; `where` filters, `-> field` extracts; null values are excluded from `-> field` results so `Set<T?>` becomes `Set<T>`; ordering is preserved (Sequence in → Sequence out)
- **Synthesis**: Query method or computed property over the relationship; null values excluded when emitting `-> field` extraction
- **Gotchas**: Predicate must use `where` (not `with`) and must not reference `this`

### derived value

- **Cues**: computed property, getter, derived attribute, formula over other fields; parameterised form taking arguments
- **Dev frame**: Read-only and automatically updated; parameterised form `can_use_feature(f): f in plan.features` exists; parameterised derived values cannot reference module `given` bindings or global state
- **Synthesis**: Computed property / getter / view; pure, no side effects; recompute on read or memoise; parameterised form must not read module `given` bindings or global state
- **Obligation categories**: (implicit via `entity_fields`)
- **Validation rules**: 10, 11
- **Gotchas**: No circular dependencies (rule 10); cannot reference `given` from parameterised form

### derived value `when` inference

- **Cues**: derived value computed from `when`-qualified inputs
- **Dev frame**: The checker infers the derived value's `when` set as the intersection of its inputs' when sets; author may annotate explicitly as documentation; mismatch with inferred set is an error; empty intersection means the derived value is unreachable
- **Obligation categories**: `derived_when_inferred`
- **Validation rules**: 7l

---

## Rules and triggers

### rule structure

- **Dev frame**: `when` (trigger) → optional `let` → `requires` (preconditions) → `ensures` (postconditions) → `@guidance` (optional, always last); `let` may appear anywhere after `when`
- **Validation rules**: 4

### pre-rule vs. resulting-state semantics in `ensures`

- **Dev frame**: In a state-change assignment `entity.field = expr`, the RHS reads pre-rule values. Inside the same `ensures` block, `if` guards and creation parameters read the *resulting* state defined by the assignments. A `let` binding inside `ensures` is visible to subsequent statements in that block.
- **PM frame**: When the system updates information, the formula uses the old values; any decisions made afterward use the new values
- **Synthesis**: Capture pre-rule snapshot before applying state changes; evaluate `if` guards against post-state; do not emit code that reads pre-rule values where the spec expects resulting state, or vice versa
- **Gotchas**: Frequent author and code-generator mistake; if pre-rule comparison is needed inside `ensures`, hoist into a `let` or `requires` before the `ensures` block

### trigger: external stimulus

- **Cues**: HTTP handler, command handler, button click, user-invoked action
- **Dev frame**: `when: ActionName(param, ...)`; optional parameters use `?` suffix and bind to `null` when omitted; multiple rules sharing a trigger must agree on parameter count and positional types
- **Validation rules**: 5, 6

### trigger: entity creation (`.created`)

- **Dev frame**: `when: e: Entity.created`; for sum types fires on any variant with the specific variant instance bound
- **Validation rules**: 5

### trigger: state transition (`transitions_to`)

- **Cues**: handler fires only when a field changes to a value from a different value
- **PM frame**: Only when the thing moves into state S from somewhere else — not when it starts there — the system does X
- **Dev frame**: `when: e: Entity.field transitions_to value`; does not fire on initial creation in that state — use `.created` for that or `becomes` for both; valid on enum, boolean, entity-reference fields; when a transition graph exists, only edges in the graph are structurally valid
- **Synthesis**: Handler fires only on transition into the state, never on creation in that state
- **Validation rules**: 5, 7a
- **Gotchas**: Confusing `transitions_to` with `becomes` produces missing or duplicated handlers

### trigger: `becomes`

- **Cues**: handler that fires both on creation in a state and on transition to it
- **Dev frame**: Fires both on creation in that state and on transition to it; equivalent to a `transitions_to` rule plus a `.created` rule with a guard
- **Synthesis**: Single handler covering creation and transition paths
- **PM frame**: Whenever the thing arrives in this state, however it got there

### trigger: temporal

- **Cues**: cron job, scheduled deadline check, expiry, timeout
- **PM frame**: Scheduled behaviour — every period (e.g. every Monday at 9 a.m.) or after a delay (e.g. N days after creation) the system does X
- **Dev frame**: `when: e: Entity.timestamp_field <= now`; fires once when the condition becomes true; always pair with a `requires` clause to prevent re-firing; `now` evaluation: in derived values re-evaluates on each read (volatile); in `ensures` is a snapshot bound to rule execution; in temporal triggers is the evaluation timestamp with fire-once semantics
- **Synthesis**: Scheduled job / time-based handler with anti-refire guard derived from the `requires` clause
- **Gotchas**: Optional timestamp fields produce null arithmetic — temporal triggers on them do not fire

### trigger: derived condition becomes true

- **Dev frame**: `when: e: Entity.is_valid` fires when the boolean derived value transitions false → true; same fire-once semantics as temporal; add `requires` if the value can revert and re-fire

### trigger: chained / trigger emission

- **Cues**: domain event subscriber (chained side); rule emitting a named event another rule listens for (emission side)
- **PM frame**: Cause and effect across the system — when X happens, the system also does Y
- **Dev frame**: Emission appears as an `ensures: EventName(field: value)` clause; chaining receiver uses `when: EventName(binding)`; bare identifiers in trigger emission parameters resolve as bindings first, then enum literals if the receiver declares a type — bare identifiers that resolve to neither produce a checker warning
- **Synthesis**: Publish/subscribe — emitter publishes a named event; chained rule is a subscriber
- **Validation rules**: 5, 6

---

## Expression language

### with vs. where discipline

- **Dev frame**: `with` declares relationships and must reference `this`; `where` filters existing collections (projections, iteration, surface context, actor `identified_by`, surface `let`) and must not reference `this`
- **Validation rules**: 3
- **Gotchas**: Inverting the two is a frequent error; predicates support the full expression language including chained navigation, boolean combinators, `in`, and bare boolean expressions

### navigation, optional navigation, null coalescing

- **Dev frame**: `.` chains field and relationship access; `?.` short-circuits to null when the left side is null; `??` provides a default; `when`-qualified field access does not require `?.` when the `requires` clause narrows to a qualifying state
- **Synthesis**: Lower `?.` to null-safe access; `??` to coalesce; do not over-guard fields the spec has narrowed via `requires`

### join lookup (`Entity{field, field}`)

- **Cues**: lookup of a join-table row by its component foreign keys
- **Dev frame**: Curly braces with field names look up the unique entity where those fields match; explicit form `{user: actor, workspace: workspace}` allows renaming; result is null if no match; ambiguous match (multiple entities satisfy) is a spec error the checker reports
- **Synthesis**: DB lookup or in-memory join; emit error path for ambiguous matches even if rare
- **Gotchas**: Accessing fields on a null lookup is an error — gate with `exists`

### collection operations (built-in dot-methods)

- **Dev frame**: Reserved set is `.count`, `.any()`, `.all()`, `.first`, `.last`, `.unique`, `.add()`, `.remove()`; lambdas in `.any`/`.all` are always explicit; `.first`/`.last` are ordered-only (warning on Set today, hard error next version); `.unique` always returns unordered Set
- **Validation rules**: 13, 14a
- **Gotchas**: Any other dot-method on a collection is rule 14a error; domain-specific operations use free-standing calls

### set arithmetic vs. mutation

- **Dev frame**: `+` and `-` are expression-level — produce a new collection without mutating; `.add()` and `.remove()` are ensures-only — mutate a relationship; applied to ordered collections, `+`/`-` produce unordered Set
- **Synthesis**: Distinguish "compute new collection" from "modify the relationship" in generated code

### `in`, `not in`, set literals

- **Dev frame**: `{value, value}` is the set literal used with `in` membership; same syntax in fields and expressions

### discard binding (`_`)

- **Cues**: unused binding placeholder where the value is ignored
- **Dev frame**: Syntactic placeholder for an unused binding; multiple `_` bindings in the same scope do not conflict with each other

### implication

- **Cues**: if-P-then-Q assertion in source code; rule formulated as a conditional implication
- **Dev frame**: `a implies b` is equivalent to `not a or b`; lowest precedence of boolean operators; available in all expression contexts; primary use is invariants but reads naturally in requires guards and derived booleans
- **Synthesis**: Lower to `!a || b`

### conditional expression (if / else if / else)

- **Dev frame**: Inline form for single-value assignments; block form for multi-statement ensures; `else if` chains; omit `else` when only the true branch has effect
- **Synthesis**: Inline → conditional expression; block → if/else if/else statement

### existence (`exists`, `not exists`)

- **Dev frame**: In `requires` checks presence; in `ensures` asserts removal (trivially satisfied if already absent — declarative, not imperative); valid as an `if` condition, narrowing the variable to non-null inside the branch
- **Synthesis**: `exists` → null check or query; `not exists` in ensures → DELETE / removal; bulk removal pattern `for x in collection: not exists x`; as an `if` condition, emit a narrowing branch where the variable is non-null

### hard delete vs. soft delete

- **Dev frame**: Hard delete uses `not exists entity` and removes the row; soft delete sets a status field (often paired with a `when` clause on `deleted_at`) and leaves the row in place
- **PM frame**: "The thing is gone" vs "The thing is marked as removed but kept for history"
- **Synthesis**: `not exists` → DELETE statement / removal; soft delete → UPDATE + `when`-clause obligations on deletion-only fields
- **Gotchas**: Code that physically deletes when the spec uses soft delete (or vice versa) is a divergence with downstream consequences (audit trails, restoration paths)

### literals (set, list, object)

- **Cues**: ordered collection field declared explicitly with retained duplicates becomes a list literal `[ ... ]` / `List<T>`
- **Dev frame**: Set literal `{ ... }` collapses duplicates; list literal `[ ... ]` produces `List<T>` and retains duplicates — the only way to populate a `List<T>` field; `Sequence` is inferred from ordered relationships, never written as a literal, and is a subtype of `Set`; object literals `{ key: value }` are anonymous records for entity creation parameters and trigger emission payloads — always require explicit `key: value` pairs
- **Synthesis**: List literal → ordered collection field retaining duplicates; object literal → anonymous record for creation parameters or trigger payloads requiring explicit `key: value` pairs
- **Validation rules**: 14b, 14c, 14d
- **Gotchas**: `{ x }` is a set literal containing `x`, never an object literal with shorthand; heterogeneous list elements are a type error

### black-box function

- **Cues**: hash, verify, parse, format, pure helper, external pure call, anything not first-class in the spec
- **PM frame**: An opaque external operation the system uses as a sealed unit (for example, hashing a password)
- **Dev frame**: Free-standing call syntax always (`hash(password)`, never `password.hash()`); collection-operating black-box functions take the collection as the first argument (`filter(events, e => ...)`); pure and deterministic for the same inputs within a rule execution; result may chain into built-in dot-methods
- **Synthesis**: Free-standing helper function or external library call; require purity and determinism
- **Gotchas**: Any dot-method on a collection not in the reserved built-in set is a checker error — domain operations must use free-standing form

### entity collections (`Users`, `Documents`)

- **Dev frame**: Pluralised type name refers to all instances; used in rule-level `for` and surface `let` bindings; natural English plurals
- **Synthesis**: Repository / global query returning all instances of the type

---

## Structural blocks

### given block

- **Cues**: module-scope dependency passed to every handler, singleton, processing engine, pipeline instance
- **Dev frame**: Module-scope instances inherited by every rule; distinct from surface `context` which binds a parametric scope; not required if rules get their entities from triggers; bindings must reference entity types declared or imported via `use`
- **Synthesis**: Module-scope singleton or constructor-injected dependency available to every rule in the module
- **Obligation categories**: `given_binding`
- **Validation rules**: 22, 23, 24

### default declaration

- **Cues**: seed data, static fixture, default role/plan/configuration
- **PM frame**: Out of the box, the system ships with these instances
- **Dev frame**: Named entity instances that exist unconditionally; type may be qualified with an import alias (`default gp/Policy my_policy = { ... }`) and the literal's field set is validated against the imported module's canonical schema
- **Synthesis**: Seed migration / fixture loaded unconditionally; validate the literal's field set against the canonical schema at emission time
- **Obligation categories**: `default_instance`
- **Validation rules**: 24a, 24b
- **Gotchas**: Renaming or removing a field surfaces as a drift error at check time on the default's literal

### config

- **Cues**: configurable threshold, tunable parameter, environment setting; parameter derived from arithmetic over other config parameters
- **PM frame**: A tunable parameter administrators can adjust, usually with a sensible default; when no default is provided, the operator must choose a value before the system runs
- **Dev frame**: Parameters declare type and optional default; parameters without defaults are mandatory and the consumer must supply a value; references use `config.name` locally and `alias/config.name` for imported; expression-form defaults combine arithmetic operators with local and qualified references per the type-compatibility table; expression defaults are evaluated once at config resolution time after overrides apply
- **Synthesis**: Configuration parameter with declared default and override mechanism; expression defaults resolve once at boot
- **Obligation categories**: `config_default`
- **Validation rules**: 25–27, 46–50
- **Gotchas**: Diamond dependency where two modules override the same shared parameter is a conflict, not a silent pick; chains longer than two levels of indirection warn; cycles error

### modular spec (`use`, qualified names)

- **Cues**: cross-module dependency, library import, shared domain concept
- **PM frame**: This domain depends on another (auth, billing, scheduling); changes there can ripple here
- **Dev frame**: `use "coord" as alias`; coordinates are immutable references (git SHAs or content hashes), not versions; qualified names `alias/Entity`, `alias/EventName`, `alias/config.param`; configure imported specs via `alias/config { ... }`; **breaking changes**: prefer accretion (add new fields, triggers, states; never remove or rename); if a breaking change is necessary, publish under a new name rather than a new version
- **Synthesis**: Import / package dependency at the coordinate's pinned hash; lift cross-module relationships into corresponding implementation imports
- **Gotchas**: Imported config parameters are overridden by qualified name; do not assume cross-domain handlers are local

---

## Invariants

### invariant: top-level

- **Cues**: property that must hold across all instances of a type; runtime assertion, class invariant, property-based test, boundary check function; for-all quantification over a collection
- **PM frame**: Across every user/order/document this is always true
- **Dev frame**: `invariant Name { for x in Collection: expr }` — universal quantifier over an entity collection; pure expression, no side effects, no `now`; checking strategy (PBT, model checker, trace validator) is a tooling concern
- **Synthesis**: Property test or contract test exercised after every state-changing rule; runtime assertion at appropriate boundaries
- **Obligation categories**: `invariant`
- **Validation rules**: 51, 53–57

### invariant: entity-level

- **Cues**: property scoped to one entity type's own state
- **PM frame**: For each individual instance this is always true
- **Dev frame**: Inside the entity body; unqualified field names resolve to the enclosing entity; `this` refers to the instance
- **Synthesis**: Class invariant / runtime assertion on the entity
- **Obligation categories**: `invariant`
- **Validation rules**: 52, 53–57

### expressibility: machine-checkable vs. prose

- **Cues**: expressible patterns — uniqueness, bounds, structural collection relationships, subset/partition (anything checkable from a single point in time); inexpressible patterns — cross-instance agreement, temporal ordering, evaluation-function contracts, counterfactual properties, monotonicity (use prose or `@invariant` inside a contract)
- **Dev frame**: Expressible (`invariant Name { expr }`): uniqueness across instances, relationships between same-entity fields, bounds on values, structural relationships between collections, subset/partition relationships — anything checkable from a single point in time. Not expressible (use prose comment or `@invariant` inside a contract): cross-instance agreement, temporal ordering, evaluation-function contracts, counterfactual properties, monotonicity. When in doubt, try writing the expression — if it requires comparing two moments in time, reasoning about another process, or referencing rule firing order, it belongs in prose.
- **PM frame**: Some promises the tooling can check automatically; others are described in prose because they refer to history or to outside systems
- **Synthesis**: For expressible invariants emit machine-checked tests; for prose-only invariants emit documentation or contract-test stubs
- **Gotchas**: Distilling code into expression-form invariants for properties that are not expressible produces false confidence; promote prose to expression form (during `gybis-spec-tend`) only when expressibility holds

---

## Surfaces and contracts

### contract declaration

- **Cues**: interface, protocol, trait, abstract class with signatures and documented invariants
- **PM frame**: An obligation between two sides of a boundary — what one party must implement, with named promises
- **Dev frame**: Module-level `contract Name { sig: (...) -> ...  @invariant Name ...  @guidance ... }`; bodies may contain only typed signatures and annotations; identity is module-qualified name; importable atomically via `use`; no type parameters
- **Synthesis**: Interface / protocol; signature-conformance test per signature; named contract test per `@invariant`
- **Obligation categories**: `contract_signature`, `contract_invariant`
- **Validation rules**: 40–45
- **Gotchas**: Two surfaces referencing identically named contracts from different modules conflict (rule 39); contract `@invariant` names must be unique within the contract

### contract reference: `demands` / `fulfils`

- **PM frame**: What integration requires of the other side (`demands`) and what this side promises in return (`fulfils`)
- **Dev frame**: Surface `contracts:` clause lists each referenced contract with direction marker — `demands` (counterpart must implement), `fulfils` (this surface supplies); each contract name appears at most once per surface
- **Synthesis**: Boundary glue — generate the implementation port for `fulfils` and the dependency declaration for `demands`
- **Obligation categories**: `surface_contract_demand`, `surface_contract_fulfilment`
- **Validation rules**: 36–39

### actor declaration (incl. `within`)

- **Cues**: role-based access, identity with membership check, scoped admin
- **PM frame**: Who counts as which role — e.g. a "workspace admin" is a user with admin rights on that workspace. With `within`, the role is scoped to a specific context (admin of *this* workspace, not everywhere)
- **Dev frame**: `actor Name { identified_by: Entity where condition }`; for context-dependent identity declare `within: ContextEntity` and reference `within` inside `identified_by` (`identified_by: User where Membership{user: this, workspace: within}.can_admin`); `this` is the entity instance being tested; `within` is the surface's `context` binding constrained to the declared type
- **Synthesis**: Identity check function; for `within`-bearing actors, take the context entity as input and verify membership/role per the predicate
- **Validation rules**: 28
- **Gotchas**: An actor with `within` can only be used in surfaces whose `context` type matches; entity types may also appear directly in `facing` for public-facing surfaces

### surface structure

- **Cues**: boundary with actor, visible data, and provided operations; boundary where any instance of an entity type can interact (facing entity type directly); per-instance scope passed to a handler or view becomes the surface context
- **PM frame**: What each role sees on a particular boundary or screen and what they can do there; named guarantees are the always-true promises about that boundary
- **Dev frame**: `facing party: ActorType` (or entity type directly), `context item: EntityType [where ...]`, `let`, `exposes`, `provides` (with optional `when` guards — parameters are per-action inputs from the party), `contracts:` (demands/fulfils), `@guarantee` (named, PascalCase), `@guidance` (no name, last), `related:` (target surfaces with context-typed expression), `timeout:` (references to temporal rules by name with optional restatement of the trigger)
- **Synthesis**: Boundary module — actor check, exposed data view, provided operations, integration tests per contract, scheduled handler per timeout; when `facing` names an entity type directly, omit the actor check (any instance of the type may interact)
- **Obligation categories**: `surface_actor`, `surface_provides`, `surface_guarantee`, `surface_timeout`, `surface_contract_demand`, `surface_contract_fulfilment`
- **Validation rules**: 28–35

### `@guarantee` vs. `@invariant` vs. `invariant { ... }`

- **Cues**: named property holding across all surface operations becomes `@guarantee`
- **Dev frame**: `@guarantee Name` (surface-scope, prose) is a boundary-wide property across all operations on the surface; `@invariant Name` (sigil prefix, in contracts) is a prose annotation scoped to that contract's signatures; `invariant Name { expr }` (no sigil, brace-delimited body) is the expression-bearing machine-checkable form used at top-level and entity-level
- **Synthesis**: Contract tests at surface boundary for `@guarantee`; documentation or contract tests for `@invariant`; machine-checked assertions for `invariant { ... }`

### surface timeout

- **PM frame**: A time-based outcome attached to this boundary — if a condition is not met within the deadline, the system does X automatically
- **Dev frame**: References an existing temporal rule by name and binds it to the surface's context; the `when` condition is optional documentation — when present the checker verifies it matches the referenced rule's trigger
- **Synthesis**: Scheduled handler per surface context instance; do not duplicate the rule body — point to the same handler
- **Obligation categories**: `surface_timeout`

### deferred specification

- **Cues**: placeholder or extension point defined elsewhere; complex sub-process referenced but not inlined
- **PM frame**: The detail of how X works lives in a separate specification rather than being repeated here
- **Dev frame**: `deferred Name.operation` declares Allium logic specified in another module; invoke at call sites via dot notation as a standalone `ensures` clause or as an expression that returns a value; distinct from a black-box function which models opaque external computation
- **Synthesis**: Call site pointing to the external module; skip local implementation

### open question

- **Cues**: TODO/FIXME comments, unresolved design decisions, ambiguous behaviour
- **PM frame**: A deliberate design decision is still open
- **Dev frame**: `open question "..."` is surfaced by the checker as a warning indicating the spec is incomplete; first-class construct, not a comment
- **Synthesis**: TODO marker in code or test, referencing the question text
- **Gotchas**: During distill or tend, prefer adding an `open question` over guessing when intent is ambiguous

---

## Cross-cutting concerns

### now evaluation model

- **Dev frame**: In derived values, `now` re-evaluates on each read (volatile, makes the derived value volatile); in `ensures` clauses, `now` is bound to the rule execution timestamp (a snapshot); in temporal triggers, `now` is the evaluation timestamp with fire-once semantics. Invariants must not reference `now` (volatile); stored timestamp fields like `created_at` are permitted.
- **Synthesis**: Capture rule execution timestamp once and reuse within `ensures`; recompute in derived value reads; temporal triggers use fire-once semantics around the evaluation timestamp

### accretion vs. breaking change

- **Dev frame**: Accretive changes (add fields, variants, triggers, states, expression-form config defaults, new surfaces) preserve consumers. Breaking changes (remove enum value, remove variant, narrow `when` set, remove contract `demands`, rename field, change contract signature) require a new module name rather than a new version per the reference's "Breaking changes" section. Tooling-impact changes (adding a `transitions` block to an existing entity, adding a `when` clause to an existing field) preserve grammar but obligate downstream rules.
- **PM frame**: Some changes ripple; some break compatibility and require a new module name
- **Gotchas**: `gybis-spec-tend` must classify proposed changes against this taxonomy before applying

---

## Quick lookup index

| Construct                                 | Section                |
| ----------------------------------------- | ---------------------- |
| entity                                    | Type definitions       |
| external entity                           | Type definitions       |
| value type                                | Type definitions       |
| sum type / variant                        | Type definitions       |
| type guard                                | Type definitions       |
| optional field `T?`                       | Type definitions       |
| `when` clause on field                    | Type definitions       |
| transition graph                          | Type definitions       |
| inline / named enum                       | Type definitions       |
| backtick-quoted enum literal              | Type definitions       |
| relationship                              | Type definitions       |
| projection                                | Type definitions       |
| derived value                             | Type definitions       |
| derived `when` inference                  | Type definitions       |
| rule structure                            | Rules and triggers     |
| pre-rule vs. resulting-state              | Rules and triggers     |
| trigger: external stimulus                | Rules and triggers     |
| trigger: entity creation                  | Rules and triggers     |
| trigger: `transitions_to`                 | Rules and triggers     |
| trigger: `becomes`                        | Rules and triggers     |
| trigger: temporal                         | Rules and triggers     |
| trigger: derived condition                | Rules and triggers     |
| trigger: chained / emission               | Rules and triggers     |
| with vs. where                            | Expression language    |
| navigation, `?.`, `??`                    | Expression language    |
| join lookup                               | Expression language    |
| built-in collection ops                   | Expression language    |
| set arithmetic vs. mutation               | Expression language    |
| `in`, set literals                        | Expression language    |
| discard binding `_`                       | Expression language    |
| implication                               | Expression language    |
| conditional expressions                   | Expression language    |
| existence                                 | Expression language    |
| hard vs. soft delete                      | Expression language    |
| literals (set, list, object)              | Expression language    |
| black-box function                        | Expression language    |
| entity collections                        | Expression language    |
| given block                               | Structural blocks      |
| default declaration                       | Structural blocks      |
| config                                    | Structural blocks      |
| modular spec                              | Structural blocks      |
| invariant top-level                       | Invariants             |
| invariant entity-level                    | Invariants             |
| expressibility                            | Invariants             |
| contract declaration                      | Surfaces and contracts |
| contract demands/fulfils                  | Surfaces and contracts |
| actor declaration (incl. within)          | Surfaces and contracts |
| surface structure                         | Surfaces and contracts |
| @guarantee vs. @invariant vs. invariant{} | Surfaces and contracts |
| surface timeout                           | Surfaces and contracts |
| deferred specification                    | Surfaces and contracts |
| open question                             | Surfaces and contracts |
| now evaluation model                      | Cross-cutting concerns |
| accretion vs. breaking change             | Cross-cutting concerns |
