# Security and Cross-Cutting Checks

Security is not a Clean Architecture layer, but security mistakes often reveal
bad boundaries: sensitive data crossing too many layers, secrets in logs, or
external side effects hidden inside domain logic.

## Sensitive Data Scan

Look for these names and variants:

- Payment cards: `cardNumber`, `card_number`, `ccNumber`, `pan`, `cvv`, `cvc`.
- Credentials: `password`, `passwd`, `pwd`, `secret`.
- Tokens and keys: `apiKey`, `api_key`, `token`, `authToken`, `refreshToken`.
- Government/banking IDs: `ssn`, `taxId`, `iban`, `accountNumber`,
  `routingNumber`.

## Layer Responsibilities

### Domain

- Use safe domain concepts: `CardToken`, `MaskedCard`, `HashedPassword`,
  `ApiCredentialId`, not raw secrets.
- Avoid storing CVV/CVC entirely.
- Expose masked display values only when needed.
- Do not put encryption implementation, vault calls, or tokenization SDK calls in
  entities. Depend on already-safe values or domain abstractions.

### Application / Use Cases

- Accept raw sensitive values only long enough to pass them to an appropriate
  boundary service, then discard them.
- Prefer tokenization before persistence.
- Do not log raw input DTOs.
- Do not include secrets in exceptions, domain events, metrics labels, or audit
  payloads.
- Keep authorization decisions explicit and testable.

### Interface Adapters

- Never put secrets or card data in URLs, query strings, route params, redirects,
  or referrers.
- Use HTTPS/TLS for sensitive endpoints.
- Read bearer tokens from headers in normal API flows. Some protocol-specific
  endpoints may use bodies; do not flag those without context.
- Validate and sanitize request DTOs before passing them inward.
- Redact response DTOs: last 4 digits only, no CVV, no password hashes.

### Infrastructure

- Load secrets from environment, vault, cloud secret manager, or managed config.
- Configure logging redaction centrally.
- Use provider-specific idempotency keys for payment and delivery APIs.
- Use timeouts and retries carefully; never retry non-idempotent side effects
  blindly.
- Consider certificate pinning only for high-assurance mobile/payment contexts
  where operational cost is justified.

## Common Findings

| Finding | Severity Guide | Fix |
|---------|----------------|-----|
| Secret hardcoded in source | P0 | Remove secret, rotate it, load through secret manager. |
| Secret/card data in URL | P0 | Move to HTTPS body/header as protocol-appropriate; redact logs. |
| CVV stored or logged | P0 | Stop storing, purge data, use payment tokenization. |
| Password stored raw | P0 | Hash with a modern password hashing scheme and migrate. |
| Token/card in error message | P0/P1 | Replace with generic errors and structured redacted context. |
| Raw card/password crosses domain | P1 | Tokenize/hash at boundary; pass safe value objects inward. |
| External side effect without idempotency | P1 | Add idempotency key, outbox, compensation, or retry guard. |
| Missing audit trail for sensitive action | P1/P2 | Add action-level audit without sensitive values. |

## Architecture Scoring Impact

- Deduct from **Cross-Cutting Risks** for security/data-integrity issues.
- Deduct from **Boundary & Layer Separation** when sensitive raw details leak
  across multiple layers.
- Deduct from **Dependency Direction** only when security/vendor infrastructure
  directly contaminates inner policy.
- Do not double-penalize every occurrence of the same root cause. Name the root
  cause, then cite representative evidence.

## Example

```typescript
// BAD: raw card travels into core and can leak through logs/errors.
type PaymentInput = {
  cardNumber: string;
  cvv: string;
  amount: number;
};

// BETTER: adapter receives raw card data, tokenizes, then use case sees a safe
// application-level command.
type ProcessPaymentCommand = {
  orderId: string;
  paymentToken: string;
  amount: Money;
};
```
