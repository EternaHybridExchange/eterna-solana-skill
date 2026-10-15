# Eterna skills for Solana agents

Two open skills an agent can install and follow. They are community skills in the same sense as the list on [solana.com/skills](https://solana.com/skills): useful to drop into an agent, not a Solana Foundation endorsement.

| Skill | What it is for | Solana piece |
| --- | --- | --- |
| [eterna-solana-usdc](skills/eterna-solana-usdc/SKILL.md) | Send official USDC from a Solana wallet into Eterna, and withdraw USDC back to that wallet | SPL transfer, official mint, Solscan |
| [eterna-mcp](skills/eterna-mcp/SKILL.md) | Trade on [ai.eterna.exchange](https://ai.eterna.exchange) through the Eterna MCP after the USDC has credited | Uses the balance that arrived from Solana |

The human product is [eterna.exchange](https://eterna.exchange). The agent product is [ai.eterna.exchange](https://ai.eterna.exchange). One skill does not click the website. The Solana skill moves USDC. The MCP skill places the trade.

## Install

Both skills:

```bash
npx skills add https://github.com/EternaHybridExchange/eterna-solana-skill
```

One skill:

```bash
npx skills add https://github.com/EternaHybridExchange/eterna-solana-skill --skill eterna-solana-usdc
npx skills add https://github.com/EternaHybridExchange/eterna-solana-skill --skill eterna-mcp
```

Works with agents that read `SKILL.md` (Cursor, Claude Code, Codex, and others supported by the skills CLI).

## What the user signs

The Solana wallet signs the USDC transfer. The agent never asks for a seed or a private key. The MCP token stays in `ETERNA_MCP_TOKEN` and never goes in chat or git.

After the transfer is credited, the balance is on a managed Eterna trading account. From that moment it is not sitting in the Solana wallet. Withdrawal back to Solana is a separate confirmed step.

## Limits

Numbers move. The snapshot and the rule to re-read them live are in [live-limits.md](skills/eterna-solana-usdc/references/live-limits.md). A 1 USDC send can fund a small trade. A round trip back to Solana needs more than the withdrawal minimum and fee. Start from 5 USDC if the USDC should return.

## License

MIT. See [LICENSE](LICENSE).
