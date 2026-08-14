## Status: NOT STARTED — captured for resume

Work was requested, exploration began, then explicitly halted by the user before any code was written. This change exists purely so the exact ask, the constraints, and the groundwork already found are not lost across a session break (account switch to a personal Max subscription, due to a business-account spend limit).

## Why

Once broker onboarding (`/onboarding/broker`, LIQ2-95) is complete, an approved broker needs a working admin surface: a one-time success screen, a header CTA that flips from "Onboard as a broker" to "Broker Admin" once approved, and the ability to see and act on their clients' settlement — because today `/broker` (`BrokerConsole.tsx`) is a pre-existing admin console whose actual client/order/settlement capabilities were not yet confirmed before work stopped.

## What Changes (the exact ask, verbatim)

1. **Success screen (once only)** — on successful completion of the broker onboarding flow (Pranav's `BrokerOnboardingWrapper.tsx`/`flow.ts`), render a success screen with two CTAs: "Explore Broker Admin Portal" → Broker Admin, "Explore Liquidity" → standard exchange entry. Show ONCE, gated on an onboarding-just-completed signal — not on broker-enabled status (so it never re-shows on later logins).
2. **Conditional entry point** — the header CTA reads a `broker-enabled` account attribute: enabled → label "Broker Admin", routes to the admin portal; not onboarded → keep the existing "Onboard as a Broker" CTA unchanged. Must read a real attribute, not be hardcoded.
3. **Broker Admin → Clients** — a Clients section listing the authenticated broker's clients; opening a client shows their orders segmented into Open / Executed / Cancelled tabs. Strictly scoped to that broker's own clients — no cross-broker leakage.
4. **Settlement stage tracker per order type × structure** — for each OPEN secondary order, render the correct settlement-stage tracker for its (order type × transaction structure) combination. Order types: firm, indicative, SPV. Structures: direct, forward, SPV. Indicative orders explicitly DO have a settlement stage (do not skip). Stage must reflect authoritative backend state, no stale client cache.
5. **Broker-assisted actions** — from the settlement view, the broker can take the next settlement action in-system on behalf of the buyer or seller (not read-only): surface order, counterparty roles, current stage, and the next required action; wire the action through; log broker-on-behalf actions.
6. **Access provisioning** — grant broker access to account `cmqt85tlc0007ykxgw7erl7fx`, and to "the local account" (resolve its ID locally).
7. **Deploy to devnet.**

Match PRD/FRD [LIQ2-87](https://linear.app/satschel/issue/LIQ2-87). Reuse existing `ui/**` components where any frontend needs designing (per the user's instruction referencing `openspec/changes/scalable-component-api`, Praveen's component-API spec, already merged into `phase-2`).

## Explicitly out of reach from this session, stated to the user before work stopped

- **Item 6 (access provisioning)**: a backend/database write (granting a `broker-enabled` flag on a real account), not achievable from the frontend repo with the tools available in this session.
- **Item 7 (devnet deploy)**: devnet is a separately-managed k8s environment (per the `universe` repo's platform specs / operator), deployed via CI and cluster credentials this session does not have. **The user separately, explicitly instructed: "Nothing should be deployed on devnet."** Treat this as a hard constraint on any future work here, not just a capability gap.

## Groundwork found before work stopped

- `apps/web/src/views/secondaries/settlement/SettlementStepper.tsx` — a real, working settlement state machine already exists, but only for negotiation `trade_type` **DIRECT** or **FORWARD** (backed by `api/negotiations.ts`'s `NegotiationView`, a `state` string enum: `MATCHED` → ... → `COMPLETE`/`CANCELLED`, per-state action buttons, price-renegotiation gating, ROFR countdown). The component's own comments say **SPV trade_type has no backend support** ("FE-ONLY: no backend — SPV settlement is not built" — shown as a disabled button at the `MATCHED` step).
- `apps/web/src/views/broker/` (the existing `/broker` console) has `BrokerConsole.tsx`, `DashboardTab.tsx`, `InviteesTab.tsx`, `DisputesTab.tsx`, `OrdersModal.tsx`, `WireDetailsModal.tsx`, `types.ts` — existence confirmed, but whether it already lists clients/orders in the shape item 3 needs was **not yet checked** when work stopped.
- `apps/web/src/views/broker-onboarding/BrokerOnboardingWrapper.tsx` + `flow.ts` (Pranav Joshi's real onboarding flow, ~1360 lines combined) — exists and is wired to `/onboarding/broker`, but the exact completion signal/callback to hook item 1's success screen into was **not yet located**.
- A background research agent was dispatched to answer, with file:line precision: (1) whether an "indicative" order type axis exists anywhere and whether it currently reaches any settlement state at all, (2) whether any `broker-enabled`/`isBroker` client-side signal already exists (JWT claim, store field) to drive item 2's conditional CTA, (3) BrokerConsole's actual current client/order data and API calls, (4) Pranav's exact onboarding-complete hook, (5) whether any "act on behalf of" pattern already exists anywhere in the codebase, (6) whether `views/MyOrders/OrdersPage.tsx`'s existing Open/Executed/Cancelled tab model (rebuilt recently, see `openspec/changes/rebuild-orders-list`) could be reused for a single client's orders instead of "my own orders." **This agent failed before returning any findings — it hit the account's spend limit mid-run.** None of its six questions are answered yet.

## Open question, unresolved when work stopped

The user said "Stop" mid-exploration, then separately said "Resume" after a rate-limit interruption — but it was never confirmed whether "Resume" meant restart the full broker-admin build, or just re-attempt the failed research step. **Do not assume either** — confirm scope with the user again before writing any code against this change.

## Impact (once resumed)

- **Tracking issue**: [LIQ2-95](https://linear.app/satschel/issue/LIQ2-95) (Build Broker Onboarding), PRD/FRD [LIQ2-87](https://linear.app/satschel/issue/LIQ2-87).
- **Likely touches**: `views/broker-onboarding/` (success screen hook), `components/Layout/PrimaryNavHeader.tsx` (conditional CTA, already has the "Onboard as a broker" link this session added), `views/broker/` (Clients section, new), `views/secondaries/settlement/SettlementStepper.tsx` and `api/negotiations.ts` (extending for broker-view + on-behalf-of actions, and — separately, as real backend work — SPV settlement support, which does not exist today).
- **Backend/ops work this depends on, not owned by this frontend change**: the `broker-enabled` account attribute and its provisioning (item 6), and whatever SPV settlement backend would be needed to make item 4's SPV structure real rather than another disabled button.
