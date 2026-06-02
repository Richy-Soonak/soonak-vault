# The Vault Discovery Problem

## Summary

Soonak Vault cannot be built using the Hyperliquid public API alone. The API does not expose a vault list or vault discovery endpoint. This document explains the problem in detail, what we've tried, and the remaining options.

## What We Need

To build Soonak Vault, we need:

1. **A list of all leader vaults** — names, leader addresses, contract addresses
2. **Per-vault performance data** — APY, TVL, drawdown, trade history, open positions
3. **User position tracking** — which vaults a user has deposited into, equity values

## What the Hyperliquid Public API Provides

Tested against `https://api.hyperliquid.xyz/info` on 2026-06-02:

### Working Endpoints

| Endpoint | Data | Notes |
|----------|------|-------|
| `userVaultEquities` | User's vault positions + equity | Only shows vaults the user is already in. Returns `[]` if no positions. |
| `vaultDetails` | Single vault metadata + performance | Works with a valid vault address. Returns `null` for invalid addresses. |
| `clearinghouseState` | User's margin account | For wallet balance tracking |
| `spotClearinghouseState` | User's spot balances | For USDC balance |
| `userFills` | User's trade/deposit/withdraw history | For tracking vault entry dates |
| `portfolio` | User's portfolio value history | Daily account value over time |

### Non-Working Endpoints (all return HTTP 422)

`vaults`, `vaultList`, `vaultSummary`, `allVaults`, `leaderVaults`, `vaultData`, `vaultInfo`, `vaultPerformance`, `vaultStats`, `vaultDeposits`, `userVaultDeposits`, `vaultHistory`, `vaultRanking`, `vaultRankings`, `vaultLeaderboard`, `vaultApr`, `vaultApy`, `vaultMetadata`, `vaultTrades`, `vaultPositions`, `vaultWithdrawals`

Every reasonable variation of vault listing was tested. None work.

## The Core Problem

**There is no public API endpoint that lists all vaults or enables vault discovery.**

The Hyperliquid app (`app.hyperliquid.xyz/vaults`) displays 3,233 vaults with full performance data. This data exists server-side. But it is not exposed through the public API.

### What the App Shows

The app's vaults page (which requires authentication) displays:

- **3,233 total vaults** (as of June 2026)
- **$435M total TVL** across all vaults
- **Per-vault metrics:** APY, TVL, age, depositor count, leader address
- **Per-vault detail:** PnL curve, max drawdown, Sharpe ratio, win rate, volume, trade history, open positions

### What We Can Get Without Auth

Even without authentication, the app's first page of vaults (sorted by TVL) is partially visible. We confirmed:

- Top vault: HyperGrowth by Systemic Strategies — $14.4M TVL, 329% APR, 276 days old
- 10th vault: FC Genesis - Quantum — $2.2M TVL, -0.76% APR, 262 days old
- Protocol vault HLP: $353M TVL (separate from leader vaults)

## What We've Tried

### 1. Public API Endpoint Discovery
- Tested 30+ variations of vault-related endpoint types against `api.hyperliquid.xyz/info`
- All return 422 except `vaultDetails` (single vault, needs address) and `userVaultEquities` (user's own positions only)

### 2. REST API Patterns
- Tested `api.hyperliquid.xyz/vaults`, `/vault/list`, `/vault/leader`, etc.
- All return 404 or 403

### 3. App Scraping (Unauthenticated)
- Loaded `app.hyperliquid.xyz/vaults` in browser
- First page renders with vault data (names, APY, TVL, age)
- Full data requires authentication
- Pagination through 323 pages (3,233 vaults × 10 per page) would be slow

### 4. On-Chain Indexing
- HyperEVM is accessible at `rpc.hyperliquid.xyz/evm`
- Vault contracts are deployed on HyperEVM
- Challenge: identifying which contracts are vaults vs other Deploys
- Would need to scan all contract deployments and filter by interface

### 5. GitBook Documentation
- Reviewed all public API documentation at `hyperliquid.gitbook.io`
- Vault documentation confirms the data model but doesn't list a discovery endpoint
- The `vaultDetails` endpoint is documented but requires a vault address

## Remaining Options

### Option A: Authenticated App Scraping
- Log in to the Hyperliquid app
- Scrape the vaults page with authentication
- Paginate through all 323 pages
- **Pros:** Gets exact same data the app shows
- **Cons:** Breaks on UI changes, requires auth, slow (11+ minutes per full scrape)

### Option B: On-Chain Indexer
- Scan HyperEVM for vault contract deployments
- Track Deposit/Withdraw events for TVL calculation
- Read vault state from contract storage
- **Pros:** Robust, doesn't depend on UI
- **Cons:** Complex, need to identify vault contract patterns, doesn't get APY/drawdown directly

### Option C: Hybrid (Recommended)
- Use authenticated scraping for MVP (fast to build)
- Build on-chain indexer in parallel (robust long-term)
- Cross-validate data between both sources
- **Pros:** Fast start, robust long-term
- **Cons:** Two systems to maintain

### Option D: Community API
- Check if any community member has reverse-engineered the internal API
- The Hyperliquid Discord or community tools might have solutions
- **Pros:** Could be the simplest solution if it exists
- **Cons:** May not exist or may be unreliable

## Data Requirements for Soonak Vault

Even with ~300-400 vaults above $100K TVL, the data pipeline needs:

| Data | Frequency | Source |
|------|-----------|--------|
| Vault list (names, addresses) | Daily | Scraping or on-chain |
| APY / TVL / Drawdown | Daily | Scraping or `vaultDetails` |
| Trade history | Daily | Scraping or on-chain events |
| Open positions | Daily | Scraping |
| User positions | Every 60s | `userVaultEquities` |
| User deposit/withdraw history | Every 60s | `userFills` |

## Estimated Data Volume

For ~350 vaults above $100K TVL with 90-day retention:

| Component | Daily | 90-Day Total |
|-----------|-------|--------------|
| Performance snapshots | ~90 KB | ~7.2 MB |
| Trade history | ~100 KB | ~8.5 MB |
| Health scores | ~28 KB | ~2.2 MB |
| **Total** | **~218 KB** | **~18 MB** |

Very manageable for Supabase (free tier: 500 MB).

## Call to Action

If you know of a public API endpoint that lists Hyperliquid vaults, or have reverse-engineered the internal API the Hyperliquid app uses, please open an issue or PR. This is the single blocker for building Soonak Vault.
