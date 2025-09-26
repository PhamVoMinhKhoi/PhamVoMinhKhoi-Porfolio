# Full Pipeline (RF → Bin-wise XGB): Tài liệu & Giải thích Code + Ý đồ chỉ số

> **Mục tiêu:** Dự đoán 2 tầng trên dữ liệu đơn hàng vận hành:
> - **Tầng 1 (Classification):** dự đoán `label` (`INTERNAL`/`POOL`/`CANCELLED`) bằng Random Forest.
> - **Tầng 2 (Regression):** dự đoán `first_pooling` (phút) cho **các đơn không bị dự đoán `CANCELLED`** bằng XGBoost theo **bin 15 phút** với **clipping**.

---

## 0) Khởi động & hằng số

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path
import warnings
warnings.filterwarnings("ignore")

from sklearn.model_selection import train_test_split
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.metrics import (
    classification_report, confusion_matrix, f1_score,
    mean_absolute_error, mean_squared_error, r2_score
)
from sklearn.ensemble import RandomForestClassifier
from xgboost import XGBRegressor

RANDOM_STATE = 42
np.random.seed(RANDOM_STATE)

def rmse(y_true, y_pred):
    return mean_squared_error(y_true, y_pred) ** 0.5
```

**Ý đồ:**  
- `warnings.filterwarnings("ignore")`: tránh log rác.  
- `RANDOM_STATE`: giữ reproducibility.  
- `rmse`: bổ sung thước đo dễ hiểu trong vận hành.

---

## 1) Chuẩn bị dữ liệu & Feature Engineering (Domain-driven)

```python
DATA_PATH = "/content/data_v9.csv"
df = pd.read_csv(DATA_PATH)

df['label'] = df['label'].fillna('CANCELLED')
df['driver_fee'] = df['driver_fee'].fillna(0)
```

- Điền thiếu `label` = CANCELLED để tránh mô hình học sai.  
- `driver_fee=0`: cho phép tính toán `margin`.  

**Tạo biến domain:**

```python
def epd(row): ...
def mpd(row): ...
df['epd_value'] = df.apply(epd, axis=1)
df['MPD'] = df.apply(mpd, axis=1)

