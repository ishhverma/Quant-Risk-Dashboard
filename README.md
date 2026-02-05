# FINANCIAL MARKET QUANTITATIVE DASHBOARD 
MULTI-ASSET, REGIME-AWARE, RISK-FIRST SYSTEM

## 1. Project Overview
This repository implements an end-to-end quantitative market intelligence and portfolio analytics system built on daily historical market data sourced from Yahoo Finance. The project covers the full research cycle: data ingestion, validation, feature engineering, regime detection, signal generation, portfolio backtesting, attribution, risk analysis, and an interactive Streamlit dashboard.
The goal is not just to visualize prices, but to detect market regimes, generate systematic trading signals, evaluate portfolio performance and fragility, and analyze tail risk and drawdowns to produce decision-ready quantitative insights.

> Core philosophy: **Risk** first, regimes matter, and returns must be explained—not assumed.

## 2. Objectives
- Build a research-grade quantitative dashboard aligned with institutional quant and risk workflows.  
- Move beyond basic indicators toward regime-aware analytics.  
- Integrate portfolio construction, attribution, and risk intelligence into a single system.  
- Deliver realistic, defensible metrics (no overfit Sharpe ratios or hidden look-ahead bias).

## 3. Asset Universe & Data

### 3.1 Multi-Asset Coverage
- Equities: AAPL, MSFT, GOOGL, TSLA  
- Commodities: GC=F (Gold), SI=F (Silver)  
- FX: USDINR=X, EURUSD=X  
- Benchmark Index: ^GSPC (S&P 500)  
In total, data was successfully fetched for 9 tickers, with 0 tickers failing to download.

### 3.2 Data Period & Frequency
- Date range: 2021-02-08 to 2026-02-05  
- Frequency: Daily (trading days)  
- Observations: 1,301 trading days in the final aligned dataset  

### 3.3 Data Quality & Validation
Before analytics, the data is passed through a dedicated validation step:
- Initial missing values (example percentages before cleaning):  
  - AAPL, MSFT, GOOGL, TSLA, ^GSPC: 3.54%  
  - GC=F, SI=F: 3.31%  
- Missing values after cleaning: 0.0% for all tickers  
- Outlier count: 0 for all tickers  
- Return spikes detected: 0 for all tickers  
- Last data date: 2026-02-05  
- Data lag: 0 days  
- Data fresh: True  
- Data quality score: 100.0  
This ensures that every subsequent backtest and risk metric is based on a clean, consistent dataset.


## 4. Feature Engineering
From the cleaned OHLCV data, the project constructs a feature matrix used for regime detection and signal generation:
- Feature matrix shape: (1301, 180)  
- Indicators per ticker: 19 technical/statistical features (excluding the raw `close` column)
- 
Feature categories include:
- Log returns and cumulative returns  
- Rolling volatility and volatility-of-volatility  
- Trend and momentum indicators (SMA, EMA, RSI, MACD)  
- Statistical descriptors (skewness, kurtosis)  
- Drawdown-related metrics and risk signals  

## 5. Market Regime Detection
The system classifies each trading day into one of four regimes:
- Low Volatility  
- High Volatility  
- Recovery  
- Crisis  

### 5.1 Regime Model
- Regime feature input shape: (1301, 4)  
Sample regime transition probabilities:
- Crisis → Crisis: 91.35%  
- High Volatility → Crisis: 83.33%  
- Low Volatility → Low Volatility: 94.12%  
- Recovery → Low Volatility: 85.18%  

### 5.2 Role in the System
The regime layer governs:
- Signal activation and deactivation  
- Risk exposure (for example, target volatility and gross exposure constraints)  
- Portfolio behavior across calm, volatile, and crisis environments  

## 6. Signal Generation
The `signal_engine.py` module generates regime-aware trading signals for each asset:
- Signals generated for 1,301 trading days  
- Entry and exit logic is systematic, driven by indicators and regime classification  
- Signals are consumed by the portfolio construction module to produce positions and trades  
Signals are evaluated on robustness, drawdown impact, and regime compatibility—not just raw performance.

## 7. Portfolio Construction & Backtesting
### 7.1 Backtest Setup
- Initial capital: 100,000 USD  
- Final portfolio value: 145,121.85 USD  
- Number of daily returns: 1300  
- Baseline allocation: Equal-weighted portfolio across selected assets  
- Rebalancing: Daily rebalancing to target weights  
This configuration provides a transparent baseline that can be compared to more advanced allocation schemes later.

### 7.2 Performance Attribution
The `attribution.py` module decomposes performance by both asset and regime.

**Total strategy PnL by asset (normalized):**
- AAPL: 0.32  
- MSFT: 0.62  
- GOOGL: 0.84  
- TSLA: -0.17  
- GC=F: 0.17  
- SI=F: 0.30  
- USDINR=X: 0.08  
- EURUSD=X: -0.10  
- ^GSPC: 0.37  

**Regime-specific annualized mean daily returns:**
- Crisis: -3.56%  
- High Volatility: 11.59%  
- Low Volatility: 26.27%  

**Benchmark-relative metrics vs ^GSPC:**
- Alpha: 0.0003  
- Beta: 0.7964  
- R-squared: 0.6981  
- Volatility harvesting alpha: NaN (not computed / not applicable in this version)  

