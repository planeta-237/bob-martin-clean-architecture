---
name: bob-martin-clean-architecture
description: >
  Review, design, and explain Robert C. Martin's layered Clean Architecture:
  Entities, Use Cases, Interface Adapters, and Frameworks & Drivers, governed by
  the Dependency Rule. Use when the user asks for architecture review, Clean
  Architecture compliance, dependency-rule validation, layer separation,
  SOLID/modularity review, architectural scoring, refactoring guidance,
  framework/database decoupling, use-case boundary design, or learning Clean
  Architecture. Supports modular TypeScript, Java, C#, Python, Kotlin, Go, and
  Ruby codebases. For Python/Pydantic/FastAPI, applies current boundary,
  validation, serialization, DI, routing, tests, settings, and DTO practices
  without making framework models the domain by default. In
  review mode, produces evidence-based findings, calibrated scoring, positives,
  and priorities. In design/teaching mode, produces canonical layered Clean
  Architecture with entities, interactors, boundaries, controllers, presenters,
  gateways, frameworks, Main, trade-offs, and clear explanations.
---

# Bob Martin Clean Architecture

Review, design, and explain architecture from concrete context. Calibrate the
result so it helps a team decide what to build, teach, or fix first, not just
whether code resembles a textbook.

## Choose the Mode

Pick one mode from the user's intent:

- **Review mode**: evaluate existing code, score architecture, find risks.
- **Design mode**: design Robert C. Martin's layered Clean Architecture for a
  feature, service, module, or project.
- **Teaching mode**: explain Clean Architecture concepts, distinguish them from
  other styles only when asked, or teach why a boundary exists.

Do not force scoring in design or teaching mode unless the user asks for it.

## Operating Principles

- Treat Clean Architecture as a dependency and boundary discipline, not a file
  naming ceremony.
- Separate core violations from style preferences and optional patterns.
- Cite concrete evidence for every major deduction: file, line, dependency,
  method, or behavior.
- Mark missing context as unknown instead of assuming failure.
- Avoid double-counting one root cause across many dimensions.
- Adapt strictness to the stated goal: production merge review, teaching
  example, legacy triage, greenfield design, or lightweight sanity check.
- Do not auto-fix unless the user explicitly asks for implementation.

## Workflow

### 1. Establish Scope and Intent

- If the user provides paths, review those paths.
- If the user provides snippets, review only the snippet and note context gaps.
- If the user asks about a project, inspect likely source roots and architecture
  boundaries before scoring.
- If the user asks a general architecture question, answer from the reference
  files without forcing a score.
- If the user asks to design, first identify business capabilities, volatile
  details, use cases, domain rules, external actors, persistence, delivery
  mechanisms, and failure modes.
- If the project uses Python or Pydantic, check whether Pydantic belongs to
  interface adapters, framework DTOs, settings, or boundary validation before
  placing it inside entities/use cases.
- If the project uses FastAPI, treat `FastAPI`, `APIRouter`, `Depends`,
  `Request`, `Response`, `HTTPException`, middleware, and lifespan code as
  outer-layer delivery/framework details unless there is explicit evidence
  otherwise.

### 2. Classify or Design the Architecture

For review mode, identify the intended style before judging:

- Robert C. Martin Clean Architecture
- Layered MVC/service architecture
- Transaction script / active record
- Modular monolith
- Microservice or package-level design
- Other / unknown / mixed

If the project is not trying to use Clean Architecture, score "fitness for Clean
Architecture" and explicitly say that a lower score may reflect style mismatch,
not necessarily bad engineering.

Do not turn another architecture style into the design target. When the user
asks for Clean Architecture, use Robert C. Martin's layered/concentric model as
the reference.

For simple CRUD, scripts, prototypes, and low-risk admin flows, do not imply
that Clean Architecture ceremony is automatically required. If the design is a
style mismatch, separate:

- **Clean Architecture fitness**: how close it is to the dependency/boundary
  model.
