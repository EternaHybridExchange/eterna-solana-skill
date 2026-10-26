---
name: eterna-solana-skill
description: >
  Use when the user wants an agent to fund Eterna with official Solana USDC
  from a wallet they sign, trade on ai.eterna.exchange through the Eterna MCP,
  and optionally withdraw USDC back to Solana. Triggers: "deposit USDC to
  Eterna", "fund Eterna from Phantom", "trade on Eterna", "Eterna MCP",
  "withdraw Eterna USDC to Solana". Do not use for generic Solana swaps or
  on-chain DEX perps without naming Eterna.
license: MIT
metadata:
  author: EternaHybridExchange
  version: "1.2.0"
---

# Eterna Solana skill

End-to-end agent flow on Eterna: Solana USDC in, MCP trading after credit, optional USDC back to Solana.

Product sites: [eterna.exchange](https://eterna.exchange) (human), [ai.eterna.exchange](https://ai.eterna.exchange) (agent access). MCP server: `https://mcp.eterna.exchange/mcp`.

## Custody

The user signs the Solana USDC transfer. After Eterna credits it, the balance is on a managed trading account, not in the Solana wallet. Say that before they sign. Orders spend that managed balance. Withdrawal to Solana is a separate confirmed step.

## When to use

- Send official Solana USDC into Eterna or withdraw USDC back to a Solana address.
- Trade on Eterna through MCP after the deposit credits (or when the account already has funds).

## When not to use

- Generic Solana swaps without Eterna. Use a Jupiter skill. If they hold SOL, swap to the official USDC mint first.
- On-chain perps (Drift and similar). This skill does not open those positions.

## Never

- Ask for a seed phrase, private key, keypair file, or pasted API secret.
- Put `ETERNA_MCP_TOKEN` or any bearer token in the chat or in a file you commit.
- Send any mint other than the official USDC mint in `references/live-limits.md`.
- Send on a chain other than Solana. Coin is `USDC`. Chain code is `SOL`.
- Add a memo when `tagDeposit` is empty. If `tagDeposit` is non-empty, stop and do not send.
- Pick an older deposit row because it is also on SOL. Match the signature.
- Withdraw to an address that starts with `0x`.
- Promise the same USDC amount back. There is a live minimum and a fee.
- Call `placeOrder` or `swapToUsdt` while the Solana deposit is still pending.
- Guess SDK method names. Call `search_sdk` first. If nothing matches, stop.
- Retry an infrastructure or connection error by rewriting the order.
- Set `positionIdx` yourself. Omit it.

## MCP connect

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

## Confirm before signing, trading, or withdrawing

Wait for an explicit yes:

1. **SPL transfer:** mint, destination, amount in USDC and in base units, chain Solana, and that the credited balance will sit on a managed Eterna account.
2. **Order:** symbol, side, quantity, and leverage. Default leverage is 1 unless the user named another number. Size must respect the instrument minimum from `getInstruments`.
3. **Withdrawal:** coin `USDC`, chain `SOL`, destination, amount, live minimum, live fee. Do not submit if the amount is below the live minimum.

## Steps

| Step | Read |
| --- | --- |
| Mint, minimums, memo, polling | `references/live-limits.md` |
| Deposit address, transfer, poll | `references/deposit-solana.md` |
| Funding wallet, margin, order | `references/trade.md` |
| USDC back to Solana | `references/withdraw-solana.md` |

## Demo size

At least 5 USDC if the user wants USDC back on Solana. Re-read `references/live-limits.md` before quoting a figure.
