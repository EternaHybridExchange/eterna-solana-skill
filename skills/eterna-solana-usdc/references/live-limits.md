# Live USDC on Solana

Re-read these figures before every send. Snapshot from `GET https://api.eterna.exchange/coins/USDC` on **2026-10-15**:

```typescript
const coins = await eterna.getAllowedDepositCoins("USDC", "SOL");
return coins;
```

Use the `SOL` row. If deposits are disabled, or the amount is below `minDepositAmount`, stop.

For withdrawal minimum and fee, call `eterna.getCoinInfo("USDC")` and read the `SOL` chain. If those fields are absent, do not guess. A withdrawal that the server rejects for size or fee is a stop, not a retry with a larger silent amount.

| Field | Snapshot 2026-10-15 |
| --- | --- |
| Chain code | `SOL` |
| chainId | `101` |
| Deposit | enabled |
| Withdraw | enabled |
| Official mint | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| Decimals | 6 |
| Min deposit | `0.005` USDC |
| Min withdraw | `1` USDC |
| Withdraw fee | `0.3` USDC |
| Confirmations before the credit is treated as safe | `200` |

## Mint

Send only `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`. Any other mint: stop. This skill does not swap or bridge.

## Memo

1. Read `tagDeposit` from `getDepositAddress("USDC", "SOL")`.
2. `null` or `""`: plain SPL transfer, no memo instruction.
3. Anything else: stop. Do not send without the tag, and do not invent a memo layout.

## Credit time

This snapshot was not timed with a live transfer. The published confirmations figure is 200. Poll `getDepositRecords` until the signature has status `3`. Status `0` unknown, `1` confirming, `2` processing, `3` success, `4` failed.

Expect minutes. Do not trade on status `1` or `2`.

## Round trip

Minimum withdrawal is 1 USDC and the fee snapshot is 0.3 USDC. Perps use USDT, so a round trip also converts. 1 USDC can fund a small trade. It does not reliably come back. For a return to Solana, start with at least 5 USDC, then re-check the live minimum and fee.
