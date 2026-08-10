## Why

Linear issue [LIQ2-73](https://linear.app/satschel/issue/LIQ2-73) defines the MPC wallet's core key architecture (Key 1 shard on device, Key 2 super admin, Key 3 restoration, Key 4 cloud HSM) and the 2-of-4 multisig / on-chain contract layer, but three things are not yet covered: a recoverable backup path for the user's key shard, frontend design ownership for the signing UI on both web and mobile, and what happens downstream once the MPC wallet is live — order placement/cancellation on-chain and settlement for private assets and private secondaries.

## What Changes

- Add a frontend key-storage backup path for the user's MPC key shard: Google Drive (Android) or iCloud Keychain (iOS).
- Add a second key-generation factor derived from a user password, as an additional recovery/security factor alongside the existing shard architecture.
- Assign frontend design ownership explicitly: web signing UI to Marshal Tavakar, mobile signing UI to Hiren Dudhat.
- Add a blocking decision point: confirm with John Hanks which MPC provider/SDK to standardize on before downstream SDK integration work proceeds.
- Once the MPC wallet is complete, extend on-chain integration to order placement (via allowance) and order cancellation.
- Add order execution and settlement flows for private assets and, separately, for private secondaries, both built on top of the MPC wallet + multisig contract layer.
- Build a Liquidity Crypto Wallet with the same deposit and whitelisted-withdrawal process as Alpaca, and transfer pipelines for BTC, ETH, USDC, USDT, and SOL.

## Impact

- **Tracking issue**: [LIQ2-73](https://linear.app/satschel/issue/LIQ2-73) (Linear, team LiquidityOnyx | P2, project "Liquidity 2.0 Roadmap | Phase 2").
- **New sub-issues created**: LIQ2-153 (key storage backup), LIQ2-154 (password-derived second key), LIQ2-155 (web design, Marshal Tavakar), LIQ2-156 (mobile design, Hiren Dudhat), LIQ2-157 (MPC provider confirmation, John Hanks — blocking), LIQ2-158 (order execution/settlement — private assets), LIQ2-159 (order execution/settlement — private secondaries), LIQ2-160 (order placement/cancel on-chain), LIQ2-161 (Liquidity Crypto Wallet — deposit/whitelisted withdrawal/BTC-ETH-USDC-USDT-SOL transfers).
- **Sequencing**: LIQ2-158, LIQ2-159, LIQ2-160, and LIQ2-161 are blocked on LIQ2-73 (the core MPC wallet + multisig layer) completing first.
- **Open decision**: MPC provider/SDK choice (LIQ2-157) blocks downstream iOS/Android SDK integration work.
