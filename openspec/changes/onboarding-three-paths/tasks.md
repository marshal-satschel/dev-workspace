## 1. Account Type Selector (done)

- [x] 1.1 Three options: Individual, Entity, Broker — with icons, descriptions, time estimates
- [x] 1.2 "Not sure which one I am?" expandable guidance
- [x] 1.3 Greet user by first name
- [x] 1.4 Individual → existing Tier 1 flow
- [x] 1.5 Entity → entity onboarding flow
- [x] 1.6 Broker → broker onboarding flow (routed in simulation)

## 2. Entity Flow — Consolidated (done)

- [x] 2.1 Before You Start — icon cards, compact layout, "no SSN needed" reassurance
- [x] 2.2 Identity — reuses existing GovernmentIdForm
- [x] 2.3 Address — reuses existing ResidentialAddressForm
- [x] 2.4 Liveness — Didit handoff (simulated in demo)
- [x] 2.5 Citizenship — 3 fields defaulting to US
- [x] 2.6 Company Step (consolidated):
  - [x] Search phase: legal name + country/state + EIN → inline Didit lookup
  - [x] Found result: green card with company details
  - [x] Not found: amber card, auto-populates manual form from search inputs
  - [x] Details section: entity type, DBA, addresses, optional website + phone
  - [x] About the business: description, purpose, source of funds, tax residency
  - [x] Tax certification: W-9/W-8BEN-E + perjury checkbox
  - [x] Documents: dynamic by entity type, drag-and-drop, signer doc + tax form
- [x] 2.7 Ownership & Control:
  - [x] Control person: self or designate someone else
  - [x] Three roles: Owner, Signer, Owner+Signer
  - [x] Phone with country code (optional)
  - [x] "No one owns 25%+" checkbox
  - [x] No SSN — verification via link
- [x] 2.8 ATS + E-Sign agreements
- [x] 2.9 MPC wallet screen (Share Issuer)
- [x] 2.10 Success screen
- [x] 2.11 Standalone simulation at /entity-flow-sim

## 3. Broker Flow (done)

- [x] 3.1 Broker type: Laferty (active) / Other firm (COMING SOON + notify me)
- [x] 3.2 Identity — reuses existing
- [x] 3.3 Address — reuses existing
- [x] 3.4 Liveness — simulated
- [x] 3.5 Licence numbers: personal CRD + firm CRD with live FINRA lookup
  - [x] Matched (firm 123456) → skip supervisor → agreements
  - [x] Mismatch → continue to supervisor
  - [x] Invalid → retry
- [x] 3.6 Supervisor: name, email, phone, title, dept + permission (upload/request)
- [x] 3.7 Seven broker-specific agreements
- [x] 3.8 MPC wallet (Broker type)
- [x] 3.9 Success screen
- [x] 3.10 Standalone simulation at /broker-flow-sim

## 4. Order Book Filters — Private Assets (done)

- [x] 4.1 Filters button in book header
- [x] 4.2 Floating panel with All/Firmness/ShareClass/Structure tags
- [x] 4.3 SPV disabled when only Firm selected (tooltip)
- [x] 4.4 Labels (CMN·DIR) adjacent to prices
- [x] 4.5 Filtered view warning banner
- [x] 4.6 Empty-side notes
- [x] 4.7 Mobile bottom sheet

## 5. Issuer Portal Landing (done)

- [x] 5.1 "List your company" header CTA
- [x] 5.2 Landing page at /list-company

## 6. Remaining (not started)

- [ ] 6.1 Backend API integration for entity KYB (Didit real lookup)
- [ ] 6.2 Backend API integration for broker FINRA BrokerCheck
- [ ] 6.3 Supervisor permission email sending
- [ ] 6.4 Real MPC wallet creation for entity/broker accounts
- [ ] 6.5 Review/waiting states (under review, need more info, declined)
- [ ] 6.6 Right-side background activity panel
- [ ] 6.7 Resume bar on trading page ("Finish setting up your account")
- [ ] 6.8 Broker admin portal (invite clients, invite brokers, tracking)