## 8. Risk & Tail Analysis
Risk analysis focuses on both traditional and tail-risk metrics, computed from portfolio returns.
- Close prices shape: (1301, 9)  
- Portfolio returns shape: (1300,)  
- Date range: 2021-02-08 to 2026-02-05  
- Average daily return: 0.0300%  

### 8.1 Risk Metrics
- Sharpe ratio: 0.674  
- Sortino ratio: 0.640  
- Calmar ratio: 0.641  
- Maximum drawdown: -11.79%  

### 8.2 Tail Risk
- Historical VaR (95%): -0.84%  
- Parametric VaR (95%): -0.83%  
- Conditional VaR (95%): -1.37%  

### 8.3 Return Distribution
- Mean: 0.03%  
- Standard deviation: 0.52%  
- Skewness: -0.4341  
- Kurtosis: 6.5979  
- Jarque-Bera p-value: 0.0000  
- Minimum daily return: -3.86%  
- Maximum daily return: 2.47%  
- Median daily return: 0.0000  
These statistics indicate fat-tailed, non-normal return behavior, consistent with real financial markets.

## 9. Fragility & Drawdown Intelligence
### 9.1 Drawdown Episodes
- Number of drawdown periods: 39  
- Average drawdown duration: 30.2 days  
- Average maximum drawdown (per episode): -1.45%  
- Worst drawdown: -11.79%  

### 9.2 Fragility Index
The `fragility.py` module defines a composite Fragility Index combining higher moments and tail risk:
- Fragility Index: 1.7909  
- Fragility_Kurtosis: 6.5979  
- Fragility_Skewness: 0.4341  
- Fragility_Max_Drawdown: 11.79%  
- Fragility_CVaR: 1.37%  

This is designed to answer:
> “How quickly does the portfolio break under stress, and how severe is the damage when it does?”

## 10. Per-Asset Metrics
The dashboard computes individual asset statistics used in the analytics panels.
**AAPL**
- Annual return: 14.66%  
- Annual volatility: 27.57%  
- Sharpe ratio: 0.53  
- Max drawdown: -35.18%  
- RSI status: Unknown (latest close not available in this snapshot)
- <img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/7f5941b0-964f-4ba9-b61f-862828c41652" />


**MSFT**
- Annual return: 11.61%  
- Annual volatility: 26.06%  
- Sharpe ratio: 0.45  
- Max drawdown: -40.61%  
- RSI status: Unknown
- <img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/863f223c-9273-441a-af6e-6c9a5faf5881" />


**GOOGL**
- Annual return: 23.45%  
- Annual volatility: 30.73%  
- Sharpe ratio: 0.76  
- Max drawdown: -47.95%  
- RSI status: Unknown  
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/6cd30d80-a4ec-404f-8c9a-07ab230658f9" />

**TSLA**
- Annual return: 7.18%  
- Annual volatility: 60.15%  
- Sharpe ratio: 0.12  
- Max drawdown: -79.88%  
- RSI status: Unknown  
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/ffd44c3c-5a79-460c-b566-a75a2fb8c6f5" />

**GC=F (Gold)**
- Current price: 4917.50  
- 20-day change: 10.52%  
- Annual return: 20.03%  
- Annual volatility: 17.24%  
- Sharpe ratio: 1.16  
- Max drawdown: -20.88%  
- RSI status: Normal  
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/d97a9c18-6d6d-43df-8393-318c7be7de45" />

**SI=F (Silver)**
- Current price: 79.11  
- 20-day change: 2.55%  
- Annual return: 21.54%  
- Annual volatility: 36.94%  
- Sharpe ratio: 0.58  
- Max drawdown: -40.94%  
- RSI status: Normal  
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/06231e2e-d834-43e1-a012-8444669b9a27" />

**USDINR=X**
- Current price: 90.15  
- 20-day change: 0.32%  
- Annual return: 4.11%  
- Annual volatility: 4.57%  
- Sharpe ratio: 0.90  
- Max drawdown: -4.06%  
- RSI status: Normal  
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/be5a9c57-0e2e-44b6-bc2f-8f6ac35d183e" />

**EURUSD=X**
- Current price: 1.18  
- 20-day change: 1.11%  
- Annual return: -0.26%  
- Annual volatility: 7.51%  
- Sharpe ratio: -0.03  
- Max drawdown: -21.98%  
- RSI status: Normal  
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/28a70479-71ad-47ef-8bd7-8c62e1b70038" />

**^GSPC (S&P 500)**
- Annual return: 11.48%  
- Annual volatility: 16.86%  
- Sharpe ratio: 0.68  
- Max drawdown: -27.11%  
- RSI status: Unknown  
(Where current price or 20-day change is not available, the dashboard can hide the value or label it as “Not available in current snapshot”.)
<img width="1593" height="956" alt="image" src="https://github.com/user-attachments/assets/07fc9d2d-04c6-4d62-9d1c-931ec029e9fe" />

## 11. Dashboard Overview
The Streamlit dashboard (`dashboard/app.py`) provides an interactive interface including:
- Market overview summary  
- Normalized price comparison across all assets  
- Risk–return scatter plots  
- Correlation heatmaps  
- Market regime timeline and distribution  
- Rolling Sharpe, volatility, and drawdowns  
- VaR / CVaR and tail-risk visualizations  
- Asset rankings and performance tables  
- RSI and return-distribution plots  

## 12. Tech Stack
- Python  
- yfinance  
- pandas, numpy  
- scipy, statsmodels  
- matplotlib, seaborn  
- streamlit
