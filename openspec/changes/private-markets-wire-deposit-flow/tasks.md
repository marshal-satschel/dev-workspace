## 1. Frontend (done, on `phase-2`)

- [x] 1.1 `PrivateMarketsWireInstructions` component — static First Montana Bank fields (name, address, ABA, account holder name, account number) + dynamic memo/fee
- [x] 1.2 Branch `WireDepositForm` on `depositCategory` ("private" vs "public") to render it
- [x] 1.3 Swap Bank Accounts / Crypto Wallet Addresses card order on the Wallet page (bundled, unrelated)
- [x] 1.4 Temporary client-side mock for memo/fee/confirmation-id generation, clearly flagged `TEMPORARY`
- [x] 1.5 Optimistic "Pending" row prepended to the Recent Transactions store on mock submit

## 2. Backend (blocked — tracked on LIQ2-172)

- [ ] 2.1 Endpoint to generate a wire-deposit intent (memo + fee) with no linked bank account required
- [ ] 2.2 Endpoint to persist the deposit intent on submit, returning a confirmation id, and surfacing a Pending record in the transactions feed
- [ ] 2.3 Decide the memo format (`FFC LIQ {account}` vs `FFC liq2-{account}` per `universe/CLAUDE.md`'s pay-app convention) and confirm it across environments
- [ ] 2.4 Settlement path: ops/reconciliation flips the record from Pending to Deposit once the wire is matched — outside frontend scope entirely

## 3. Once backend lands

- [ ] 3.1 Swap the two `TEMPORARY` blocks in `wireDeopsitForm.tsx` for real `getWireInstructions`/`WireDepositData` calls
- [ ] 3.2 Confirm the memo displayed matches whatever format the real endpoint returns
- [ ] 3.3 Remove the optimistic-row injection once the real submit call causes the row to appear via the normal `getTransactions()` refetch
