# Soonak Vault

Hyperliquid leader vault discovery, analysis, and management.

## What Is Soonak Vault?

Soonak Vault is a dashboard for discovering and managing positions in Hyperliquid leader vaults. Leader vaults are onchain trading vaults where a skilled trader (the "leader") manages deposited capital and takes 10% of profits. Depositors share pro-rata in the PnL.

### Features

- **Discover** — Browse and filter leader vaults by APY, TVL, Sharpe ratio, max drawdown, age, and win rate
- **Analyze** — Deep dive into vault performance: PnL curves, trade history, open positions, risk metrics
- **Track** — Monitor your positions across multiple vaults with real-time equity tracking
- **Exit** — Get alerts when vaults degrade (drawdown breaches, negative APY streaks, inactivity)
- **Manage** — One-click deposit and withdraw with 4-day lockup tracking

### Rationale

Hyperliquid has **3,233 leader vaults** with a combined TVL of ~$435M. But there is no good tool for discovering which vaults are worth depositing into. The Hyperliquid app shows basic stats but lacks:

- Risk-adjusted return metrics (Sharpe ratio)
- Historical performance trends
- Cross-vault comparison and ranking
- Automated exit alerts
- Portfolio-level tracking across multiple vaults

Soonak Vault fills this gap.

### Tech Stack

- **Frontend** — React + TypeScript + Tailwind CSS
- **Database** — Supabase (Postgres)
- **Data Ingestion** — Hyperliquid API + HyperEVM on-chain indexing + app scraping (see [#1](issues/1))
- **Deployment** — Vercel (frontend) + Railway/Render (indexer)

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Hyperliquid   │     │   Soonak Vault   │     │   Supabase      │
│   API + EVM     │────▶│   Indexer        │────▶│   (Postgres)    │
│   + App Scrape  │     │   Service        │     │                 │
└─────────────────┘     └──────────────────┘     └────────┬────────┘
                                                          │
                                                          ▼
                                                 ┌─────────────────┐
                                                 │   Dashboard     │
                                                 │   (React)       │
                                                 │   + Supabase    │
                                                 └─────────────────┘
```

## Data Schema

Soonak Vault tracks leader vaults with these metrics:

| Metric | Source | Purpose |
|--------|--------|---------|
| APY (7d/30d/90d/all-time) | App scraping | Performance assessment |
| TVL | `vaultDetails` + scraping | Size and capacity |
| Max drawdown | App scraping | Risk assessment |
| Sharpe ratio | Calculated | Risk-adjusted returns |
| Win rate | Trade history | Consistency |
| Volume (24h/7d/30d) | App scraping | Activity level |
| Age | On-chain | Track record length |
| Depositor count | App scraping | Social proof |
| Open positions | App scraping | Current exposure |

## Roadmap

### Phase 1: Discovery (MVP)
- [ ] Vault list with key metrics (APY, TVL, Sharpe, drawdown)
- [ ] Filter and sort by risk profile (Aggressive/Normal/Passive)
- [ ] Vault detail page with performance history
- [ ] Paper tracking mode (track vaults without depositing)
- [ ] Supabase schema and data pipeline

### Phase 2: Management
- [ ] Wallet connection and position tracking via `userVaultEquities`
- [ ] Deposit and withdraw with lockup countdown
- [ ] Portfolio dashboard (total equity, P&L, per-vault breakdown)
- [ ] Daily performance alerts

### Phase 3: Intelligence
- [ ] Automated exit alerts based on risk thresholds
- [ ] Vault health scoring (multi-signal composite)
- [ ] Leader reputation tracking across vaults
- [ ] Auto-rebalance suggestions

### Phase 4: Automation
- [ ] Auto-deposit into top-performing vaults
- [ ] Auto-exit on health score degradation
- [ ] Undo window for automated actions

## Contributing

Soonak Vault is open source. We welcome contributions, especially around:

- Better vault discovery methods (the [#1](issues/1) problem)
- On-chain indexer improvements
- UI/UX for vault comparison

## License

MIT
