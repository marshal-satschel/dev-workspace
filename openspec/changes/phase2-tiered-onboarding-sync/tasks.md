## 1. Tier Pipeline Sync

- [ ] 1.1 Define Tier 0 → Tier 1 → Tier 2 as a single synchronized pipeline (state transitions, ownership handoffs between systems)
- [ ] 1.2 Document where each tier's state lives today and who/what updates it

## 2. Tier 1 — Devnet Sync

- [ ] 2.1 Identify the recent devnet changes to email, phone number, CIP details, and SSN updates
- [ ] 2.2 Carry those changes into the Phase 2 flow

## 3. Tier 2 — Onyx Plus

- [ ] 3.1 Push Onyx Plus Tier 2 support work to a new, dedicated branch
- [ ] 3.2 Hardcode the flow specifically for Phase 2 (not general Onyx Plus)

## 4. Tier 2 — Simplici

- [ ] 4.1 Evaluate whether a new environment is required for Tier 2 / Phase 2
- [ ] 4.2 Decide and document the environment approach before building against it

## 5. Tier 2 — Liquidity

- [ ] 5.1 Define BE support needed once Simplici/Onyx Plus hand off

## 6. Verification

- [ ] 6.1 Test cases covering all tier transitions and the 10 accreditation scenarios already on LIQ2-100
- [ ] 6.2 Acceptance criteria fulfilled across tiers 0–2
- [ ] 6.3 Security review of SSN/CIP/identity data handling across Simplici, Onyx Plus, and Liquidity
