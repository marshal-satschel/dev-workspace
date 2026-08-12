## Why

Tier-1-approved investors need an MPC wallet before they can hold or trade anything, but nothing in the onboarding flow actually created one at the right moment — `WalletStep.tsx` existed but was wired only into a post-hoc profile-page nudge (`WalletSetupBanner`), never into Tier-1 itself. The source spec (a Claude-design prototype at `claude.ai/design/p/86efec5d.../Canvas.dc.html`, `onyx/onyx-wallet.jsx`) called for a hard, non-skippable wallet-creation overlay immediately after Tier-1 completion and before the success screen, with granular shard-level custody UX (Google Drive / WebAuthn passkey / iCloud Keychain for one share, password/biometric-encrypted for the other).

## What Changes

- Insert a wallet-creation step in `Tier1Page.tsx` between Tier-1 approval and the unlock screen. Tier-1's own KYC/identity backend contract is untouched — this is a purely client-side routing change plus the existing, separate MPC provisioning surface (`getMpcConfig`/`key1.generate`/`createWallet`).
- `WalletCreationOverlay`: granular phases (initializing → generating shards → recovery key / Shard 2 → custody / Shard 1 → finalizing), with a 2:00 countdown that switches to "taking longer than usual" messaging past that mark rather than failing or capping.
- Shard 1 (device share1, user custody): access-checked first (already-connected Drive, or an existing custody passkey, skips the picker). Options: Google Drive OAuth (`useGoogleDrive`, existing) or a Chrome passkey using the WebAuthn `largeBlob` extension (new).
- Shard 2 (device share2, encrypted restoration key): password (PBKDF2 → AES-256-GCM) or biometrics (WebAuthn `prf` → AES-256-GCM), replacing the manual copy-paste-a-raw-share UX that existed elsewhere in the codebase (`mpc-demo/WalletSetup.tsx`'s recovery-phrase flow).
- Shard 3 (node3 / platform party): unchanged — server-side, silent, no client action.
- Every failure re-presents the same step with Retry as the only way forward. No "continue without a wallet" escape on this path.
- `WalletResumePrompt`: a single resume overlay when a prior run started but never finished, detected via a local `inProgress` flag — restarts the whole ceremony rather than attempting a partial resume.
- Gate extension: an approved-but-walletless investor is now blocked from trading the same way an unapproved one is (`exchange.tsx`'s existing `useOnboardingStatus`-driven gate, extended with `useMpcWallet.needsSetup`). Landing back on `/onboarding/tier1` with Tier-1 already done skips straight to the wallet step instead of re-running KYC.
- `WalletSetupBanner` now opens this same overlay instead of the separate `WalletStep.tsx`, so there is one ceremony implementation instead of two.

## Impact

- **Tracking issue**: [LIQ2-86](https://linear.app/satschel/issue/LIQ2-86) (Web — MPC Wallet: SDK Integration, Wallet Ready State & Signing Confirmation UI), under [LIQ2-73](https://linear.app/satschel/issue/LIQ2-73).
- **New files**: `services/mpc-wallet/shard-crypto.ts` (password encryption), `services/mpc-wallet/webauthn-shard.ts` (passkey/biometric custody), `services/mpc-wallet/wallet-creation-store.ts` (resume-flag + encrypted-shard-2 storage, deliberately separate from the existing `mpc-store.ts` schema), `views/onboarding/tier1/steps/WalletCreationOverlay.tsx`, `views/onboarding/tier1/steps/WalletResumePrompt.tsx`.
- **Modified**: `Tier1Page.tsx` (new local `walletCreation`/`walletResume` steps, not part of the server-driven `Tier1Step` gate chain), `exchange.tsx` (trade-gate extension), `WalletSetupBanner.tsx` (points at the new overlay).
- **Deliberately not touched**: `WalletStep.tsx` itself (left in place, now unused, in case something else still references it), Tier-1's KYC/identity API surface, the MPC provisioning backend contract.
- **Deviation from the source spec, and why**: the spec asked for a literal `beforeunload`-guarded, background-disabled hard modal lock. Built instead as a route/gate-level lock reusing this app's existing verification-gate pattern (non-dismissible banner + inline trade block, both already used for unapproved Tier-1) — the user can navigate away in the ordinary browser sense; they just can't do anything else productive until the wallet exists. This was an explicit product decision (not a shortcut) — see the Linear comment on LIQ2-86 for the full reasoning.
- **Mobile / iCloud Keychain**: explicitly out of scope here — that path belongs to the native mobile app's own codebase, not this web repo.
- **Browser support**: Chrome only, per the spec. Passkey/biometric options self-hide via feature detection on unsupported browsers rather than presenting a button that will always fail.
