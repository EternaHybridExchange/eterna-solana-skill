# Trade after the Solana credit

Only after deposit `status` is `3`.

Call `search_sdk` for the method names below before `execute_code`. Copy signatures from the search result if they differ from this note.

## 1. Funding to trading

```typescript
const moved = await eterna.transferToTrading("USDC", amount);
return moved; // { transferId }
```

`amount` is a decimal string of the USDC that just credited, not a guess.

## 2. USDT margin

Linear perps on this MCP are USDT. USDC in the trading wallet does not margin them.

```typescript
const swapped = await eterna.swapToUsdt("USDC");
return swapped;
```

Omitting the amount sells the available USDC balance. Tell the user the result (`qty`, `success`). If `success` is false, stop. Do not place a perp.

## 3. One order

Read the instrument first:

```typescript
const info = await eterna.getInstruments("BTCUSDT");
return info; // min order qty, qty step, tick size, max leverage
```

Pick a size at or above the minimum, at low leverage (1x unless the user named a number). Show symbol, side, qty, leverage, and wait for yes.

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

Do not pass spot fields (`market: "spot"`, `marketUnit`) on a perp. Do not invent `positionIdx`. Omit it so the server resolves one-way vs hedge.

To flatten:

```typescript
const closed = await eterna.closePosition("BTCUSDT");
return closed;
```

`closePosition` throws if nothing is open. Read `eterna.getPositions("BTCUSDT")` first.

## 4. Errors

If the tool returns an infrastructure or connection error, tell the user. Do not rewrite the order and retry. If `search_sdk` returns no methods, stop. Do not guess method names.
