# Deposit USDC on Solana

Read `live-limits.md` first. You need the MCP from the `eterna-mcp` skill only to read the deposit address and the deposit rows. The wallet still signs the transfer.

## 1. Address

```typescript
const addr = await eterna.getDepositAddress("USDC", "SOL");
return addr;
```

Read `addressDeposit` and `tagDeposit` from `addr.chains`. If `chains` is a list, use the entry for chain `SOL`.

Stop if `addressDeposit` is missing or starts with `0x`.

Memo rule from `live-limits.md`: empty tag means a plain SPL transfer. A non-empty tag means stop.

## 2. Transfer

The user signs. You never hold the key.

One SPL token transfer:

- Mint: the official USDC mint in `live-limits.md`
- Destination owner: `addressDeposit`
- If that owner has no USDC token account yet, the user can create the associated token account in the same transaction and pay the rent
- Amount: integer base units, 6 decimals. `1.5` USDC is `1500000`. Compute the integer from a decimal string. Do not use binary floating point
- No memo instruction when the tag is empty

Prefer `@solana/kit` or the user's wallet. Do not load a secret key. If you cannot build the transaction here, show mint, destination, amount, and chain, and let the user send from a wallet they already unlocked.

Show the confirmation from `SKILL.md` and wait.

After broadcast, give `https://solscan.io/tx/<signature>`.

## 3. Poll

```typescript
const records = await eterna.getDepositRecords("USDC");
const row = records.rows.find((r) => r.txID === signature);
return row
  ? { status: row.status, amount: row.amount, txID: row.txID, chain: row.chain }
  : null;
```

Match `txID` to the Solana signature only. A missing row means "not visible yet", not "pick another SOL deposit".

Repeat until `status` is `3` or `4`. Wait between polls, starting at a few seconds and capping around 30 seconds. Do not use one fixed sleep as proof of credit.

`3` is success. `4` is failed: stop.

The credit lands in the funding wallet. Do not place an order in this step. Hand off to `eterna-mcp`.
