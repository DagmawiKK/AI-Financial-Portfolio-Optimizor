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