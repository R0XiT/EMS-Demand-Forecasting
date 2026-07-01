# 🚑 NYC EMS Demand Forecasting — ARIMA vs Prophet

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Prophet](https://img.shields.io/badge/Prophet-1.1+-orange)
![Statsmodels](https://img.shields.io/badge/Statsmodels-0.14+-green)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

## 📌 Project Overview

New York City EMS responds to over **4,400 emergency incidents every single day**. 
Accurately forecasting daily demand allows hospital administrators and dispatch 
managers to optimize staff scheduling — reducing both understaffing risk and 
unnecessary overtime costs.

This project analyzes **4.8 million NYC EMS dispatch records (2023–2025)** and 
builds two forecasting models — ARIMA and Prophet — evaluated against a naive 
rolling mean benchmark to predict daily incident volume.

---

## 🎯 Business Problem

> *"How many EMS incidents will occur on a given day so that NYC dispatch centers 
> can plan staffing shifts 30 days in advance?"*

**Impact of poor forecasting:**
- Understaffing → longer response times → patient deterioration
- Overstaffing → budget waste → staff burnout

---

## 📊 Dataset

| Property | Detail |
|---|---|
| Source | [NYC OpenData — EMS Incident Dispatch Data](https://data.cityofnewyork.us/Health/EMS-Incident-Dispatch-Data/76xm-jjuj) |
| Period | January 2023 — December 2025 |
| Raw rows | 4,855,843 incidents |
| After aggregation | 1,094 daily records |
| Key columns used | `INCIDENT_DATETIME`, `INITIAL_CALL_TYPE`, `BOROUGH` |

---

## 🔬 Methodology

### 1. Data Cleaning & Aggregation
- Trimmed 31 columns to 4 relevant features
- Parsed datetime and extracted date component
- Aggregated 4.8M rows → 1,094 daily incident counts
- Removed 2 partial-day outliers (Jan 1 2023, Dec 31 2025)

### 2. Exploratory Data Analysis
- Daily trend visualization across 3 years
- Monthly and day-of-week demand patterns
- Key finding: **Friday peak (~4,480), Sunday lowest (~4,180)**

### 3. Stationarity Testing (ADF Test)
- ADF Statistic: **-4.098**, p-value: **0.001**
- Series confirmed stationary → d = 0, no differencing required
- Unlike stock price data, EMS volume does not trend upward indefinitely

### 4. Time Series Decomposition
- Separated series into Trend + Weekly Seasonality + Residuals
- Trend: Peaked late 2023, gradually declining since
- Seasonality: Clear ±200 incident weekly cycle confirmed

### 5. ACF & PACF Analysis
- PACF cutoff at lag 1 → p = 1
- ACF cutoff at lag 1 → q = 1
- Confirmed ARIMA order: **(1, 0, 1)**

### 6. Model Building
| Approach | Description |
|---|---|
| Naive Benchmark | 7-day rolling mean — simplest possible reference forecast |
| ARIMA(1,0,1) | Classical statistical forecasting |
| Prophet | Meta's time series model with US holiday effects |

> **Note:** The naive benchmark is not a model — it is a reference point.
> It represents the simplest possible prediction (recent average) and exists 
> to give ARIMA and Prophet results meaningful business context.

### 7. Evaluation
- 80/20 train-test split (1,004 train / 90 test days)
- Metrics: MAE, RMSE, MAPE
- All forecasting models compared against the naive benchmark

---

## 📈 Results

*All models evaluated on a 90-day holdout test set (Oct–Dec 2025).*
*Results compared against a naive 7-day rolling mean benchmark.*

| Approach | MAE | RMSE | MAPE |
|---|---|---|---|
| Naive Benchmark (7-day avg) | 294.79 | 364.17 | 7.05% |
| ARIMA(1,0,1) | 246.88 | 317.10 | 5.84% |
| **Prophet** ✅ | **185.82** | **231.92** | **4.32%** |

- **ARIMA improved over benchmark by 16%** on MAE
- **Prophet improved over benchmark by 37%** on MAE
- **Prophet outperformed ARIMA by 25%** on MAE

**Why Prophet won:** Standard ARIMA produces near-flat forecasts and cannot 
capture multiple seasonality layers simultaneously. Prophet explicitly models 
weekly cycles, annual patterns, and US holiday effects — all of which are 
strong signals in EMS demand data.

---

## 💡 Business Recommendations

1. **Staff 8–12% more on Fridays vs Sundays** — consistent 300-incident 
   gap confirmed across all 3 years

2. **Surge planning for July–August** — heat-related emergencies drive 
   demand ~240 incidents above annual average

3. **Reduce staffing around major holidays** — Thanksgiving and Christmas 
   show consistent −600 to −800 incident dips

4. **Deploy Prophet for 30-day rolling forecasts** — 4.32% MAPE is 
   operationally sufficient for shift-level planning at borough level

---

## 📌 Key Learnings

- Prophet significantly outperforms ARIMA on data with **multiple 
  seasonality layers** (weekly + yearly + holidays)
- Standard ARIMA produces near-flat forecasts when seasonal patterns 
  dominate — a fundamental limitation for this use case
- A naive benchmark is essential for context — without it, a 5.84% MAPE 
  sounds impressive but is only marginally better than a flat average
- Aggregating 4.8M raw transactional rows into a clean daily time series 
  is often more impactful than model selection itself

---
