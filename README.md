# AI-Financial-Portfolio-Optimization

# Time Series Forecasting for Portfolio Management Optimization

## Task 1: Data Preprocessing and Exploration

This document details the first phase of the **Time Series Forecasting for Portfolio Management Optimization** project: Data Preprocessing and Exploration. The objective of this task was to load, clean, and understand the historical financial data for selected assets in preparation for subsequent forecasting and portfolio optimization.

---

## Objective

The primary goals for Task 1 were to:

- **Extract historical financial data** for specified assets using `yfinance`.
- **Perform essential data cleaning**, including handling missing values and ensuring correct data types.
- **Conduct Exploratory Data Analysis (EDA)** to visualize trends, patterns, and volatility.
- **Analyze stationarity** of the time series data using statistical tests.
- **Calculate foundational risk metrics** to understand potential losses and risk-adjusted returns.

---

## Data Used

Historical financial data for the following three key assets was utilized:

- **TSLA (Tesla):** High-growth, high-risk stock in the consumer discretionary sector.
- **BND (Vanguard Total Bond Market ETF):** Provides stability and income from U.S. investment-grade bonds.
- **SPY (S&P 500 ETF):** Offers broad U.S. market exposure with diversified, moderate risk.

**Data Source:** [yfinance](https://github.com/ranaroussi/yfinance)  
**Period Covered:** July 1, 2015 – July 31, 2025

---

## 🛠️ Methodology and Findings

### 1. Data Extraction and Cleaning

- Downloaded data for `'Adj Close'` prices, `'Volume'`, `'Open'`, `'High'`, and `'Low'` using `yfinance`.
- Processed initial MultiIndex columns to extract `'Adj Close'` prices, resulting in a cleaner DataFrame.
- Checked for missing values: **No missing values found in 'Adj Close' prices after initial load.**

### 2. Exploratory Data Analysis (EDA)

#### Historical Price Trends:
- **TSLA:** Significant upward growth, especially post-2020, confirming its high-growth nature.
- **SPY:** Steady upward trend, indicating overall U.S. market growth.
- **BND:** Relatively stable, consistent with bond ETF characteristics.

#### Daily Returns Distribution:
- **TSLA:** Widest daily returns distribution, indicating higher volatility.
- **SPY:** Moderate risk with a narrower distribution.
- **BND:** Lowest volatility and narrowest distribution.

#### Rolling Statistics (Volatility):
- **TSLA:** Highest and most volatile rolling standard deviation, especially after 2020.
- **SPY:** Moderate fluctuations in rolling volatility.
- **BND:** Consistently low and stable rolling volatility.

#### Outlier Detection:
- Significant outliers (extreme returns) observed across all assets, notably around March 2020 (COVID-19 market impact).
- **TSLA** displayed the most extreme positive and negative daily returns.

### 3. Stationarity Analysis (Augmented Dickey-Fuller Test)

- **Price Series:** TSLA, BND, and SPY were all **non-stationary** (_p-value > 0.05_), indicating trends or seasonality.
- **Daily Returns Series:** TSLA, BND, and SPY were all **stationary** (_p-value < 0.05_), allowing direct application of ARIMA and similar models.

### 4. Risk Metrics

**Value at Risk (VaR) at 95% Confidence:**
- **BND:** -0.0049 _(lowest risk)_
- **SPY:** -0.0172 _(moderate risk)_
- **TSLA:** -0.0547 _(highest risk)_

**Sharpe Ratio (Annualized, 0% risk-free rate):**
- **BND:** 0.357
- **SPY:** 0.794
- **TSLA:** 0.778

_SPY provided the best risk-adjusted return, followed closely by TSLA, while BND offered lower returns commensurate with its lower risk._

---

## Conclusion

Task 1 has successfully laid the groundwork for the next stages of the project. The data is cleaned, its characteristics are well understood through EDA, its stationarity properties have been assessed, and risk metrics provide a foundational understanding of each asset's performance. These insights are crucial for selecting appropriate forecasting models and for effective portfolio optimization.

---