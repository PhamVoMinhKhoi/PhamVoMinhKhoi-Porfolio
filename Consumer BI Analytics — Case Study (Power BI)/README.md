# Consumer BI Analytics — Case Study (Power BI)

> Bộ phân tích cho **Data Analyst**: KPI kinh doanh, Funnel (Signup→Card), hiệu quả theo Channel, Feature Adoption (30D), Cohort/Retention. Dùng **Power BI** với dữ liệu CSV sẵn có.

---

## 1) Mục tiêu phân tích

- Theo dõi **KPI cốt lõi**: `GTV`, `Fee`, `DAU/MAU`, `ARPU`, `Completed Txns`.
- Phân tích **Funnel**: `Signed Up → KYC Submitted → KYC Approved → Activated → Card Activated`.
- Đo **hiệu quả kênh** (channel) & `GTV per Active`.
- **Feature adoption (30D)**: Users, Conversion, ARPU, GTV theo feature.
- **Cohort/Retention** theo tháng signup (heatmap % active theo tháng hoạt động) + đường **Retention by Offset (M+0…M+12)**.

---

## 2) Dữ liệu đầu vào

- `customers.csv`
- `app_events.csv`
- `transactions.csv`
- `date_dim.csv`
- `Fintech Dash.pbix` (Power BI)
- `powerbi_measures.txt` (DAX tổng hợp)
- `data_dictionary.md` (mô tả schema)

**Lưu ý**
- `transactions.status = "COMPLETED"` để tính chỉ số liên quan doanh thu.
- `app_events.event_time` dùng cho DAU/MAU & retention.
- Các cột ngày `*_date` trong `customers` dùng cho các mốc Funnel.

---

## 3) Định nghĩa KPI (thống nhất giữa DAX & Dashboard)

### 3.1 Người dùng & hoạt động
- **Users** = `DISTINCTCOUNT(customers[customer_id])`
- **DAU** = `DISTINCTCOUNT(app_events[customer_id])` trên 1 ngày
- **MAU (30D)** = `DISTINCTCOUNT(app_events[customer_id])` trong cửa sổ 30 ngày lăn
- **ARPU** = `GTV / Users`

### 3.2 Giao dịch
- **GTV** = `SUM(transactions[amount])` (*status* = `COMPLETED`)
- **Fee** = `SUM(transactions[fee])` (*status* = `COMPLETED`)
- **Completed Txns** = `COUNTROWS(transactions)` (*status* = `COMPLETED`)
- **Avg Txn Value** = `GTV / Completed Txns`

### 3.3 Funnel (theo mốc ngày ở `customers`)
- **Signed Up / KYC Submitted / KYC Approved / Activated / Card Activated**  
  = `DISTINCTCOUNT(customers[customer_id])` **kích hoạt theo cột ngày tương ứng** (qua `USERELATIONSHIP` hoặc `TREATAS`)
- **Signup→KYC %** = `KYC Submitted / Signed Up`  
- **KYC→Activate %** = `Activated / KYC Approved`  
- **Activate→Card %** = `Card Activated / Activated`

### 3.4 Feature (30D)
- **Feature Users 30D** = `DISTINCTCOUNT(app_events[customer_id])` với `event_name = feature` trong 30D
- **Feature Txn Users 30D** = `DISTINCTCOUNT(transactions[customer_id])` với `product = map(feature)`, `status="COMPLETED"`, 30D
- **Feature GTV 30D** = `SUM(transactions[amount])` trong cùng filter
- **Feature Conversion % 30D** = `Txn Users 30D / Users 30D`
- **Feature ARPU 30D** = `GTV 30D / Users 30D`

---

## 4) Mô hình quan hệ (Power BI)

- **Date table:** `date_dim` (Mark as date table; key = `date`)
- **Active (1→*)**  
  `date_dim[date] → app_events[event_date]`  
  `date_dim[date] → transactions[txn_date]`  
  `customers[customer_id] → app_events[customer_id]`  
  `customers[customer_id] → transactions[customer_id]`
- **Inactive (1→*) cho Funnel**  
  `date_dim[date] → customers[signup_date] / kyc_submit_date / kyc_approved_date / activate_date / card_activate_date`
- **Cross filter:** *Single* (tránh ambiguous).  
- **Slicer ngày:** `date_dim[date]` kiểu *Between* hoặc *Relative*; **Sync** toàn report.

---

## 5) Thiết kế Dashboard (4 trang)

### Page 1 — **Tổng quan KPI & GTV**
- **Cards:** `GTV`, `Fee`, `ARPU`, `DAU`, `Completed Txns`, `Avg Txn Value`
- **Combo (cột+đường):** `GTV` (cột) & `DAU` (line) theo ngày/tháng
- **Bar:** `GTV` theo `transactions[product]`
- **Table:** **Top 20** customers by `GTV` (dựa `Rank by GTV`)

