# On-Chain Verified Findings
# Facts established by inspecting real transactions, not just docs.
# Verification tag legend: [OFFICIAL_DOCS] [ON_CHAIN_VERIFIED] [INFERRED]

## 1. pump.fun bonding-curve buy fee = 1.25% [ON_CHAIN_VERIFIED + OFFICIAL_DOCS]
Real pump.fun buy transaction (fee_payer `C7j1DPtX...`, tx fee 15,000 lamports):

| Destination | Lamports | % of buy | Meaning |
|-------------|----------|----------|---------|
| bonding curve | 195,555,555 | 100.000% | buy amount |
| creator fee acct | 586,667 | 0.300% | creator fee |
| protocol fee acct A | 928,889 | 0.475% | protocol fee (half) |
| protocol fee acct B | 928,889 | 0.475% | protocol fee (half) |

- Sum of fee transfers = 2,444,445 lamports = **0.012500 × buy amount** = 1.25%.
- Matches official pump.fun fee doc: creator 0.300% + protocol 0.950% = 1.25% total.
- How to reproduce: pull any successful pump.fun buy, list system `transfer`
  instructions sourced from the fee-payer, divide each by the largest (the buy).

### Accounting consequence
- True buy cost = buy_amount × 1.0125 (+ tx base/priority fee).
- Isolating only the bonding-curve transfer understates cost by 1.235% of true cost.
- Fix applied in `live_rpc_source.py`: `PUMP_BONDING_CURVE_FEE_RATE = 0.0125`,
  `economic_spend = round(curve_transfer × 1.0125)`. Verified to land within 1 lamport
  of the on-chain true cost (197,999,999 vs 198,000,000).

## 2. Buys route through aggregators as nested CPIs [ON_CHAIN_VERIFIED]
In a sampled buy, top-level instructions were: ComputeBudget, ComputeBudget,
`FLASHX8DrLbg...` (a router/aggregator), and a small system transfer (tip). The actual
pump.fun buy and all 10 of its transfers were INNER instructions under the router.
- Lesson: never assume pump.fun is a top-level instruction. Detect by program presence
  in `accountKeys`; always walk `meta.innerInstructions`.
- Common routers seen in the wild: FLASHX, Photon, BullX, Trojan, Jupiter, Pump UI.

## 3. Bonding-curve PDA derivation [ON_CHAIN_VERIFIED in code]
`bonding_curve = findProgramAddress(["bonding-curve", mint_bytes], PUMP_FUN_PROGRAM)`
where `PUMP_FUN_PROGRAM = 6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P`.
Its associated token account holds the curve's token supply.

## 4. SOL balance delta already nets fees [ON_CHAIN_VERIFIED]
The wallet's `postBalances - preBalances` already reflects buy + 1.25% fee + tips +
rent − received SOL. The only adjustment needed is to add the tx base/priority fee back
(it is reported separately in `meta.fee`) so it isn't double-counted, then re-subtract
it once at the accounting layer. Net result: one tx fee counted, pump fee included.
This is why SELLS need no uplift — they use the balance delta directly.

## 5. Real-trade vs noise rules [ON_CHAIN_VERIFIED in code + tests]
- Real BUY: tx success + token balance ↑ + SOL/WSOL ↓ + explicit swap context.
- Real SELL: tx success + token balance ↓ + SOL/WSOL ↑ + explicit swap context.
- Excluded: failed tx, missing meta, wrong mint, transfer-only (no economic SOL move),
  fee-only SOL delta, ATA rent refund (CloseAccount SOL increase).

## Open items requiring on-chain verification before use [INFERRED]
- Graduation threshold in SOL (commonly cited as ~85 SOL / ~$69k mcap) — NOT confirmed
  in official docs; must be measured from a migration transaction.
- PumpSwap post-graduation fee tier boundaries — confirm against real PumpSwap swaps.
- Pump fee account addresses — only 3 samples seen; treat as rotating, do not hardcode.
