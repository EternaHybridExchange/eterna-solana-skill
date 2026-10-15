# Withdraw USDC to Solana

Read `live-limits.md` again. Live minimum and fee win over the snapshot.

## 1. Hold USDC, not only USDT

Perp margin is USDT. `swapToUsdt` sells into USDT. After the position is closed, buy USDC back only if the SDK lists the spot pair.

```typescript
const info = await eterna.getInstruments("USDCUSDT", "spot");
return info;
```

If that list is empty, stop. Do not withdraw a different coin to a Solana USDC address.

When the pair exists, search the SDK and place a spot buy. `qty` is USDT to spend when `marketUnit` is `quoteCoin`:

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

Confirm that order with the user before sending it.

## 2. Size

```typescript
const w = await eterna.getWithdrawableAmount("USDC");
return w;
```

The amount must be at least the live minimum (snapshot: 1 USDC). Tell the user the live fee (snapshot: 0.3 USDC). If the balance is short, stop.

## 3. Submit

Destination is the Solana address that funded the deposit, unless the user typed a different base58 address and confirmed it. Reject `0x`.

```typescript
const req = await eterna.submitWithdrawal("USDC", amount, solanaAddress, "SOL");
return req;
```

Chain is `SOL`.

## 4. Status

```typescript
const status = await eterna.getWithdrawalStatus(withdrawalRequestId);
return status;
```

Tell the user the USDC is back only when status is `COMPLETED`. For any other status, show `status` and `errorMessage`. Do not submit a second withdrawal for the same funds while the first request is still open.
