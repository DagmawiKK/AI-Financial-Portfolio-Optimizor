# Time Series Forecasting for Portfolio Management Optimization

End‑to‑end workflow to (1) preprocess and explore market data, (2) develop and compare forecasting models (ARIMA vs. LSTM), (3) produce a 12‑month TSLA forecast, and (4) construct an optimized portfolio informed by the forecast.

- Task 1: Data Preprocessing and Exploration
- Task 2: Develop Time Series Forecasting Models (ARIMA vs. LSTM) and model selection
- Task 3: Forecast Future Market Trends (12‑month TSLA outlook)
- Task 4: Optimize Portfolio Based on Forecast (Efficient Frontier, key portfolios, recommendation)

---

## Project Structure

```
financial_portfolio_optimization/
├─ data/
│  ├─ raw/
│  │  └─ financial_data.csv                  # Raw yfinance data (Task 1 input)
│  ├─ processed/
│  │  ├─ adj_close.csv                       # Clean Adjusted Close (Task 1 output → Tasks 2/4 input)
│  │  └─ tsla_12month_forecast.csv           # 12‑month TSLA forecast (Task 3 output → Task 4 input)
│  └─ interim/
├─ models/
│  └─ lstm_tsla_forecast_model.keras         # Trained LSTM (Tasks 2/3 output)
├─ notebooks/
│  ├─ EDA_and_Preprocessing.ipynb            # Task 1 details & plots
│  ├─ Forecasting_Models.ipynb               # Tasks 2/3 modeling, evaluation & forecast
│  └─ Portfolio_Optimization.ipynb           # Task 4 optimization & figures
├─ scripts/
│  ├─ data_ingestion.py                      # Task 1: fetch & clean data
│  ├─ model_training.py                      # Tasks 2/3: ARIMA & LSTM + forecast
│  └─ portfolio_analysis.py                  # Task 4: MPT simulation & outputs
├─ reports/
│  └─ figures/                               # EDA, model, and optimization charts
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
  or
  ```
  pip install -r requirements.txt
  ```

---

## How to Run

1) Task 1 — Data ingestion and preprocessing
- From project root:
  ```
  python scripts/data_ingestion.py
  ```
- Outputs:
  - data/raw/financial_data.csv
  - data/processed/adj_close.csv

2) Tasks 2 & 3 — Model training, evaluation, and 12‑month forecast
- From project root:
  ```
  python scripts/model_training.py
  ```
- Outputs:
  - models/lstm_tsla_forecast_model.keras
  - data/processed/tsla_12month_forecast.csv
  - reports/figures/tsla_test_forecast_vs_actual.png
  - reports/figures/tsla_future_forecast_12m.png

3) Task 4 — Portfolio optimization
- From project root:
  ```
  python scripts/portfolio_analysis.py
  ```
- Outputs:
  - Console summary of key portfolios and recommended weights
  - Efficient Frontier plots saved under reports/figures/

4) Optional notebooks (interactive reproduction)
```
jupyter notebook notebooks/EDA_and_Preprocessing.ipynb
jupyter notebook notebooks/Forecasting_Models.ipynb
jupyter notebook notebooks/Portfolio_Optimization.ipynb
```

---

## Task 1 — Data Preprocessing and Exploration (Summary)

Data
- Tickers: TSLA, BND, SPY
- Source: yfinance
- Period: 2015‑07‑01 to 2025‑07‑31
- Focus: Adjusted Close (daily)

Processing and EDA
- Extracted and flattened Adjusted Close from MultiIndex raw; validated dtypes and aligned trading days.
- No missing values in adj_close.csv for the study window.
- Price trends: TSLA strong post‑2020 growth; SPY steady uptrend; BND stable.
- Returns distribution: TSLA widest (highest volatility); SPY moderate; BND narrowest.
- 30‑day rolling volatility: TSLA highest/most variable; SPY moderate; BND consistently low.
- Outliers: Large daily return shocks around March 2020 across assets.

Stationarity (ADF)
- Prices: non‑stationary (p > 0.05).
- Daily returns: stationary (p < 0.05).

Risk Metrics
- Daily 95% VaR: BND −0.0049; SPY −0.0172; TSLA −0.0547.
- Annualized Sharpe (rf = 0%): BND 0.3569; SPY 0.7941; TSLA 0.7783.

---

## Task 2 — Develop Time Series Forecasting Models (TSLA)

Objective
- Build, evaluate, and compare ARIMA (classical) and LSTM (deep learning) models for TSLA daily Adjusted Close; select the best by MAE, RMSE, and MAPE.

Methodology
- Split: Train (2015‑07–2023‑12), Test (2024‑01–2025‑07).
- ARIMA (statsmodels):
  - ARIMA(5,1,0) on prices (d=1 to difference once).
  - Produced dynamic forecasts over the test period.
- LSTM (TensorFlow/Keras):
  - MinMax scaling to [0,1]; 60‑day rolling window to predict next day.
  - Architecture: LSTM → Dropout → LSTM → Dropout → Dense(1).
  - Loss: MSE; Optimizer: Adam; EarlyStopping to mitigate overfitting.
  - Inverse scaling for evaluation and plots.

Performance (Test)
- ARIMA: MAE 62.97; RMSE 77.99; MAPE 24.08% (relatively flat bias).
- LSTM: MAE 11.33; RMSE 15.93; MAPE 4.00% (tracks trend and volatility well).

Decision
- LSTM selected as the best‑performing model.

Artifacts
- models/lstm_tsla_forecast_model.keras
- reports/figures/tsla_test_forecast_vs_actual.png

---

## Task 3 — Forecast Future Market Trends (12‑Month TSLA Outlook)

Objective
- Use the LSTM model to generate a 12‑month TSLA price forecast and assess trends, volatility, and risk.

Methodology
- Retrained LSTM on full history (2015‑07–2025‑07).
- Recursive multi‑step forecast for ~252 trading days; each prediction feeds the next.
- Inverse scaling to price levels; dates aligned and saved.

Key Findings
- Trend: Downward trajectory from roughly $314 to about $144 over 12 months.
- Volatility: Early horizon reflects recent swings; longer horizon smooths (typical of recursive forecasts).
- Risks & opportunities: Indicates notable downside risk; tactical short/hedging may be considered, with strong caveats on long‑horizon uncertainty and regime shifts.

Deliverables
- data/processed/tsla_12month_forecast.csv
- reports/figures/tsla_future_forecast_12m.png

---

## Task 4 — Optimize Portfolio Based on Forecast (TSLA, BND, SPY)

Objective
- Apply MPT to construct an efficient portfolio using the TSLA forecast (Task 3) and historical characteristics of BND/SPY.

Methodology
- Expected returns:
  - TSLA: Mean of the 12‑month forecast (−0.5400 annualized assumption).
  - BND: Historical average daily return annualized (0.0196).
  - SPY: Historical average daily return annualized (0.1448).
- Risk model:
  - Annualized covariance matrix from historical daily returns (TSLA, BND, SPY).
- Simulation:
  - 50,000 random portfolios; compute expected return, volatility, and Sharpe Ratio (rf = 0%).
- Identification:
  - Maximum Sharpe Ratio Portfolio and Minimum Volatility Portfolio.

Key Findings
- Efficient Frontier: Clear trade‑off curve between risk and expected return.
- Maximum Sharpe Ratio Portfolio:
  - Return 0.1394; Volatility 0.5704; Sharpe 0.2445
  - Weights: TSLA 0.0000; BND 0.0429; SPY 0.9571
  - Interpretation: Best risk‑adjusted return; excludes TSLA given negative expected return.
- Minimum Volatility Portfolio:
  - Return −0.5053; Volatility 0.0540; Sharpe −9.3586
  - Weights: TSLA 0.9381; BND 0.0619; SPY 0.0001
  - Interpretation: Lowest variance but highly negative expected return due to TSLA allocation.

Recommendation
- Adopt the Maximum Sharpe Ratio Portfolio for efficient risk‑adjusted growth, primarily allocating to SPY with a small BND position and excluding TSLA given the bearish forecast.

Artifacts
- Console/CSV summary of portfolio stats (if implemented).
- Efficient Frontier figures in reports/figures/.

---

## Reproducibility & Assumptions

- Time‑based split; no shuffling.
- Annualization assumes 252 trading days; Sharpe uses rf = 0%.
- EarlyStopping and random seeds introduce minor run‑to‑run variance; set seeds for stricter reproducibility.
- yfinance data can be revised; minor differences may appear across runs.

---