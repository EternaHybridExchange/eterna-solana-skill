---
name: eterna-solana-usdc
description: >
  Use when the user wants an agent to deposit USDC on Solana and trade through
  Eterna MCP (ai.eterna.exchange), or withdraw that USDC back to the same
  Solana address. Triggers: "deposit USDC on Solana to Eterna", "fund Eterna
  from Phantom", "trade on Eterna with Solana USDC", "withdraw Eterna USDC to
  Solana". Do not use for generic Solana swaps, Jupiter routes, or DEX perps
  (Drift, Raydium, Orca). Do not use when the user only wants to build a
  Solana program.
license: MIT
metadata:
  author: EternaHybridExchange
  version: "1.0.0"
---

# Eterna: Solana USDC in, MCP trade, USDC out

## Custody (read this first)

Eterna is a hybrid exchange. The user signs a Solana SPL transfer from their own wallet. After the venue credits it, the balance is on a **managed trading account** (a custodial sub-account). It is not a DEX. The agent cannot move those funds except through the MCP methods below, and only after the user confirms each spend and each withdrawal.

Do not describe this as self-custody, on-chain perps, or a decentralized exchange.

## When to use

- The wallet already holds official Solana USDC, or the user is ready to send it.
- The user wants that USDC on Eterna and then a trade through MCP.
- The user wants USDC withdrawn back to a Solana address.

## When not to use

- The user said buy, sell, or swap on Solana and did not name Eterna. Use a Jupiter (or other DEX) skill. If they hold SOL and need USDC first, that swap happens there, into the official mint, and only then this skill starts.
- The user wants Drift, Raydium, Orca, or any fully on-chain perp.
- The task is "write a Solana program" or "set up Anchor".

## Never

- Ask for a seed phrase, private key, or keypair file. The wallet signs. You prepare the transfer or you show the address and amount for the user to send.
- Paste `ETERNA_MCP_TOKEN` or any bearer token into the chat, the skill, or a commit.
- Send any mint other than official USDC. The mint check is in `references/live-limits.md`.
- Send on Arbitrum, Ethereum, or any chain other than Solana for this skill. Ignore older MCP examples that default to USDT on Arbitrum.
- Add a Solana memo when `tagDeposit` is empty. If `tagDeposit` is non-empty, **stop and do not send**. This product path does not attach a memo on Solana. A required tag that you drop will not be credited.
- Place an order or call `swapToUsdt` before deposit `status` is `3`.
- Withdraw to an address that starts with `0x`. Solana addresses are base58.
- Promise that the same number of USDC comes back. There is a withdrawal fee, a minimum, and a USDC to USDT conversion for perps.

## Two confirmations

Show this and wait for an explicit yes before the matching action:

1. **Before the SPL transfer:** mint, destination address, amount (USDC and base units), chain `Solana`, and the sentence "after this send, funds are on a managed Eterna account."
2. **Before `placeOrder`:** symbol, side, qty, leverage, and that margin is USDT.
3. **Before `submitWithdrawal`:** coin `USDC`, chain `SOL`, destination, amount, and the live fee and minimum. Skip the withdrawal if the amount is below the live minimum.

## Golden path

Read a reference only when that step starts.

| Step | Read |
| --- | --- |
| Limits, mint, memo rule, why 1 USDC cannot round-trip | `references/live-limits.md` |
| Deposit address, SPL transfer, poll until credited | `references/deposit-solana.md` |
| Funding to trading, USDC to USDT, one confirmed order, close | `references/mcp-trade.md` |
| USDC back to the same Solana address | `references/withdraw-solana.md` |

## Connect MCP

Human creates access at https://ai.eterna.exchange. Token stays in the environment as `ETERNA_MCP_TOKEN`.

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

Auth reference: https://github.com/EternaHybridExchange/eterna-mcp/blob/main/docs/authentication.md

Inside MCP, call `search_sdk` before writing code, then `execute_code` with the `eterna.*` SDK. On an infrastructure error, report it. Do not invent a retry loop.

## Demo size

Use at least 5 USDC if the user wants funds back on Solana. Live withdrawal minimum is higher than a 1 USDC send once the fee is included. Re-read limits before quoting a number. Details: `references/live-limits.md`.
