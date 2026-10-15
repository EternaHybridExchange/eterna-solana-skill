# Deposit USDC on Solana

Read `live-limits.md` first.

## 1. Address

```typescript
const addr = await eterna.getDepositAddress("USDC", "SOL");
return addr; // chains.addressDeposit, chains.tagDeposit, chains.chainType
```

`chainType` must be the Solana chain. If it is not `SOL`, stop.

Apply the memo rule in `live-limits.md` before building a transaction.

## 2. Transfer

The user signs. You never hold the key.

Build a single SPL token transfer:

- Mint: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`
- Destination: `addressDeposit` (the token account owner is that address; create the associated token account if the venue address has none)
- Amount: integer base units. 6 decimals. `1.5` USDC is `1500000`. Do not use floating point to compute the integer.
- No memo instruction when `tagDeposit` is empty.

Prefer `@solana/kit` (or the user's wallet adapter) over legacy `@solana/web3.js` examples that load a secret key. If you cannot build the transaction in this session, show mint, destination, amount, and chain, and let the user send from Phantom or another wallet they already unlocked. Do not ask them to export a key so you can send it.

Show the confirmation block from `SKILL.md` and wait.

After broadcast, give `https://solscan.io/tx/<signature>`.

## 3. Poll

```typescript
const records = await eterna.getDepositRecords("USDC");
const row = records.rows.find((r) => r.txID === signature || r.chain === "SOL");
return row ? { status: row.status, amount: row.amount, txID: row.txID } : null;
```

Match on `txID` when you have the signature. Do not treat an older SOL row as this deposit.

Loop until `status === 3` or `status === 4`. Between polls, wait with backoff (start at a few seconds, cap around 30 seconds). A fixed short sleep is not a credit.

Deposits land in the **funding** wallet, not the trading wallet. Do not call `placeOrder` in this step.
