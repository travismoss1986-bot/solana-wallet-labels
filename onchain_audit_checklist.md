# On-Chain Audit Checklist
# What must be verified against actual chain data before trusting any PnL calculation

## Transaction Classification
- [ ] Real buy = token balance INCREASED + SOL/WSOL DECREASED + explicit swap context + tx succeeded
- [ ] Real sell = token balance DECREASED + SOL/WSOL INCREASED + explicit swap context + tx succeeded
- [ ] Rent refund (CloseAccount) must NOT count as sell proceeds
- [ ] Fee-only SOL decrease must NOT count as a sell
- [ ] Transfer-in (no SOL decrease) must NOT count as a buy
- [ ] Transfer-out (no SOL increase) must NOT count as a sell
- [ ] Failed transactions (meta.err != null) must be excluded entirely

## Fee Deduction (pump.fun)
- [ ] Pre-graduation trades: subtract 1.25% total fee from every buy and sell
- [ ] Post-graduation PumpSwap trades: subtract tiered fee (0.30%-1.25%) based on market cap at trade time
- [ ] Transaction base fee (5,000 lamports × signatures) subtract from SOL balance delta
- [ ] Priority fee (from Compute Budget instruction) subtract from SOL balance delta

## SOL Balance Delta Calculation
- [ ] Use `meta.preBalances[account_index]` and `meta.postBalances[account_index]`
- [ ] Use `meta.preTokenBalances` and `meta.postTokenBalances` for token amounts
- [ ] Account index must match the correct wallet address in `transaction.message.accountKeys`

## Price Calculation
- [ ] Price per token = SOL spent / tokens received (buy) or SOL received / tokens sold (sell)
- [ ] Use actual on-chain amounts from balance deltas, not instruction data alone
- [ ] WSOL: unwrap WSOL balance changes to SOL equivalent before computing price

## PnL Calculation (FIFO)
- [ ] Cost basis = actual SOL paid including fees
- [ ] Realized PnL = sell proceeds (after fees) - FIFO cost basis
- [ ] Unrealized PnL = current price × remaining tokens - FIFO cost basis
- [ ] Do NOT include open positions in win rate or ROI
- [ ] Partial sells reduce open position proportionally

## Graduation / Migration Events
- [ ] Detect when bonding curve account closes → this signals graduation to PumpSwap
- [ ] After graduation: resolve PumpSwap pool address for this mint
- [ ] PumpSwap trades use different program (`pAMMBay6oceH9fJKBRHGP5D4bD4sWpmSwMn52FMfXEA`)
- [ ] Both pre and post-graduation trades needed for complete wallet history on a coin

## Data Completeness
- [ ] Signature pagination: must fetch ALL signatures (not just first page)
- [ ] Cache signatures by (source_account, oldest_cursor) to avoid window shift
- [ ] Flag history as PARTIAL if pagination was capped before reaching genesis
- [ ] Unmatched sells (no preceding buy) = incomplete history, do not infer buy price

## Audit Results (2026-06-09)
- [x] CORRECTED: tx base/priority fees ARE deducted (once, correctly) via the
      balance-delta + meta.fee machinery. Earlier "ROI is gross" claim was wrong.
- [x] FIXED: pump.fun 1.25% bonding-curve fee was dropped on the explicit-spend path
      (wallet→curve transfer only). Now uplifted by PUMP_BONDING_CURVE_FEE_RATE=0.0125
      in live_rpc_source.py. Verified to within 1 lamport of on-chain ground truth.
      Buys on the balance-delta fallback path already included the fee.

## Remaining Gaps in Current Build (as of 2026-06-09)
- [ ] PumpSwap post-graduation coverage is partial; fees there are tiered (0.30%–1.25%)
      and NOT the flat bonding-curve 1.25% — needs market-cap tier lookup, not yet built
- [ ] Pagination cap means most scans return PARTIAL_CA_TRADE_HISTORY
- [ ] Copy sim slippage uses aggregate formula, not per-trade price replay
- [ ] Graduation threshold (SOL) still unverified on-chain — do not treat as fact
