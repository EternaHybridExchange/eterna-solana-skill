# Eterna Solana skill

One open agent skill: fund an Eterna trading account with official Solana USDC, trade through the Eterna MCP at [ai.eterna.exchange](https://ai.eterna.exchange), and withdraw USDC back to Solana when needed.

Community skill in the same sense as the list on [solana.com/skills](https://solana.com/skills): useful to drop into an agent, not a Solana Foundation endorsement.

The human product is [eterna.exchange](https://eterna.exchange). The agent entry point is [ai.eterna.exchange](https://ai.eterna.exchange).

## Install

```bash
npx skills add https://github.com/EternaHybridExchange/eterna-solana-skill
```

Works with agents that read `SKILL.md` (Cursor, Claude Code, Codex, and others supported by the skills CLI).

Skill path: [skills/eterna-solana-skill/SKILL.md](skills/eterna-solana-skill/SKILL.md).

## What the user signs

The Solana wallet signs the USDC transfer. The agent never asks for a seed or a private key. The MCP token stays in `ETERNA_MCP_TOKEN` and never goes in chat or git.

After the transfer is credited, the balance is on a managed Eterna trading account. From that moment it is not sitting in the Solana wallet. Withdrawal back to Solana is a separate confirmed step.

## Limits

Numbers move. The snapshot and the rule to re-read them live are in [live-limits.md](skills/eterna-solana-skill/references/live-limits.md). A 1 USDC send can fund a small trade. A round trip back to Solana needs more than the withdrawal minimum and fee. Start from 5 USDC if the USDC should return.

## License

MIT. See [LICENSE](LICENSE).
