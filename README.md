# Time Series Forecasting for Portfolio Management Optimization

A three-stage pipeline to: (1) prepare and understand market data, (2) develop and compare forecasting models (ARIMA vs. LSTM), and (3) produce a 12‑month Tesla (TSLA) price forecast for downstream portfolio decisions.

- Task 1: Data Preprocessing and Exploration
- Task 2: Time Series Forecasting Models (ARIMA vs. LSTM) and model selection
- Task 3: Forecast Future Market Trends (12‑month TSLA forecast using the best model)

---

## Project Structure (key paths)

```
financial_portfolio_optimization/
├─ data/
│  ├─ raw/
│  │  └─ financial_data.csv                 # Raw yfinance dump
│  ├─ processed/
│  │  ├─ adj_close.csv                      # Clean Adjusted Close (Task 1)
│  │  └─ tsla_12month_forecast.csv          # 12‑month TSLA forecast (Tasks 2/3)
│  └─ interim/
├─ notebooks/
│  ├─ EDA_and_Preprocessing.ipynb           # Task 1 details & plots
|  ├─ Forecast_future_market_trends.ipynb   # task 3
│  └─ Forecasting_Models.ipynb              # Tasks 2/3 modeling & analysis
├─ scripts/
│  ├─ data_ingestion.py                     # Task 1 core script
│  └─ model_training.py                     # Tasks 2/3 ARIMA & LSTM + forecasting
├─ models/
│  └─ lstm_tsla_forecast_model.keras        # Saved LSTM (best model)
├─ reports/
│  └─ figures/
│     ├─ image1.jpg                         # Historical Adj Close (Task 1)
│     ├─ image2.jpg                         # Returns distributions (Task 1)
│     ├─ image3.jpg                         # Rolling mean & volatility (Task 1)
│     ├─ tsla_test_forecast_vs_actual.png   # Test-set comparison (Task 2)
│     └─ tsla_future_forecast_12m.png       # 12‑month forecast (Task 3)
├─ README.md
└─ requirements.txt
```

---

## Environment

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

2) Tasks 2 & 3 — Model training, evaluation, and 12‑month forecast
- From the project root:
  ```
  python scripts/model_training.py
  ```
- What it does:
  - Loads adj_close.csv.
  - Trains and evaluates ARIMA and LSTM on TSLA (time-based split).
  - Selects best model by MAE, RMSE, MAPE.
  - Retrains best model (LSTM) on full history and generates a 12‑month forecast (~252 trading days).
- Outputs:
  - models/lstm_tsla_forecast_model.keras
  - data/processed/tsla_12month_forecast.csv
  - reports/figures/tsla_test_forecast_vs_actual.png
  - reports/figures/tsla_future_forecast_12m.png

3) Optional notebooks
- Reproduce analysis and figures interactively:
  ```
  jupyter notebook notebooks/EDA_and_Preprocessing.ipynb
  jupyter notebook notebooks/Forecasting_Models.ipynb
  ```

---

## Task 1 — Data Preprocessing and Exploration (Summary)

Data
- Tickers: TSLA (Tesla), BND (Vanguard Total Bond Market ETF), SPY (S&P 500 ETF)
- Source: yfinance
- Period: 2015‑07‑01 to 2025‑07‑31
- Frequency: Daily (trading days)
- Focus: Adjusted Close prices

Processing
- Loaded multi-index raw data; extracted and flattened Adjusted Close for each ticker.
- Aligned trading calendars and validated dtypes.
- Result: data/processed/adj_close.csv with no missing values in the study window.

EDA Highlights
- Price trends:
  - TSLA: pronounced post‑2020 growth.
  - SPY: steady market-wide uptrend.
  - BND: stable bond-like behavior.
- Daily returns distribution:
  - TSLA widest (highest volatility), SPY moderate, BND narrowest.
- 30‑day rolling volatility:
  - TSLA highest and most variable; SPY moderate; BND consistently low.
