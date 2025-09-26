# Tài liệu kỹ thuật — Full Pipeline: RF (label) → Bin-wise XGB (first_pooling)

## 0) Mục tiêu tổng quát
Pipeline được thiết kế thành **2 tầng** nhằm mô phỏng đúng luồng dự đoán trong thực tế vận hành:

1. **Tầng 1 (RandomForestClassifier)**  
   - Nhiệm vụ: phân loại `label` cho mỗi đơn hàng thành 3 nhóm:  
     - `INTERNAL`: đơn có lợi nhuận nội bộ.  
     - `POOL`: đơn ghép với các đơn khác.  
     - `CANCELLED`: đơn bị huỷ.  
   - Tại sao cần tầng này:  
     - Nếu đơn bị huỷ, không cần dự đoán pooling → tiết kiệm tài nguyên.  
     - Nếu đơn hợp lệ, kết quả `label` làm input cho tầng tiếp theo.

2. **Tầng 2 (Bin-wise XGBoost Regressor)**  
   - Nhiệm vụ: dự đoán `first_pooling` (thời gian ghép đơn đầu tiên).  
   - Đặc điểm:  
     - Thay vì hồi quy toàn bộ dải 0–∞ phút, dữ liệu được chia thành **bin 15 phút**.  
     - Mỗi bin huấn luyện một mô hình XGB riêng → giảm nhiễu & xử lý outlier tốt hơn.  
     - Dự đoán được **clip** trong range bin để tránh sai lệch.

---

## 1) Môi trường & Phụ thuộc
- **Ngôn ngữ & thư viện**:  
  - `numpy`, `pandas` → xử lý dữ liệu.  
  - `matplotlib`, `seaborn` → trực quan.  
  - `scikit-learn` → pipeline, preprocessing, RandomForest.  
  - `xgboost` → hồi quy với XGBRegressor.  

- **Thiết lập reproducibility**:  
  - `RANDOM_STATE = 42`, `np.random.seed(42)`.  
  - Đảm bảo chạy nhiều lần vẫn ra kết quả gần như giống nhau.

- **Tối ưu tốc độ**:  
  - RandomForest: `n_jobs=-1` tận dụng toàn bộ CPU.  
  - XGB: `tree_method='hist'` tối ưu tốc độ/ram cho dữ liệu lớn.

---

## 2) Dữ liệu đầu vào & Chuẩn hoá theo quy tắc
### 2.1 File & cột bắt buộc
- **File đầu vào**: `/content/data_v9.csv`
- **Cột cần thiết**:  
  `order_id, label, driver_fee, partner_fee, distance, service, region, hour, weekday_number, demand, first_pooling, pickup_district, dropoff_district, buoi`

### 2.2 Xử lý missing value
- `label`: NaN → `"CANCELLED"`.  
- `driver_fee`: NaN → `0`.

### 2.3 Feature engineering (theo quy tắc domain)
- **`epd_value`**: earning per delivery (hệ số lợi nhuận theo `service`, `region`, `hour`).  
  - Ví dụ: `SAMEDAY-HAN <16h → 6.6`, `≥16h → 5.6`.
- **`MPD`**: money per distance (đồng/km, đặt số âm để thể hiện chi phí/km).  
  - Ví dụ: `SAMEPRICE-HAN/SGN → -500`.
- **`margin`**: xấp xỉ lợi nhuận sau khi trừ chi phí:  
  - `margin = MPD * distance + driver_fee`
- **`is_internal`**: 1 nếu `margin > partner_fee`, ngược lại 0.  
  - Biến này cho biết đơn có lợi nhuận nội bộ hay không.

---

## 3) Khám phá dữ liệu (EDA)
### 3.1 Phân bố nhãn `label`
- Countplot → cho thấy tỷ lệ giữa `INTERNAL`, `POOL`, `CANCELLED`.

### 3.2 Boxplot các biến numeric
- **Distance theo Label**: phát hiện đơn dài/ngắn phân bố ở đâu.  
- **Driver_fee theo Label**: xem khác biệt chi phí theo nhóm.  
- **First_pooling theo Label**: phân tích sự khác biệt thời gian ghép đơn.

### 3.3 Phân bố categorical
- **Region vs Label**: đơn nào thuộc HAN, SGN phân bố ra sao.  
- **Service vs Label**: dịch vụ nào dễ cancel/ghép hơn.

### 3.4 Phân bố theo thời gian
- **Demand by Label**: biến động cầu theo nhóm.  
- **Hour by Label**: histogram giờ đặt đơn để xem pattern theo thời gian trong ngày.

---

## 4) Feature set & Preprocessing
### 4.1 Tập biến đầu vào
`pickup_district, dropoff_district, distance, hour, service, region, driver_fee, partner_fee, weekday_number, demand, epd_value, MPD, margin, is_internal, buoi`

### 4.2 Preprocessing
- **Categorical**:  
  - `SimpleImputer(strategy="most_frequent")`  
  - `OneHotEncoder(handle_unknown="ignore")`  
  - → đảm bảo không crash khi có nhãn mới.
- **Numeric**:  
  - `SimpleImputer(strategy="median")` → robust với outlier.  
  - `StandardScaler()` → chuẩn hoá thang đo, phù hợp khi kết hợp nhiều mô hình khác nhau.

➡ Tất cả đóng gói trong `ColumnTransformer` → nhất quán giữa train/test.

