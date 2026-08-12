## 1. Crypto & custody primitives (done)

- [x] 1.1 Password-based shard encryption (PBKDF2 → AES-256-GCM) — `shard-crypto.ts`
- [x] 1.2 WebAuthn passkey custody for Shard 1 (`largeBlob` extension, feature-detected) — `webauthn-shard.ts`
- [x] 1.3 WebAuthn biometric key derivation for Shard 2 (`prf` extension, feature-detected) — `webauthn-shard.ts`
- [x] 1.4 Resume-flag + encrypted-shard-2 local storage, separate from `mpc-store.ts` — `wallet-creation-store.ts`

## 2. Overlay UI (done)

- [x] 2.1 `WalletCreationOverlay` — granular phase state machine, 2:00 countdown with overflow messaging
- [x] 2.2 Shard 1 panel — access-check first, then Drive/passkey picker
- [x] 2.3 Shard 2 panel — password or biometrics
- [x] 2.4 Retry-only failure handling on every phase, no skip
- [x] 2.5 `WalletResumePrompt` — single resume overlay

## 3. Wiring (done)

- [x] 3.1 `Tier1Page.tsx`: route through the overlay before `unlock`, local `walletCreation`/`walletResume` steps kept out of the server-driven gate chain
- [x] 3.2 `Tier1Page.tsx`: mount-time jump straight to the wallet step for an already-Tier1-approved return visit (skips KYC re-run)
- [x] 3.3 `WalletSetupBanner.tsx`: points at the new overlay instead of `WalletStep.tsx`
- [x] 3.4 `exchange.tsx`: trade-gate extended with `useMpcWallet.needsSetup`

## 4. Verification

- [x] 4.1 Project-wide `tsc --noEmit` clean
- [ ] 4.2 Manual browser walkthrough: happy path (password + Drive), happy path (biometrics + passkey), a forced failure + retry on each phase, resume prompt after an abandoned run
- [ ] 4.3 Cross-check on a non-Chrome browser that passkey/biometric options correctly stay hidden
- [ ] 4.4 Confirm with backend/Hiren that `GET /v1/bd/wallet` polling and `POST /v1/bd/wallet/create` behave as expected under this new call pattern (unchanged contract, but not yet exercised end-to-end against a live ceremony from this new entry point)
