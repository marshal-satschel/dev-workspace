## Why

Customers need a way to link an external bank account and fund their Liquidity wallet.

**Decision record (2026-08-10): Option 2 — Plaid-Led Deposits (Plaid Transfer), selected.** Internally this was scoped as two options ("How Customer Deposits Will Work: Two Options" — LIQ2 cross-functional review):

- **Option 1 (Bank-Led)**: Plaid verifies the account only; First Montana originates the ACH debit directly into the FBO account. This was the original design in this proposal.
- **Option 2 (Plaid-Led / Plaid Transfer)**: Plaid both verifies *and* pulls the funds into a Plaid-operated holding account (the Ledger balance), then forwards them to our FBO account at First Montana. **This is the selected option**, per direction on [LIQ2-97](https://linear.app/satschel/issue/LIQ2-97).

The tradeoff, stated plainly: Option 1 means First Montana is the only party ever holding customer funds, at the cost of Liquidity building and maintaining the ACH origination machinery (submission format, daily deadlines, testing, returns/NSF risk) ourselves. Option 2 means Plaid builds and holds that machinery — including fraud-scoring, refunds, repeat deposits, and automatic use of faster payment rails — at the cost of funds sitting with Plaid first, an extra hop before money is usable, paying both Plaid and First Montana, and record-keeping that has to reconcile against Plaid's Ledger rather than bank records alone. Full comparison lives in the source document; the summary above is preserved here as the record of *why* Option 2 was chosen, not just *that* it was.

**Unresolved risk carried over from that document, not yet closed out**: Plaid's own documentation states Transfer does not support marketplaces or money-transfer apps, and Liquidity pools customer deposits into a single omnibus account with internal ownership tracking — structurally similar to what Plaid excludes. **Liquidity has not yet asked Plaid whether it qualifies for Transfer at all.** Until that's confirmed, production build-out (LIQ2-164) is at risk of being blocked entirely, not just delayed. This is the single highest-priority open question in this change — see design.md Open Questions.

- Plaid Transfer holds settled funds in a Plaid-owned **Ledger balance** before they are swept to our own bank account — accepted deliberately in exchange for not building ACH origination ourselves.
- Transfer requires a **Custom plan with a 12-month minimum contract** (not available pay-as-you-go) and a separate **originator approval** (Transfer Application/Questionnaire, ACH use-case and processing-history review, and — per the qualification risk above — a marketplace/money-transfer-app eligibility check) before production access — tracked as [LIQ2-164](https://linear.app/satschel/issue/LIQ2-164).
- Plaid processor tokens remain ruled out for the same reason as before: Liquidity has no signed partnership with Plaid as a processor destination.
- Legal counsel still needs to define the exact authorization wording the customer agrees to before a debit is pulled — needed under either option, not yet resolved, and out of scope for this engineering spec to draft.

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
