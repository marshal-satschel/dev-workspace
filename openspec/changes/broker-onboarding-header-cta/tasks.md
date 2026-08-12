## 1. Header CTA (done)

- [x] 1.1 "Onboard as a broker" link in `PrimaryNavHeader.tsx`'s utility cluster, before the theme toggle
- [x] 1.2 Route to `ROUTES.BROKER_ONBOARDING` (Pranav's real route, not this change's original placeholder)

## 2. Reconciliation with the concurrent flow build (done)

- [x] 2.1 Merge `origin/phase-2`, resolve the `constants.ts` conflict in favor of Pranav's `BROKER_ONBOARDING` naming
- [x] 2.2 Remove this change's placeholder page, route constant, and route registration
- [x] 2.3 Verify Pranav's `BrokerOnboardingWrapper.tsx`/`flow.ts`/`panels.tsx` survived intact post-merge (restored after an accidental `rm -rf` caught via `git status`, confirmed via `git diff origin/phase-2`)
- [x] 2.4 Project-wide `tsc --noEmit` clean post-merge

## 3. Remaining (owned by Pranav on LIQ2-95, not this change)

- [ ] 3.1 KYB/CRD-link wizard — firm details, licensing/registration capture, agreement signing, review
- [ ] 3.2 Reviewer-outcome handling
- [ ] 3.3 Real compliance backend (currently FE-only reviewer-outcome controls stand in for it)