---

## 5) Model 1 — RandomForestClassifier
### 5.1 Cơ chế
- RF = tập hợp nhiều cây quyết định (decision tree).  
- Mỗi cây học trên dữ liệu bootstrap + chọn random feature.  
- Dự đoán cuối cùng = voting từ tất cả cây.  

### 5.2 Ưu điểm
- Chịu được dữ liệu categorical (sau OHE).  
- Khó overfit khi số cây lớn.  
- Ít cần chuẩn hoá dữ liệu.

### 5.3 Thiết lập
- `n_estimators=300` → số cây nhiều giúp ổn định.  
- `max_depth=None` → cho cây phát triển tối đa.  
- Train/test split với `stratify=y` để giữ tỉ lệ lớp.

### 5.4 Đầu ra
- **Báo cáo đánh giá**: classification_report (precision, recall, F1), confusion matrix.  
- **Dự đoán cho toàn bộ dataset**: tạo `pred_label`.  
- **Xuất file**: `predictions_label_rf.csv`.

---

## 6) Model 2 — XGBRegressor theo Bin 15 phút
### 6.1 Ý tưởng
- Trực tiếp hồi quy `first_pooling` dễ gây sai số lớn vì:  
  - Dữ liệu lệch phải (nhiều giá trị lớn).  
  - Nhiễu từ outlier.  
- Giải pháp: **chia dải thời gian thành bin 15 phút**, huấn luyện XGB riêng cho từng bin.  

### 6.2 Binning
- Các khoảng: `[0–15), [15–30), [30–45), [45–60), [60–90), [90–120), [120+)`.  
- Gán `fp_bin` từ `first_pooling` thật (chỉ dùng trong train).

### 6.3 Cơ chế huấn luyện
- Với mỗi bin đủ dữ liệu (≥200 mẫu):  
  - Chia train/test 80/20.  
  - Pipeline: preprocessor → XGBRegressor.  
- Tham số XGB:  
  - `n_estimators=200`, `max_depth=4`.  
  - `learning_rate=0.08`, `subsample=0.9`, `colsample_bytree=0.9`.  
  - `tree_method="hist"` cho dữ liệu lớn.  
- Dự đoán được **clip** về khoảng bin → tránh outlier sai lệch.

### 6.4 Đánh giá
- **Per-bin metrics**: MAE, RMSE, R².  
- **Overall metrics**: gộp toàn bộ test → tính MAE/RMSE/R² tổng.

### 6.5 Kết quả
- File: `predictions_first_pooling_binwise.csv`.  
- Cột: `order_id, bin, first_pooling_true, first_pooling_pred`.

---

## 7) Phân tích đặc trưng theo bin
### 7.1 Numeric summary
- Trung bình các biến numeric (`distance`, `hour`, `driver_fee`, …) theo bin.  
- Xuất: `bin_numeric_summary.csv`.

### 7.2 Phân bố categorical
- Service/Region/Buổi → count & phần trăm theo bin.  
- Xuất:  
  - `bin_service_count.csv`, `bin_service_pct.csv`  
  - `bin_region_count.csv`, `bin_region_pct.csv`  
  - `bin_buoi_count.csv`, `bin_buoi_pct.csv`

### 7.3 Top-5 pickup/dropoff district
- Liệt kê top quận có nhiều đơn nhất trong mỗi bin.  
- Xuất: `bin_top5_pickup_district.csv`, `bin_top5_dropoff_district.csv`.

---

## 8) Artifacts đầu ra
- `predictions_label_rf.csv`  
- `predictions_first_pooling_binwise.csv`  
- `bin_numeric_summary.csv`  
- `bin_service_count.csv`, `bin_service_pct.csv`  
- `bin_region_count.csv`, `bin_region_pct.csv`  
- `bin_buoi_count.csv`, `bin_buoi_pct.csv`  
- `bin_top5_pickup_district.csv`, `bin_top5_dropoff_district.csv`  
- (Tuỳ chọn) `final_predictions_all.csv`

---

## 9) Đánh giá mô hình
### 9.1 Tầng 1 (RF)
- **Macro-F1**: đo lường cân bằng giữa các lớp.  
- **Confusion Matrix**: kiểm tra lỗi nhầm giữa `POOL`/`INTERNAL` và tỷ lệ false cancel.  
- RF mạnh với tabular data nhưng khó diễn giải chi tiết từng feature.

### 9.2 Tầng 2 (XGB bin-wise)
- **MAE/RMSE**: đơn vị phút → dễ diễn giải trong vận hành.  
- **R²**: đo mức độ mô hình giải thích biến thiên dữ liệu.  
- Mỗi bin cho biết độ khó dự đoán riêng → bin `120+` thường khó hơn.

---

## 10) Giả định & Rủi ro
- Quy tắc tính `epd_value`, `MPD`, `margin` phản ánh **chính sách giá hiện tại** → nếu thay đổi cần update.  
- False cancel từ RF khiến pipeline bỏ qua dự đoán pooling.  
- Nếu dữ liệu trong bin ít → mô hình dễ overfit hoặc không huấn luyện được.  
- Nguy cơ **drift** theo mùa, khu vực, khung giờ → cần retrain định kỳ.

---

## 11) Hướng dẫn chạy lại
1. Cài thư viện:  
   ```bash
   pip install numpy pandas matplotlib seaborn scikit-learn xgboost
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
