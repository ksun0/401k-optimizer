# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a 401(k) Portfolio Optimizer built for the 2022 Microsoft Hackathon. The tool uses Modern Portfolio Theory (MPT) to recommend optimal asset allocations across mutual funds available in 401(k) plans, maximizing risk-adjusted returns based on the efficient frontier.

The project is implemented entirely in Jupyter notebooks using Python for financial data analysis, LSTM-based price forecasting, and portfolio optimization.

## Architecture

### Data Pipeline

1. **Data Scraping** (Step 1): Historical price data is fetched from Yahoo Finance using `yfinance` for various fund categories
2. **Machine Learning Prediction** (Step 2): LSTM models forecast future fund prices based on historical data
3. **Portfolio Optimization**: Scipy optimization (`scipy.optimize`) calculates efficient frontier and identifies maximum Sharpe ratio portfolios

### Fund Categories

The codebase organizes mutual funds into categories:

- **Blended funds**: Target-date funds (e.g., LINIX, LIBIX, STLFX)
- **Large-cap stocks**: VRGWX, VRVIX, FCNTX, FDGRX
- **Mid-cap stocks**: ARTMX
- **Small-cap stocks**: VRTGX, SEVAX
- **International stocks**: FIGFX, FIVLX
- **Bonds**: PTTRX, VBIPX
- **Short-term**: BTCFX

Additional fund collections include HSA funds, Vanguard funds, HealthEquity funds, Roth IRA funds, and experimental swing trading tickers.

### Core Functions

**`fund_pred(data, fundname, n_lookback, n_forecast, agg=False)`** (2022Hack_401K_Opt.ipynb:cell-12)

- LSTM-based price forecasting for individual funds
- Parameters:
  - `n_lookback`: Historical window size (typically 60 days)
  - `n_forecast`: Forecast horizon (typically 7 days)
  - `agg`: Whether to aggregate data weekly
- Returns dataframe with actual prices, predictions, and forecasts

**`portfolio_annualized_performance(weights, mean_returns, cov_matrix)`** (kevin.ipynb:cell-19)

- Calculates annualized returns and volatility for a given portfolio weight allocation
- Returns: (standard_deviation, returns)

**`random_portfolios(num_portfolios, mean_returns, cov_matrix, risk_free_rate)`** (kevin.ipynb:cell-19)

- Monte Carlo simulation generating random portfolio allocations
- Returns portfolio metrics (std dev, returns, Sharpe ratio) and weights

**`max_sharpe_ratio(mean_returns, cov_matrix, risk_free_rate)`** (kevin.ipynb:cell-24)

- Uses scipy SLSQP optimizer to find portfolio weights that maximize the Sharpe ratio
- Constraint: weights sum to 1, each weight between 0 and 1

**`min_variance(mean_returns, cov_matrix)`** (kevin.ipynb:cell-24)

- Finds minimum volatility portfolio allocation

**`efficient_frontier(mean_returns, cov_matrix, returns_range)`** (kevin.ipynb:cell-24)

- Calculates points along the efficient frontier curve

**`display_calculated_ef_with_random(...)`** (kevin.ipynb:cell-24)

- Master visualization function that:
  - Runs Monte Carlo simulation
  - Calculates max Sharpe ratio and min variance portfolios
  - Plots efficient frontier with highlighted optimal portfolios
  - Prints portfolio allocation recommendations

## Development Commands

### Environment Setup

```bash
pip install -r requirements.txt
```

Key dependencies: pandas, numpy, yfinance, matplotlib, seaborn, keras, scikit-learn, scipy

### Running Analysis

Start Jupyter:

```bash
jupyter notebook
```

Primary notebooks:

- `kevin.ipynb`: Main portfolio optimization analysis (10-year lookback, efficient frontier)
- `2022Hack_401K_Opt.ipynb`: Original hackathon submission with LSTM forecasting (2-year lookback)

### Typical Analysis Workflow

1. Set date range (adjust `beg` and `end` variables)
2. Download fund data using `yf.download()`
3. Calculate returns and covariance matrix
4. Run `display_calculated_ef_with_random()` to get optimal allocations
5. Interpret results: maximum Sharpe ratio portfolio shows recommended allocation percentages

### Key Parameters

**Risk-free rate**: Currently set to 3.10% (10-Year Treasury Note yield) in `kevin.ipynb:cell-19`

**Lookback periods**:

- 401(k) analysis: 5-10 years
- Swing trading: 12 months
- LSTM training split: 70% train, 30% test

**Monte Carlo simulations**: 5,000,000 portfolios generated for visualization

## Important Notes

- The notebooks contain multiple ticker lists (HSA, Vanguard, HealthEquity, Roth IRA). To switch between them, uncomment the relevant `all_df` and `all_ticks` assignments in kevin.ipynb:cell-11
- LSTM models are trained with 1 epoch and batch_size=1 for demonstration purposes
- Standard scaling is applied for visualization but NOT for optimization calculations
- The efficient frontier optimization assumes no transaction costs or taxes
- Results should be backtested before real investment decisions
- Certain funds like BTCFX (Bitcoin Strategy) have higher volatility due to crypto exposure

## Data Cleaning

Some funds have missing data (NaN values). The analysis:

- Uses `fillna(method='bfill')` for LSTM training data
- Drops NaN rows with `dropna()` for portfolio optimization
- Filters correlated funds to reduce redundancy (e.g., VRVIX and VSPVX track each other)

## Output Interpretation

The optimization outputs two key portfolios:

1. **Maximum Sharpe Ratio Portfolio**: Best risk-adjusted returns
2. **Minimum Volatility Portfolio**: Lowest risk

Each shows:

- Annualized Return (e.g., 0.16 = 16%)
- Annualized Volatility (standard deviation)
- Allocation percentages for each fund
