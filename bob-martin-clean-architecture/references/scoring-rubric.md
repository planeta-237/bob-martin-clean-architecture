# Scoring Rubric

Use this rubric to assign a calibrated 0-10 architecture score. The score is
for decision-making: how safe, understandable, testable, and changeable the
architecture is for its stated goal.

## Scoring Method

1. Classify the intended architecture style.
2. List evidence-backed findings by severity.
3. Assign each dimension independently.
4. Avoid double-counting one root cause.
5. Adjust for missing context: unknown is not the same as failed.
6. Compare the final score with the anchors before publishing it.

If the code is a compact CRUD flow, prototype, script, or Active Record-style
module, state whether the score is **Clean Architecture fitness** rather than
overall engineering quality. A low fitness score can coexist with an acceptable
contextual design when risk, change rate, and domain complexity are low.

## Weighted Dimensions

| Dimension | Max | Full Credit | Major Deductions |
|-----------|-----|-------------|------------------|
| Dependency Direction | 2.0 | Core policy has no dependency on DB, HTTP, UI, queues, frameworks, or vendor SDKs. Ports invert control flow at volatile boundaries. | Framework/ORM imports in domain; use cases take HTTP types; direct vendor SDK calls in core; service locator/global container in core. |
| Boundary & Layer Separation | 2.0 | Domain, application/use cases, adapters, and infrastructure have distinct responsibilities. Mapping/translation happens at edges. | Business logic in controllers/repositories/views; use cases construct HTTP responses; persistence models used as domain models; layer bypassing. |
| Domain & Use-Case Modeling | 1.5 | Important invariants live in entities/value objects/domain services. Use cases orchestrate one system action and express transaction intent. | Anemic domain in a domain-heavy system; validation scattered across controllers; fat procedural services; unclear transaction/consistency model. |
| Modularity & SOLID | 1.5 | Classes/modules are cohesive, substitutable where needed, and easy to extend. Interfaces are focused and motivated by volatility. | God classes; fat interfaces; shotgun surgery; circular dependencies; speculative abstraction; concrete dependencies where replacement matters. |
| Testability & Change Safety | 1.5 | Core behavior can be unit-tested without infrastructure. Time, IDs, IO, randomness, and external APIs have deterministic seams. | Live DB/web server required for business rules; static/global dependencies; hidden IO; no place to mock or substitute external details. |
| Cross-Cutting Risks | 1.0 | Sensitive data, external side effects, observability, idempotency, and data integrity are handled deliberately. | Secrets in logs/URLs/errors; payment/email/event side effects without idempotency or compensation; missing rollback/outbox strategy where risk is material. |
| Documentation & Narrative | 0.5 | Naming, ADRs, README, package layout, or PR context make boundaries understandable. | Architecture only exists in comments or tribal knowledge; cryptic names; no rationale for unusual boundaries. |

## Grade Anchors

| Score | Grade | Meaning |
|-------|-------|---------|
| 9.3-10.0 | A+ | Reference-quality implementation. Only small polish remains. |
| 8.5-9.2 | A | Strong, production-ready structure with minor issues. |
| 7.5-8.4 | B | Good architecture with notable but manageable gaps. |
| 6.0-7.4 | C | Acceptable design, but important changes should be requested. |
| 4.5-5.9 | D | Significant rework needed; risks are structural. |
| 0.0-4.4 | F | Fundamental architecture failure or unsafe coupling. |

## Calibration Examples

Use these anchors to prevent score drift:

- **9.5**: clean dependency direction, rich domain where needed, deterministic
  tests, clear transaction/outbox or consistency strategy, good docs.
- **8.5**: strong ports/layers and testability, but a few pragmatic shortcuts
  exist: controller formats responses, simple DI composition, partial docs.
- **7.5**: recognizable Clean Architecture, but use cases are getting fat,
  boundary DTOs/mappers are inconsistent, or package cycles need cleanup.
- **6.5**: mixed architecture; business rules are partly isolated, but framework
  types or persistence details leak into application code.
- **5.0**: maintainable only with care; major services/controllers mix
  orchestration, persistence, and external calls.
