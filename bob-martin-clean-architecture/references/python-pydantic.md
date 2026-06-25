# Python and Pydantic Guide

Use this reference when reviewing, designing, or explaining a Python codebase,
especially one using Pydantic. Keep Robert C. Martin's layered Clean
Architecture as the architectural model; treat Python and Pydantic as
implementation choices inside that model.

## Contents

- [Placement Rules](#placement-rules)
- [Pydantic v2 Practices](#pydantic-v2-practices)
- [Python Design Practices](#python-design-practices)
- [Layer Guidance](#layer-guidance)
- [Review Signals](#review-signals)
- [Design Defaults](#design-defaults)

## Placement Rules

- Put Pydantic models at boundaries by default: HTTP request/response schemas,
  CLI payloads, message payloads, vendor DTOs, persistence DTOs, and settings.
- Do not make `BaseModel` classes the default Entity type. Entities should own
  business invariants and behavior without depending on validation frameworks.
- Use Pydantic inside the domain only as a deliberate trade-off for simple,
  stable value objects where the dependency is accepted and does not pull in
  framework concerns.
- Convert boundary DTOs into domain commands/value objects before entering a Use
  Case Interactor.
- Convert use-case response models into Pydantic response schemas in presenters
  or delivery adapters, not inside the interactor.

## Pydantic v2 Practices

- Prefer Pydantic v2 APIs:
  - `Model.model_validate(data)` for Python data.
  - `Model.model_validate_json(raw_json)` for JSON input.
  - `model.model_dump()` for Python dictionaries.
  - `model.model_dump_json()` for JSON output.
- Prefer `ConfigDict` for model configuration.
- Use strict validation where coercion would hide bugs, security issues, money
  mistakes, IDs, permissions, dates, or external contract drift:

```python
from pydantic import BaseModel, ConfigDict, Field


class CreateOrderRequest(BaseModel):
    model_config = ConfigDict(strict=True)

    customer_id: str
    quantity: int = Field(strict=True, ge=1)
```

- Use `Field(...)` constraints for boundary validation that belongs to an
  external contract. Put business invariants in entities/value objects.
- Use `TypeAdapter` for validating non-model types such as `list[Dto]`,
  primitives, dataclasses, or typed collections without creating wrapper
  `BaseModel` classes only for validation.
- Keep validators focused on shape normalization and boundary checks. Do not put
  use-case orchestration, persistence, external calls, authorization, or complex
  business policy in Pydantic validators.
- Avoid Pydantic v1 habits in new code: `.dict()`, `.json()`, `parse_obj`, and
  class-based `Config` unless maintaining legacy code.

## Python Design Practices

- Prefer explicit type hints on public functions, use cases, gateways, and
  presenters.
- Use `Protocol` from `typing` for gateway/output-boundary interfaces when
  structural typing fits the codebase.
- Keep imports pointing inward. Inner layers may define protocols; outer layers
  implement them.
- Prefer dataclasses, frozen dataclasses, or plain classes for domain value
  objects when they keep invariants clear without framework coupling.
- Avoid global containers, module-level mutable state, hidden IO, and direct
  environment reads inside entities/use cases.
- Use dependency injection through constructors or function arguments; keep
  framework DI in Main or outer adapters.
- Treat async as a boundary/runtime concern. Do not make domain policy async only
  because a database or HTTP client is async.

## Layer Guidance

### Entities

- Plain Python classes, dataclasses, or deliberate value-object models.
- Own invariants: money, quantity, status transitions, ownership, limits.
- No FastAPI, SQLAlchemy, Django ORM, HTTP request objects, Pydantic settings, or
  vendor SDK imports.

### Use Cases / Interactors

- Accept request models that express application intent, not raw framework
  request objects.
- Depend on protocols/gateway interfaces that the use case owns.
- Return response models or call output boundaries. Do not return ORM rows,
  vendor responses, or HTTP responses.

### Interface Adapters

- Pydantic request/response schemas usually live here.
- Controllers map Pydantic schemas into use-case requests.
- Presenters map use-case responses into Pydantic/API response schemas.
- Gateways map domain objects to persistence/vendor DTOs and back.

### Frameworks & Drivers

- FastAPI/Django/Flask routes, SQLAlchemy/Django ORM models, Redis, Celery,
  external SDK clients, Pydantic settings, and concrete DI wiring live here.
- Main/composition root wires concrete adapters into interactors.

## Review Signals

Positive signals:

- Pydantic schemas are near framework/API boundaries.
- Domain entities can be tested without Pydantic, FastAPI, DB, or vendor clients.
- `model_validate`/`model_dump` are used in adapter mapping code.
- Strict mode or constrained fields protect external contracts where needed.
- Gateway protocols are defined inward and implemented outward.

Risk signals:

- Use cases accept FastAPI `Request`, Pydantic API schemas, ORM models, or raw
  vendor payloads directly.
- Entities inherit from ORM models or Pydantic models only for convenience.
- Validators perform database queries, HTTP calls, authorization, or workflow
  orchestration.
- Settings/environment reads happen inside entities/use cases.
- Async database/client details force async into pure domain behavior.
- Pydantic schemas are reused across API, persistence, and domain boundaries
  without mapping.

## Design Defaults

For a Python/Pydantic service, default to this shape:

```text
src/
  entities/
    order.py
    money.py
  use_cases/
    create_order/
      interactor.py
      request.py
      response.py
      boundaries.py
  interface_adapters/
    controllers/
      create_order_controller.py
    presenters/
      create_order_presenter.py
    schemas/
      order_api.py          # Pydantic API schemas
    gateways/
      sqlalchemy_order_gateway.py
  frameworks_drivers/
    web/
      fastapi_routes.py
    persistence/
      orm_models.py
    settings.py             # Pydantic settings or environment adapter
  main/
    composition_root.py
```

Use Pydantic schemas to protect the edges. Use entities and interactors to
protect the policy.
