# Wallet Label Taxonomy
# Every label: what it means, HOW TO DETECT it on-chain, and verification provenance.
# Verification tags: [OFFICIAL_DOCS] [ON_CHAIN_VERIFIED] [INFERRED]
# A label is only trustworthy at the confidence of its weakest detection signal.

## How to use this file
Each label below has: Definition / Detection (the on-chain or doc-based test) /
Verification (how we know the detection is correct). When you add a wallet to
`labels.csv` or `pumpfun_labels.csv`, record the `confidence` per the scale at the
bottom. Never label from price/UI screenshots — only from chain data or official docs.

---

## A. Account-role labels (structural, from chain) [ON_CHAIN_VERIFIED + OFFICIAL_DOCS]
These come directly from an account's `owner` program and `executable` flag.

### system_account
- Definition: a normal wallet that holds SOL.
- Detection: `owner == 11111111111111111111111111111111` and `executable == false`.
- Verification: [OFFICIAL_DOCS] solana.com/docs/core/accounts.

### token_account
- Definition: holds an SPL token balance for one mint.
- Detection: `owner == TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`; struct has
  `mint`, `owner`, `amount`.
- Verification: [OFFICIAL_DOCS] solana-program.com/docs/token.

### mint_account
- Definition: the token itself (supply, decimals, authorities).
- Detection: owned by Token Program; struct has `supply`, `decimals`, `mint_authority`,
  `freeze_authority`.
- Verification: [OFFICIAL_DOCS].

### ata (associated token account)
- Definition: the canonical token account for (wallet, mint).
- Detection: PDA of seeds [wallet, TokenProgramID, mint] under
  `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL`.
- Verification: [OFFICIAL_DOCS] solana-program.com/docs/associated-token-account.

### pda
- Definition: program-controlled address with no private key.
- Detection: address is off the ed25519 curve / derived via findProgramAddress.
- Verification: [OFFICIAL_DOCS].

---

## B. pump.fun on-chain role labels [ON_CHAIN_VERIFIED]

### pump_deployer / dev_wallet
- Definition: wallet that created the token.
- Detection: signer of the pump.fun create instruction for the mint (first authority);
  also the recipient of the 0.300% creator fee on every buy.
- Verification: [ON_CHAIN_VERIFIED] creator fee transfer observed at 0.300% of buy.

### pump_bonding_curve
- Definition: PDA that holds the token's bonding-curve reserves pre-graduation.
- Detection: `findProgramAddress(["bonding-curve", mint], 6EF8...F6P)`.
- Verification: [ON_CHAIN_VERIFIED] derivation matches the buy-amount transfer target.

### creator_fee_account / protocol_fee_account
- Definition: destinations of the 0.30% / 0.95% pump fees.
- Detection: transfer targets that receive exactly 0.300% and 0.475%×2 of the buy.
- Verification: [ON_CHAIN_VERIFIED] (3 samples). Treat addresses as rotating — identify
  by proportion, not by hardcoded pubkey.

### pumpswap_pool
- Definition: AMM pool after graduation.
- Detection: account owned by `pAMMBay6oceH9fJKBRHGP5D4bD4sWpmSwMn52FMfXEA`.
- Verification: [OFFICIAL_DOCS] + program ID in code.

---

## C. Trading-behavior labels (from trade history) [INFERRED unless noted]
These require reconstructing a wallet's buys/sells. Inference confidence rises with
the number of coins observed. A label here is a hypothesis, not a structural fact.

### smart_money
- Definition: consistently profitable across many coins.
- Detection: realized ROI > 0 on a majority of ≥N closed positions across ≥M distinct
  mints, ranked by ROI (not total PnL). Use FIFO, fee-inclusive cost basis.
- Verification: [INFERRED] — only as good as trade-history completeness. Flag
  `PARTIAL_CA_TRADE_HISTORY` results as low confidence.

### early_buyer
- Definition: bought within the first N transactions of a launch.
- Detection: buy appears among the first N pump buys by block time / slot for the mint.

### sniper
- Definition: bought at or near bonding-curve open (slot 0 / block 0).
- Detection: buy in the same or adjacent slot as the create instruction.
- Verification: [ON_CHAIN_VERIFIED] possible via slot comparison.

### bundle_wallet
- Definition: part of a coordinated multi-wallet launch buy.
- Detection: multiple wallets buying the same mint in the same block/slot with similar
  sizes, often funded from a common source. See pumpfun bundle knowledge.

### dev_side_wallet
- Definition: wallet linked to a known dev that appears across multiple of their launches.
- Detection: recurring co-occurrence with a `pump_deployer` across launches; shared
  funding source.
- Verification: [INFERRED] — corroborate with funding-trail analysis.

### bot
- Definition: automated trader; fixed sizes, fixed timing, repeats across launches.
- Detection: near-constant buy amounts, sub-second reaction, high launch count.

### copy_trader
- Definition: mirrors another wallet's entries.
- Detection: repeatedly buys the same mints seconds AFTER a specific leader wallet.

### whale
- Definition: large position sizes.
- Detection: buys ≥ 10 SOL (tunable).

### flipper
- Definition: very short holds, high turnover.
- Detection: median hold < 5 minutes (the FAST_PROFIT_EXIT / FAST_LOSS_CUT classes).

### exchange_hot_wallet / evasion_wallet
- Definition: CEX wallet, or fresh wallet funded from a CEX to break the trail.
- Detection: funding tx from a known exchange wallet; brand-new wallet age.
- Verification: [INFERRED] — exchange-funded fresh wallet = elevated risk signal.

### known_winner
- Definition: verified high-ROI trader promoted by the CA scan pipeline.
- Detection: passed the scanner's clean-closed ROI gate on real (non-partial) history.

---

## Confidence scale (record in CSV `confidence` column)
| Level | Meaning |
|-------|---------|
| `ON_CHAIN_VERIFIED` | derived directly from tx data, no inference |
| `OFFICIAL_DOCS` | defined by Solana/pump.fun official documentation |
| `PATTERN_INFERRED` | inferred from behavior across multiple coins |
| `MANUAL` | hand-labeled by the operator |
| `UNVERIFIED` | added without verification — hint only, do not act on alone |
