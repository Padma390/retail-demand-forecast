# Retail Demand Forecasting & Inventory Optimization

## Executive Summary

This project builds an end-to-end machine learning pipeline to:

* forecast the next 7 days of sales at the store-item level,
* optimize inventory using safety stock and reorder point logic,
* generate automated restocking alerts.

The final solution improves forecasting accuracy by about **30% over a baseline model** and connects predictions directly to inventory decisions. It also includes a **fallback mechanism** that uses forecasted values when the latest actual sales data is unavailable, allowing weekly forecasting to continue without interruption. 

## Business Problem

Retail chains face two major risks:

* **Stockouts** → lost sales and poor customer experience
* **Overstocking** → higher holding cost and locked working capital

To address this, businesses need:

* accurate short-term demand forecasts,
* a data-driven reorder strategy.

This project solves both in one integrated workflow. 

## Solution Overview

The solution has two main components:

1. **Demand Forecasting** using machine learning
2. **Inventory Optimization** using statistical and rule-based logic

## Methodology

### 1. Data Preprocessing

* Converted the date column to datetime format
* Sorted data by store, item, and date
* Applied log transformation to stabilize variance

### 2. Feature Engineering

#### Time-Based Features

* day of week
* week of year
* month

#### Lag Features

* lag 1
* lag 7
* lag 14

#### Rolling Statistics

* 7-day rolling mean
* 14-day rolling mean
* 28-day rolling mean
* 7-day rolling standard deviation

These features help capture:

* seasonality,
* recent demand trends,
* demand variability. 

## Models Evaluated

### 1. Naive Baseline (Lag-7)

Uses previous week’s demand as the prediction.

**Strength:** captures weekly seasonality
**Limitation:** cannot capture trends or nonlinear patterns

### 2. Linear Regression

Uses engineered lag and rolling features in a linear model.

**Strength:** simple and interpretable
**Limitation:** struggles with nonlinear demand behavior

### 3. Random Forest Regressor

Captures nonlinear relationships and feature interactions.

**Strength:** robust and better at modeling complex patterns
**Result:** significant improvement over Linear Regression

### 4. XGBoost (Final Model)

A gradient boosting model that sequentially learns residual errors.

**Strength:** strong performance on structured tabular data
**Result:** best performance among all evaluated models 

## Model Performance

### RMSE on Actual Scale

* **Baseline:** 11.63
* **Linear Regression:** 8.61
* **Random Forest:** 8.33
* **XGBoost:** ~8.1

This represents an improvement of approximately **30% over the baseline**.

**Final model selected:** XGBoost. 

## Model Interpretability

SHAP was used to interpret the XGBoost model.

### Key Insights

* Rolling mean features (`roll_mean_7`, `roll_mean_14`, `roll_mean_28`) had the highest impact
* Demand was influenced more by recent smoothed trends than by raw lag values
* Higher rolling averages generally led to higher predicted demand
* Calendar features had relatively low impact

This suggests that recent demand behavior is more important than simple date-based effects in this use case. 

## 7-Day Recursive Forecasting

The forecasting pipeline uses a recursive strategy:

1. Take the latest available history
2. Predict day 1
3. Append the prediction to history
4. Recompute lag and rolling features
5. Repeat for the next 7 days

This simulates real-world forecasting, where actual future sales are not yet known. 

## Inventory Optimization Logic

### Lead Time Demand

Lead Time Demand is calculated as the sum of predicted demand over the lead time.

### Safety Stock

[
\text{Safety Stock} = Z \times \sigma_d \times \sqrt{\text{Lead Time}}
]

Where:

* ( Z = 1.65 ) for a 95% service level
* ( \sigma_d ) = standard deviation of historical demand

### Reorder Point

[
\text{Reorder Point} = \text{Lead Time Demand} + \text{Safety Stock}
]

### Decision Rule

If:

**Current Inventory < Reorder Point**

→ generate a reorder alert

This helps reduce stockout risk while keeping inventory decisions data-driven. 

## Production Deployment Logic

The pipeline is designed for a realistic weekly forecasting setup.

* If the latest actual sales data is available, it is used to compute lag and rolling features
* If the latest actual sales is unavailable, previously forecasted values are used instead

This ensures:

* forecasting can continue even when real-time data is delayed,
* lag and rolling features can still be computed,
* the pipeline remains operational in production-like conditions.

**Note:** forecasted values are used only to support ongoing forecasting and are **not** used for model retraining. 

## Business Impact

* Improved short-term demand forecasting accuracy
* Reduced stockout risk
* More informed inventory planning
* End-to-end reproducible ML workflow

## Key Learnings

* Feature engineering had a major impact on performance
* Tree-based models outperformed linear models
* XGBoost delivered the best predictive accuracy
* SHAP improved explainability
* Recursive forecasting better reflects real deployment conditions

## Future Improvements

* Add LightGBM and other advanced models
* Use time-series cross-validation
* Incorporate external signals such as holidays and promotions
* Automate retraining and deployment pipelines
