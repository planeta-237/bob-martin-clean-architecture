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
| ORM annotations on domain entity | `@Entity`, `@Column`, ActiveRecord base class in domain | P0/P1 |
| HTTP types in use case | Express `Request`, Spring `ResponseEntity`, Flask request | P1 |
| Vendor SDK in core policy | Stripe/S3/SMTP client imported in application/domain | P1 |
| Repository direct from controller | controller -> repo, no use case for business action | P1 |
| Service locator in core | `container.get(...)`, `ServiceLocator.get(...)` | P1 |
| Domain throws framework exception | `HttpException`, `SQLException`, ORM exception | P1/P2 |
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

```typescript
// Application owns the need.
export interface PaymentGateway {
  authorize(amount: Money, token: PaymentToken): Promise<PaymentAuthorization>;
}

export class ProcessPaymentUseCase {
  constructor(private readonly payments: PaymentGateway) {}
}

// Infrastructure owns the detail.
export class StripePaymentGateway implements PaymentGateway {
  async authorize(amount: Money, token: PaymentToken): Promise<PaymentAuthorization> {
    // Stripe SDK details live here.
  }
}
```

## Package Structure Examples

Layer-first:

```text
src/
  domain/
  application/
  adapters/
  infrastructure/
```

Feature-first with internal layers:

```text
src/
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

TypeScript:

```javascript
{
  "rules": {
    "boundaries/element-types": ["error", {
      "default": "disallow",
      "rules": [
        { "from": "domain", "allow": ["domain"] },
        { "from": "application", "allow": ["domain", "application"] },
        { "from": "adapters", "allow": ["domain", "application", "adapters"] },
        { "from": "infrastructure", "allow": ["domain", "application", "infrastructure"] }
      ]
    }]
  }
}
```

Java:

```java
layeredArchitecture()
  .layer("Domain").definedBy("..domain..")
  .layer("Application").definedBy("..application..")
  .layer("Adapters").definedBy("..adapters..")
  .layer("Infrastructure").definedBy("..infrastructure..")
  .whereLayer("Domain").mayNotAccessAnyLayer()
  .whereLayer("Application").mayOnlyAccessLayers("Domain");
```

Use tooling suggestions as recommendations unless the user asks to implement
architecture enforcement.