- **Engineering adequacy for context**: whether the current simpler style may be
  acceptable for the risk and expected change rate.

For design mode, propose a structure around:

- **Entities**: enterprise business rules and invariants.
- **Use Cases / Interactors**: application operations, input boundaries, output
  boundaries, request/response models.
- **Interface Adapters**: controllers, presenters, gateways, mappers.
- **Frameworks & Drivers**: web framework, DB, ORM, UI, queues, vendors,
  filesystem, external services.
- **Main / Composition Root**: where concrete implementations are wired.
- **Tests**: isolated domain/use-case tests plus adapter/integration tests.

### 3. Load References as Needed

Load only the reference files needed for the current task:

| File | Load When |
|------|-----------|
| `references/design-and-teaching.md` | designing an architecture or explaining Clean Architecture concepts |
| `references/scoring-rubric.md` | assigning or calibrating the numeric score |
| `references/layer-checklists.md` | reviewing domain, use cases, adapters, infrastructure |
| `references/dependency-rule.md` | dependency direction, ports, inversion, boundary violations |
| `references/solid-principles.md` | SOLID, CQS, class/interface design |
| `references/code-smells.md` | smells and refactoring recommendations |
| `references/component-principles.md` | package/module cohesion, cycles, stability |
| `references/security-checks.md` | sensitive data, credentials, payment, auth, audit concerns |
| `references/python-pydantic.md` | Python or Pydantic codebases, DTOs, validation, settings, serialization |
| `references/fastapi.md` | FastAPI apps, APIRouter, Depends, response models, lifespan, TestClient |

### 4. For Review Mode: Score with Calibration

Use the 10-point weighted model. Anti-patterns inform deductions inside the
dimensions instead of creating a separate double-penalty bucket.

| Dimension | Max | What to Check |
|-----------|-----|---------------|
| Dependency Direction | 2.0 | Source dependencies point inward; ports isolate outer details |
| Boundary & Layer Separation | 2.0 | responsibilities are separated; DTOs/mappers at boundaries when useful |
| Domain & Use-Case Modeling | 1.5 | business rules, invariants, use-case orchestration, transaction intent |
| Modularity & SOLID | 1.5 | SRP/OCP/LSP/ISP/DIP, cohesion, coupling, code smells |
| Testability & Change Safety | 1.5 | core testable without infrastructure; deterministic seams; safe change paths |
| Cross-Cutting Risks | 1.0 | security, data integrity, observability, external side effects |
| Documentation & Narrative | 0.5 | ADRs/README/PR context/naming where relevant |

Scoring anchors:

- **9.3-10.0**: exemplary, production-grade reference implementation.
- **8.5-9.2**: strong architecture with minor, non-blocking gaps.
- **7.5-8.4**: good structure; several improvements remain.
- **6.0-7.4**: usable but important architectural gaps need attention.
- **4.5-5.9**: significant rework needed before relying on the design.
- **0.0-4.4**: fundamental boundary failure, unsafe coupling, or no meaningful architecture.

Reserve **0-1** for code that is unreviewable, dangerous, or has almost no
separable structure. Typical "bad architecture teaching examples" usually land
around **2-4**, unless they also have severe security/data-loss behavior.

### 5. Apply Severity

- **P0**: blocks merge or production use. Examples: core depends directly on
  frameworks/DB, secrets leak, payment/data integrity can break, business rules
  cannot be tested without live infrastructure.
- **P1**: important redesign or refactor. Examples: fat use cases/controllers,
  missing ports for external services, unclear transaction boundaries, anemic
  domain in a domain-heavy system.
- **P2**: improvement. Examples: naming, package layout, optional presenter,
  line-count heuristics, better docs, more focused interfaces.

When the main issue is style mismatch rather than concrete risk, phrase findings
as "Clean Architecture fitness gaps" and avoid treating every missing layer/port
as a merge blocker. Escalate only when the missing boundary harms testing,
change safety, data integrity, security, or expected feature growth.

