# Withdraw USDC back to Solana

Read `live-limits.md` again. Use the live minimum and fee, not the snapshot, if they differ.

## 1. USDT back to USDC

Perp PnL and margin are USDT. `swapToUsdt` only sells into USDT. To leave as USDC, buy USDC on the spot market after the position is closed.

Search the SDK, then a spot buy (shape from `placeOrder`):

```typescript
const order = await eterna.placeOrder({
  symbol: "USDCUSDT",
  side: "Buy",
  orderType: "Market",
  qty: usdtToSpend,
  market: "spot",
  marketUnit: "quoteCoin",
});
return order;
```

`qty` here is USDT to spend. Confirm the symbol exists via `getInstruments("USDCUSDT", "spot")` or `search_sdk`. If USDC spot is not listed, stop. Do not withdraw USDT to a Solana USDC mint address.

## 2. Check size

```typescript
const w = await eterna.getWithdrawableAmount("USDC");
return w;
```

Need `withdrawableAmount` greater than or equal to the live minimum (snapshot: 1 USDC) and enough left that the user still understands the fee (snapshot: 0.3 USDC). If not, say so and do not submit.

## 3. Submit

Destination is the same Solana address that signed the deposit, unless the user typed a different base58 address and confirmed it. Reject `0x` addresses.

```typescript
const req = await eterna.submitWithdrawal("USDC", amount, solanaAddress, "SOL");
return req; // withdrawalRequestId, status
```

Chain argument is `SOL`, the same code as the deposit.

## 4. Status

```typescript
const status = await eterna.getWithdrawalStatus(withdrawalRequestId);
return status;
```

`COMPLETED` is done. `FAILED`, `REVERTED`, or `REVERT_FAILED`: show `errorMessage` and stop. Do not submit a second withdrawal for the same funds until the first request is terminal.

Withdrawal is asynchronous. Do not tell the user the USDC is in Phantom until status is `COMPLETED` or the signature is on Solscan.
