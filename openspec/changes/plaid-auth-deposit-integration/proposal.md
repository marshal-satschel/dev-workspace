## Why

Customers need a way to link an external bank account and fund their Liquidity wallet. We already originate ACH debits through our sponsor bank, First Montana, so Plaid's role is verification only — confirming routing/account numbers are real and consented — not moving money. Two integration paths were considered and ruled out before this proposal: Plaid processor tokens (Liquidity has no signed partnership with Plaid as a processor destination, and we already originate through our own sponsor bank, so there is no third party to delegate to) and Plaid Transfer (would duplicate the First Montana origination relationship we already pay for and add Plaid's ledger/hold period on top of it). This proposal builds the `/auth/get`-based verification path instead.

## What Changes

- Add Plaid Link token issuance for authenticated customers (`products: ["auth"]`), minted on demand and never cached (4-hour expiry).
- Add funding source persistence: exchange the Link `public_token` for an `access_token`, confirm real ACH numbers via `/auth/get`, and store only the encrypted access token, item id, account id, institution name, and mask — never raw account/routing numbers at rest.
- Add deposit initiation: fetch current routing/account numbers at transaction time (never stored), submit the ACH debit through a **stubbed** `AchOriginator` interface pending bank-team confirmation of the actual First Montana origination path, and record the pending deposit in a subledger keyed on our internal customer id (not the ACH memo field, which banks truncate/rewrite).
- Add Plaid webhook handling with signature verification, covering `ITEM_ERROR`, `PENDING_EXPIRATION`, and `SESSION_FINISHED`.
- Add consent-expiry detection and re-link: funding sources are marked `requires_reauth` when `/auth/get` fails with an item error or `consent_expiration_time` has passed, deposits against them are blocked with a clear error, and a Link token in **update mode** can be minted to re-establish consent.
- Require an idempotency key on deposit initiation — duplicate deposits are the primary failure mode this design must prevent.

## Capabilities

### New Capabilities
- `plaid-funding-source-linking`: Link token creation, `public_token` exchange, `/auth/get` verification, and encrypted persistence of a customer's external bank funding source.
- `ach-deposit-initiation`: Idempotent deposit creation that re-fetches live ACH numbers at transaction time, submits through the stubbed `AchOriginator`, and records a subledger entry attributing the (commingled, FBO-omnibus) funds to the correct customer.
- `plaid-webhook-handling`: Signature-verified webhook ingestion for item errors, pending consent expiration, and Hosted Link session completion.
- `funding-source-reauth`: Detection of expired/broken consent, blocking of deposits against affected funding sources, and update-mode Link token issuance to restore them.

### Modified Capabilities
(none — greenfield project, nothing existing to modify)

## Impact

- **New service**: `plaid-deposit-service` (Node + TypeScript + Express + Postgres), standalone for now.
- **External dependency**: Plaid API (sandbox during development; `https://sandbox.plaid.com`).
- **Stubbed dependency**: `AchOriginator.submitDebit(...)` — the actual First Montana origination call is not implemented pending confirmation from the banking team. This is a hard boundary called out explicitly in design.md and in code comments; nothing here submits a real ACH debit yet.
- **Secrets**: Plaid credentials via environment variables only. `access_token` encrypted at rest using local envelope encryption (AES-256-GCM, master key from env) with a documented extension point to swap in a real KMS (AWS/GCP) later — there is no existing shared KMS setup for this standalone project to plug into yet.
- **Future integration point** (out of scope for this change): the Phase 2 wallet page's "Private Markets" deposit flow will call `POST /api/funding-sources/link-token` to open Plaid Link when a customer wants to fund a private-markets purchase. Not built here.
