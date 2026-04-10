# Inventory Management — Spare Parts Demand Forecasting

A time series analysis and forecasting project for a Bajaj motorcycle service center. The goal is to analyse historical spare parts usage and forecast weekly demand to support inventory planning.

## About

The project pulls service invoice data from a live MySQL database, cleans and separates spare parts from service labour charges, and applies time series models to forecast how many spare parts will be needed in the coming weeks.

## Data Source

- MySQL database: `project_service_data` → table `service_data`
- 28,482 raw records spanning May 2017 – January 2019
- 7 columns: invoice date, job card date, customer name, vehicle number, vehicle model, km reading, invoice line text

> Note: Do not hardcode database credentials in the notebook. Use environment variables or a `.env` file and add it to `.gitignore`.

## Dataset Summary

| Metric | Value |
|---|---|
| Raw records | 28,482 |
| After cleaning | 21,298 |
| Vehicle models | 28 (all Bajaj) |
| Unique spare parts | 431 |
| Date range | May 2017 – Jan 2019 (84 weeks) |

**Top spare parts by demand:** Engine Oil (3,804) · Air Filter (1,713) · 3M Oil (1,628) · Consumables (1,595) · Polish (1,245)

**Most serviced vehicle:** Bajaj Pulsar 150 — 6,480 records

## Data Cleaning Steps

- Dropped 34 null rows in `invoice_line_text` (0.12% of data)
- Removed 35 outlier rows where `current_km_reading > 100,000`
- Corrected typos in spare part names (e.g. OVERHUAL → OVERHAUL, WIELDING → WELDING)
- Separated 67 service/labour items from physical spare parts
- Retained only `date`, `vehicle_model`, `spare_part` columns for modelling

## Time Series Analysis

- Resampled data to weekly and monthly frequency
- Applied rolling mean (4-week window) and cumulative mean smoothing
- **ADF test on raw series:** p-value = 0.685 → non-stationary
- **Fix:** First-order differencing
- **ADF test after differencing:** p-value = 1.3e-11 → stationary

## Forecasting Models

| Model | MSE |
|---|---|
| Triple Exponential Smoothing (Holt-Winters) | 8,998.67 |
| SARIMA(5,1,1)(1,0,0,12) | 10,703.76 |

Triple Exponential Smoothing (multiplicative trend, additive seasonality, 26-week period) produced a lower MSE and is the recommended baseline model. The SARIMA order was selected automatically using `auto_arima`.

A 16-week ahead forecast was generated using the full dataset.

## Tech Stack

- Python, pandas, NumPy, Matplotlib, seaborn
- pymysql (database connection)
- statsmodels (ExponentialSmoothing, SARIMAX, adfuller)
- pmdarima (auto_arima)
- scikit-learn (MAE, MSE)
- ydata-profiling, sweetviz (automated EDA reports)

## Setup

```bash
git clone https://github.com/<your-username>/inventory-management.git
cd inventory-management
pip install pandas numpy matplotlib seaborn pymysql statsmodels pmdarima scikit-learn ydata-profiling sweetviz
```

Set your database credentials as environment variables before running:

```python
import os
host = os.getenv("DB_HOST")
user = os.getenv("DB_USER")
password = os.getenv("DB_PASSWORD")
```

## Run

Open `Inventory_Management.ipynb` and run all cells. The notebook will:

1. Pull data from MySQL and save to `service_data.csv`
2. Clean and preprocess the data
3. Generate EDA reports (HTML files)
4. Run stationarity tests and apply differencing
5. Train and evaluate Triple Exponential Smoothing and SARIMA models
6. Produce a 16-week demand forecast plot

## Project Structure

```
inventory-management/
├── Inventory_Management.ipynb   # Main notebook
├── service_data.csv             # Exported data (auto-generated)
├── SWEETVIZ_REPORT.html         # Sweetviz EDA report (auto-generated)
└── README.md
```

## Author

K Manoj
