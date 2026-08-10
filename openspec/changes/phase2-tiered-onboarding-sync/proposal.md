## Why

Linear issue [LIQ2-100](https://linear.app/satschel/issue/LIQ2-100) ("Build 506b/506c onboarding UI + eligibility engine") currently covers the onboarding UI and eligibility engine, but the tiered accreditation flow (Tier 0 → Tier 1 → Tier 2) is not yet specified as a single synchronized pipeline across the systems that own each tier. Recent devnet changes to Tier 1 (email, phone number, CIP details, SSN updates) need to be carried into Phase 2 explicitly, and Tier 2 spans three separate systems — Simplici, Onyx Plus, and Liquidity — that currently have no defined integration contract for this flow.

## What Changes

- Define Tier 0 → Tier 1 → Tier 2 as one synchronized onboarding pipeline rather than three independently evolving flows.
- Carry the recent Tier 1 devnet changes (email, phone number, CIP, SSN updates) into the Phase 2 flow.
- Onyx Plus: push the Tier 2 support work to a new, dedicated branch and hardcode the flow specifically for Phase 2 (not the general Onyx Plus flow).
- Simplici: evaluate and decide whether a new environment is required to support Tier 2 for Phase 2, before building against it.
- Liquidity: define the BE support needed to close the loop once Simplici/Onyx Plus hand off.
- All test cases and acceptance criteria for the tiered flow must be fulfilled before this is considered done.
- Security review required given the flow touches SSN, CIP, and other identity-verification data across three systems.

## Impact

- **Tracking issue**: [LIQ2-100](https://linear.app/satschel/issue/LIQ2-100) (Linear, team LiquidityOnyx | P2, project "Liquidity 2.0 Roadmap | Phase 2", milestone M2 — Build & Integration).
- **Systems affected**: Onyx Plus (new dedicated branch), Simplici (possible new environment), Liquidity BE.
- **Open decisions**: who owns the Onyx Plus branch work and the Simplici environment decision — not yet assigned as of this proposal.
- **Not yet broken into sub-issues in Linear** — this OpenSpec change is the durable record of the requirement; sub-issue creation is pending assignee decisions.
