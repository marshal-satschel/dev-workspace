## Context

Greenfield standalone service. No existing HTTP framework, persistence layer, secrets infrastructure, or auth system to integrate with yet — this will eventually be called from the Phase 2 wallet app's Private Markets deposit flow, but that integration is out of scope here (see proposal.md - Impact). Plaid's role is verification-only (`/auth/get`); money movement is a separate, partially-unconfirmed integration with our sponsor bank, First Montana.

## Goals / Non-Goals

**Goals:**
- A working, testable backend against Plaid sandbox for all four capabilities in specs/.
- A clean seam between "verified account numbers" (Plaid) and "money movement" (First Montana), so the unconfirmed piece doesn't block everything else.
- Encryption and secret-handling that meets the spec's constraints today, with an explicit, narrow point to swap in real KMS later.

**Non-Goals:**
- Real ACH origination to First Montana (stubbed; see Open Questions).
- Real customer authentication (stubbed via a header-based placeholder middleware).
- Any frontend or Plaid Link UI — this service returns tokens only.
- Background job infrastructure — consent-expiry checks are reactive (at deposit time and via webhook), not a scheduled sweep.
- Per-customer virtual account numbers — an extension point is left in the subledger design, not implemented.

## Decisions

### Endpoint shape
Five endpoints as specified: `POST /api/funding-sources/link-token`, `POST /api/funding-sources`, `POST /api/deposits`, `POST /api/webhooks/plaid`, and a re-auth link-token endpoint (`POST /api/funding-sources/:id/reauth-link-token`) for update-mode Plaid Link. Kept as thin Express routes delegating to services — no framework-level reason to deviate from what the spec already lays out.

### Idempotency: dedicated table, not an in-memory cache
A `idempotency_keys` table (key, deposit_id, response_snapshot, created_at) rather than an in-memory or Redis cache. **Why**: duplicate deposits are called out as the worst failure mode; an in-memory cache is lost on restart and a Redis dependency adds infrastructure for a single-instance prototype. A DB row means the guarantee survives a crash-restart, at the cost of one extra table and one extra query per deposit. **Alternative considered**: a unique constraint directly on `deposits.idempotency_key` with a catch-and-return-existing pattern — simpler, but doesn't cleanly support returning the exact original response shape if the deposit's own state has since changed (e.g., status updated by a webhook). The dedicated table snapshots the response at creation time.

### Encryption: local envelope encryption now, KMS-shaped interface
AES-256-GCM with a master key from an environment variable, behind a small `EnvelopeEncryption` interface (`encrypt(plaintext): {ciphertext, iv, authTag}`, `decrypt(...)`). **Why**: there is no existing shared KMS/secrets setup for this standalone project (confirmed — this is a new, unattached service). The interface shape mirrors how a real KMS envelope-encryption call would be used (wrap/unwrap a data key), so swapping the implementation later doesn't touch call sites. **Trade-off**: the master key's own protection is only as good as wherever the env var lives — this is explicitly weaker than real KMS and is flagged as such in the README, not presented as equivalent.

### Webhook verification: Plaid's JWT scheme, implemented directly
Plaid signs webhooks with a JWT in the `Plaid-Verification` header, verified against a rotating public key fetched via `/webhook_verification_key/get` (cached by `kid`). Implemented directly per Plaid's documented scheme rather than searching for a wrapper library, since correctness here is a hard security requirement (the spec says "do not skip this") and a small, auditable implementation is easier to trust than an unfamiliar dependency.

### Subledger: keyed on customer id, with a virtual-account extension point
Deposits are recorded against `customer_ref` (our internal customer id) rather than any ACH-network-supplied field. The `deposits` table includes a nullable `virtual_account_ref` column, unused today, as the named extension point if First Montana turns out to support per-customer virtual account numbers — the proposal explicitly asks for this seam to exist even though it isn't implemented.

### AchOriginator: interface + stub, isolated to one module
`AchOriginator.submitDebit({ debitRouting, debitAccount, amount, customerRef })` is the only surface the deposit service calls. The stub implementation returns a synthetic pending result and logs clearly that no real origination occurred. **Why here specifically**: this is the one piece explicitly not specified by the user pending bank-team confirmation — isolating it to a single swappable module means every other capability (linking, webhooks, reauth, idempotency, subledger) is fully real and testable without waiting on that confirmation.

### Auth: placeholder header middleware
A middleware reads a customer id from a header (e.g. `x-customer-id`) and attaches it to the request. **Why**: no real auth system exists in this standalone context, and inventing one is out of scope — the eventual integration point (Phase 2 wallet) already has its own auth, which this service will need to trust or validate against once wired in. This is a clearly marked stand-in, not a security boundary.

## Risks / Trade-offs

- **[Risk]** Local envelope encryption's master key is only as protected as its env var → **Mitigation**: documented in README as explicitly provisional; `EnvelopeEncryption` interface isolates the swap to one module when real KMS is available.
- **[Risk]** Placeholder auth middleware could be mistaken for a real security boundary if this code is copied elsewhere → **Mitigation**: named and commented explicitly as a placeholder; README calls it out under constraints.
- **[Risk]** Reactive-only consent-expiry checking means an expired funding source is only caught when a deposit is attempted or a webhook happens to arrive, not proactively → **Mitigation**: acceptable for a prototype without job infrastructure; noted in README as a natural follow-up once one exists.
- **[Risk]** `AchOriginator` stub means step 3 (deposit initiation) cannot be exercised end-to-end against a real bank → **Mitigation**: everything up to and including the origination call is real and tested; the interface boundary is the intentional, visible edge of what's confirmed.

## Migration Plan

Two migrations via `node-pg-migrate`: `funding_sources` and `deposits` (which includes the `idempotency_keys` table, or a third migration if kept separate — decided at implementation time based on what reads more clearly). No existing data to migrate from; this is a new schema. Rollback is a straight `down` migration since nothing else depends on these tables yet.

## Open Questions

- The actual First Montana origination call (protocol, auth, request/response shape) is pending confirmation from the banking team — tracked via the `AchOriginator` stub, not resolved here. Does not change any spec, approach, or task in this change; swapping the stub for a real implementation is future work once confirmed.
