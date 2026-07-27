---
name: Stake HYPE via Hyperbeat
description: Move HYPE from spot to staking balance and delegate it to the Hyperbeat validator on Hyperliquid, signing and broadcasting each transaction.
api: openapi/hyperbeat-staking-openapi.yml
operations: [hyperliquid-transfer, hyperliquid-delegate, hyperliquid-send]
---

# Stake HYPE via Hyperbeat

Institutional HYPE staking on Hyperliquid through Hyperbeat's validator (Staking API powered by P2P.org). Base URL `https://api.p2p.org` (mainnet) or `https://api-test.p2p.org` (testnet); the `{network}` path segment is `mainnet` or `testnet`.

## Auth
Send `Authorization: Bearer <JWT>`. Obtain a token per https://docs.p2p.org/docs/authentication. Missing token -> `NoTokenException` (101111); invalid -> `WrongTokenException` (101109).

## Steps
1. **Transfer to staking balance** (if needed) — `POST /api/v1/hyperliquid/{network}/staking/transfer` (`hyperliquid-transfer`) with `{ amount, delegatorAddress }`. Returns `unsignedTransaction`. Insufficient funds -> `NotEnoughSpotBalanceException` (129108). Transfers spot->staking are instant.
2. **Sign & send the transfer** — sign the `unsignedTransaction`, then `POST /api/v1/hyperliquid/{network}/transaction/send` (`hyperliquid-send`) with `{ signedTransaction }`. Invalid signature -> `InvalidSignedTransactionException` (129113).
3. **Delegate** — `POST /api/v1/hyperliquid/{network}/staking/delegate` (`hyperliquid-delegate`) with `{ amount, delegatorAddress }` (amount >= 0.1 HYPE). Returns an `unsignedTransaction`. Not enough staking balance -> `NotEnoughStakingBalanceException` (129109).
4. **Sign & send the delegation** — repeat step 2 with the delegate transaction.

## Rules
- Every write returns an `unsignedTransaction`; sign and broadcast it exactly once (build-sign-send; there is no API idempotency key). See conventions/hyperbeat-conventions.yml.
- Errors use the `{ error: { code, message, name }, result }` envelope. See errors/hyperbeat-problem-types.yml.
- Verify success with the staking-info skill (`hyperliquid-info`).
