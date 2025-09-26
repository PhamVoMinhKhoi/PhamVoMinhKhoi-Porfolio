# Tài liệu kỹ thuật — Full Pipeline: RF (label) → Bin-wise XGB (first_pooling)

## 0) Mục tiêu tổng quát
- Dự đoán 2 tầng:
- 1) Phân loại `label` cho mỗi đơn (`INTERNAL`/`POOL`/`CANCELLED`) bằng RandomForest.
- 2) Với các đơn không bị dự đoán `CANCELLED`, dự đoán thời gian `first_pooling` (phút) bằng XGBoost Regressor theo nhóm (bin) 15 phút và ràng buộc kết quả trong khoảng của bin (clipping).

## 1) Môi trường & Phụ thuộc
- Python, NumPy, Pandas, Matplotlib, Seaborn (EDA), scikit-learn, XGBoost.
- Thiết lập ngẫu nhiên: `RANDOM_STATE = 42`, `np.random.seed(42)`.
- Hiệu năng: RandomForestClassifier(..., n_jobs=-1) tận dụng CPU; XGBRegressor(tree_method='hist', n_jobs=-1) tối ưu bộ nhớ/tốc độ.

## 2) Dữ liệu đầu vào & Chuẩn hoá theo quy tắc dự án
- File: `/content/data_v9.csv`.
- Cột bắt buộc: order_id, label, driver_fee, partner_fee, distance, service, region, hour, weekday_number, demand, first_pooling, pickup_district, dropoff_district, buoi.
- Điền thiếu: label→'CANCELLED', driver_fee→0.
- Tạo biến domain: epd_value (theo service, region, ngưỡng 16h), MPD (đồng/km, số âm), margin = MPD×distance + driver_fee, is_internal = 1(margin > partner_fee).

## 3) Thăm dò dữ liệu (EDA) & trực quan
- Biểu đồ phân bố label; boxplot distance/driver_fee/first_pooling/demand theo label; histogram hour theo label; phân bố label theo region/service.
- Mục tiêu: kiểm tra lệch lớp, outlier, pattern theo khu vực/dịch vụ/giờ; xác thực giả thuyết domain.

## 4) Tập biến đầu vào & Tiền xử lý dùng chung
- selected_features: pickup_district, dropoff_district, distance, hour, service, region, driver_fee, partner_fee, weekday_number, demand, epd_value, MPD, margin, is_internal, buoi.
- Categorical: pickup_district, dropoff_district, service, region, buoi; Numeric: phần còn lại.
- Preprocessor: Categorical → SimpleImputer(most_frequent) → OneHotEncoder(handle_unknown='ignore'); Numeric → SimpleImputer(median) → StandardScaler().

## 5) Model 1 — RandomForestClassifier (dự đoán label)
- Split 80/20 với stratify=y_cls.
- Pipeline: preprocessor → RandomForestClassifier(n_estimators=300, max_depth=None, random_state=42, n_jobs=-1).
- Đánh giá: classification_report (macro-F1), confusion_matrix.
- Dự đoán toàn tập dữ liệu để có pred_label mô phỏng production; xuất predictions_label_rf.csv.

## 6) Model 2 — XGBRegressor theo bin 15 phút với clipping
- Lọc dữ liệu hồi quy: pred_label != 'CANCELLED' & first_pooling không NaN.
- Định nghĩa bins phút: [0–15), [15–30), [30–45), [45–60), [60–90), [90–120), [120+).
- Huấn luyện mỗi bin (đủ mẫu ≥200): split 80/20; Pipeline preprocessor → XGBRegressor(n_estimators=200, max_depth=4, learning_rate=0.08, subsample=0.9, colsample_bytree=0.9, tree_method='hist', n_jobs=-1).
- Clipping dự đoán theo biên của bin. Đánh giá per-bin (MAE, RMSE, R²) và Overall; xuất predictions_first_pooling_binwise.csv.

## 7) Tổng hợp đặc trưng theo bin (giải thích mô hình)
- Numeric summary theo bin (mean): distance, hour, driver_fee, partner_fee, weekday_number, demand, epd_value, MPD, margin, is_internal → bin_numeric_summary.csv.
- Phân bố categorical: service/region/buoi (count & %) → bin_service_count/pct.csv, bin_region_count/pct.csv, bin_buoi_count/pct.csv.
- Top-5 pickup/dropoff_district theo bin → bin_top5_pickup_district.csv, bin_top5_dropoff_district.csv.

## 8) Kết quả đầu ra (artifacts)
- predictions_label_rf.csv; predictions_first_pooling_binwise.csv; các CSV summary theo bin; có thể merge thành final_predictions_all.csv.

## 9) Đánh giá chất lượng & diễn giải
- Tầng 1: Macro-F1, Confusion Matrix; chú ý false-cancel.
- Tầng 2: MAE/RMSE (KPI chính), R²; theo dõi khó ở bin 120+; cân nhắc gom bin/tăng dữ liệu.

## 10) Giả định, ràng buộc & rủi ro
- Quy tắc epd_value/MPD phản ánh chính sách hiện tại; cần cập nhật khi chính sách đổi.
- margin là xấp xỉ; không bao gồm chi phí khác.
- Sai số tầng 1 (false CANCELLED) khiến mất dự đoán tầng 2; mẫu bin mỏng dễ overfit; drift theo mùa/vùng/giờ.

## 11) Hướng dẫn chạy & tái lập
- Cài: `pip install numpy pandas matplotlib seaborn scikit-learn xgboost`.
- Đặt `/content/data_v9.csv`; chạy tuần tự các khối mã.
- Artifacts xuất ở thư mục làm việc. `RANDOM_STATE=42` đảm bảo tái lập gần như ổn định.

## 12) Gợi ý cải tiến
- Tầng 1: class weights, LightGBM/CatBoost, calibrate xác suất & điều chỉnh ngưỡng.
- Tầng 2: gom bin theo phân vị, quantile regression (P10/P50/P90), monotonic constraints.
- Feature: tương tác region×hour, service×weekday, demand theo zone×15’; thêm biến về incentive/bán kính matching.
- Triển khai: joblib lưu pipeline; batch job tầng 1→2; monitoring drift & KPI theo vùng/dịch vụ/khung giờ.

## 13) Sơ đồ luồng tổng quát
- Raw CSV → Chuẩn hoá & FE (fillna, epd_value, MPD, margin, is_internal) → preprocessor → RF (label) → lọc pred_label != CANCELLED → gán bin theo ground truth → XGB per-bin + clipping → xuất predictions & summaries.

## 14) Checklist nhanh (QA)
- label NaN→CANCELLED; driver_fee NaN→0; tạo đúng epd_value/MPD/margin/is_internal; preprocessor đúng; RF có stratify & macro-F1; lọc pred_label != CANCELLED trước hồi quy; bins [0,15,30,45,60,90,120,∞); đủ mẫu hoặc có fallback; clipping theo bin; xuất đủ CSV.
