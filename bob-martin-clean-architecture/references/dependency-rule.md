# Dependency Rule

The dependency rule is the core Clean Architecture test: source code
dependencies should point from volatile details toward stable policy.

## Mental Model

```text
Frameworks / Drivers / Infrastructure
        -> Interface Adapters
        -> Application / Use Cases
        -> Domain / Entities
```

Source imports point inward. Runtime control flow may go outward through ports:
an inner use case calls an interface it owns, and an outer adapter implements it.

## Evidence to Collect

For each suspicious dependency, identify:

- **Source dependency**: what file imports what?
- **Layer ownership**: which layer owns the caller and callee?
- **Volatility**: is the callee a framework, database, SDK, queue, UI, or vendor?
- **Test impact**: does the dependency force framework/IO setup to test policy?
- **Replacement impact**: could the detail be swapped without changing core code?

## Allowed Cross-Boundary Movement

| Movement | Allowed? | Notes |
|----------|----------|-------|
| Outer adapter imports inner use case/domain | Yes | Normal inward source dependency. |
| Use case imports domain entity/value object | Yes | Domain is more stable policy. |
| Use case calls an interface it defines/owns | Yes | Control flow can go outward through DIP. |
| Infrastructure implements application/domain port | Yes | Concrete detail depends on abstraction. |
| DTO/data crosses boundaries | Yes | Keep it simple and stable for the boundary. |
| Domain imports ORM/HTTP/vendor SDK | No | Detail contaminates policy. |
| Use case takes `Request`/`Response` | Usually no | Transport concern leaks inward. |
| Controller calls repository directly | Usually no | Bypasses application policy. |

## Common Violations

| Violation | Evidence | Typical Severity |
|-----------|----------|------------------|
| ORM annotations on domain entity | SQLAlchemy `Mapped`/`mapped_column`, SQLModel table model, or Active Record-style base in domain | P0/P1 |
| HTTP types in use case | FastAPI/Starlette `Request`, `Response`, `Depends`, `HTTPException`, or `BackgroundTasks` | P1 |
| Vendor SDK in core policy | `stripe`, `boto3`, `smtplib`, `httpx`, OpenAI/PydanticAI client, or queue client imported in application/domain | P1 |
| Repository direct from controller | controller -> repo, no use case for business action | P1 |
| Service locator in core | `container.get(...)`, `ServiceLocator.get(...)` | P1 |
| Domain throws framework exception | `HTTPException`, `SQLAlchemyError`, driver error, ORM exception | P1/P2 |
| Config read inside entity | environment/config provider in domain object | P2/P1 |

Escalate severity when the dependency hides IO, blocks unit testing, or creates
data/security risk.

## Inversion Process

When core code depends on a detail:

1. Name the capability the core needs.
2. Define a small port in the inner layer that owns the policy.
3. Change the core code to depend on the port.
4. Implement the port in an outer adapter.
5. Wire the implementation in the composition root.

```python
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True)
class PaymentAuthorization:
    id: str


class PaymentGateway(Protocol):
    async def authorize(self, amount: "Money", token: "PaymentToken") -> PaymentAuthorization:
        """Application-owned port for payment authorization."""


class ProcessPaymentUseCase:
    def __init__(self, payments: PaymentGateway) -> None:
        self._payments = payments

    async def execute(self, amount: "Money", token: "PaymentToken") -> PaymentAuthorization:
        return await self._payments.authorize(amount, token)


class StripePaymentGateway:
    async def authorize(self, amount: "Money", token: "PaymentToken") -> PaymentAuthorization:
        # Stripe SDK details live in infrastructure.
        ...
```

## Package Structure Examples

Layer-first:

```text
app/
  domain/
  application/
  adapters/
  infrastructure/
```

Feature-first with internal layers:

```text
app/
  orders/
    domain/
    application/
    adapters/
    infrastructure/
  payments/
    domain/
    application/
    adapters/
    infrastructure/
```

Neither structure is automatically superior. Score by dependency direction,
cohesion, and change locality.

## Enforcement Ideas

Use import-linter, grimp, pydeps, or a small pytest architecture test. Keep
tooling as a guardrail; evidence from code still matters more than tool output.

Example pytest check for a layer-first Python project:

```python
from pathlib import Path


DOMAIN_ROOT = Path("app/domain")
FORBIDDEN_IN_DOMAIN = (
    "from app.api",
    "from app.infrastructure",
    "from fastapi",
    "from sqlalchemy",
    "import fastapi",
    "import sqlalchemy",
)


def test_domain_does_not_import_outer_layers() -> None:
    offenders: list[str] = []
    for path in DOMAIN_ROOT.rglob("*.py"):
        text = path.read_text(encoding="utf-8")
        for forbidden in FORBIDDEN_IN_DOMAIN:
            if forbidden in text:
                offenders.append(f"{path}: {forbidden}")

    assert offenders == []
```

Use tooling suggestions as recommendations unless the user asks to implement
architecture enforcement.
