## 0. Resume checkpoint — do this first

- [ ] 0.1 Confirm with the user what "Resume" covers: full build, or just re-run the failed research step
- [ ] 0.2 Re-confirm the standing constraint is still in force: **nothing gets deployed to devnet**
- [ ] 0.3 Re-run the six-question research pass (see proposal.md's "Groundwork found" section) that failed mid-run — inline, without a sub-agent, to avoid re-hitting a spend cap

## 1. Success screen (once only)

- [ ] 1.1 Find Pranav's exact onboarding-complete signal in `BrokerOnboardingWrapper.tsx`/`flow.ts`
- [ ] 1.2 Success screen: "Explore Broker Admin Portal" + "Explore Liquidity" CTAs
- [ ] 1.3 Gate on an onboarding-just-completed signal (not broker-enabled status) so it never re-shows

## 2. Conditional entry point

- [ ] 2.1 Locate (or, if absent, ask backend for) a real `broker-enabled` account attribute — do not hardcode
- [ ] 2.2 Header CTA: broker-enabled → "Broker Admin" → admin portal; not onboarded → existing "Onboard as a Broker" CTA (unchanged)

## 3. Broker Admin → Clients

- [ ] 3.1 Confirm whether `views/broker/` already has a clients list / per-client orders API
- [ ] 3.2 Clients list, scoped strictly to the authenticated broker (no cross-broker leakage)
- [ ] 3.3 Per-client orders, segmented Open / Executed / Cancelled — evaluate reusing `views/MyOrders/OrdersPage.tsx`'s tab model

## 4. Settlement stage tracker (order type × structure)

- [ ] 4.1 Confirm the order-type axis (firm/indicative/SPV) — does it exist today, and does "indicative" currently reach any settlement state?
- [ ] 4.2 Extend/reuse `SettlementStepper.tsx` for a broker-view context (read + act on someone else's negotiation)
- [ ] 4.3 SPV structure: flag to backend that this needs real settlement support — it does not exist today (confirmed: FE-only disabled button)
- [ ] 4.4 No stale client cache — stage must reflect live backend state

## 5. Broker-assisted actions

- [ ] 5.1 Confirm whether any "act on behalf of" pattern exists anywhere already
- [ ] 5.2 Wire broker-invoked actions through the existing per-state action functions in `api/negotiations.ts`, with an on-behalf-of param
- [ ] 5.3 Audit logging for every broker-on-behalf action

## 6. Access provisioning (not a frontend task)

- [ ] 6.1 Backend/ops: grant `broker-enabled` to account `cmqt85tlc0007ykxgw7erl7fx`
- [ ] 6.2 Backend/ops: grant `broker-enabled` to the local dev account (resolve its ID locally)

## 7. Deploy

- [ ] 7.1 **Do not deploy to devnet** — explicit user instruction. Confirm the actual target/process with the user before any deploy step.
