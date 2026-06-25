# FastAPI Guide

Use this reference when reviewing, designing, or explaining a Python service that
uses FastAPI. Keep Robert C. Martin's Clean Architecture as the architectural
model. FastAPI is a Frameworks & Drivers detail plus delivery adapter tooling;
it is not the architecture.

## Contents

- [Placement Rules](#placement-rules)
- [Routing and Controllers](#routing-and-controllers)
- [Dependency Injection](#dependency-injection)
- [Response Models and Presenters](#response-models-and-presenters)
- [Lifespan, Settings, and Resources](#lifespan-settings-and-resources)
- [Background Work and Side Effects](#background-work-and-side-effects)
- [Security Dependencies](#security-dependencies)
- [Testing](#testing)
- [Review Signals](#review-signals)
- [Design Defaults](#design-defaults)

## Placement Rules

- Keep `FastAPI`, `APIRouter`, `Depends`, `Request`, `Response`,
  `HTTPException`, middleware, and lifespan functions outside Entities and Use
  Cases.
- Put routes and FastAPI dependency providers in Frameworks & Drivers or thin
  Interface Adapter modules.
- Controllers map FastAPI/Pydantic input into use-case request models and call
  input boundaries/interactors.
- Presenters map use-case responses into API response schemas or view models.
- Main/composition root wires interactors, gateways, presenters, settings, and
  FastAPI dependencies.

## Routing and Controllers

- Use `APIRouter` to group delivery endpoints by capability or API surface.
- Keep path operation functions thin: parse HTTP input, call a controller or
  interactor boundary, return a presenter/API schema.
- Do not put business decisions, transaction orchestration, vendor calls, or
  persistence rules in route functions.
- Convert path/query/body/header values into application request models before
  entering the use case.
- Raise or map `HTTPException` only at the delivery boundary. Inner layers should
  use domain/application errors.

## Dependency Injection

- Use `Depends` and `Annotated[..., Depends(...)]` for FastAPI delivery
  dependencies: auth context, request-scoped resources, sessions, current user,
  and controller factories.
- Do not let Use Cases depend on `Depends` or FastAPI's dependency system.
- Keep framework DI as an outer adapter. Use constructors or explicit function
  arguments for interactors and gateways.
- Use reusable dependency aliases for repeated delivery concerns, but pass clean
  values inward:

```python
CurrentUser = Annotated[UserContext, Depends(get_current_user)]
```

- Prefer dependency providers that return ready-to-use boundary objects, such as
  a controller, interactor, gateway, or unit of work.

## Response Models and Presenters

- Use return type annotations or `response_model` to define HTTP contracts.
- Treat response schemas as Interface Adapter models, not use-case return types
  by default.
- Do not return ORM models, domain entities, or raw vendor payloads directly from
  route handlers when the public API contract is different.
- Let presenters decide API-facing shape when output formatting, localization,
  permissions, redaction, or multiple views matter.

## Lifespan, Settings, and Resources

- Use FastAPI lifespan for startup/shutdown resources such as DB pools, HTTP
  clients, caches, ML models, or message producers.
- Keep lifespan code in Frameworks & Drivers or Main.
- Load settings in an outer settings/config adapter. Do not read environment
  variables inside entities/use cases.
- Store concrete clients and pools in app state or composition root facilities,
  then pass abstractions inward through gateways.
- Do not hide business workflows in startup/shutdown hooks.

## Background Work and Side Effects

- Use FastAPI background tasks only for simple, non-critical delivery-adjacent
  side effects.
- For money movement, emails that must be reliable, webhooks, audit events,
  integration events, or anything requiring retry/idempotency, prefer an outbox,
  queue, workflow, or explicit use-case side-effect boundary.
- Do not put core business policy in a `BackgroundTasks` callback.

## Security Dependencies

- Put token parsing, session extraction, OAuth/JWT verification, and request auth
  mechanics in FastAPI dependencies or middleware.
- Pass a clean `Actor`, `UserContext`, `Permissions`, or similar application
  model into the use case.
- Keep authorization policy that is business-specific in the use case/domain,
  not only in route decorators.
- Avoid logging raw tokens, authorization headers, cookies, or sensitive request
  bodies.

## Testing

- Use `TestClient` or async HTTP clients for API adapter tests.
- Use `app.dependency_overrides` to replace FastAPI dependencies with fakes in
  route/controller tests.
- Keep entity and use-case tests independent of FastAPI, ASGI, HTTP clients, and
  live infrastructure.
- Test mapping separately when API schemas differ from use-case request/response
  models.
- Reset dependency overrides between tests to avoid cross-test leakage.

## Review Signals

Positive signals:

- Routes are thin and delegate to controllers/interactors.
- `Depends` is limited to delivery/composition concerns.
- Use cases can be tested without FastAPI or ASGI.
- `response_model` or return annotations document API contracts while presenters
  handle output mapping.
- Lifespan owns infrastructure startup/shutdown, not business policy.
- API tests override dependencies cleanly.

Risk signals:

- Use cases import `fastapi`, `Request`, `Response`, `Depends`, or
  `HTTPException`.
- Route handlers contain business rules, transaction coordination, or direct DB
  and vendor SDK calls.
- Pydantic API schemas are passed deep into the domain.
- `BackgroundTasks` hides critical workflows without retries or idempotency.
- Auth dependency decisions are the only place where business authorization
  exists.
- Lifespan or middleware mutates global state consumed by inner policy.

## Design Defaults

For FastAPI, default to this shape:

```text
src/
  entities/
    order.py
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
      order_api.py
  frameworks_drivers/
    web/
      app.py
      routers/
        orders.py
      dependencies.py
      exception_handlers.py
      middleware.py
      lifespan.py
    settings.py
  main/
    composition_root.py
```

Typical request path:

```text
FastAPI route -> dependency/controller -> input boundary/interactor -> entity
Interactor -> output boundary/presenter -> API schema/response
Interactor -> gateway protocol -> concrete infrastructure adapter
```

Use FastAPI to expose and compose the system. Use Clean Architecture to decide
where policy lives.
