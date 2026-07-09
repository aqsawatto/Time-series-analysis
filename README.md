# Forecasting German Weekly Electricity Demand

A single, reproducible notebook that forecasts German electricity demand two years
ahead. Demand is expressed as weekly energy in terawatt-hours, and eight models are
compared against a seasonal-naive baseline.

## Aim

Forecast the final 104 weeks of German demand and judge whether a SARIMA, a
weather-driven SARIMAX, a random forest, or a Conv1D-LSTM can beat the seasonal
naive, using RMSE and a demand-weighted MAPE.

## Data

- **Load:** hourly German load (`DE_load_actual_entsoe_transparency`) from Open
  Power System Data, January 2015 to October 2020, summed to weekly energy (TWh)
  and daily totals (GWh).
- **Temperature:** daily Berlin temperature from the Open-Meteo archive, turned
  into a heating-degree feature whose base (12 &#176;C) is chosen on the training
  data. Because the test window uses observed temperature, the weather-driven
  forecasts are conditional.
- **Calendar:** a holiday-week indicator and Fourier terms for the annual cycle.

## Running the notebook

Open `electricity_demand_forecasting.ipynb` in Google Colab (a GPU runtime helps
the neural step) and run the cells in order.

- The load CSV is read locally if present, otherwise downloaded from OPSD.
- A `quick_grid` flag in the `Config` cell controls the SARIMA search; it is set to
  **False** for the full mandated grid (p, q in [0, 6]). The full grid fits several
  hundred models and takes around 25 minutes.
- Dependencies are listed in `requirements.txt` (or `environment.yml`).

## What the notebook covers

- Data loading, gap repair, and weekly/daily aggregation.
- Exploratory analysis: a year-by-week heatmap, monthly boxplots, a periodogram,
  and an additive decomposition.
- Stationarity testing with the ADF test across differencing variants.
- Benchmark forecasts (mean, naive, seasonal naive, drift).
- SARIMA: a full grid ranked by the small-sample-corrected AIC (AICc), with
  residual diagnostics and a two-year forecast with intervals.
- SARIMAX: the same model with a heating-degree feature and a holiday indicator.
- A random forest on annual lags plus weather and calendar features.
- A Conv1D-LSTM on the hourly series, tuned and rolled forward day by day.
- A seasonal error breakdown and a final comparison figure and table.

## Evaluation

Every model is scored on the same 104-week window with RMSE (TWh), MAPE, and a
demand-weighted MAPE, all compared against the seasonal naive.

## Avoiding data leakage

- The heating-degree base and any fitted transforms use training data only.
- The random forest's one-year lag is filled with its own first-year predictions in
  the second test year, so no test actual leaks into the horizon.
- The neural scaler is fitted on training hours only, and the forecast is produced
  block by block without consuming test values.
- The train/test split is the final 104 weeks in time order (no shuffling).

## Outputs

Running the notebook writes to an `artifacts/` folder next to it:

```
artifacts/metrics_summary.csv       # RMSE, MAPE, wMAPE per model
artifacts/comparison_of_models.png  # all forecasts vs actual
```

## Files

```
electricity_demand_forecasting.ipynb   # the analysis
report_draft.html / report.docx        # written report (draft)
requirements.txt / environment.yml     # dependencies
time_series_60min_singleindex.csv      # load data (optional local copy)
```
