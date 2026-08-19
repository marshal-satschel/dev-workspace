## 1. Header CTA (done)

- [x] 1.1 Remove "Broker Admin" and "Onboard as a broker" links from `PrimaryNavHeader.tsx`
- [x] 1.2 Add "List your company" primary button routing to `ROUTES.LIST_COMPANY`

## 2. Route setup (done)

- [x] 2.1 Add `LIST_COMPANY: "/list-company"` to `routes/constants.ts`
- [x] 2.2 Register lazy-loaded route in `routes/routes.tsx`
- [x] 2.3 Add `IssuerPortalLanding` lazy import

## 3. Phase 1 — Verify your identity (done)

- [x] 3.1 Issuer Portal header with "Liquidity Issuer Portal" branding
- [x] 3.2 Left sidebar with 4 phases and step-level navigation
- [x] 3.3 Progress bar segmented by total steps
- [x] 3.4 SSN input with auto-formatting (XXX-XX-XXXX) and visibility toggle
- [x] 3.5 Government-issued ID type dropdown (Passport, Driver's license, State ID)
- [x] 3.6 Continue button

## 4. Phase 2 — Your Company (not started)

- [ ] 4.1 Additional details step — entity name, legal structure, jurisdiction, address
- [ ] 4.2 Ownership & control step — beneficial owners (25%+), control persons
- [ ] 4.3 Tax certification step — W-9 / W-8BEN-E upload or inline form

## 5. Phase 3 — Review (not started)

- [ ] 5.1 Agreements & submit step — terms acceptance, compliance disclosures, submit for review
- [ ] 5.2 Review status screen — pending / approved / rejected states

## 6. Phase 4 — Your Asset (not started)

- [ ] 6.1 Asset listing creation — asset name, type, description, documents
- [ ] 6.2 Minting flow — on-chain token creation on approval

## 7. Backend integration (not started)

- [ ] 7.1 Connect identity verification to KYC/AML provider
- [ ] 7.2 Connect company onboarding to entity management API
- [ ] 7.3 Connect review step to compliance workflow
- [ ] 7.4 Connect asset listing to token issuance pipeline
