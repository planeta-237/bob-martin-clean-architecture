# Layer Checklists

Use these checklists to find evidence. Treat "critical" items as likely score
drivers and "guideline" items as context-sensitive.

## Domain Layer

### Critical

- [ ] Avoid imports from web frameworks, ORMs, queues, storage, UI, or vendor SDKs.
- [ ] Keep business invariants in entities, value objects, or domain services.
- [ ] Represent important domain concepts explicitly, not only as primitives.
- [ ] Keep external references by identity or stable domain abstraction.
- [ ] Prevent invalid state transitions through domain methods or constructors.

### Guidelines

- [ ] Use value objects where validation, equality, units, formatting, or safety
  matter: `Money`, `Email`, `UserId`, `CardToken`.
- [ ] Keep entities cohesive. Size alone is not a violation; unrelated reasons to
  change are.
- [ ] Use domain services when behavior spans multiple entities and does not
  naturally belong to one aggregate.
- [ ] Keep domain events as facts about the domain, not instructions to adapters.

### Red Flags

- ORM decorators/annotations on domain classes.
- Entity methods executing SQL, HTTP, filesystem, cache, or queue operations.
- Domain errors that expose HTTP status codes or framework exceptions.
- Entities with only public fields/getters while all rules live elsewhere in a
  domain-heavy problem.

## Application / Use-Case Layer

### Critical

- [ ] Express one user/system action per use case or handler.
- [ ] Depend on domain policy and ports, not concrete infrastructure.
- [ ] Translate input into domain concepts before executing business behavior.
- [ ] Define transaction, consistency, or side-effect ordering deliberately.
- [ ] Return stable output data, not framework response objects.

### Guidelines

- [ ] Keep use cases as orchestrators. They may contain workflow decisions, but
  domain invariants should remain in the domain.
- [ ] Inject time, ID generation, randomness, external services, and repositories
  when determinism or replacement matters.
- [ ] Prefer explicit command/query intent. CQS is a useful heuristic, not an
  absolute Clean Architecture law.
- [ ] Let application code publish domain events or enqueue outbox records after
  persistence succeeds.

### Transaction and Side-Effect Checks

External APIs cannot be rolled back by a database transaction. Review the actual
failure modes:

- [ ] Multiple database writes are wrapped in a transaction or unit of work.
- [ ] Payment/charge/email/queue calls are idempotent or compensated.
- [ ] Events/messages that depend on committed state use outbox, post-commit
  hooks, or documented eventual consistency.
- [ ] Notifications happen after the state they describe is durable.
- [ ] Partial failure paths are observable and recoverable.

### Example: Better Payment Orchestration

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class ProcessPaymentCommand:
    order_id: str
    payment_token: str


@dataclass(frozen=True)
class ProcessPaymentResult:
    order_id: str
    payment_id: str


class ProcessPaymentUseCase:
    def __init__(
        self,
        orders: "OrderRepository",
        payments: "PaymentGateway",
        unit_of_work: "UnitOfWork",
        outbox: "Outbox",
    ) -> None:
        self._orders = orders
        self._payments = payments
        self._unit_of_work = unit_of_work
        self._outbox = outbox

    async def execute(self, command: ProcessPaymentCommand) -> ProcessPaymentResult:
        order = await self._orders.get(OrderId(command.order_id))
        if order is None:
            raise OrderNotFoundError(command.order_id)

        charge = await self._payments.authorize(order.total, PaymentToken(command.payment_token))

        async with self._unit_of_work:
            order.mark_payment_authorized(charge.id)
            await self._orders.save(order)
            await self._outbox.add(PaymentAuthorizedEvent(order.id, charge.id))

        return ProcessPaymentResult(order_id=order.id.value, payment_id=charge.id.value)
```

This does not pretend the payment API is part of the database transaction. It
makes the ordering, persistence, and follow-up delivery explicit.

## Interface Adapters

### Critical

- [ ] Controllers translate transport input to use-case input.
- [ ] Controllers map application/domain errors to transport responses.
- [ ] Repository implementations map between persistence models and domain models.
- [ ] Presenters/serializers shape output where UI/API needs diverge from use-case
  output.
- [ ] Input validation, authorization extraction, and deserialization stay near
  the boundary.

### Guidelines

- [ ] A dedicated presenter is optional when response formatting is trivial.
- [ ] DTOs are required at volatile boundaries; internal DTOs are optional when
  they add no clarity.
- [ ] Controllers can depend on concrete use-case classes if those classes are
  stable and easy to fake. Interfaces are more valuable when substitution is
  expected.

### Red Flags

- Controller calls repository directly and skips the use case.
- Adapter decides business policy instead of translating and delegating.
- Persistence rows are returned directly to API clients.
- Domain objects carry serialization annotations only for API convenience.

## Infrastructure Layer

### Critical

- [ ] Keep framework configuration, DI composition, ORM entities, HTTP clients,
  queues, caches, and filesystem access here.
- [ ] Implement ports defined in inner layers.
- [ ] Convert external schemas and exceptions into application/domain concepts.
- [ ] Keep secrets in environment/vault/config providers, not source code.

### Guidelines

- [ ] Add timeouts, retries, circuit breakers, and idempotency keys according to
  the risk of the dependency.
- [ ] Keep ORM models separate from domain models when domain behavior matters.
  Active Record may be acceptable in simple CRUD systems, but score it as a
  style mismatch for strict Clean Architecture.
- [ ] Keep composition root explicit enough that dependencies are discoverable.

## Layer-by-Layer Report Hints

For each layer, report:

- What belongs there and is correct.
- What belongs elsewhere.
- Whether the issue is a core violation or a pragmatic shortcut.
- The smallest refactor that would improve the boundary.
