# Solana + pump.fun Wallet Labels — Working Knowledge

This repo is the master reference for labeling Solana / pump.fun wallets and for
building a **copy-trade simulator** that scores memecoin traders. A fresh Claude
session should read this file first, then the linked docs. Every fact here is tagged
with how it was verified: `[OFFICIAL_DOCS]`, `[ON_CHAIN_VERIFIED]`, or `[INFERRED]`.

## What we are building
A simulator that, given a pump.fun contract address (CA), finds the top traders by
ROI and reports the **net profit of a $1 copy trade** with 10% buy slippage / 20% sell
slippage. It must handle multiple buys and partial sells per coin. Memecoin traders
target ~2x, so break-even ROI for a copier is not the headline metric — realized
copy profit is. Output is a CSV on the user's Desktop, NOT Discord.

**Hard rule from the user: everything must be verified on-chain or in official docs.
No assumptions, no guessed numbers, no paid APIs.**

## Files in this repo
- `CLAUDE.md` — this file (read first)
- `wallet_label_taxonomy.md` — every label, how to detect it, verification status
- `onchain_verified_findings.md` — facts proven against real Polygon/Solana tx data
- `solana_program_addresses.md` — native + pump.fun program IDs `[OFFICIAL_DOCS]`
- `solana_account_types.md` — account structs and fields `[OFFICIAL_DOCS]`
- `pumpfun_mechanics.md` — fees, graduation, bonding curve `[OFFICIAL_DOCS]`
- `solana_fees.md` — base/priority fee, rent `[OFFICIAL_DOCS]`
- `onchain_audit_checklist.md` — what must be checked before trusting any PnL number
- `labels.csv` / `pumpfun_labels.csv` — the actual wallet label data
- `categories.md` — CSV schema and source enums

## The single most important accounting fact `[ON_CHAIN_VERIFIED]`
A pump.fun **buy** debits the wallet in SEPARATE on-chain transfers:
1. wallet → bonding curve = the buy amount
2. wallet → creator fee account = **0.300%** of the buy
3. wallet → protocol fee account(s) = **0.950%** of the buy (often split in two)

Total fee = **1.25%** of the buy amount — confirmed both in pump.fun's official fee
docs AND by measuring real transactions (fee transfers = exactly 0.0125 × buy amount).

Implication for cost basis: if you isolate only the wallet→bonding-curve transfer you
UNDERSTATE buy cost by 1.25% and OVERSTATE ROI. The simulator's RPC source
(`live_rpc_source.py::_pump_buy_sol_spend_lamports`) does isolate that transfer, so the
isolated amount is uplifted by `PUMP_BONDING_CURVE_FEE_RATE = 0.0125`. Sells already
net the fee because they are measured from the wallet's SOL balance delta.

## On-chain realities that break naive parsing `[ON_CHAIN_VERIFIED]`
- Most buys route through **third-party aggregators** (e.g. `FLASHX...`, Photon, BullX,
  Trojan, Jupiter). The pump.fun `buy` is a **nested CPI**, NOT a top-level instruction.
  Any logic that assumes pump is a top-level instruction will miss most trades. Detect
  pump involvement by presence of the program in `accountKeys`, then read inner
  instructions.
- Transaction base fee is 5,000 lamports per signature; priority fee is separate and
  read from `meta.fee`. The fee-payer is `accountKeys[0]`.
- SOL balance deltas come from `meta.preBalances` / `meta.postBalances` by account index.
- Token deltas come from `meta.preTokenBalances` / `meta.postTokenBalances` by owner.
- ATA creation shows as a `createAccount`/`create` instruction (rent ~0.002 SOL), NOT a
  `transfer`, so summing `transfer` instructions naturally excludes rent.

## Codebase pointers (the live system, separate repo at /home/travi/ai_system)
- Discovery package: `core/trading_system/ca_wallet_discovery/`
  - `live_rpc_source.py` — fetches + normalizes on-chain txs (Helius RPC)
  - `classifier.py` / `trade_event_source.py` — real-buy / real-sell rules
  - `accounting.py` — FIFO cost basis, realized PnL, ROI (fee-inclusive)
  - `ca_scanner.py` — ranks wallets by ROI, computes `copy_profit_usd`
  - `discord_output.py` — formatter
- Discord trigger channel: `#meme-copy`. On a CA drop it scans and writes a CSV to
  `/mnt/c/Users/travi/Desktop/ca_scan_<ca8>_<timestamp>.csv`.
- Copy-profit formula (per wallet): `((1 + roi) × (1 − sell_slip) / (1 + buy_slip)) − 1`
  × $copy_size, with buy_slip=0.10, sell_slip=0.20, copy_size=$1.

## Known gaps / next work
- Post-graduation PumpSwap fees are market-cap-tiered (0.30%–1.25%), NOT flat 1.25%.
  The flat rate only applies pre-graduation (bonding curve). PumpSwap cost handling
  still needs tier lookup. See `pumpfun_mechanics.md`.
- Pagination cap means most live scans still return `PARTIAL_CA_TRADE_HISTORY`.
- Graduation threshold (SOL raised that triggers migration) is NOT in official docs —
  marked `[INFERRED]` / NEEDS_ONCHAIN_VERIFICATION, do not treat as fact.
