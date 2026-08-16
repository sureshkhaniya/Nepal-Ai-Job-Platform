# eSewa Payment Architecture

## Scope

eSewa is the primary payment gateway for the MVP. It supports employer plan purchases and, if enabled, candidate premium upgrades.

## Security invariant

A successful browser redirect is never sufficient proof of payment. Paid entitlement is granted only after server-side validation/verification according to the active eSewa integration contract.

## Flow

```text
Candidate / Employer
        |
        v
POST /v1/payments/esewa/initiate
        |
        +-- authenticate
        +-- validate plan
        +-- load price from DB
        +-- create unique transaction UUID
        +-- INSERT payment=PENDING
        +-- create required signed request
        |
        v
      eSewa
        |
   customer completes payment
        |
   success/failure return
        |
        v
 server validates response
        |
        +-- signature/response verification
        +-- transaction UUID check
        +-- product/merchant check
        +-- amount check
        +-- authoritative status/reconciliation check when required
        |
        v
 verified success?
     /      \
   yes       no/unknown
    |            |
 payment=PAID    keep failed/pending/reconciling
    |
 activate subscription atomically
```

## Payment table blueprint

```sql
CREATE TABLE esewa_payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  company_id UUID REFERENCES companies(id),
  plan_id UUID NOT NULL REFERENCES plans(id),
  transaction_uuid VARCHAR(120) NOT NULL UNIQUE,
  amount_npr NUMERIC(12,2) NOT NULL CHECK(amount_npr > 0),
  product_code VARCHAR(100) NOT NULL,
  transaction_code VARCHAR(150),
  status VARCHAR(30) NOT NULL DEFAULT 'CREATED',
  esewa_response JSONB,
  signature_verified BOOLEAN NOT NULL DEFAULT FALSE,
  initiated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  verified_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CHECK(user_id IS NOT NULL OR company_id IS NOT NULL)
);

CREATE UNIQUE INDEX uq_esewa_transaction_code
ON esewa_payments(transaction_code)
WHERE transaction_code IS NOT NULL;

CREATE INDEX idx_esewa_pending
ON esewa_payments(status,created_at)
WHERE status IN ('PENDING','VERIFYING');
```

## API

```text
POST /v1/payments/esewa/initiate
GET  /v1/payments/esewa/success
GET  /v1/payments/esewa/failure
GET  /v1/payments/:paymentId

Internal/reconciliation:
POST /internal/payments/esewa/:paymentId/verify
```

## Defensive rules

- Price and plan come from server-side database records, never from trusted client totals.
- Generate a unique merchant transaction reference for every payment attempt.
- Keep UAT and production endpoints/credentials in environment configuration.
- Never expose merchant secrets to browser JavaScript.
- Verify the exact fields required by the current eSewa specification.
- Compare monetary values using normalized decimal representations.
- Duplicate callbacks/status checks are safe.
- Lock/guard payment state transitions to prevent double entitlement.
- Persist sanitized provider response metadata for reconciliation.
- Provider timeouts leave the transaction reconcilable; they do not imply failure or success.

## Idempotent entitlement activation

Within one transaction:
1. Lock payment row.
2. If already PAID, return existing success state.
3. Validate verified provider transaction identity.
4. Transition payment to PAID.
5. Create/extend entitlement using a uniqueness/idempotency key tied to payment ID.
6. Commit.

## Reconciliation

A scheduled/queued reconciliation job should inspect payments stuck in pending/verifying states beyond a defined threshold and query the supported eSewa status mechanism. Manual admin reconciliation should also be possible with an audit trail.

## Integration contract

Before production implementation, re-check the current official eSewa developer documentation for required fields, signing rules, UAT endpoints, production endpoints, response format, and status verification. These external details can change and should not be hard-coded from old examples.
