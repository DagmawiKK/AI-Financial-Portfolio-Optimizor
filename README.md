# Time Series Forecasting for Portfolio Management Optimization

This repository implements a two-stage workflow for data-driven portfolio management:
- Task 1: Data Preprocessing and Exploration
- Task 2: Time Series Forecasting Models (ARIMA vs. LSTM) and 12‑month TSLA forecast

The outputs from Task 1 feed Task 2, and Task 2’s best model forecast will be used by subsequent portfolio construction and optimization.

---

## Project Structure

```
financial_portfolio_optimization/
├── data/
│   ├── raw/
│   │   └── financial_data.csv                 # Raw data from yfinance
│   ├── processed/
│   │   ├── adj_close.csv                      # Cleaned Adjusted Close prices (Task 1)
│   │   └── tsla_12month_forecast.csv          # 12-month TSLA forecast (Task 2)
│   └── interim/
├── notebooks/
│   ├── EDA_and_Preprocessing.ipynb            # Detailed Task 1 EDA
│   └── Forecasting_Models.ipynb               # Task 2 model training & analysis
├── scripts/
│   ├── data_ingestion.py                      # Task 1 data fetch & processing
│   └── model_training.py                      # Task 2 ARIMA & LSTM training/evaluation
|
├── reports/
│   └── figures/
│       ├── Historical Adj Close prices.jpg           # Historical Adj Close (Task 1)
│       ├── distribution of daily returns.jpg                         # Returns distributions (Task 1)
│       ├── 30 day rolling standard deviation.jpg                         # Rolling mean & volatility (Task 1)
│       ├── tsla_test_forecast_vs_actual.png   # ARIMA/LSTM vs actuals on test set (Task 2)
│       └── tsla_future_forecast_12m.png       # 12‑month TSLA forecast (Task 2)
├── README.md
└── requirements.txt
```

---

## Environment and Requirements

- Python 3.8+
- Install dependencies:
  ```
  pip install pandas numpy matplotlib seaborn scikit-learn yfinance statsmodels tensorflow
  ```
- Optional: Jupyter for notebooks.

---

## How to Run

1) Task 1 — Data ingestion and preprocessing
- From the project root:
  ```
  python scripts/data_ingestion.py
  ```
- Outputs:
  - data/raw/financial_data.csv
  - data/processed/adj_close.csv

2) Task 2 — Model training, evaluation, and forecasting
- From the project root:
  ```
  python scripts/model_training.py
  ```
- Outputs:
  - models/lstm_tsla_forecast_model.keras
  - data/processed/tsla_12month_forecast.csv
  - reports/figures/tsla_test_forecast_vs_actual.png
  - reports/figures/tsla_future_forecast_12m.png

3) Optional (reproduce figures and analysis interactively)
- Open the notebooks and run cells in order:
  ```
  jupyter notebook notebooks/EDA_and_Preprocessing.ipynb
  jupyter notebook notebooks/Forecasting_Models.ipynb
  ```

---

## Task 1: Data Preprocessing and Exploration (Summary)

Objective
- Extract, clean, and understand historical financial data for TSLA, BND, and SPY to support forecasting and optimization.

Data
- Source: yfinance
- Tickers: TSLA, BND, SPY
- Period: 2015-07-01 to 2025-07-31
- Focus: Adjusted Close prices, daily frequency

Key Steps
- Cleaned MultiIndex raw structure, extracted Adjusted Close columns.
- Ensured aligned trading dates and correct dtypes.
- Verified no missing values in adj_close.csv for the study window.

Exploratory Findings
- Price trends:
  - TSLA: strong growth, especially post‑2020.
  - SPY: steady upward trend.
  - BND: stable profile, characteristic of bond exposure.
- Returns distribution:
  - TSLA widest dispersion (highest volatility), SPY moderate, BND narrowest.
- Rolling volatility (30‑day std):
  - TSLA highest and most variable; SPY moderate; BND consistently low.
- Outliers:
  - All assets show extremes near March 2020; TSLA has most pronounced swings.

Stationarity
- Prices: non‑stationary (ADF p > 0.05).
- Daily returns: stationary (ADF p < 0.05).

Risk Metrics
- Daily 95% VaR:
  - BND: -0.0049
  - SPY: -0.0172
  - TSLA: -0.0547
