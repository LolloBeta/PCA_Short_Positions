# S&P 500 Correlation Analysis: Short Positions in PCA Portfolios

Analysis of S&P 500 return correlations and classification of correlation matrices based on short positions in the first principal component, following the methodology from "Short Positions in the First Principal Component Portfolio".

## Overview

This project investigates when and why Principal Component Analysis (PCA) portfolios require short positions. We analyze S&P 500 data across four periods (1994-2013) to understand the relationship between correlation structure and portfolio composition.

### Key Concepts

**PCA Portfolio**: The first principal component (PC1) represents the direction of maximum variance in stock returns. The PCA portfolio weights are derived from the first eigenvector v₁:

```
w_PCA = v₁ / (1ᵀv₁)
```

**Correlation Matrix Classes**:
- **Class I**: All correlations positive → v₁ > 0 (Perron-Frobenius theorem) → Long-only portfolio
- **Class II**: Some negative correlations, but v₁ ≥ 0 → Long-only portfolio  
- **Class III**: v₁ contains negative elements → **Short positions required**

**MNC (Majority Negative Correlation)**: A stock is MNC if it's negatively correlated with the majority of other stocks. MNC stocks drive the appearance of Class III matrices and short positions.

## Project Structure

```
.
├── build_dataset.py           # Downloads S&P 500 data from Yahoo Finance
├── analyze_correlations.py    # Analyzes correlations and classifies matrices
├── sp500_intervals.csv         # S&P 500 membership intervals (input)
└── data/                       # Output directory
    ├── sp500_prices_long.csv      # Long format: date, ticker, OHLCV
    ├── sp500_adj_close_wide.csv   # Wide format: date × tickers
    └── logs/
        └── failed_tickers.txt
```

## Requirements

```bash
pip install pandas numpy yfinance tqdm matplotlib seaborn scipy
```

## Usage

### 1. Build Dataset

Download S&P 500 historical data based on membership intervals:

```bash
python build_dataset.py --intervals sp500_intervals.csv --outdir ./data
```

**Options**:
- `--start YYYY-MM-DD`: Force global start date
- `--end YYYY-MM-DD`: Force global end date  
- `--retry N`: Number of download retries (default: 2)
- `--sleep S`: Sleep time between requests in seconds (default: 0.5)

**Outputs**:
- `sp500_prices_long.csv`: Long format with all OHLCV data
- `sp500_adj_close_wide.csv`: Wide format (dates × tickers) for analysis

### 2. Analyze Correlations

Classify correlation matrices and generate statistics:

```bash
python analyze_correlations.py --data ./data/sp500_adj_close_wide.csv
```

**Options**:
- `--no-plots`: Skip heatmap generation

**Outputs**:
- Correlation heatmaps (PNG files) for each period
- Three summary tables printed to console

## Analysis Parameters

- **Coverage threshold**: 80% minimum valid observations per stock
- **Submatrix size**: 40 stocks randomly sampled
- **Samples per period**: 60 independent samples
- **Analysis periods**: 
  - 1994-1998 (Pre dot-com)
  - 1999-2003 (Dot-com bubble & burst)
  - 2004-2008 (Financial crisis)
  - 2009-2013 (Recovery)

## Results

### Expected Outputs

**Table 1**: Percentage of negative correlations in full correlation matrix per period

**Table 2**: Classification counts (Class I, II, III) for 60 submatrices per period

**Table 3**: Mean statistics for Classes II and III:
- Mean % of negative correlations
- Mean magnitude of negative correlations

### Key Findings

From empirical analysis (1994-2013):

| Period | Class I | Class II | Class III | Observations |
|--------|---------|----------|-----------|--------------|
| 1994-1998 | 4 | 34 | 22 | Pre-crisis stability |
| 1999-2003 | 2 | 33 | 25 | Dot-com volatility |
| 2004-2008 | 11 | 19 | 30 | **Peak instability** |
| 2009-2013 | 54 | 6 | 0 | Strong recovery |

**Interpretation**:
- Class III matrices (short positions) emerge primarily during market instability (1999-2008)
- Recovery period (2009-2013) shows almost no Class III cases
- Short positions arise from MNC stocks that behave oppositely to the market

## References

Based on the paper methodology analyzing PCA as an alternative to Markowitz mean-variance optimization, particularly relevant for institutional investors (pension funds, insurance companies) with short-selling constraints.
