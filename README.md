**Retail Demand Forecasting & Inventory Optimization**

**Executive Summary:**
This project builds an end-to-end machine learning pipeline to:

* Forecast next 7 days of sales at store-item level
* Optimize inventory using safety stock & reorder point logic
* Generate automated restocking alerts


The system reduces forecasting error by ~30% over baseline and connects predictions directly to business inventory decisions.
The pipeline also includes a production-ready fallback mechanism to handle missing real-time sales data using forecasted values

**Business Problem:**
Retail chains face two major risks:
    Stockouts → Lost sales & customer dissatisfaction
    Overstocking → High holding cost & capital lock-in

Requirement
    Accurate short-term demand forecasts
    A data-driven reorder strategy


This project solves both problems in one integrated pipeline.


**Solution Approach:**

The solution consists of two major components:
    Demand Forecasting (ML-based)
    Inventory Optimization (rule + statistical approach)

Methodology

1. Data Preprocessing

    * Converted date column to datetime format
    * Sorted data by store, item, and date
    * Applied log transformation to stabilize variance

2. Feature Engineering

Time-Based Features
    1. Day of week
    2. Week of year
    3. Month

Lag Features
    1. Lag 1 (previous day)
    2. Lag 7 (previous week)
    3. Lag 14 (two weeks prior)

Rolling Statistics
    1. 7-day rolling mean
    2. 14-day rolling mean
    3. 28-day rolling mean
    4. 7-day rolling standard deviation

These features capture:
    * seasonality
    * short-term trends
    * demand variability


**Model Selection**
**Models Evaluated:**

* Naive Baseline (Lag-7)
* Linear Regression
* Random Forest Regressor
* XGBoost


**1. Naive Baseline (Lag-7)**

* Uses previous week demand
* Captures weekly seasonality

Limitation:
    Cannot capture trends or complex patterns

**2. Linear Regression**

* Assumes linear relationships
* Uses engineered lag & rolling features

Limitation:
    Cannot capture nonlinear demand patterns

**3. Random Forest**

* Captures nonlinear relationships
* Handles feature interactions
* Robust to noise

**Result:**
    Significant improvement over Linear Regression

**4. XGBoost (Final Model)**

* Gradient boosting model
* Sequentially learns residual errors
* Captures complex nonlinear patterns

**Result:**
    Best performance among all models

Model Performance (Actual Scale)

* Baseline RMSE → 11.63
* Linear Regression RMSE → 8.61
* Random Forest RMSE → 8.33
* XGBoost RMSE → ~8.1 (best)

~30% improvement over baseline

Final Model Selected: XGBoost

**Model Interpretability (SHAP)**

    SHAP analysis was used to interpret the XGBoost model.

**Key Insights:**

* Rolling mean features (roll_mean_7, roll_mean_14, roll_mean_28) had highest impact
* Demand is driven more by recent trends than raw lag values
* Higher rolling averages → higher predicted demand
* Calendar features had minimal impact

**Conclusion:**
    Demand forecasting depends heavily on smoothed recent behavior

**7-Day Recursive Forecasting**
**Forecasting strategy:**

* Use latest 14-day history
* Predict day 1
* Append prediction to history
* Recalculate rolling features
* Repeat for 7 days

This mimics real-world deployment where future actuals are unknown.

**Inventory Optimization Logic**

* Lead Time Demand
* Sum of predicted demand over lead time
* Safety Stock

Safety Stock = Z × Demand Std Dev × √Lead Time
Where:
Z = 1.65 (95% service level)

Demand std dev from historical actual sales

Reorder Point
    Reorder Point = Lead Time Demand + Safety Stock

Decision Rule

If:
Current Inventory < Reorder Point
→ Generate Reorder Alert

This ensures:
* Reduced stockouts
* Controlled inventory risk
* Business Impact
* Improved short-term demand prediction
* Reduced stockout risk
* Data-driven inventory decisions
* End-to-end reproducible ML workflow

**Production - Deplyment Logic**
The forecasting pipeline is designed to operate in real-world setting with continuously updating data
The model is trained on historical actual sales data
For weekly forecasting:
* If latest actual sales are available, they are used to compute features
* If actual sales are not yet available, previously forecasted values are used

This ensures that:
* Lag and rolling features can always be computed
* Forecasting can continue even when real-time data is delayed

Note: Forecasted values are used only for operational continuity and are not used for model retraining

**Key Learnings**

* Feature engineering drives most performance
* Tree-based models outperform linear models
* XGBoost provides best predictive performance
* SHAP improves model explainability
* Recursive forecasting simulates real deployment

**Future Improvements**

* Add LightGBM / advanced models
* Time-series cross-validation
* Add external features (holidays, promotions)
* Automate model retraining pipeline

