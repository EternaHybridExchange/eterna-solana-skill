# Live USDC on Solana (re-read before every send)

Snapshot from `GET https://api.eterna.exchange/coins/USDC` on **2026-10-15**. These numbers move. Before a transfer or withdrawal, re-read them with MCP:

```typescript
const coins = await eterna.getAllowedDepositCoins("USDC", "SOL");
return coins;
```

Use the row whose chain is `SOL`. If deposit is disabled, or the amount is below `minDepositAmount`, stop.

Also call `eterna.getCoinInfo("USDC")` and use the `SOL` chain row for withdraw flags. Public snapshot of that chain:

| Field | Snapshot 2026-10-15 |
| --- | --- |
| Chain code for MCP | `SOL` |
| chainId | `101` |
| Deposit enabled | yes |
| Withdraw enabled | yes |
| Official mint | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| Decimals | 6 |
| Min deposit | `0.005` USDC |
| Min withdraw | `1` USDC |
| Withdraw fee | `0.3` USDC |
| Confirmations | `200` |
| Safe confirm number | `200` |

## Mint

Send only `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`. If the wallet's USDC account has a different mint, stop. Do not bridge or swap inside this skill.

## Memo

Bybit documents tags for coins such as XRP, EOS, and TON, not as a blanket Solana rule. Eterna's own deposit sender ignores memo on Solana (memo is required for TON only).

Still do not guess:

1. `const addr = await eterna.getDepositAddress("USDC", "SOL")`.
2. If `tagDeposit` is `null` or `""`, send a plain SPL transfer. Do not add a Memo program instruction.
3. If `tagDeposit` is anything else, **stop**. Do not send. Tell the user the venue returned a tag and this skill will not drop it.

## Credit time

We did not time a live deposit. The venue asks for **200 confirmations** (`safeConfirmNumber` 200) before it treats the deposit as safe. Do not `sleep` for a fixed number of seconds. Poll `eterna.getDepositRecords("USDC")` until the matching `txID` has `status === 3`.

Status: `0` unknown, `1` to confirm, `2` processing, `3` success, `4` failed. On `4`, stop.

Expect minutes, not one slot. Do not trade while status is 1 or 2.

## Why 1 USDC cannot round-trip

Withdrawal minimum is 1 USDC and the fee snapshot is 0.3 USDC. Perps use USDT, so the path also converts USDC to USDT and back, which is not 1:1. A 1 USDC demo can deposit and trade. It cannot reliably withdraw. For a round trip, start with **at least 5 USDC**, then re-check the live minimum and fee and only withdraw an amount that clears both.
