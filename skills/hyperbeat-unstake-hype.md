---
name: Unstake HYPE via Hyperbeat
description: Undelegate HYPE from the Hyperbeat validator, wait out the Hyperliquid unstaking queue, then withdraw it back to spot balance.
api: openapi/hyperbeat-staking-openapi.yml
operations: [hyperliquid-undelegate, hyperliquid-withdraw, hyperliquid-send, hyperliquid-info]
---

# Unstake HYPE via Hyperbeat

Reverse the staking flow on Hyperliquid. Base URL `https://api.p2p.org` (mainnet) / `https://api-test.p2p.org` (testnet); `{network}` = `mainnet` | `testnet`. Send `Authorization: Bearer <JWT>`.

## Steps
1. **Check what is delegated** — `GET /api/v1/hyperliquid/{network}/staking/info` (`hyperliquid-info`) with `{ delegatorAddress }`. Read `delegations[]` and each delegation's `lockedUntil`.
2. **Undelegate** — `POST /api/v1/hyperliquid/{network}/staking/undelegate` (`hyperliquid-undelegate`) with `{ amount, delegatorAddress }`. Returns `unsignedTransaction`. If the delegation is still locked -> `DelegationLockedException` (129110); if amount exceeds delegated -> `NotEnoughDelegatedBalanceException` (129111).
3. **Sign & send** — sign, then `POST /api/v1/hyperliquid/{network}/transaction/send` (`hyperliquid-send`) with `{ signedTransaction }`.
4. **Wait the unstaking queue** — undelegated HYPE enters a ~7-day queue and shows as `pendingWithdrawal` in staking info until available.
5. **Withdraw to spot** — once available, `POST /api/v1/hyperliquid/{network}/staking/withdraw` (`hyperliquid-withdraw`) with `{ amount, delegatorAddress }`, then sign & send (step 3). Not enough staking balance -> `NotEnoughStakingBalanceException` (129109).

## Rules
- Respect `lockedUntil` before undelegating; the 7-day queue cannot be skipped.
- Build-sign-send, once each; no API idempotency key (conventions/hyperbeat-conventions.yml).
- Error envelope + codes: errors/hyperbeat-problem-types.yml.