- Annualized Sharpe (rf = 0%):
  - BND: 0.3569
  - SPY: 0.7941
  - TSLA: 0.7783

---

## Task 2: Develop Time Series Forecasting Models

Objective
- Build, evaluate, and compare at least two models—ARIMA (classical) and LSTM (deep learning)—for TSLA daily Adjusted Close forecasting, then produce a 12‑month (≈252 trading days) future forecast using the best model.

Data Split
- Train: 2015-07-01 to 2023-12-31
- Test: 2024-01-01 to 2025-07-31
- Chronological split with no shuffling.

Models

1) ARIMA (statsmodels)
- Specification: ARIMA(5, 1, 0) on price series with first differencing (d=1).
- Rationale: address non‑stationarity by differencing.
- Forecasting: generated dynamic predictions over the test window.

2) LSTM (TensorFlow/Keras)
- Scaling: MinMaxScaler to [0, 1] on TSLA prices.
- Windowing: sequence_length = 60 (use prior 60 days to predict next day).
- Architecture:
  - LSTM layer
  - Dropout
  - LSTM layer
  - Dropout
  - Dense output (1 unit)
- Training:
  - Loss: mean_squared_error
  - Optimizer: adam
  - EarlyStopping to curb overfitting
- Multi‑step future forecast:
  - Retrained on full history (train + test).
  - Recursive strategy to predict 252 trading days beyond last date.
- Inverse scaling applied before metric computation and saving outputs.

Evaluation Metrics (Test Set: 2024‑01 to 2025‑07)
- ARIMA:
  - MAE: 62.97
  - RMSE: 77.99
  - MAPE: 24.08%
  - Behavior: relatively flat forecast from the last training point; limited ability to capture TSLA’s strong trend and volatility under this specification.
- LSTM:
  - MAE: 11.33
  - RMSE: 15.93
  - MAPE: 4.00%
  - Behavior: closely tracks trend and volatility; markedly lower forecast errors.

Key Findings
- LSTM significantly outperformed ARIMA on all metrics for the test period, reflecting its capacity to learn non‑linear dynamics and long‑range dependencies in TSLA’s price behavior.
- The chosen LSTM model was retrained on the full historical dataset and used to produce a 12‑month forecast.

12‑Month TSLA Forecast (from last observed price ≈ $314)
- Direction: pronounced downward trajectory toward ≈ $144 across the 12‑month horizon.
- Volatility: initially evident, with smoothing over longer horizons.
- This bearish outlook will be a primary input to the portfolio optimization phase (e.g., tilting allocations, hedging considerations, scenario analysis).

Deliverables (Task 2)
- models/lstm_tsla_forecast_model.keras — saved Keras model
- data/processed/tsla_12month_forecast.csv — dates, predicted prices (inverse‑scaled)
- reports/figures/tsla_test_forecast_vs_actual.png — test window comparison
- reports/figures/tsla_future_forecast_12m.png — 12‑month projection

Reproducibility Notes
- Train/test split is time‑based and deterministic.
- EarlyStopping may cause small run‑to‑run differences unless seeds are fixed.
- yfinance may revise historical data; re‑runs can yield minor deviations.

---

## Design Choices and Considerations

- Modeling prices vs returns:
  - ARIMA on differenced prices is simple, but for assets like TSLA with regime shifts, richer models or exogenous signals can help.
  - LSTM benefits from longer lookbacks and non‑linear mapping but requires careful scaling and validation.
- Annualization assumptions:
  - 252 trading days/year.
  - Sharpe Ratio computed with rf = 0% (can be parameterized later).
- Risk and performance:
  - Combine point forecasts with uncertainty estimates and risk metrics for robust portfolio decisions (future work).

---

## Next Steps

- Uncertainty modeling (prediction intervals via bootstrapping or Monte Carlo with LSTM residuals).
- Incorporate exogenous variables (macro factors, indices, market sentiment) via SARIMAX or multivariate deep models.
- Portfolio optimization using forecasted returns/risk (mean‑variance, Black‑Litterman, risk‑parity), plus sensitivity and stress testing.
- Model risk governance: backtesting over rolling windows, stability and drift monitoring, and scheduled retraining.

---