df['margin'] = df['MPD'] * df['distance'] + df['driver_fee']
df['is_internal'] = (df['margin'] > df['partner_fee']).astype(int)
```

- `epd_value`: earning per delivery theo region/service/hour.  
- `MPD`: chi phí mỗi km.  
- `margin`: xấp xỉ lợi nhuận.  
- `is_internal`: binary → dễ hiểu & giàu ý nghĩa.  

---

## 2) EDA

- **Countplot `label`**: phân bố lớp.  
- **Boxplot `distance`, `driver_fee`, `first_pooling`, `demand` theo label**.  
- **Histogram `hour` theo label**.  
- **Countplot `region`/`service` theo label**.  

**Ý đồ:** kiểm tra data imbalance, outlier, pattern domain.

---

## 3) Feature set & Preprocessing

```python
selected_features = [...]
categorical_cols = [...]
numeric_cols = [...]
preprocessor = ColumnTransformer([...])
```

- `SimpleImputer(most_frequent)` cho categorical, tránh crash khi missing.  
- `OHE(handle_unknown='ignore')` → robust khi inference.  
- `SimpleImputer(median)` cho numeric → robust outlier.  
- `StandardScaler()` → scale đồng nhất (có lợi khi mở rộng model).  

**Ý đồ:** đóng gói sạch, tránh rò rỉ dữ liệu.  

---

## 4) Tầng 1 — RandomForest Classification

```python
rf_clf = Pipeline(steps=[
    ('prep', preprocessor),
    ('rf', RandomForestClassifier(
        n_estimators=300,
        max_depth=None,
        random_state=RANDOM_STATE,
        n_jobs=-1
    ))
])
rf_clf.fit(X_train_c, y_train_c)
y_pred_c = rf_clf.predict(X_test_c)
```

- **RF cơ chế:** nhiều decision tree học trên bootstrap sample → bỏ phiếu đa số.  
- **Chỉ số:**  
  - `classification_report`: chi tiết precision, recall, F1.  
  - `Macro-F1`: cân bằng khi lớp lệch.  
  - `confusion_matrix`: xem loại nhầm lẫn.  

**Ý đồ:** đảm bảo mô hình không chỉ học lớp đông (`INTERNAL`).  

---

## 5) Tầng 2 — Bin-wise XGB Regression

### 5.1 Binning

```python
bin_edges = [0,15,30,45,60,90,120,np.inf]
bin_labels = ['0-15','15-30','30-45','45-60','60-90','90-120','120+']
df_work['fp_bin'] = pd.cut(df_work['first_pooling'], bins=bin_edges, labels=bin_labels)
```

- Chia dải pooling thành bins 15 phút.  
- Giảm variance, tránh nhiễu.  

### 5.2 Huấn luyện per-bin

```python
model = Pipeline(steps=[
    ('prep', preprocessor),
    ('xgb', XGBRegressor(
        n_estimators=200, max_depth=4, learning_rate=0.08,
        subsample=0.9, colsample_bytree=0.9,
        tree_method='hist', n_jobs=-1
    ))
])
model.fit(X_tr, y_tr)
y_hat = np.clip(model.predict(X_te), lo, hi)
```

- **XGB ưu điểm:** mạnh cho tabular, xử lý phi tuyến tốt.  
- **Clipping:** ép dự đoán vào miền bin → an toàn, đúng nghiệp vụ.  

### 5.3 Chỉ số đánh giá
- **MAE**: sai số tuyệt đối trung bình → dễ hiểu (phút).  
- **RMSE**: nhạy outlier → phát hiện case sai lệch nặng.  
- **R²**: phần phương sai giải thích được.  

**Ý đồ:** MAE = KPI chính; RMSE để giám sát outlier; R² để so sánh tổng quan.  

---

## 6) Phân tích đặc trưng theo bin
- **Numeric summary**: trung bình distance, driver_fee, demand theo bin.  
- **Categorical distribution**: service/region/buổi theo bin (%).  
- **Top-5 pickup/dropoff district** theo bin.  

**Ý đồ:** hiểu đặc điểm của đơn hàng trong mỗi bin → hỗ trợ vận hành.  

---

## 7) Artifacts đầu ra
- `predictions_label_rf.csv`  
- `predictions_first_pooling_binwise.csv`  
- `bin_numeric_summary.csv`  
- `bin_service_count.csv`, `bin_service_pct.csv`  
- `bin_region_count.csv`, `bin_region_pct.csv`  
- `bin_buoi_count.csv`, `bin_buoi_pct.csv`  
- `bin_top5_pickup_district.csv`, `bin_top5_dropoff_district.csv`  

---

## 8) Giả định & Rủi ro
- Các rule EPD/MPD phản ánh chính sách giá hiện tại → phải update nếu thay đổi.  
- Sai số tầng 1 (false CANCELLED) khiến tầng 2 bỏ sót pooling.  
- Bin nhỏ dễ overfit → cần tối thiểu số mẫu.  
- Drift vùng/giờ/mùa → cần retrain định kỳ.  

---

## 9) Hướng dẫn chạy lại
1. Cài thư viện:  
   ```bash
   pip install numpy pandas matplotlib seaborn scikit-learn xgboost
   ```
2. Đặt dữ liệu: `/content/data_v9.csv`.  
3. Chạy các cell theo thứ tự.  
4. Lấy output CSV trong thư mục làm việc.  

---

## 10) Sơ đồ luồng pipeline

```
Raw CSV
   │
Chuẩn hoá & Feature Engineering
   │
Preprocessor (OHE + Scaling)
   │
RandomForest (label)
   │
Lọc pred_label != CANCELLED
   │
Chia bin theo ground truth
   │
XGBoost Regressor theo bin + clipping
   │
Xuất predictions + summary CSV
```
