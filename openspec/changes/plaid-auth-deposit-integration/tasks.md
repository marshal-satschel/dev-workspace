## 1. Project Scaffolding

- [ ] 1.1 Initialize package.json, tsconfig.json, Express + TypeScript dependencies
- [ ] 1.2 Add `plaid` npm SDK, `pg`, `node-pg-migrate`, `jest` + `ts-jest`, `dotenv`
- [ ] 1.3 Set up `src/` structure per design.md (config, db, crypto, plaid, routes, services, middleware, types)
- [ ] 1.4 Add `.env.example` with all required variables (Plaid client id/secret/env, `KMS_MASTER_KEY`, `DATABASE_URL`, `PLAID_WEBHOOK_URL`)
- [ ] 1.5 Add `.gitignore` (`.env`, `node_modules`, `dist`)

## 2. Database & Migrations

- [ ] 2.1 Migration: `funding_sources` table (id, customer_ref, encrypted_access_token, iv, auth_tag, item_id, account_id, institution_name, account_mask, consent_expiration_time, requires_reauth, created_at, updated_at)
- [ ] 2.2 Migration: `deposits` table (id, funding_source_id, customer_ref, amount, status, ach_originator_ref, created_at, updated_at) — include the nullable `virtual_account_ref` extension point from design.md
- [ ] 2.3 Migration: `idempotency_keys` table (key, deposit_id, response_snapshot, created_at)
- [ ] 2.4 Verify migrations run clean up and down against a local Postgres instance

## 3. Secrets & Encryption

- [ ] 3.1 Implement `EnvelopeEncryption` interface (`encrypt`/`decrypt`) using AES-256-GCM with `KMS_MASTER_KEY`
- [ ] 3.2 Fail fast on startup if `KMS_MASTER_KEY` is missing or the wrong length
- [ ] 3.3 Unit tests: encrypt→decrypt round trip, tampered ciphertext/auth-tag rejected

## 4. Plaid Client & Webhook Verification

- [ ] 4.1 Bootstrap Plaid client from env (`PLAID_CLIENT_ID`, `PLAID_SECRET`, `PLAID_ENV=sandbox`)
- [ ] 4.2 Implement webhook JWT signature verification against Plaid's `/webhook_verification_key/get`, with key caching by `kid`
- [ ] 4.3 Unit tests: valid signature accepted, invalid/missing signature rejected, expired/wrong key rejected

## 5. Funding Source Linking

- [ ] 5.1 Placeholder customer-auth middleware (`x-customer-id` header), clearly commented as a stand-in
- [ ] 5.2 `POST /api/funding-sources/link-token`: call `/link/token/create` with `products: ["auth"]`, `country_codes: ["US"]`, stable `client_user_id`, webhook URL; return only `link_token`
- [ ] 5.3 `POST /api/funding-sources`: exchange `public_token` via `/item/public_token/exchange`; call `/auth/get` to confirm ACH numbers exist; encrypt and persist; return only funding source id, institution name, mask
- [ ] 5.4 Reject persistence (no funding source created) when `/auth/get` cannot return ACH numbers
- [ ] 5.5 Audit every response/log path in this module for access-token or full-account-number leakage (spec: `plaid-funding-source-linking` confidentiality requirements)

## 6. Deposit Initiation

- [ ] 6.1 Idempotency middleware: look up key in `idempotency_keys` before proceeding; on hit, return the stored response snapshot without re-executing
- [ ] 6.2 Reject deposit requests missing an idempotency key
- [ ] 6.3 `POST /api/deposits`: look up funding source, reject if `requires_reauth`
- [ ] 6.4 Decrypt access token, call `/auth/get` for current routing/account numbers (never read from storage)
- [ ] 6.5 Implement `AchOriginator` interface + stubbed implementation (`submitDebit`); log clearly that this is a stub, not a real submission
- [ ] 6.6 Record subledger entry keyed on `customer_ref`, not on any ACH memo/description field
- [ ] 6.7 Store the idempotency key → deposit response mapping after a successful (stubbed) submission
- [ ] 6.8 Return deposit id and status

## 7. Webhook Handling

- [ ] 7.1 `POST /api/webhooks/plaid`: verify signature first; reject unverified payloads before any parsing of webhook-specific fields
- [ ] 7.2 Handle `ITEM_ERROR` → mark funding source `requires_reauth`
- [ ] 7.3 Handle `PENDING_EXPIRATION` → mark funding source `requires_reauth`
- [ ] 7.4 Handle `SESSION_FINISHED` (Hosted Link) → record session outcome
- [ ] 7.5 Unhandled webhook types: acknowledge (2xx) without erroring, per Plaid's expected behavior

## 8. Consent Expiry & Reauth

- [ ] 8.1 Reactive expiry check: before any `/auth/get` call, compare stored `consent_expiration_time` against now; mark `requires_reauth` if passed
- [ ] 8.2 On `/auth/get` item error at deposit time (not just via webhook), mark `requires_reauth`
- [ ] 8.3 `POST /api/funding-sources/:id/reauth-link-token`: mint a Link token in update mode for the funding source's existing item
- [ ] 8.4 On successful update-mode Link completion (via the existing exchange endpoint or a dedicated path — decide during implementation), clear `requires_reauth`

## 9. Tests

- [ ] 9.1 Happy path: link token → exchange → deposit → subledger entry recorded
- [ ] 9.2 Expired consent: deposit blocked with clear error, no ACH numbers fetched
- [ ] 9.3 Item error (via webhook and via reactive `/auth/get` failure): funding source marked `requires_reauth`
- [ ] 9.4 Missing ACH numbers on exchange: funding source not persisted, clear error returned
- [ ] 9.5 Duplicate idempotency key: second request returns first request's response, no second deposit/subledger entry created
- [ ] 9.6 All tests run against Plaid sandbox (`sandbox_public_token/create`, `user_good`/`pass_good`) — confirm no test hits production Plaid

## 10. Documentation

- [ ] 10.1 README: required environment variables, with a one-line description of each
- [ ] 10.2 README: sandbox end-to-end walkthrough (create sandbox public token → exchange → deposit → inspect subledger row)
- [ ] 10.3 README: explicit callout that `AchOriginator` is stubbed pending bank-team confirmation, and where to swap in the real implementation
- [ ] 10.4 README: explicit callout that encryption is local envelope encryption, not a real KMS, with the swap point named
