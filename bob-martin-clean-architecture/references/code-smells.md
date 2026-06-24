# Code Smells for Architecture Review

Use smells as clues, not verdicts. A smell becomes an architecture finding when
it increases coupling, hides policy, blocks testing, or makes change risky.

## Severity Calibration

- **P0/P1**: smell causes boundary violation, data/security risk, or makes core
  behavior untestable.
- **P1**: smell forces broad changes for ordinary feature work.
- **P2**: smell hurts readability or maintainability but does not threaten the
  architecture.

Size thresholds are prompts for inspection. They are not automatic failures.

## Bloaters

### Long Method

- **Signal**: method is hard to scan, mixes levels of abstraction, or requires
  many mocks to test.
- **Heuristic**: > 30 lines deserves a look; > 75 lines often needs splitting.
- **Architecture impact**: use case/controller/repository may be hiding multiple
  responsibilities.
- **Refactor**: extract domain behavior, extract policy object, split command
  from side effects, introduce helper only when it clarifies intent.

### Large Class / God Service

- **Signal**: class changes for unrelated reasons or coordinates many external
  dependencies.
- **Heuristic**: method count and line count matter less than change reasons.
- **Architecture impact**: often masks missing use cases, ports, or domain model.
- **Refactor**: split by user action, domain concept, or adapter responsibility.

### Long Parameter List / Data Clumps

- **Signal**: same group of values travels together across boundaries.
- **Architecture impact**: missing command DTO, value object, or aggregate.
- **Refactor**: introduce parameter object, command object, value object, or
  keep whole object when it already represents the concept.

### Primitive Obsession

- **Signal**: strings/numbers represent important concepts with validation or
  units: money, IDs, status, email, card token, date range.
- **Architecture impact**: validation and business rules scatter across layers.
- **Refactor**: introduce value objects for important concepts. Do not wrap every
  primitive mechanically.

## OO and Modularity Smells

### Conditional Type Switching

- **Signal**: `switch`/`if` chains on type select behavior.
- **Architecture impact**: adding variants requires editing central policy.
- **Refactor**: strategy, polymorphism, registry, pattern matching, or table
  dispatch. Keep simple conditionals when the set is small and stable.

### Refused Bequest / Broken Substitution

- **Signal**: subclass throws unsupported operation, strengthens preconditions,
  or forces callers to check concrete types.
- **Architecture impact**: ports/interfaces cannot be trusted.
- **Refactor**: composition, smaller interface, or separate hierarchy.

### Fat Interface

- **Signal**: implementations throw no-op/not-supported, clients use only a small
  subset, or test doubles implement irrelevant methods.
- **Architecture impact**: violates ISP and makes adapter replacement noisy.
- **Refactor**: split by role/client/use case.

## Change Preventers

### Divergent Change

- **Signal**: one class changes for payments, reporting, notification, storage,
  and HTTP concerns.
- **Architecture impact**: SRP failure, high regression risk.
- **Refactor**: split by responsibility and move outer details behind ports.

### Shotgun Surgery

- **Signal**: one domain change requires edits across controllers, repositories,
  DTOs, service methods, and UI formatting.
- **Architecture impact**: boundaries are not isolating volatility.
- **Refactor**: centralize domain rule, define stable port/DTO, or reorganize
  package cohesion.

### Circular Dependencies

- **Signal**: package A imports B and B imports A, directly or through a chain.
- **Architecture impact**: dependency rule and independent deploy/test are at
  risk.
- **Refactor**: extract stable interface, merge inseparable modules, or move
  shared policy inward.

## Dispensables

### Duplicate Code

- **Signal**: same policy or mapping logic appears in multiple places.
- **Architecture impact**: business rules drift.
- **Refactor**: extract when duplication represents the same concept. Keep
  accidental similarity separate until the abstraction is clear.

### Dead Code

- **Signal**: unused classes, branches, adapters, or ports.
- **Architecture impact**: false architecture; reviewers think a boundary exists
  when it does not.
- **Refactor**: remove or mark as intentionally retained with reason.

### Speculative Generality

- **Signal**: abstractions exist for imagined future variants.
- **Architecture impact**: more ceremony without replacing actual volatility.
- **Refactor**: inline or simplify until a second real implementation appears.

## Couplers

### Feature Envy / Inappropriate Intimacy

- **Signal**: one class manipulates another object's internals or uses another
  module's data more than its own.
- **Architecture impact**: behavior belongs in the wrong layer or aggregate.
- **Refactor**: move behavior to the data owner, expose intent-revealing method,
  or introduce a domain service.

### Message Chains

- **Signal**: `a.b().c().d()` or repeated navigation through nested DTOs.
- **Architecture impact**: callers know too much about representation.
- **Refactor**: hide delegate, add query method, map to a stable DTO.

### Service Locator / Hidden Dependencies

- **Signal**: core code asks a container/global registry for collaborators.
- **Architecture impact**: dependencies are invisible and tests fail at runtime.
- **Refactor**: constructor injection or explicit factory at composition root.

## Architecture-Specific Anti-Patterns

| Anti-Pattern | Evidence | Typical Fix |
|--------------|----------|-------------|
| Anemic Domain in domain-heavy app | entities are data bags; services hold rules | move invariants and state transitions into entities/value objects/domain services |
| Fat Controller | HTTP handler validates, decides policy, queries DB, formats response | controller maps request/response; use case owns workflow |
| Repository with Business Rules | repository approves orders, applies discounts, sends events | repository only persists/maps; move policy inward |
| Active Record in strict Clean Architecture | ORM entity is also domain entity | separate persistence model from domain model |
| Missing Port | use case directly calls Stripe/S3/SMTP/queue | define application port; implement adapter outside |
| Unbounded Side Effects | save, charge, email, and publish happen with no recovery model | idempotency, outbox, compensation, or documented eventual consistency |
