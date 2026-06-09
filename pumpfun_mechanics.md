# pump.fun Mechanics Reference
# Source: https://pump.fun/docs/fees (official pump.fun docs only)

## Token Creation
| Item | Value |
|------|-------|
| Cost to create token | 0 SOL |
| Graduation fee (to PumpSwap) | 0.015 SOL |

## Bonding Curve Fees (pre-graduation)
All tokens on the bonding curve (before graduation to PumpSwap):

| Fee Type | Rate |
|----------|------|
| Creator fee | 0.300% |
| Protocol fee | 0.950% |
| LP fee | 0% |
| **Total** | **1.25%** |

## PumpSwap Canonical Pool Fees (post-graduation, SOL-denominated)
Fee tiers scale down as market cap rises:

| Market Cap (SOL) | Creator Fee | Protocol Fee | LP Fee | Total |
|-----------------|-------------|-------------|--------|-------|
| 0 – 420 SOL | 0.30% | 0.05% | 0.20% | **1.25%** |
| (mid tiers) | decreasing | 0.05% | 0.20% | decreasing |
| 98,240+ SOL | 0.05% | 0.05% | 0.20% | **0.30%** |

## PumpSwap Non-Canonical Pool Fees
| Fee Type | Rate |
|----------|------|
| Creator fee | 0% |
| Protocol fee | 0.05% |
| LP fee | 0.25% |
| **Total** | **0.30%** |

## PumpSwap Canonical Pool Fees (USDC-denominated)
Similar tiering to SOL-denominated:

| Market Cap (USDC) | Total Fee |
|------------------|-----------|
| 0 – 59,000 USDC | 1.25% |
| 20,000,000+ USDC | 0.30% |

## Mobile Fee Note
Mobile users may see fee increases of up to 0.1% on certain transactions.

## Key Account Labels for pump.fun On-Chain
| Label | Description |
|-------|-------------|
| `pump_bonding_curve` | PDA managing bonding curve for a specific mint |
| `pump_bonding_curve_ata` | Token account holding token supply in bonding curve |
| `pumpswap_pool` | AMM pool account after graduation |
| `pumpswap_vault_sol` | SOL vault in PumpSwap pool |
| `pumpswap_vault_token` | Token vault in PumpSwap pool |

## Audit Notes (On-Chain Verification Required)
- Bonding curve fees must be subtracted from every buy/sell when computing wallet PnL
- 1.25% total fee applies to all pre-graduation trades
- Post-graduation fees are tiered — need to look up market cap at time of trade
- Graduation threshold: 85 SOL raised on bonding curve triggers migration (NOT in official docs — NEEDS ON-CHAIN VERIFICATION)
- Creator fee recipient = the wallet that deployed the token
