## Why

Brokers need a way to apply to join the platform, and there was no discoverable entry point for it — the admin console (`/broker`) is for an already-approved broker, not a prospective one. A header-level CTA, always visible in the utility cluster, is the recruitment surface.

## What Changes

- Add "Onboard as a broker" as a standalone-style link in the signed-in header's utility cluster, before the theme toggle.
- Add `ROUTES.BROKER_ONBOARDING` (`/onboarding/broker`) as an unprotected route — a prospective broker is not necessarily a signed-in investor.
- **Reconciled with a concurrent build**: this was built against a placeholder page (`BrokerOnboardingPage.tsx`, a "coming soon" stub) since no real flow existed on `phase-2` at the time. While pushing, Pranav Joshi's real KYB/CRD-link wizard (`BrokerOnboardingWrapper.tsx` + `flow.ts` + `panels.tsx`, tracked on the same ticket, LIQ2-95) landed at the identical route with a differently-named route constant (`BROKER_ONBOARDING` vs. this change's original `ONBOARDING_BROKER`). Resolved during merge: dropped the placeholder page and its route constant entirely, pointed the header CTA at Pranav's real route instead. One implementation, no dangling dead route.

## Impact

- **Tracking issue**: [LIQ2-95](https://linear.app/satschel/issue/LIQ2-95) (Build Broker Onboarding), reassigned to Pranav Joshi — he was already building the real flow in parallel.
- **New**: header CTA in `PrimaryNavHeader.tsx`, `ROUTES.BROKER_ONBOARDING` route constant (Pranav's, kept), route registration (Pranav's, kept).
- **Removed during reconciliation**: `views/broker-onboarding/BrokerOnboardingPage.tsx` (placeholder, superseded), `ROUTES.ONBOARDING_BROKER` (this change's original constant name, superseded by Pranav's `BROKER_ONBOARDING`).
- **Near-miss worth naming**: resolving this by hand included an `rm -rf` on the shared `views/broker-onboarding/` directory intended to remove only the placeholder file, which also deleted Pranav's three real flow files from disk. Caught immediately via `git status` (showed as `AD` — added by the merge, deleted from disk) and restored with `git restore`, confirmed byte-identical to his pushed version via `git diff origin/phase-2`. No data was lost, but it's the reason to `rm` a single file by exact path rather than a shared directory when a merge has just placed unfamiliar files into it.