### Page 2 — **Funnel & Channel**
- **Funnel:** `Stages[Stage]` + measure `Funnel Value`
- **3 Cards tỉ lệ:** `Signup→KYC %`, `KYC→Activate %`, `Activate→Card %`
- **Activation by Channel (combo):** `Activated` (cột) + `Activation Rate = Activated/Signed Up` (line)
- **GTV by Channel (bar):** `GTV` + tooltip `Active Users`, `GTV per Active`
- **Thời gian chuyển đổi (tuỳ chọn):** `Avg Days Signup→KYC`, `Avg Days KYC→Activate`

### Page 3 — **Feature Adoption (30D)**
- **Users by Feature (30D)**  
- **Conversion % (cột) & ARPU (line) theo Feature**  
- **GTV by Feature (30D)**  
- **Bảng chi tiết:** Users 30D, Txn Users 30D, Conversion %, GTV 30D, ARPU 30D  
  *(Map `event_name → product`, ví dụ `savings_deposit → savings`)*

### Page 4 — **Cohort / Retention**
- **Heatmap:** Rows = Cohort (YYYY-MM signup), Cols = Activity month, Values = `% retention`  
  *(Conditional formatting → Background color → Color scale 0→1)*
- **Line:** **Retention by Offset (M+0…M+12)**
- **Cohort Size / Cohort GTV**
- **Activation Time Distribution** (bucket 0–1d, 2–3d, 4–7d, 8–14d, 15–30d, >30d)

---

## 6) DAX mẫu (copy nhanh)

```DAX
-- KPI giao dịch
GTV :=
CALCULATE( SUM(transactions[amount]), transactions[status] = "COMPLETED" )

Fee :=
CALCULATE( SUM(transactions[fee]), transactions[status] = "COMPLETED" )

Completed Txns :=
CALCULATE( COUNTROWS(transactions), transactions[status] = "COMPLETED" )

Avg Txn Value := DIVIDE( [GTV], [Completed Txns] )
ARPU := DIVIDE( [GTV], DISTINCTCOUNT(customers[customer_id]) )

-- Funnel (kích hoạt các quan hệ Inactive bằng USERELATIONSHIP)
Signed Up :=
CALCULATE( DISTINCTCOUNT(customers[customer_id]),
    USERELATIONSHIP(customers[signup_date], date_dim[date]),
    FILTER(customers, NOT ISBLANK(customers[signup_date])) )

KYC Submitted :=
CALCULATE( DISTINCTCOUNT(customers[customer_id]),
    USERELATIONSHIP(customers[kyc_submit_date], date_dim[date]),
    FILTER(customers, NOT ISBLANK(customers[kyc_submit_date])) )

KYC Approved :=
CALCULATE( DISTINCTCOUNT(customers[customer_id]),
    USERELATIONSHIP(customers[kyc_approved_date], date_dim[date]),
    FILTER(customers, NOT ISBLANK(customers[kyc_approved_date])) )

Activated :=
CALCULATE( DISTINCTCOUNT(customers[customer_id]),
    USERELATIONSHIP(customers[activate_date], date_dim[date]),
    FILTER(customers, NOT ISBLANK(customers[activate_date])) )

Card Activated :=
CALCULATE( DISTINCTCOUNT(customers[customer_id]),
    USERELATIONSHIP(customers[card_activate_date], date_dim[date]),
    FILTER(customers, NOT ISBLANK(customers[card_activate_date])) )

Signup → KYC %     := DIVIDE([KYC Submitted], [Signed Up])
KYC → Activate %   := DIVIDE([Activated], [KYC Approved])
Activate → Card %  := DIVIDE([Card Activated], [Activated])
Activation Rate    := DIVIDE([Activated], [Signed Up])

-- Feature 30D (ví dụ 'pay'; các feature khác thay event/product tương ứng)
Feature Users 30D (pay) :=
CALCULATE( DISTINCTCOUNT(app_events[customer_id]),
           DATESINPERIOD(date_dim[date], MAX(date_dim[date]), -30, DAY),
           app_events[event_name] = "pay" )

Feature Txn Users 30D (pay) :=
CALCULATE( DISTINCTCOUNT(transactions[customer_id]),
           DATESINPERIOD(date_dim[date], MAX(date_dim[date]), -30, DAY),
           transactions[status] = "COMPLETED",
           transactions[product] = "pay" )

Feature GTV 30D (pay) :=
CALCULATE( SUM(transactions[amount]),
           DATESINPERIOD(date_dim[date], MAX(date_dim[date]), -30, DAY),
           transactions[status] = "COMPLETED",
           transactions[product] = "pay" )

Feature Conversion % 30D (pay) := DIVIDE([Feature Txn Users 30D (pay)], [Feature Users 30D (pay)])
Feature ARPU 30D (pay)        := DIVIDE([Feature GTV 30D (pay)], [Feature Users 30D (pay)])
