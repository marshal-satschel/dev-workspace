## Why

Approved entities need to turn their business into tradeable shares. After entity onboarding + wallet creation, there was no path to create and list an asset. This flow covers the full lifecycle: create → review → approve → publish → sell.

## What Changes

### Asset Issuance Page (`/asset-issuance`)

**Entry point**: Green "Create your first asset" button on the entity wallet success screen. Also accessible directly via URL for returning users.

**Empty state**: Explanation of what an asset is + one button to begin. **List state**: Asset cards with status pills (DRAFT, UNDER REVIEW, CHANGES REQUESTED, APPROVED, LIVE).

**Create form** — single page with collapsible sections:
1. **What the shares are called**: name, ticker (live availability check — "LIQ" is taken in demo), asset type, logo upload
2. **The numbers**: total shares, price per share, currency, minimum investment, use of proceeds + live total calculation (shares × price)
3. **Account settings** (read-only): wallet address (auto-filled), offering type (from sign-up choice)
4. **Documents**: offering document, risk factors, financial statements (drag-and-drop)

DRAFT pill visible from first moment. "All changes saved" indicator.

**Review summary** before submit — all fields in one view, last chance to edit.

**Under review**: three-badge progress (Submitted → Being reviewed → Final approval). Copy distinguishes this from the business approval ("your business is already approved — this is about the listing").

**Changes requested**: specific field-level items with amber cards, "Edit this field" links, everything else preserved. Resubmit button.

**Approved (not live)**: distinction made unmistakable — approved to publish, not yet published.

**Preview**: buyer-facing view showing logo, name, ticker, price, shares, total raising.

**Publish**: one-click with confirmation. Genuinely irreversible.

**Live**: celebration + link to sell shares + link to trading page.

**Sell shares**: standard order form with transparent 3% fee breakdown (subtotal, fee, net amount). Settlement note about transfer agent.

### Header Change

Removed "List your company" button from PrimaryNavHeader. Entity onboarding now handles this through the account type selector → entity flow → asset issuance path.

### Entity Flow Integration

Entity wallet success screen now shows green "Create your first asset" button as the primary CTA, with "Go to dashboard" as secondary.

## Branch

`phase-2` on `liquidity-alt/exchange2.0`
