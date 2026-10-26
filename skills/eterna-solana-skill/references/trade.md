# Trade after USDC has credited

Only after deposit status `3` for the Solana transfer, or the user already has a trading balance.

Call `search_sdk` before `execute_code`. If the live signature differs from this note, follow the search result.

## 1. Funding wallet to trading wallet

```typescript
const moved = await eterna.transferToTrading("USDC", amount);
return moved;
```

`amount` is the decimal string that just credited.

## 2. USDT margin

Perps use USDT margin. USDC left in the trading wallet does not open them.

```typescript
const swapped = await eterna.swapToUsdt("USDC");
return swapped;
```

Omitting the amount sells the available USDC. If `success` is false, stop. Do not place a perp.

## 3. One order

```typescript
const info = await eterna.getInstruments("BTCUSDT");
return info;
```

Use `minOrderQty` and `qtyStep` from that result. Show the order and wait for yes.

```typescript
const order = await eterna.placeOrder({
  symbol: "BTCUSDT",
  side: "Buy",
  orderType: "Market",
  qty: qty,
  leverage: 1,
});
return order;
```

Do not pass `market: "spot"` on this perp order.

To flatten, read positions first. `closePosition` throws when nothing is open.

```typescript
const positions = await eterna.getPositions("BTCUSDT");
return positions;
```

```typescript
const closed = await eterna.closePosition("BTCUSDT");
return closed;
```

## 4. Errors

Report infrastructure and connection errors as-is. Do not invent a second order to "fix" them. If `search_sdk` is empty, stop.

To send USDC back to Solana, read `references/withdraw-solana.md`.
