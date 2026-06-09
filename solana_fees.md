# Solana Transaction Fees
# Source: https://solana.com/docs/core/fees (official Solana docs)

## Transaction Fee Components

### 1. Base Fee
| Item | Value |
|------|-------|
| Amount | 5,000 lamports per signature |
| Distribution | 50% burned, 50% to validator |
| Purpose | Compensates validators for signature verification |

### 2. Prioritization Fee (optional)
| Item | Formula |
|------|---------|
| Calculation | `ceil(compute_unit_price × compute_unit_limit / 1,000,000)` lamports |
| Distribution | 100% to validator |
| Purpose | Increases likelihood transaction is processed ahead of competing ones |
| Set via | Compute Budget Program instruction |

## Rent
| Term | Definition |
|------|------------|
| Rent | Fee for storing data on-chain |
| Rent-exempt | Account maintains minimum lamport balance to avoid periodic rent charges |
| CloseAccount | Reclaims rent lamports back to owner when account is closed |

## Compute Units
| Term | Definition |
|------|------------|
| Compute unit limit | Max compute units a transaction may consume |
| Compute unit price | Micro-lamports per compute unit for prioritization |

## Fee Audit Implications for Copy Sim
- Every transaction costs at least 5,000 lamports (~0.000005 SOL) per signature
- Priority fees vary by network congestion — must be read from actual transaction meta
- Fees paid by the fee-payer account (first signer in transaction)
- When computing wallet PnL, subtract: base fee + priority fee + pump.fun trading fee
- Rent refunds from CloseAccount must NOT be counted as trade proceeds
