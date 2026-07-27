---
name: Read Hyperbeat staking position
description: Fetch a delegator's HYPE spot/stake balances, active delegations, and pending withdrawals from the Hyperbeat Staking API.
api: openapi/hyperbeat-staking-openapi.yml
operations: [hyperliquid-info]
---

# Read Hyperbeat staking position

Look up the full staking position for one Hyperliquid delegator. Base URL `https://api.p2p.org` (mainnet) / `https://api-test.p2p.org` (testnet); `{network}` = `mainnet` | `testnet`. Send `Authorization: Bearer <JWT>`.

## Step
- `GET /api/v1/hyperliquid/{network}/staking/info` (`hyperliquid-info`) with `{ delegatorAddress }`.

## Response fields
- `spotBalance` — HYPE available to transfer into staking balance.
- `stakeBalance` — HYPE available to delegate.
- `delegations[]` — each `{ amount, validator, lockedUntil }`.
- `pendingWithdrawal` — HYPE in the ~7-day unstaking queue, not yet withdrawable.

## Rules
- Read-only; safe to poll. Use it to gate the stake and unstake skills (check balances and `lockedUntil` before acting).
- Errors use the `{ error, result }` envelope: errors/hyperbeat-problem-types.yml.
