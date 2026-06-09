# Wallet Label Taxonomy
# Combined: official Solana account types + pump.fun roles + trading archetypes

## Solana Account Role Labels
These come directly from on-chain account structure (owner program + executable flag).

| Label | How to Detect |
|-------|--------------|
| `system_account` | owner = `11111111111111111111111111111111` |
| `program_account` | executable = true |
| `token_account` | owner = Token Program, holds token balance |
| `mint_account` | owner = Token Program, has `supply` + `decimals` fields |
| `ata` | token account at deterministic PDA address |
| `pda` | address derived by program, no private key exists |
| `sysvar` | special cluster-state account |

## pump.fun On-Chain Role Labels
| Label | How to Detect |
|-------|--------------|
| `pump_deployer` | wallet that called InitializeMint on pump.fun program |
| `pump_bonding_curve` | PDA owned by pump.fun program for a specific mint |
| `pump_bonding_curve_ata` | token account holding bonding curve supply |
| `pumpswap_pool` | account owned by PumpSwap program post-graduation |
| `pumpswap_lp` | LP position holder in PumpSwap pool |

## Wallet Behavior Labels (Trading Archetypes)
| Label | Definition |
|-------|------------|
| `smart_money` | Consistently profitable across many trades, high ROI |
| `early_buyer` | Buys within first 50 transactions of a coin launch |
| `sniper` | Block-0 or slot-0 buyer on bonding curve open |
| `bundle_wallet` | Wallet coordinated in a multi-wallet launch bundle |
| `dev_wallet` | Wallet that deployed one or more tokens |
| `dev_side_wallet` | Wallet linked to a known dev, appears across multiple launches |
| `bot` | Fixed buy amounts, fixed timing, repeating pattern |
| `copy_trader` | Detected pattern of copying another wallet's entries |
| `whale` | Large SOL position sizes (10+ SOL buys) |
| `flipper` | Holds <5 minutes, high turnover |
| `diamond_hands` | Holds through major price swings |
| `rugged_victim` | Frequently holds to near-zero |
| `known_winner` | Verified high-ROI from CA scan pipeline |
| `exchange_hot_wallet` | CEX deposit/withdrawal wallet |
| `evasion_wallet` | Newly created wallet funded from exchange to break trail |

## Confidence Levels for Labels
| Level | Meaning |
|-------|---------|
| `ON_CHAIN_VERIFIED` | Derived directly from transaction data, no inference |
| `PATTERN_INFERRED` | Inferred from behavioral patterns across multiple coins |
| `MANUAL` | Manually labeled by operator |
| `UNVERIFIED` | Added without verification — treat as hint only |
