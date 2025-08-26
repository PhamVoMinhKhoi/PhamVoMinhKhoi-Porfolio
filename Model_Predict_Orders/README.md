# 🚚 Pooling Prediction Pipeline (RF + Bin-wise XGB)

## 📌 Project Description
The project’s goal is to build a predictive pipeline for logistics pooling using a two-stage model:  
1. **Random Forest Classification** to predict whether an order is **INTERNAL**, **POOL**, or **CANCELLED**.  
2. **Bin-wise XGB Regression** to estimate the **first pooling time** in 15-minute bins, ensuring realistic and stable predictions.  

The pipeline simulates production where true labels are unknown, so predicted labels guide pooling time estimation.

---

## ⚙️ Workflow
1. **Data preparation & feature engineering**
   - Filled missing values (`label` → CANCELLED, `driver_fee` → 0).  
   - Created domain-driven features:  
     - `epd_value`, `MPD`, `margin`, `is_internal`.  
   - Preprocessing with `SimpleImputer`, `OneHotEncoder`, `StandardScaler`.

2. **Model 1 – RandomForest (classification)**
   - Output: predicted order label (`INTERNAL`, `POOL`, `CANCELLED`).  
   - Evaluation: classification report, confusion matrix, macro-F1.

3. **Model 2 – Bin-wise XGB (regression)**
   - Output: predicted `first_pooling` clipped into bins:  
     - [0–15), [15–30), [30–45), [45–60), [60–90), [90–120), [120+].  
   - Evaluation: MAE, RMSE, R² per bin and overall.

---

## 📤 Outputs
- **predictions_label_rf.csv**  
  - `order_id`, true `label`, predicted `label`.  
  - Used to analyze classification accuracy.  

- **predictions_first_pooling_binwise.csv**  
  - `order_id`, pooling `bin`, `first_pooling_true`, `first_pooling_pred`.  
  - Used to analyze pooling speed distribution.  

- **Scatter plot** comparing true vs predicted pooling times.  

---

## 📊 Output Analysis

### 1. Classification Results
- Majority of orders are predicted as **INTERNAL**, consistent with real-world distribution.  
- Model successfully separates **CANCELLED** orders → allowing resources to be reallocated away from high-risk orders.  
- **POOL** remains a minority class → macro-F1 ensures balanced evaluation across all labels.  

**Impact:**  
- Prevent wasted effort on orders likely to be cancelled.  
- Prioritize INTERNAL orders → improve driver efficiency and platform reliability.  

---

### 2. Regression Results
- Orders pooled within **0–15 minutes** are mostly: short distance, SAMEDAY, HAN region, morning demand.  
- Slow-pooling orders (>60 minutes) often: long distance, SAMEPRICE, SGN region, high COD, high demand.  
- Bin-wise regression reduces extreme outlier errors and provides interpretable time ranges.  

**Impact:**  
- Identify configurations where pooling is too slow → adjust system to accelerate.  
- Proactively manage driver allocation during high-demand periods.  
- Predict which orders may need **config adjustments or price incentives** to be matched earlier.  

---

## 📊 Summary Table

| Stage              | Key Outputs                          | Business Impact |
|--------------------|---------------------------------------|-----------------|
| **Classification** | INTERNAL / POOL / CANCELLED labels    | Avoid wasted effort on cancelled orders, focus on INTERNAL |
| **Regression**     | First pooling time prediction (bins)  | Detect slow orders, adjust pooling configs, manage driver allocation |

---

## 💡 Insights & Applications
- **Fast-pooling vs Slow-pooling patterns:** extracted from features like distance, service type, region, demand, COD.  
- **Actionable recommendations:**  
  - Adjust pooling configs for slow orders.  
  - Anticipate cancellations and minimize wasted time.  
  - Optimize driver acceptance via pricing and COD policies.  

---
