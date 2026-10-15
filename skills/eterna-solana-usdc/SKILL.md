---
name: eterna-solana-usdc
description: >
  Use when the user wants to deposit official Solana USDC into Eterna from a
  wallet they sign, or withdraw Eterna USDC back to a Solana address. Triggers:
  "deposit USDC on Solana to Eterna", "fund Eterna from Phantom", "withdraw
  Eterna USDC to Solana". Do not use for generic Solana swaps or on-chain DEX
  perps. For orders after the deposit credits, use the eterna-mcp skill.
license: MIT
metadata:
  author: EternaHybridExchange
  version: "1.1.0"
---

# Eterna Solana USDC

Move official Solana USDC between a user-signed wallet and Eterna. Trading after the credit is the `eterna-mcp` skill, not this one.

## Custody

The user signs the Solana transfer. After Eterna credits it, the balance is on a managed trading account. It is not in the Solana wallet anymore. Say that before the user signs. Withdrawal is a later step, with its own confirmation.

## When to use

- The wallet holds official Solana USDC, or the user will send it.
- The user wants that USDC on Eterna.
- The user wants USDC withdrawn to a Solana address.

## When not to use

- The user wants a Solana swap and did not name Eterna. Use a Jupiter skill. If they hold SOL, swap to the official USDC mint there first.
- The user wants an on-chain perp (Drift and similar). This skill does not open those positions.
- The user wants to place an Eterna order. Switch to `eterna-mcp` once deposit status is success.

## Never

- Ask for a seed phrase, private key, or keypair file.
- Put `ETERNA_MCP_TOKEN` or any bearer token in the chat or in a file you commit.
- Send any mint other than the official USDC mint in `references/live-limits.md`.
- Send on a chain other than Solana. Coin is `USDC`. Chain code is `SOL`.
- Add a memo when `tagDeposit` is empty. If `tagDeposit` is non-empty, stop and do not send.
- Pick an older deposit row because it is also on SOL. Match the signature.
- Withdraw to an address that starts with `0x`.
- Promise the same USDC amount back. There is a live minimum and a fee.

## Confirm before signing or withdrawing

Wait for an explicit yes:

1. **SPL transfer:** mint, destination, amount in USDC and in base units, chain Solana, and that the credited balance will sit on a managed Eterna account.
2. **Withdrawal:** coin `USDC`, chain `SOL`, destination, amount, live minimum, live fee. Do not submit if the amount is below the live minimum.

## Steps

| Step | Read |
| --- | --- |
| Mint, minimums, memo, polling | `references/live-limits.md` |
| Deposit address, transfer, poll | `references/deposit-solana.md` |
| USDC back to Solana | `references/withdraw-solana.md` |

MCP connection and orders: skill `eterna-mcp` in this repo.

## Demo size

At least 5 USDC if the user wants USDC back on Solana. Re-read `references/live-limits.md` before quoting a figure.
