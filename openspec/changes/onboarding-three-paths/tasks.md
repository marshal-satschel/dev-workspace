## 1. Account Type Selector (done)

- [x] 1.1 Three options: Individual, Entity, Broker — with icons, descriptions, time estimates
- [x] 1.2 "Not sure which one I am?" expandable guidance
- [x] 1.3 Greet user by first name
- [x] 1.4 Individual → existing Tier 1 flow
- [x] 1.5 Entity → entity onboarding flow
- [x] 1.6 Broker → broker flow (navigates to /broker-flow-sim from Tier 1)

## 2. Entity Flow — Consolidated (done)

- [x] 2.1 Before You Start — icon cards, compact layout, "no SSN needed" reassurance
- [x] 2.2 Identity — reuses existing GovernmentIdForm
- [x] 2.3 Address — reuses existing ResidentialAddressForm
- [x] 2.4 Liveness — Didit handoff (simulated in demo)
- [x] 2.5 Citizenship — 3 fields (citizenship, birth, tax residence) defaulting to US
- [x] 2.6 Company Step (consolidated):
  - [x] Search phase: legal name + country/state + EIN → inline Didit lookup
  - [x] Found result: green card with company details
  - [x] Not found: amber card, auto-populates manual form from search inputs
  - [x] Details section: entity type, DBA, addresses, optional website + phone
  - [x] About the business: description, purpose, source of funds, tax residency
  - [x] Tax certification: W-9/W-8BEN-E + perjury checkbox
  - [x] Documents: dynamic by entity type, drag-and-drop, signer doc + tax form
- [x] 2.7 Ownership & Control:
  - [x] Control person: self or designate someone else (name, title, email)
  - [x] Three roles: Owner, Signer, Owner+Signer
  - [x] Phone with country code (optional)
  - [x] "No one owns 25%+" checkbox
  - [x] No SSN — verification via link
- [x] 2.8 ATS + E-Sign agreements
- [x] 2.9 MPC wallet screen (Share Issuer) — works without backend
- [x] 2.10 Marks onboarding complete (clearTier1Progress + primeOnboardingStatus)
- [x] 2.11 "Create your first asset" → /asset-issuance
- [x] 2.12 Standalone simulation at /entity-flow-sim

## 3. Broker Flow (done)

- [x] 3.1 Broker type: Laferty (active) / Other firm (COMING SOON + notify me)
- [x] 3.2 Identity — reuses existing
- [x] 3.3 Address — reuses existing
- [x] 3.4 Liveness — simulated
- [x] 3.5 Licence numbers: personal CRD + firm CRD with live FINRA lookup
  - [x] Matched (firm 123456) → skip supervisor → agreements
  - [x] Mismatch → can continue to supervisor
  - [x] Invalid → retry
- [x] 3.6 Supervisor: name, email, phone, title, dept + permission (upload/request)
- [x] 3.7 Seven broker-specific agreements
- [x] 3.8 MPC wallet (Broker type) — works without backend
- [x] 3.9 Marks onboarding complete (primeOnboardingStatus)
- [x] 3.10 Navigates to exchange on completion
- [x] 3.11 Standalone simulation at /broker-flow-sim

## 4. Order Book Filters — Secondaries (done)

- [x] 4.1 Filters button in SecondaryAsset order book header
- [x] 4.2 Floating panel with All/Firmness/ShareClass/Structure tags
- [x] 4.3 SPV disabled when only Firm selected (tooltip)
- [x] 4.4 Labels (CMN·DIR) adjacent to prices
- [x] 4.5 Filtered view warning banner
- [x] 4.6 Empty-side notes
- [x] 4.7 Mobile bottom sheet
- [x] 4.8 Removed from PrivateAsset (was wrong location, moved to secondaries only)

## 5. Completion & Status (done)

- [x] 5.1 Entity/broker wallet screens call clearTier1Progress + primeOnboardingStatus
- [x] 5.2 No more "finish setup" banner after completing entity/broker flow
- [x] 5.3 Account settings reflect completed status
- [x] 5.4 Header CTA "List your company" removed (entity flow handles this)

## 6. Asset Issuance (done)

- [x] 6.1 Asset list / empty state at /asset-issuance
- [x] 6.2 Create form with collapsible sections (shares, numbers, settings, docs)
- [x] 6.3 Live ticker availability check
- [x] 6.4 Live total calculation (shares × price)
- [x] 6.5 Review summary before submit
- [x] 6.6 Under review with three-badge progress
- [x] 6.7 Changes requested with field-level items
- [x] 6.8 Approved (not live) state
- [x] 6.9 Preview before publish
- [x] 6.10 Live confirmation
- [x] 6.11 Sell shares with 3% fee breakdown
- [x] 6.12 Entity wallet → "Create your first asset" → /asset-issuance

## 7. Remaining (not started)

- [ ] 7.1 Backend API integration for entity KYB (Didit real lookup)
- [ ] 7.2 Backend API integration for broker FINRA BrokerCheck
- [ ] 7.3 Supervisor permission email sending
- [ ] 7.4 Real MPC wallet creation for entity/broker accounts
- [ ] 7.5 Review/waiting states (under review, need more info, declined)
- [ ] 7.6 Right-side background activity panel
- [ ] 7.7 Resume bar on trading page ("Finish setting up your account")
- [ ] 7.8 Broker admin portal (invite clients, invite brokers, tracking)
- [ ] 7.9 Backend: asset CRUD API + admin review workflow
- [ ] 7.10 Backend: on-chain token minting on publish
- [ ] 7.11 Ongoing asset dashboard (issued vs held, open orders, activity)
