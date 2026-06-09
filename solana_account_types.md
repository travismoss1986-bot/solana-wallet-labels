# Solana Account Types & Labels
# Source: https://solana.com/docs/terminology + https://solana.com/docs/core/accounts
# Source: https://www.solana-program.com/docs/token + https://www.solana-program.com/docs/associated-token-account

## Core Account Fields (all accounts share these)
| Field | Type | Description |
|-------|------|-------------|
| `lamports` | u64 | SOL balance in lamports (1 SOL = 1,000,000,000 lamports) |
| `data` | bytes | Stored data, max 10 MiB |
| `owner` | pubkey | Program that controls modifications to this account |
| `executable` | bool | True if account contains deployed program code |
| `rent_epoch` | u64 | Tracks rental obligation status |

## Account Type Classification
Classification is determined by the `owner` program and `executable` flag.

| Label | Owner | Executable | Description |
|-------|-------|-----------|-------------|
| `system_account` | System Program (`1111...1111`) | false | Standard wallet / SOL holder |
| `program_account` | BPF Upgradeable Loader | true | Deployed on-chain program |
| `token_account` | Token Program | false | Holds SPL token balance for one mint |
| `token_2022_account` | Token-2022 Program | false | Token account with extensions |
| `mint_account` | Token Program | false | Token mint (supply, decimals, authorities) |
| `ata` | Token Program | false | Token account at deterministic PDA address |
| `pda` | Any program | false | Program Derived Address — no private key |
| `sysvar` | Sysvar Program | false | Cluster-wide state (clock, rent, etc.) |

## Mint Account Fields (SPL Token)
# Source: https://www.solana-program.com/docs/token
| Field | Description |
|-------|-------------|
| `mint_authority` | Pubkey that can create new tokens (null = fixed supply) |
| `supply` | Total tokens in circulation |
| `decimals` | Number of decimal places |
| `is_initialized` | Whether mint is active |
| `freeze_authority` | Pubkey that can freeze token accounts (optional) |

## Token Account Fields (SPL Token)
| Field | Description |
|-------|-------------|
| `mint` | The token mint this account belongs to |
| `owner` | Authority over this account (the wallet) |
| `amount` | Current token balance |
| `delegate` | Delegated spending authority (optional) |
| `state` | `Initialized`, `Frozen`, or `Uninitialized` |
| `is_native` | Set if this is a wrapped-SOL account |
| `delegated_amount` | Amount approved to delegate |
| `close_authority` | Who can close this account and reclaim rent |

## Associated Token Account (ATA) Derivation
# Source: https://www.solana-program.com/docs/associated-token-account
ATA address = PDA derived from:
1. Wallet public key
2. Token Program ID (`TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`)
3. Token mint address

Program: `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL`

## Token Program Instructions
# Source: https://www.solana-program.com/docs/token
| Instruction | Description |
|-------------|-------------|
| `InitializeMint` | Creates a new token mint |
| `InitializeAccount` | Sets up an account to hold token balances |
| `Transfer` | Moves tokens between accounts |
| `MintTo` | Creates new tokens (requires mint authority) |
| `Burn` | Removes tokens from circulation |
| `Approve` | Delegates authority over tokens |
| `Revoke` | Cancels delegated authority |
| `FreezeAccount` | Renders an account unusable |
| `ThawAccount` | Re-enables a frozen account |
| `SetAuthority` | Changes mint or account authorities |
| `CloseAccount` | Closes account and reclaims SOL rent |
| `InitializeMultisig` | Creates multisignature authorities |
