# Label Categories

## Wallet Categories
- `smart_money` — consistently profitable traders, high ROI across multiple coins
- `dev` — coin deployer / project wallet
- `dev_side_wallet` — recurring wallet linked to a known dev
- `sniper` — block-0 or early sniper wallet
- `bot` — automated trading bot
- `bundle_wallet` — wallet used in coordinated bundle buy
- `exchange` — centralized exchange hot/cold wallet
- `copy_trader` — known copy trading wallet
- `whale` — large position holder ($10k+ entries)
- `rugged` — wallet associated with rug pulls
- `known_winner` — verified high-ROI trader from copy sim

## Sources
- `manual` — added by hand
- `ca_wallet_discovery` — surfaced by CA scan pipeline
- `dev_profile_seeder` — sourced from dev profile seeder
- `dune` — sourced from Dune Analytics
- `leaderboard` — from pump.fun leaderboard scrape
- `community` — community-sourced label

## Files
- `labels.csv` — general Solana wallet labels
- `pumpfun_labels.csv` — pump.fun specific wallet labels (devs, snipers, bundlers)
