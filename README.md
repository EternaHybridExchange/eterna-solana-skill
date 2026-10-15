# Eterna Solana USDC skill

Open agent skill: send **USDC on Solana** from a wallet you control, trade through the [Eterna MCP](https://mcp.eterna.exchange/mcp), and withdraw USDC back to the same Solana address.

This is a hybrid venue. The Solana transfer is signed by the user's wallet. After it confirms, the balance sits on a **managed trading account**, not in that wallet. It is not a DEX and it is not self-custody after the send.

## Install

```bash
npx skills add https://github.com/EternaHybridExchange/eterna-solana-skill
```

The skill directory is `skills/eterna-solana-usdc/`.

## What the agent is allowed to do

1. Read a deposit address for `USDC` on chain `SOL`.
2. Ask the user to sign an SPL transfer of the official USDC mint. No seed, no private key, no MCP token in the chat.
3. Wait until the deposit status is success, move funds to the trading wallet, convert to USDT (perps are USDT-margined), then place one order the user confirmed.
4. Withdraw USDC on `SOL` back to the same address, after a second confirmation, if the live minimum and fee allow it.

Generic Solana swaps belong to a Jupiter (or other DEX) skill. This skill starts once the wallet already holds official USDC.

## Humans

Create agent access at [ai.eterna.exchange](https://ai.eterna.exchange). Put the bearer token in `ETERNA_MCP_TOKEN`. Never commit it.

Auth details: [eterna-mcp authentication](https://github.com/EternaHybridExchange/eterna-mcp/blob/main/docs/authentication.md).

## Limits

Live numbers change. The snapshot and the rule "re-read before sending" are in `skills/eterna-solana-usdc/references/live-limits.md`. A 1 USDC round trip does not clear the withdrawal minimum and fee.

## License

MIT. See [LICENSE](LICENSE).