- Outliers:
  - Notable extremes around March 2020 (COVID‑19 shock), strongest in TSLA.

Stationarity (ADF)
- Prices: non‑stationary (p > 0.05).
- Daily returns: stationary (p < 0.05).

Risk Metrics (daily unless noted)
- 95% VaR: BND −0.0049, SPY −0.0172, TSLA −0.0547.
- Sharpe Ratio (annualized, rf = 0%): BND 0.3569, SPY 0.7941, TSLA 0.7783.

---

## Task 2 — Model Development and Selection (TSLA)

Objective
- Build, evaluate, and compare a classical ARIMA model and a deep LSTM model for TSLA daily Adjusted Close; select the best by error metrics, then prepare for the 12‑month forecast.

Data Split
- Train: 2015‑07‑01 to 2023‑12‑31
- Test: 2024‑01‑01 to 2025‑07‑31

Models

- ARIMA (statsmodels)
  - Spec: ARIMA(5, 1, 0) on prices (d=1 to address non‑stationarity).
  - Forecasting: dynamic predictions over the test set.

- LSTM (TensorFlow/Keras)
  - Scaling: MinMaxScaler to [0, 1] on prices.
  - Windowing: sequence_length = 60 (use prior 60 days to predict the next day).
  - Architecture: LSTM → Dropout → LSTM → Dropout → Dense(1).
  - Training: loss = MSE, optimizer = Adam, EarlyStopping to mitigate overfitting.
  - Inference: inverse scaling for metrics and plots.

Performance (Test: 2024‑01 to 2025‑07)
- ARIMA: MAE 62.97, RMSE 77.99, MAPE 24.08% (relatively flat bias from last training point).
- LSTM: MAE 11.33, RMSE 15.93, MAPE 4.00% (captures trend and volatility more effectively).

Decision
- LSTM selected as best-performing model across all evaluation metrics.

Artifacts
- Saved model: models/lstm_tsla_forecast_model.keras
- Figure: reports/figures/tsla_test_forecast_vs_actual.png

---

## Task 3 — Forecast Future Market Trends (12‑Month TSLA Outlook)

Objective
- Use the best model (LSTM) to produce a 12‑month TSLA price forecast and assess potential trends and risks.

Methodology
- Retrained the LSTM on the full history (2015‑07 to 2025‑07) to exploit all information.
- Employed recursive multi‑step forecasting: each predicted day is appended to the input window to predict the next, iterated for ~252 trading days.
- Applied inverse scaling to return price‑level forecasts; saved with calendar dates.

Key Findings
- Trend: Forecast projects a sharp downward trajectory from approximately $314 to about $144 over 12 months.
- Volatility: Early horizon reflects recent variability; longer horizon smooths—common in recursive forecasts as errors propagate.
- Implications: Indicates notable downside risk. Could inform hedging or tactical short exposure for high risk‑tolerant investors, with strong caveats on uncertainty and regime sensitivity.

Deliverables
- data/processed/tsla_12month_forecast.csv
- reports/figures/tsla_future_forecast_12m.png

---

## Reproducibility and Notes

- Time-based split with no shuffling ensures causality.
- EarlyStopping and random initializations can introduce minor run‑to‑run variance; set seeds for stricter reproducibility.
- yfinance may revise historical data; minor differences can occur across runs.
- Annualization assumes 252 trading days; Sharpe uses rf = 0%.

---

## Next Steps

- Add uncertainty estimates (prediction intervals via residual/bootstrapped simulation or quantile approaches).
- Incorporate exogenous features (macro factors, indices) with SARIMAX or multivariate deep models.
- Feed forecasts into portfolio optimization (mean‑variance, Black‑Litterman, risk‑parity) with constraints and transaction costs.

---

## Disclaimer

For research and educational purposes only; not investment advice. Forecasts are uncertain and may be materially wrong, especially over long horizons or regime changes.