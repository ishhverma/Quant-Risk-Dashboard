# FINANCIAL MARKET QUANTITATIVE DASHBOARD  
### Multi-Asset, Regime-Aware, Risk-First System

## 1. Project Overview
This repository implements an end-to-end quantitative market intelligence and portfolio analytics system built on daily historical market data sourced from Yahoo Finance. The system covers the full institutional quant workflow: data ingestion, validation, feature engineering, regime detection, signal generation, portfolio backtesting, attribution, risk analysis, and visualization via an interactive Streamlit dashboard. 
The goal is not just to visualize prices, but to detect market regimes, generate systematic trading signals, evaluate portfolio performance and fragility, and analyze tail risk and drawdowns to produce decision-ready quantitative insights. The core philosophy is **risk** first, regimes matter, and returns must be explained—not assumed.

## 2. Objectives
- Build a research-grade quantitative dashboard aligned with institutional quant and risk workflows. 
- Move beyond basic indicators toward regime-aware analytics and state-dependent behavior.  
- Integrate portfolio construction, attribution, and risk intelligence into a single, reproducible system.  
- Deliver realistic, defensible metrics (no overfit Sharpe ratios, no hidden look-ahead bias, and clear assumptions).

## 3. Asset Universe & Data
### 3.1 Multi-Asset Coverage
Tracked instruments:
- Equities: `AAPL`, `MSFT`, `GOOGL`, `TSLA`  
- Commodities: `GC=F` (Gold), `SI=F` (Silver)  
- FX: `USDINR=X`, `EURUSD=X`  
- Benchmark Index: `^GSPC` (S&P 500)

Data summary:
- Data fetched: 9 tickers  
- Failed downloads: 0 tickers  

### 3.2 Data Period & Frequency
- Date range: 2021-02-08 to 2026-02-05  
- Frequency: Daily (trading days)  
- Observations: 1,301 trading days  

### 3.3 Data Quality & Validation
Before any analytics or backtests, the raw Yahoo Finance data is validated and cleaned to ensure robustness:
- Initial missing values (before cleaning):  
  - Equities & benchmark: 3.54%  
  - Commodities: 3.31%  
- Missing values after cleaning: 0.0%  
- Outlier count: 0  
- Return spikes detected: 0  
- Last data date: 2026-02-05  
- Data lag: 0 days  
- Data fresh: True  
- Data quality score: 100.0  
All metrics and backtests are therefore computed on a clean, consistent dataset.

## 4. Feature Engineering
From the cleaned OHLCV data, the project builds a rich, multi-asset feature matrix for regime detection and signal generation.
- Feature matrix shape: `(1301, 180)`  
- Indicators per ticker: 19 technical/statistical features  

Feature families:
- Log returns and cumulative returns  
- Rolling volatility and volatility-of-volatility  
- Trend and momentum indicators (SMA, EMA, RSI, MACD)  
- Statistical descriptors (skewness, kurtosis)  
- Drawdown-related metrics and risk-sensitive features  
These features feed both the regime detection engine and signal generation layer.

## 5. Market Regime Detection
### 5.1 Regime Definition & Model
The system classifies each trading day into one of four market regimes:
- Low Volatility  
- High Volatility  
- Recovery  
- Crisis
    
Regime model inputs:
- Regime feature input shape: `(1301, 4)`
   
Sample transition probabilities:
- Crisis → Crisis: 91.35%  
- High Volatility → Crisis: 83.33%  
- Low Volatility → Low Volatility: 94.12%  
- Recovery → Low Volatility: 85.18%  
These transition probabilities encode persistence in calm/crisis regimes and mean-reversion toward normal conditions.

### 5.2 Role in the System
The regime layer is central to the architecture and governs:
- Signal activation and deactivation (e.g., turning signals off in deep crises or high-stress states)  
- Risk exposure constraints (caps on leverage, position size, or gross exposure by regime)  
- Portfolio behavior across calm, volatile, and crisis environments, enabling regime-aware allocation and scaling.

## 6. Signal Generation
Signals are generated via `signal_engine.py`, which consumes both feature-engineered data and regime classifications.
Key properties:
- Signals generated for 1,301 trading days.  
- Entry and exit logic is systematic, rule-based, and regime-aware.  
- Signals are produced per asset and handed off to the portfolio module as desired position weights and trades.  

Signal evaluation focuses on:
- Robustness to different regimes.  
- Drawdown impact and tail behavior.  
- Regime compatibility and stability, not just raw PnL.

## 7. Portfolio Construction & Backtesting
### 7.1 Backtest Setup
The portfolio engine takes signals and simulates a daily rebalanced, multi-asset strategy.
- Initial capital: 100,000 USD  
- Final portfolio value: 145,121.85 USD  
- Number of daily returns: 1,300  
- Baseline allocation: Equal-weighted across the asset universe  
- Rebalancing: Daily  

### 7.2 Performance Attribution
The `attribution.py` module decomposes performance by asset and regime to explain where returns come from.
Total strategy PnL by asset (normalized):
- AAPL: 0.32  
- MSFT: 0.62  
- GOOGL: 0.84  
- TSLA: -0.17  
- GC=F: 0.17  
- SI=F: 0.30  
- USDINR=X: 0.08  
- EURUSD=X: -0.10  
- ^GSPC: 0.37  

Regime-specific annualized mean daily returns:
- Crisis: -3.56%  
- High Volatility: 11.59%  
- Low Volatility: 26.27%  

