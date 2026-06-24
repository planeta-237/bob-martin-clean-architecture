# Design and Teaching Guide

Use this reference when the user asks to design or explain Robert C. Martin's
Clean Architecture. The target design is the canonical layered/concentric model,
with Entities, Use Cases, Interface Adapters, Frameworks & Drivers, and Main.
Do not substitute neighboring architecture styles or framework/package
conventions for these circles.

## Contents

- [Core Mental Model](#core-mental-model)
- [The Four Circles](#the-four-circles)
- [Design Procedure](#design-procedure)
- [Boundary Anatomy](#boundary-anatomy)
- [Canonical Project Shape](#canonical-project-shape)
- [Boundary Decision Rules](#boundary-decision-rules)
- [Explaining Concepts](#explaining-concepts)
- [Common Design Mistakes](#common-design-mistakes)

## Core Mental Model

Clean Architecture protects high-level policy from low-level details.

The Dependency Rule:

```text
Source code dependencies point inward.
Frameworks & Drivers -> Interface Adapters -> Use Cases -> Entities
```

Control flow may cross outward through interfaces, but source code dependencies
still point inward because the inner layer owns the interface.

## The Four Circles

### 1. Entities

Enterprise business rules. They would still matter if the app had no web
framework, database, or UI.

Examples:

- `Order`, `Invoice`, `Account`, `TransferPolicy`
- Value objects: `Money`, `Email`, `AccountId`, `PaymentToken`
- Domain errors and domain events as business facts

### 2. Use Cases

Application-specific business rules. Each use case is one system operation and
orchestrates entities plus boundary interfaces.

Examples:

- `CreateOrderInteractor`
- `ApproveExpenseInteractor`
- `TransferFundsInteractor`

Use cases define request/response models, input boundaries, output boundaries,
and gateway interfaces they need.

### 3. Interface Adapters

Translate between the inner model and external forms.

Examples:

- Controllers: HTTP/CLI/message input -> use case request model
- Presenters: use case response model -> view model/API response
- Gateways/repositories: use case gateway interface -> DB/vendor mapping
- Mappers: persistence/API schemas <-> inner models

### 4. Frameworks & Drivers

Details: web framework, database, ORM, UI, queues, external SDKs, filesystem,
framework DI, and the composition root.

Main is a plugin: it wires concrete adapters into use cases and starts the
framework.

## Design Procedure

1. Name the business capability.
2. List actors and use cases as commands: `PlaceOrder`, `CancelOrder`,
   `TransferFunds`.
3. Identify entities and invariants.
4. Define use case request and response models.
5. Define input boundary for each use case.
6. Define output boundary/presenter needs.
7. Define gateway interfaces for persistence and external services.
8. Place controllers, presenters, and gateways in Interface Adapters.
9. Place framework implementations and Main in Frameworks & Drivers.
10. Define tests for entities, interactors with fakes, presenters/controllers,
    and framework integration.

## Boundary Anatomy

Typical request flow:

```text
HTTP Route
  -> Controller
  -> UseCaseInputBoundary
  -> UseCaseInteractor
  -> Entity
  -> GatewayInterface
  -> GatewayImplementation
  -> Database
```

Typical response flow:

```text
UseCaseInteractor
  -> OutputBoundary
  -> Presenter
  -> ViewModel / HTTP response body
```

The use case can call outward only through interfaces it owns:

```text
Use Case owns: OrderRepository, PaymentGateway, CreateOrderOutputBoundary
Adapters implement: SqlOrderRepository, StripePaymentGateway, JsonPresenter
```

## Canonical Project Shape

Use a layered shape by default:

```text
src/
  entities/
    order/
      Order.ts
      Money.ts
      OrderPolicy.ts
  use-cases/
    create-order/
      CreateOrderInputBoundary.ts
      CreateOrderInteractor.ts
      CreateOrderRequestModel.ts
      CreateOrderResponseModel.ts
      CreateOrderOutputBoundary.ts
      OrderGateway.ts
  interface-adapters/
    controllers/
      CreateOrderController.ts
    presenters/
      CreateOrderPresenter.ts
    gateways/
      SqlOrderGateway.ts
      StripePaymentGateway.ts
    view-models/
      CreateOrderViewModel.ts
  frameworks-drivers/
    web/
      routes.ts
    persistence/
      orm/
      migrations/
    vendors/
      stripe/
  main/
    composition-root.ts
```

For larger systems, grouping by business capability is acceptable only if each
capability still preserves the same four circles internally. Do not replace the
circles with framework folders as the architectural model.

## Boundary Decision Rules

Draw a full boundary when:

- The detail is a framework, database, vendor SDK, queue, filesystem, clock, or
  ID generator that affects tests or policy.
- Business behavior must be tested without infrastructure.
- Failure handling, idempotency, security, transactions, or output formatting
  matter.
- You need independent presenters, gateways, or delivery mechanisms.

Use a partial boundary when:

- There is a real volatility point, but a full input/output boundary would be
  excessive right now.
- A strategy/facade interface gives enough independence.

Do not create boundaries solely for decoration. In Clean Architecture, the
boundary is valuable because it protects policy from details.

## Explaining Concepts

Explain in this order:

1. **Policy vs detail**: entities/use cases are policy; DB/web/UI are details.
2. **Dependency Rule**: imports point inward.
3. **Boundaries**: use cases own the interfaces that details implement.
4. **Adapters**: controllers/presenters/gateways translate data.
5. **Main**: wires everything; frameworks are plugins.
6. **Cost**: boundaries add code, so draw them at volatility and risk points.

Small example:

```text
Route -> Controller -> CreateOrderInputBoundary
                         |
                         v
                 CreateOrderInteractor -> Order
                         |
                         v
                  OrderGateway interface
                         ^
                         |
                    SqlOrderGateway
```

## Common Design Mistakes

- Treating ORM models as Entities.
- Passing HTTP request/response objects into Use Cases.
- Returning database rows from Use Cases.
- Letting controllers contain business rules.
- Letting repositories/gateways contain business rules.
- Skipping presenters when output formatting has real policy or multiple views.
- Creating interfaces in outer layers instead of inner use case layers.
- Calling microservices "architecture"; deployment boundaries are not the same
  as Clean Architecture boundaries.
