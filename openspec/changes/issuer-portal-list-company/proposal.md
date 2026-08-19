## Why

Companies that want to list their assets on Liquidity had no self-service entry point. The existing header only surfaced broker-related actions. An issuer-facing portal — reachable from a prominent header CTA — gives companies a guided path through identity verification, entity onboarding, compliance review, and asset listing.

## What Changes

- **Header**: Replace "Broker Admin" and "Onboard as a broker" links with a single "List your company" primary button in `PrimaryNavHeader.tsx`. Routes to `/list-company`.
- **Route**: Add `ROUTES.LIST_COMPANY` (`/list-company`) in `routes/constants.ts` and register it in `routes/routes.tsx` with lazy loading.
- **Issuer Portal page** (`views/issuer-portal/IssuerPortalLanding.tsx`): A multi-step wizard with:
  - **Left sidebar** showing 4 phases: You (~5 min), Your Company (~20 min), Review (1–3 days), Your Asset (~10 min), with step-level navigation.
  - **Progress bar** segmented by total steps across all phases.
  - **Phase 1 — Verify your identity**: SSN input with auto-formatting (XXX-XX-XXXX), government-issued ID type selector (Passport, Driver's license, State ID), and Continue button.
  - Phases 2–4 are defined in the sidebar but not yet implemented (next iterations).

## Design Reference

Source design: Claude Design project `86efec5d-387c-4f15-b738-2eca0cff2729`, file `Issuer Portal.dc.html`.

## Impact

- **New files**: `views/issuer-portal/IssuerPortalLanding.tsx`, `ROUTES.LIST_COMPANY` constant.
- **Modified**: `PrimaryNavHeader.tsx` (removed broker CTAs, added "List your company" button), `routes/routes.tsx` (new lazy route), `routes/constants.ts` (new constant).
- **No backend changes** — this is a FE-only static wizard. Identity verification, company onboarding, and compliance review will connect to backend APIs in future iterations.
- **Branch**: `phase-2` on `liquidity-alt/exchange2.0`.
