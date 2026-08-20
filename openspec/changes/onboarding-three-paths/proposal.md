## Why

Liquidity serves three distinct user types — individual investors, entities/businesses, and brokers — each with different regulatory requirements (CIP, CDD, KYB, FINRA). The existing Tier 1 onboarding only handled individuals. Entities and brokers had no path.

## What Changes

### Account Type Selector (Tier 1 entry point)

The first screen in Tier 1 now asks "How will you be using Liquidity?" with three options:
- **Individual** — existing flow, unchanged
- **Entity or business** — new consolidated KYB flow (~25 min)
- **Broker** — new FINRA-compliant flow (~8 min)

Includes a "Not sure which one I am?" expandable with plain guidance. Greets the user by name.

### Entity Flow (8 steps, consolidated)

1. **Before You Start** — two-card checklist (about you / about the company) with reassurance that partners' SSN/ID is not needed
2. **Identity** — SSN + Gov ID (reuses existing Tier 1 step)
3. **Address** — residential (reuses existing)
4. **Liveness** — Didit handoff (reuses existing)
5. **Citizenship** — 3 fields (citizenship, country of birth, tax residence), all defaulting to US
6. **Company** — consolidated single step with collapsible sections:
   - **Search**: legal name, country/state, EIN → inline Didit lookup. If found, pre-fills details with green "from official records" tags. If not found, auto-populates manual form from search inputs
   - **Company details**: entity type (determines required docs), DBA, registered/business/mailing address, optional website + phone
   - **About the business**: description, purpose, source of funds, tax residency
   - **Tax certification**: W-9/W-8BEN-E selection + perjury certification
   - **Documents**: dynamic list by entity type, drag-and-drop upload, plus authorized signer doc + tax form
7. **Ownership & Control** — control person (self or designate someone else with name/title/email), owners/signers with three roles (Owner 25%+, Signer, Owner+Signer), optional phone with country code, "no one owns 25%+" checkbox. No SSN — people verify themselves via link
8. **Agreements** — ATS + E-Sign
9. **MPC Wallet** — same ceremony as individual, labeled as "Share Issuer"
10. **Success**

### Broker Flow (7 steps)

1. **Broker Type** — Laferty (active) / Other firm (greyed out, COMING SOON tag, "notify me" inline note)
2. **Identity** — SSN + Gov ID (reuses existing)
3. **Address** — residential (reuses existing)
4. **Liveness** — Didit (reuses existing)
5. **Licence Numbers** — personal CRD + firm CRD, live FINRA BrokerCheck lookup with three outcomes:
   - **Matched** (firm CRD `123456`) → skips supervisor, goes to agreements
   - **Mismatch** → amber note ("we're checking something"), can still continue to supervisor
   - **Invalid** → must retry
6. **Supervisor** — name, email, phone, title, department + employer permission (upload file or "contact my supervisor" with 5-day window) + trade records consent
7. **Agreements** — 7 broker-specific acknowledgements
8. **MPC Wallet** — labeled "Broker — Trading + Client Management"
9. **Success**

### Order Book Filters (Private Assets)

- "Filters" button in order book header opens floating panel
- Filter groups: All (default), Firmness (Firm/Indicative), Share Class (Common/Preferred/Others), Structure (Direct/Forward/SPV)
- SPV disabled when only Firm selected (tooltip explains why)
- Labels (CMN·DIR, PRF·FWD) adjacent to prices
- Amber warning banner "Filtered view — showing X of Y orders"
- Empty-side notes ("No sellers match — try widening the filter")
- Mobile: bottom sheet

### Issuer Portal Landing Page

- Header CTA "List your company" in PrimaryNavHeader (replaced broker links)
- Landing page at `/list-company` with Issuer Portal wizard layout

## Standalone Simulations

- `/entity-flow-sim` — walkthrough of the complete entity flow
- `/broker-flow-sim` — walkthrough of the complete broker flow

## Data Sources

- `Liquidity_io_Entity_Questionnaire_Fillable.xlsx` — 88-field CIP/CDD/KYB/AML questionnaire (Parts A–H)
- `Liquidity.io Entity Onboarding.pdf` — 12-step regulatory flow with document requirements by entity type

## Branch

`phase-2` on `liquidity-alt/exchange2.0`
