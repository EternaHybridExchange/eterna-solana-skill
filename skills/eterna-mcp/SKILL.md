---
name: eterna-mcp
description: >
  Use when the user wants an agent to trade on Eterna through the MCP at
  ai.eterna.exchange after Solana USDC has credited: move funds to the trading
  wallet, convert USDC to USDT margin, place one confirmed order, and close it.
  Triggers: "trade on Eterna", "Eterna MCP", "ai.eterna.exchange". Do not use
  for the Solana USDC transfer itself (use eterna-solana-usdc) or for generic
  Solana swaps and on-chain DEX perps.
license: MIT
metadata:
  author: EternaHybridExchange
  version: "1.1.0"
---

# Eterna MCP

Trade on Eterna from an agent. The server is `https://mcp.eterna.exchange/mcp`. People open access at https://ai.eterna.exchange. The exchange site is https://eterna.exchange.

The Solana USDC send and the withdrawal back to Solana are the `eterna-solana-usdc` skill. Start here only after that deposit status is success, or when the trading account already has funds.

## Custody

Orders spend a managed Eterna balance. They do not spend the Solana wallet directly. Confirm the order with the user before `placeOrder`.

## Never

- Ask for a seed, a private key, or a pasted API secret.
- Put the bearer token in the chat or in git. It belongs in `ETERNA_MCP_TOKEN`.
- Call `placeOrder` or `swapToUsdt` while the Solana deposit is still pending.
- Guess SDK method names. Call `search_sdk` first. If nothing matches, stop.
- Retry an infrastructure or connection error by rewriting the order.
- Set `positionIdx` yourself. Omit it.

## Connect

```json
{
  "mcpServers": {
    "eterna": {
      "type": "streamable-http",
      "url": "https://mcp.eterna.exchange/mcp",
      "headers": {
        "Authorization": "Bearer ${ETERNA_MCP_TOKEN}"
      }
    }
  }
}
```

Auth: https://github.com/EternaHybridExchange/eterna-mcp/blob/main/docs/authentication.md

Inside the server, use `search_sdk`, then `execute_code` with the `eterna.*` SDK.

## Confirm before the order

Show symbol, side, quantity, and leverage. Wait for an explicit yes. Default leverage is 1 unless the user named another number. Size must respect the instrument minimum from `getInstruments`.

## Steps

Read `references/trade.md` when the deposit has credited and the user wants an order.

To fund from Solana or to withdraw, switch to `eterna-solana-usdc`.
