# Component and Package Principles

Use package-level principles when reviewing modules, monolith boundaries,
microservices, libraries, or larger codebases. Apply them as design pressure,
not rigid math.

## Cohesion Principles

### REP - Reuse/Release Equivalence

Reusable components should be versioned and released as units.

Review checks:

- [ ] Classes reused together live together.
- [ ] Published packages have clear version/release boundaries.
- [ ] Consumers are not forced to pull unrelated features.

This matters most for libraries and deployable packages. It matters less inside
a small application with no independent package releases.

### CCP - Common Closure

Classes that change together should live together.

Review checks:

- [ ] A typical feature change touches a small, predictable set of modules.
- [ ] Business rule changes do not require edits across many adapters.
- [ ] Package boundaries match likely change boundaries.

Vertical feature modules are often strong:

```text
orders/
  domain/
  application/
  adapters/
  infrastructure/

products/
  domain/
  application/
  adapters/
  infrastructure/
```

Layer-first structure can also be acceptable in small systems or teams with
strong conventions:

```text
domain/
application/
adapters/
infrastructure/
```

Score based on change locality and dependency direction, not folder fashion.

### CRP - Common Reuse

Do not force clients to depend on things they do not use.

Review checks:

- [ ] Avoid grab-bag `common`, `shared`, or `utils` modules.
- [ ] Shared modules have a clear domain or technical purpose.
- [ ] Importing one helper does not pull a large dependency tree.

## Coupling Principles

### ADP - Acyclic Dependencies

Package dependencies should not cycle.

Review checks:

- [ ] No cycles in package/module graph.
- [ ] If a cycle exists, determine whether to merge modules or extract an
  inward-facing abstraction.
- [ ] Framework composition code does not create source-level cycles.

Detection examples:

```bash
dependency-cruise src --include-only "^src"
pydeps --show-cycles src/
mvn dependency:analyze
```

### SDP - Stable Dependencies

Depend in the direction of stability. Volatile code should depend on stable
policy, not the reverse.

Review checks:

- [ ] Domain policy is stable and has few outgoing dependencies.
- [ ] Infrastructure/adapters are volatile and depend inward.
- [ ] A stable shared package does not depend on volatile feature details.

### SAP - Stable Abstractions

Stable packages should expose abstractions where consumers need extension or
replacement. Do not interpret this as "domain must be all interfaces"; domain
entities are often concrete and still stable.

Review checks:

- [ ] Stable packages expose clear contracts.
- [ ] Concrete implementation details stay in volatile packages.
- [ ] Abstractions correspond to real extension points.

## Metrics

Use metrics to guide inspection:

```text
Instability I = Ce / (Ca + Ce)

Ca = afferent coupling, incoming dependencies
Ce = efferent coupling, outgoing dependencies
```

- `I` near 0: many depend on it, it depends on little.
- `I` near 1: depends on many, few depend on it.

Do not require exact target ranges for every project. Use the metric to ask:

- Is stable code depending on volatile code?
- Is a heavily depended-on package concrete and hard to change?
- Are package cycles hiding the real architecture?

## Common Findings

| Finding | Why It Matters | Fix |
|---------|----------------|-----|
| Feature change touches many packages | poor closure, shotgun surgery | regroup by feature or move policy inward |
| `shared/utils` imports infrastructure | shared code becomes volatile | split stable helpers from adapter helpers |
| Package cycle | dependency direction is unclear | merge cohesive modules or extract interface |
| Stable package imports adapter | policy depends on detail | invert dependency through a port |
| Too many tiny packages | navigation and release overhead | merge modules that change/release together |

## Reporting Guidance

When package design is the main issue, include a small dependency summary:

```text
Current: orders/application -> payments/infrastructure -> orders/domain
Risk: application policy depends on payment adapter detail.
Fix: define PaymentGateway port in application; implement in infrastructure.
```