## Design Mode Output

Default format for architecture design:

```markdown
## Architecture Shape
- Robert C. Martin layered Clean Architecture and why it fits.

## Core Policies
- Entities/value objects/domain services.
- Use cases/interactors with input and output boundaries.

## Boundaries
| Boundary | Inner Owner | Outer Implementer | Why It Exists |
|----------|-------------|-------------------|---------------|

## Suggested Structure
- `src/entities/...`
- `src/use-cases/...`
- `src/interface-adapters/...`
- `src/frameworks-drivers/...`
- `src/main/...`

## Flow
1. Actor -> Controller -> Input Boundary -> Use Case Interactor -> Entity.
2. Interactor -> Output Boundary -> Presenter -> View Model/Response.
3. Interactor -> Gateway Interface -> Gateway Implementation -> DB/vendor.

## Trade-offs
- Boundaries to draw now.
- Boundaries to defer.

## Tests
- Domain tests.
- Use-case tests with fakes.
- Adapter/integration tests.
```

Ask at most two clarifying questions if missing information would materially
change the architecture. Otherwise state assumptions and proceed.

## Teaching Mode Output

Default format for teaching/explanation:

```markdown
## Short Answer
...

## Mental Model
...

## Small Example
...

## Common Mistakes
...

## How To Apply It Here
...
```

Use the user's code or domain when available. Prefer concrete examples over
abstract slogans.

## Output Guidance

When the user asks for a "review", follow code-review style: lead with findings
ordered by severity, then score and recommendations. When the user explicitly
asks for scoring, put the score near the top.

Default format:

```markdown
## Findings
- [P0] [File:Line] Problem. Why it matters. Concrete fix.
- [P1] [File:Line] Problem. Why it matters. Concrete fix.

## Score
Overall: X.X/10 ([Grade])

| Dimension | Score | Max | Notes |
|-----------|-------|-----|-------|

## Positive Aspects
- ...

## Priority Fix List
1. [P0] ...
2. [P1] ...
3. [P2] ...

## Assumptions / Unknowns
- ...
```

For quick checks, keep the report short and explain that the score is approximate.
For full project reviews, include a layer-by-layer checklist and dependency
direction summary.

## Calibration Rules

- Do not mechanically subtract the quick diagnostic from 10 for the final score.
- Do not require a presenter layer, DTO for every internal call, 80% coverage,
  or specific line-count limits unless the project's standards require them.
- Treat CQS, object calisthenics, package metrics, and line counts as heuristics.
- Penalize transaction and side-effect handling based on actual risk. External
  APIs cannot be rolled back by a database transaction; look for idempotency,
  compensation, outbox, or documented eventual consistency.
- Give credit for pragmatic, testable designs even if they are not textbook
  Clean Architecture.
- Deduct for framework convenience only when it crosses into core policy or
  makes testing/change materially harder.
- In Python/Pydantic projects, do not reward Pydantic usage by itself. Reward it
  when it improves boundary validation, serialization, settings, or adapter
  clarity without leaking framework concerns into entities or use cases.
- In FastAPI projects, do not reward framework convenience by itself. Reward it
  when routes, dependencies, response models, lifespan, and tests stay at the
  delivery/composition edge and preserve use-case/domain independence.

## Quick Diagnostic

Use these questions to orient the review, not to compute the final score:

1. Can business rules be tested without a DB, web server, or external API?
2. Do source dependencies point from volatile details toward stable policy?
3. Are ports/interfaces defined where outer details must be replaceable?
4. Are entities/value objects responsible for important invariants?
5. Are use cases thin enough to orchestrate rather than hide business rules?
6. Are external side effects isolated, idempotent, or compensated?
7. Can the database, HTTP framework, payment provider, or queue be swapped with
   limited changes?
8. Is sensitive data tokenized, masked, or kept out of logs/URLs/errors?
9. Are package/module cycles absent or explicitly justified?
10. Would a new maintainer understand the boundaries from names and docs?