Benchmark-relative metrics vs `^GSPC`:
- Alpha: 0.0003  
- Beta: 0.7964  
- R-squared: 0.6981  
- Volatility harvesting alpha: NaN (not computed or not meaningful for this configuration)

## 8. Risk & Tail Analysis
The risk engine focuses on both standard risk-adjusted metrics and explicit tail-risk diagnostics.
Data shapes:
- Close prices shape: `(1301, 9)`  
- Portfolio returns shape: `(1300,)`  
- Average daily return: 0.0300%  

### 8.1 Risk Metrics
- Sharpe ratio: 0.674  
- Sortino ratio: 0.640  
- Calmar ratio: 0.641  
- Maximum drawdown: -11.79%  
These metrics quantify reward per unit of volatility, downside risk, and drawdown depth.

### 8.2 Tail Risk
- Historical VaR (95%): -0.84%  
- Parametric VaR (95%): -0.83%  
- Conditional VaR (95%): -1.37%  
This provides both historical and model-based estimates of left-tail loss, as well as expected loss beyond VaR.

### 8.3 Return Distribution
Distributional statistics of daily portfolio returns:
- Mean: 0.02%  
- Standard deviation: 0.55%  
- Skewness: -0.0875  
- Kurtosis: 14.2706  
- Jarque-Bera p-value: 0.0000  
- Minimum daily return: -4.88%  
- Maximum daily return: 3.40%  
- Median daily return: 0.0000  
These statistics indicate fat-tailed, non-normal return behavior, underscoring the need for explicit tail-risk analysis.

## 9. Fragility & Drawdown Intelligence
### 9.1 Drawdown Episodes
The system identifies and characterizes drawdown regimes.
- Number of drawdown periods: 39  
- Average drawdown duration: 30.2 days  
- Average max drawdown per episode: -1.45%  
- Worst drawdown: -11.79%  
This allows you to study how the strategy behaves during stress and how long recovery typically takes.

### 9.2 Fragility Index
The `fragility.py` module aggregates tail and drawdown information into a composite Fragility Index.
- Fragility Index: 3.6234  
- Fragility_Kurtosis: 14.2706  
- Fragility_Skewness: 0.0875  
- Fragility_Max_Drawdown: 12.17%  
- Fragility_CVaR: 1.37%  
Higher values indicate more fragile, tail-sensitive performance, while lower values indicate more robust behavior.

## 10. Per-Asset Metrics
The dashboard also exposes single-asset statistics, which can be used for both standalone analysis and portfolio construction.
### AAPL
- Annual return: 14.66%  
- Annual volatility: 27.57%  
- Sharpe ratio: 0.53  
- Max drawdown: -35.18%  
- RSI status: Unknown  

### MSFT
- Annual return: 11.61%  
- Annual volatility: 26.06%  
- Sharpe ratio: 0.45  
- Max drawdown: -40.61%  
- RSI status: Unknown  

### GOOGL
- Annual return: 23.45%  
- Annual volatility: 30.73%  
- Sharpe ratio: 0.76  
- Max drawdown: -47.95%  
- RSI status: Unknown  

### TSLA
- Annual return: 7.18%  
- Annual volatility: 60.15%  
- Sharpe ratio: 0.12  
- Max drawdown: -79.88%  
- RSI status: Unknown  

### GC=F (Gold)
- Current price: 4917.50  
- 20-day change: 10.52%  
- Annual return: 20.03%  
- Annual volatility: 17.24%  
- Sharpe ratio: 1.16  
- Max drawdown: -20.88%  
- RSI status: Normal  

### SI=F (Silver)
- Current price: 79.11  
- 20-day change: 2.55%  
- Annual return: 21.54%  
- Annual volatility: 36.94%  
- Sharpe ratio: 0.58  
- Max drawdown: -40.94%  
- RSI status: Normal  

### USDINR=X
- Current price: 90.15  
- 20-day change: 0.32%  
- Annual return: 4.11%  
- Annual volatility: 4.57%  
- Sharpe ratio: 0.90  
- Max drawdown: -4.06%  
- RSI status: Normal  

### EURUSD=X
- Current price: 1.18  
- 20-day change: 1.11%  
- Annual return: -0.26%  
- Annual volatility: 7.51%  
- Sharpe ratio: -0.03  
- Max drawdown: -21.98%  
- RSI status: Normal  

### ^GSPC (S&P 500)
- Annual return: 11.48%  
- Annual volatility: 16.86%  
- Sharpe ratio: 0.68  
- Max drawdown: -27.11%  
- RSI status: Unknown  

## 11. Dashboard Overview
The Streamlit app (e.g., `dashboard/app.py`) exposes an interactive dashboard for exploration and decision support. [github](https://github.com/shwetanaren/streamlit-financial-dashboard)
Key views:
- Market overview summary (headline stats, current regimes, and risk snapshot).  
- Normalized price comparison across assets to visualize relative performance.  
- Risk–return scatter plots for cross-sectional risk–reward analysis.  
- Correlation heatmaps to inspect diversification and clustering.  
- Market regime timeline and distribution plots to understand state persistence.  
- Rolling Sharpe, volatility, and drawdown charts for time-varying performance.  
- VaR / CVaR and tail-risk visualizations.  
- Asset rankings and performance tables.  
- RSI and return-distribution plots for microstructure and overbought/oversold context.

## 13. Tech Stack
Core stack:
- Python  
- `yfinance` for historical market data  
- `pandas`, `numpy` for data handling and numerical computing  
- `scipy`, `statsmodels` for statistics and distributional tests  
- `matplotlib`, `seaborn` for static plots  
- `streamlit` for the interactive dashboard front-end 
