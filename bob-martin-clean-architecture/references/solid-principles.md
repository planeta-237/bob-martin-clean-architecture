# SOLID and CQS Review Guide

Use SOLID to explain why a design is easy or hard to change. Do not apply it as
a checklist of ceremony.

## S - Single Responsibility Principle

Ask: **What reason would make this module change?**

Strong signs:

- A class/module has one clear job in the architecture.
- Policy, orchestration, translation, persistence, and delivery are separated.
- Tests for the class read like one responsibility.

Red flags:

- Description needs "and" across unrelated concerns: "creates orders and sends
  Slack messages and exports CSV".
- One change area repeatedly risks unrelated features.
- A use case owns domain rules, DB details, and HTTP formatting.

Review guidance:

- Do not penalize size alone. Penalize mixed reasons to change.
- Prefer splitting by user action, domain concept, adapter role, or external
  dependency.

## O - Open/Closed Principle

Ask: **Can expected variants be added without editing stable policy?**

Strong signs:

- New payment provider, repository, notification channel, or rule variant can be
  added through a port/strategy/handler.
- Stable code depends on a stable abstraction.

Red flags:

- Central `switch`/`if` chain changes for every new provider/type.
- Use case imports concrete vendor implementations.
- Adding one feature requires edits across many unrelated modules.

Review guidance:

- Do not require abstraction for hypothetical variants.
- A simple conditional is acceptable when the variant set is small and stable.

## L - Liskov Substitution Principle

Ask: **Can every implementation be used through the abstraction without
surprises?**

Strong signs:

- Implementations preserve the interface contract.
- Errors, nullability, side effects, and transactions match caller expectations.

Red flags:

- Implementation throws "not supported" for required methods.
- Subclass strengthens preconditions or weakens guarantees.
- Callers check concrete types before using an interface.

Review guidance:

- LSP applies to interfaces, base classes, and ports, not only classical
  inheritance.

## I - Interface Segregation Principle

Ask: **Does each client depend only on the capabilities it uses?**

Strong signs:

- Ports are role-specific: `PaymentGateway`, `OrderRepository`,
  `NotificationService`.
- Test doubles are easy to write.
- Interface methods change together.

Red flags:

- Generic `Service`/`Manager` interfaces with unrelated methods.
- Implementations use no-op methods or throw unsupported errors.
- Clients import large facades for one operation.

Review guidance:

- Method count is a smell, not a rule. A cohesive interface with eight methods
  can be better than five fragmented one-method abstractions.

## D - Dependency Inversion Principle

Ask: **Do high-level policies depend on abstractions that they own or control?**

Strong signs:

- Application/domain code defines ports for volatile external details.
- Infrastructure implements those ports.
- Composition root wires concrete implementations.

Red flags:

- Domain/use case imports SQLAlchemy models/session, FastAPI objects, vendor SDK
  clients, SMTP clients, queue clients, or the application container.
- Core code calls a service locator or global container.
- Tests must boot the framework to exercise policy.

Review guidance:

- Interfaces are most valuable at volatility, IO, and testing seams.
- Do not require an interface for every class. Concrete dependencies inside one
  stable module may be fine.

## CQS - Command Query Separation

CQS is a useful reasoning heuristic: a method should usually either change state
or answer a question. It is not a universal Clean Architecture requirement.

High-value CQS findings:

- A query method mutates persistent state: `findById()` updates last access time.
- A getter triggers lazy-loading with hidden IO.
- A validator writes audit records or sends messages.
- A factory/constructor calls external APIs.

Context-sensitive findings:

- A command returning a DTO or ID is often appropriate.
- A repository `save()` returning generated ID/status can be acceptable when it
  is documented and not returning a mutable domain object.
- A use case returning output after mutating state is normal application design.

Recommended language:

- Say "This mixes command and query behavior and makes effects surprising" when
  there is hidden mutation or IO.
- Avoid claiming every command must return `void`.

## Review Phrases

Use precise claims:

- "This violates DIP because the application layer imports a concrete SQLAlchemy
  repository instead of an application-owned port."
- "This is an SRP issue because payment routing and CSV export change for
  unrelated reasons."
- "This interface is too broad for its clients; split by role."
- "This conditional is acceptable unless payment methods are expected to grow."
