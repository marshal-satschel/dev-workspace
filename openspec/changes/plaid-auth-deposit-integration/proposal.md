## Why

Customers need a way to link an external bank account and fund their Liquidity wallet.

**Revision (2026-08-10): superseded the original Auth-only decision.** This proposal originally used Plaid `/auth/get` for verification only, with First Montana originating the ACH debit directly, explicitly ruling out Plaid Transfer ("would duplicate the First Montana origination relationship we already pay for and add Plaid's ledger/hold period on top of it"). Per direction on [LIQ2-97](https://linear.app/satschel/issue/LIQ2-97), that decision is now reversed: Liquidity will use **Plaid Transfer** directly rather than self-originating through First Montana. This is a real trade-off, not a free upgrade — documented in full under Decisions in design.md — and it is taken deliberately:

- Plaid Transfer holds settled funds in a Plaid-owned **Ledger balance** before they are swept to our own bank account. This is exactly the "Plaid ledger/hold period" the original decision avoided; it is now accepted in exchange for not having to build and maintain our own ACH origination path.
- Transfer requires a **Custom plan with a 12-month minimum contract** (not available pay-as-you-go) and a separate **originator approval** (Transfer Application/Questionnaire, ACH use-case and processing-history review) before production access — tracked as [LIQ2-164](https://linear.app/satschel/issue/LIQ2-164).
- Plaid processor tokens remain ruled out for the same reason as before: Liquidity has no signed partnership with Plaid as a processor destination.

## What Changes

- Add Plaid Link token issuance for authenticated customers (`products: ["transfer"]`), minted on demand and never cached (4-hour expiry).
- Add funding source persistence: exchange the Link `public_token` for an `access_token`, confirm real ACH numbers via `/auth/get`, and store only the encrypted access token, item id, account id, institution name, and mask — never raw account/routing numbers at rest.
- Add deposit initiation via **Plaid Transfer**: request a transfer authorization (`/transfer/authorization/create`), create the transfer (`/transfer/create`) if approved, and record the pending deposit in a subledger keyed on our internal customer id (not the ACH memo field, which banks truncate/rewrite). No self-built `AchOriginator` is needed — Plaid originates the debit.
- Add a ledger-sweep step: settled Transfer funds land in Plaid's Ledger balance first and must be withdrawn/distributed (`/transfer/ledger/withdraw`) to our FBO omnibus account at First Montana — this replaces First Montana as *originator* with First Montana as the *sweep destination*.
- Add Plaid webhook handling with signature verification, covering `ITEM_ERROR`, `PENDING_EXPIRATION`, `SESSION_FINISHED`, and Transfer-specific events (e.g. transfer status updates).
- Add consent-expiry detection and re-link: funding sources are marked `requires_reauth` when `/auth/get` fails with an item error or `consent_expiration_time` has passed, deposits against them are blocked with a clear error, and a Link token in **update mode** can be minted to re-establish consent.
- Require an idempotency key on deposit initiation — duplicate deposits are the primary failure mode this design must prevent.

## Capabilities

### New Capabilities
- `plaid-funding-source-linking`: Link token creation, `public_token` exchange, `/auth/get` verification, and encrypted persistence of a customer's external bank funding source.
- `ach-deposit-initiation`: Idempotent deposit creation that re-fetches live ACH numbers at transaction time, submits through **Plaid Transfer** (no stub), sweeps settled funds from Plaid's Ledger to the FBO account, and records a subledger entry attributing the (commingled, FBO-omnibus) funds to the correct customer.
- `plaid-webhook-handling`: Signature-verified webhook ingestion for item errors, pending consent expiration, Hosted Link session completion, and Transfer status events.
- `funding-source-reauth`: Detection of expired/broken consent, blocking of deposits against affected funding sources, and update-mode Link token issuance to restore them.

### Modified Capabilities
(none — greenfield project; the Auth-only → Transfer revision above happened before this change was ever implemented, so it's captured as a revision to this same proposal rather than a separate modify-change)

## Impact

- **New service**: `plaid-deposit-service` (Node + TypeScript + Express + Postgres), standalone for now.
- **External dependency**: Plaid API, specifically the **Transfer** product (sandbox during development; `https://sandbox.plaid.com`). Production requires Plaid originator approval — see [LIQ2-164](https://linear.app/satschel/issue/LIQ2-164) (Marshal Tavakar) and the Custom-plan/12-month-contract requirement above.
- **Sandbox integration**: tracked as [LIQ2-162](https://linear.app/satschel/issue/LIQ2-162) (Akhil Bharti) — start with `/sandbox/transfer/...` flows before production access lands.
- **No more `AchOriginator` stub**: Plaid Transfer is the real origination path once sandbox/production access exists. There is no unimplemented boundary here anymore — the only blocker is Plaid's own originator-approval timeline, not an internal design gap.
- **Secrets**: Plaid credentials via environment variables only. `access_token` encrypted at rest using local envelope encryption (AES-256-GCM, master key from env) with a documented extension point to swap in a real KMS (AWS/GCP) later — there is no existing shared KMS setup for this standalone project to plug into yet.
- **Frontend/on-chain integration point**: the Phase 2 wallet page's "Private Markets" deposit flow will call `POST /api/funding-sources/link-token` to open Plaid Link when a customer wants to fund a private-markets purchase. Per [LIQ2-163](https://linear.app/satschel/issue/LIQ2-163) (Karishma Agrawal), that frontend deposit must also register on the blockchain wallet, and wallet-page balance calculations must reconcile against this service's subledger. Not built here — tracked as a downstream dependency.
