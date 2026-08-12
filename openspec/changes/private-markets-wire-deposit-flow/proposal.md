## Why

Customers investing in private markets need a way to fund their wallet by wire transfer, not just ACH/Plaid Transfer. Unlike the public-markets wire flow (customer's bank details are collected, Alpaca's per-transaction bank instructions are fetched from the backend), the private-markets wire deposit is a **notification-only, memo-attributed** flow: the customer wires from any bank of their choosing into Liquidity's single FBO omnibus account at First Montana, and the memo is the only thing tying that wire back to their account — there is no linked bank account on this path at all.

## What Changes

- Add a Private Markets variant of the existing dark-theme wire deposit modal (3 steps: amount → bank wire instructions → success), reusing the public-markets flow's chrome, stepper, and success screen exactly.
- Step 2 (Bank Wire Instructions) renders **static** First Montana Bank details (name, address, ABA routing number, FBO account holder name, account number) instead of the per-transaction Alpaca details the public flow fetches from the backend — these never vary per customer or per deposit.
- Only the wire memo and fee are dynamic, intended to be generated server-side per deposit.
- Swap the Bank Accounts / Crypto Wallet Addresses card positions on the Wallet page (unrelated small UI change bundled into the same work session).

## Impact

- **Tracking issue**: [LIQ2-97](https://linear.app/satschel/issue/LIQ2-97) (Build Omnibus Account ledger + reconciliation service).
- **Blocking gap found**: [LIQ2-172](https://linear.app/satschel/issue/LIQ2-172) — no backend endpoint exists yet to create this kind of deposit intent. Confirmed live against `api.phase2.satschel.com`:
  - `/v1/funding/deposit` — no GET route (404 on any query); POST requires a linked `bankAccountId`, which this flow doesn't have.
  - `/v1/bd/deposits` — 404 on both GET and POST. This is the endpoint `universe/CLAUDE.md` documents for the separate "pay" app's deposit flow; it hasn't shipped to this environment's BD build.
- **Memo format not yet reconciled**: the UI this was built against uses `FFC LIQ {account}`. `universe/CLAUDE.md` already documents a different, decided convention for the pay app — `FFC liq1-{account}` (testnet) / `FFC liq2-{account}` (devnet) / `FFC liq3-{account}` (mainnet), lowercase and env-prefixed. Whichever format the real endpoint ships with is what the frontend must match exactly — currently mocked as `FFC LIQ {account}` pending that decision.
- **Frontend state**: built and pushed to `phase-2` (`apps/web/src/views/MyWallet/conponent/deposit/component/wireDepositForm/wireDeopsitForm.tsx`, new `PrivateMarketsWireInstructions` component). Runs on a clearly-flagged temporary client-side mock — memo/fee/confirmation-id generated locally, an optimistic "Pending" row is prepended to the Recent Transactions store on submit — so the UI is reviewable before the real endpoint exists. Nothing is persisted; settlement (Pending → Deposit) cannot be simulated client-side since it's an ops/bank reconciliation event, not something the frontend can trigger.
