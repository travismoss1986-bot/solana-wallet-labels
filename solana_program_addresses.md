# Solana Native Program Addresses
# Source: https://solana.com/docs/terminology (official Solana docs)

## Core Native Programs

| Program | Address | Purpose |
|---------|---------|---------|
| System Program | `11111111111111111111111111111111` | Creates accounts, allocates data, transfers SOL |
| Token Program (SPL) | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` | SPL token mint, transfer, freeze, burn |
| Token Extensions Program (Token-2022) | `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` | Token Program with extensions (metadata, etc.) |
| Associated Token Account Program | `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL` | Derives deterministic token account addresses |
| BPF Upgradeable Loader | `BPFLoaderUpgradeab1e11111111111111111111111` | Owns and loads all deployed programs |
| Compute Budget Program | `ComputeBudget111111111111111111111111111111` | Sets compute unit limits and priority fees |

## Wrapped SOL

| Token | Mint Address | Notes |
|-------|-------------|-------|
| Wrapped SOL (WSOL) | `So11111111111111111111111111111111111111112` | Native SOL wrapped as SPL token |

## pump.fun Programs
# Source: https://pump.fun/docs/fees (official pump.fun docs) + on-chain verification

| Program | Address | Purpose |
|---------|---------|---------|
| pump.fun Program | `6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P` | Bonding curve trading (pre-graduation) |
| PumpSwap AMM | `pAMMBay6oceH9fJKBRHGP5D4bD4sWpmSwMn52FMfXEA` | AMM pool trading (post-graduation) |
