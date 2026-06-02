# Soonak Vault — Data Sources

This document tracks all known sources of Hyperliquid vault data.

## Public API (Confirmed Working)

| Endpoint | Type | Data | Limitations |
|----------|------|------|-------------|
| `userVaultEquities` | User | Vault positions + equity | Only shows user's own positions |
| `vaultDetails` | Vault | Single vault metadata | Needs vault address |
| `clearinghouseState` | User | Margin account | Balance tracking |
| `userFills` | User | Deposit/withdraw history | Entry date tracking |
| `portfolio` | User | Portfolio value history | Daily snapshots |

## Public API (Confirmed NOT Working)

All return HTTP 422:
`vaults`, `vaultList`, `vaultSummary`, `allVaults`, `leaderVaults`, `vaultData`, `vaultInfo`, `vaultPerformance`, `vaultStats`, `vaultDeposits`, `vaultHistory`, `vaultRanking`, `vaultRankings`, `vaultLeaderboard`, `vaultApr`, `vaultApy`, `vaultMetadata`, `vaultTrades`, `vaultPositions`, `vaultWithdrawals`

## Known Data Sources

### Hyperliquid App
- **URL:** `https://app.hyperliquid.xyz/vaults`
- **Data:** Full vault list (3,233 vaults), APY, TVL, age, depositor count, leader address
- **Per-vault:** PnL curve, max drawdown, Sharpe ratio, win rate, volume, trade history, open positions
- **Access:** Requires authentication
- **Reliability:** Subject to UI changes

### HyperEVM On-Chain
- **RPC:** `https://rpc.hyperliquid.xyz/evm`
- **Data:** Contract deployments, Deposit/Withdraw events, vault state
- **Challenge:** Need to identify vault contract patterns vs other contracts
- **Reliability:** Fully decentralized, doesn't depend on any API

## Community Contributions

If you know of additional data sources, please open an issue with the `[DATA]` label.

See [PROBLEM.md](../PROBLEM.md) for the full problem statement.