- **4.0-5.0**: common range for small low-risk CRUD/Active Record code that is
  understandable but not shaped like Clean Architecture. Explain style mismatch
  instead of framing every missing layer as urgent redesign.
- **3.0**: typical teaching example of bad architecture: ORM-decorated domain,
  HTTP request/response in business code, direct external calls, hard to test.
- **1.0**: barely separable or unsafe: no meaningful boundaries plus severe
  security/data-integrity risks or unreviewable code.

## Dimension Guidance

### Dependency Direction (0-2.0)

- **2.0**: Inner policy imports only language/runtime primitives and stable
  domain abstractions. Outer details implement inner ports.
- **1.5**: Mostly correct, with small convenience leaks that do not affect core
  policy or tests.
- **1.0**: Some application logic depends directly on frameworks or concrete
  adapters, but domain remains mostly clean.
- **0.5**: Core code frequently imports infrastructure, HTTP, ORM, or vendor SDKs.
- **0.0**: No dependency discipline; business policy is embedded in details.

### Boundary & Layer Separation (0-2.0)

- **2.0**: Responsibilities are clear and boundaries are explicit enough for the
  project size.
- **1.5**: Minor adjacent-layer mixing; no major business rule leakage.
- **1.0**: Controllers, repositories, or views contain meaningful business logic.
- **0.5**: Most features are transaction scripts or active-record procedures.
- **0.0**: No recognizable boundaries.

### Domain & Use-Case Modeling (0-1.5)

- **1.5**: Domain model protects important invariants; use cases coordinate flow
  and consistency.
- **1.0**: Domain has some behavior, but services still own important decisions.
- **0.5**: Entities are mostly data bags; use cases/controllers hold rules.
- **0.0**: Business rules are scattered across IO code.

### Modularity & SOLID (0-1.5)

- **1.5**: Responsibilities are cohesive; abstractions are purposeful; changes
  are localized.
- **1.0**: Several smells exist, but the direction of change is still clear.
- **0.5**: God classes, shotgun surgery, or cycles make change risky.
- **0.0**: Tight coupling dominates the design.

### Testability & Change Safety (0-1.5)

- **1.5**: Business behavior can be tested in memory with simple fakes.
- **1.0**: Mostly testable; a few deterministic seams are missing.
- **0.5**: Heavy integration setup is needed for ordinary business rules.
- **0.0**: No meaningful unit testing path.

### Cross-Cutting Risks (0-1.0)

- **1.0**: Sensitive data, IO, transactions, retries, idempotency, and logging are
  appropriate for the domain risk.
- **0.7**: Minor gaps or undocumented assumptions.
- **0.3**: Important operational or security risks are present.
- **0.0**: Severe leakage, data-loss, payment, or credential risk.

### Documentation & Narrative (0-0.5)

- **0.5**: Boundaries and decisions are easy to understand from names/docs.
- **0.3**: Naming is clear but rationale is thin.
- **0.1**: Some comments exist, but architecture is implicit.
- **0.0**: A maintainer must reverse-engineer everything.

## Quick Diagnostic Use

The quick diagnostic is an orientation tool. Do not sum its deductions as the
final score. Use it to decide where to inspect next, then score from evidence.

Strong "No" answers to dependency, testability, and sensitive-data questions
usually imply a score below 6. Multiple strong "No" answers in core boundaries
usually imply a score below 4.

## Style-Mismatch Severity

Use lower severity when a finding is mainly "not Clean Architecture":

- Missing explicit use cases in a tiny CRUD endpoint is often **P2** unless the
  route contains real business policy or is expected to grow.
- Active Record or ORM-backed entities are **P1** for domain-heavy systems, but
  may be **P2/style mismatch** for simple persistence screens.
- Repository commits inside a repository are **P1** only when workflows need to
  coordinate multiple writes/side effects. For single-record CRUD, call it a
  future flexibility concern.
- Escalate to **P1/P0** when the style mismatch creates concrete risk: hard-to-
  test policy, money movement, sensitive data, external side effects, or high
  change frequency.
