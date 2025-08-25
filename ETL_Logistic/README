# 🚚 Amazon Logistics – ETL + KPI Views (Python → Google Sheets → Looker Studio)

---

## 🌐 Giới thiệu (Tiếng Việt)

### 1. Tại sao tạo ra project này?
Trong lĩnh vực **logistics**, dữ liệu thường nằm rải rác ở nhiều bảng (khách hàng, đơn hàng, nhân viên, thanh toán, trạng thái...).  
Mục tiêu của project là:
- **Chuẩn hóa dữ liệu** từ nhiều bảng → tạo thành **mô hình FACT/DIM** dễ phân tích.  
- **Tự động hóa ETL** bằng Python → giảm thao tác thủ công.  
- **Kết nối Google Sheets & Looker Studio** để xây dựng **dashboard động** cho quản trị.

### 2. Dữ liệu sử dụng
- Bộ dữ liệu giả lập gồm 7 bảng: `Customer`, `Employee_Details`, `employee_manages_shipment`, `Membership`, `Payment_Details`, `Shipment_Details`, `Status`.

### 3. Các bước xử lý
1. **Python (pandas)**: join 7 bảng → `fact_shipments`.  
2. Sinh ra các bảng KPI:
   - `kpi_overview`
   - `kpi_by_region`
   - `kpi_by_employee`
   - `kpi_by_membership`
   - `kpi_by_payment`
   - `kpi_daily_status`
3. **Push** các bảng này lên Google Sheets (qua Service Account).  
4. **Looker Studio** kết nối Google Sheets để dựng dashboard.

### 4. Kết quả đạt được
- Một pipeline **tự động**: từ raw CSV → Google Sheets → Dashboard.  
- Dashboard trực quan, theo dõi:
  - Tổng số đơn, tỉ lệ giao đúng hạn, thời gian giao trung bình, doanh thu.  
  - Phân tích theo khu vực, nhân viên, loại membership, phương thức thanh toán.  
  - Xu hướng trạng thái đơn theo ngày.  

### 5. Ý nghĩa
- Thể hiện năng lực **ETL + Data Modeling + Data Visualization** trong bối cảnh logistics.  
- Làm case study tốt để đưa vào **portfolio Business Analyst / Data Analyst**.

---

## 🌐 Introduction (English)

### 1. Why this project?
In the **logistics domain**, data is usually spread across multiple tables (customers, shipments, employees, payments, statuses...).  
The project aims to:
- **Standardize and join data** into a clean **FACT/DIM schema** for analysis.  
- **Automate ETL** with Python to minimize manual steps.  
- **Integrate with Google Sheets & Looker Studio** for **interactive dashboards**.

### 2. Data used
- Synthetic dataset with 7 tables: `Customer`, `Employee_Details`, `employee_manages_shipment`, `Membership`, `Payment_Details`, `Shipment_Details`, `Status`.

### 3. Workflow
1. **Python (pandas)**: join 7 tables → `fact_shipments`.  
2. Generate KPI views:
   - `kpi_overview`
   - `kpi_by_region`
   - `kpi_by_employee`
   - `kpi_by_membership`
   - `kpi_by_payment`
   - `kpi_daily_status`
3. **Push** these views to Google Sheets via Service Account.  
4. **Looker Studio** connects to Google Sheets → build dashboards.

### 4. Results
- A fully automated pipeline: raw CSV → Google Sheets → Dashboard.  
- Interactive dashboards showing:
  - Total orders, on-time delivery rate, avg. delivery days, revenue.  
  - Performance by region, employee, membership type, payment method.  
  - Daily order status trends.

### 5. Significance
- Demonstrates **ETL, Data Modeling, and Data Visualization** skills in a logistics context.  
- Serves as a strong case study for a **Business Analyst / Data Analyst portfolio**.

---

## 📂 Project Structure
- `scripts/join_and_build_views_pdf.py` – join 7 CSV → fact + KPI views
- `scripts/push_to_sheet.py` – push `outputs/*.csv` → Google Sheets
- `data/` – raw CSV (not committed)
- `outputs/` – ETL results (ignored in git)
- `.env` – contains `GOOGLE_SHEET_KEY` (ignored in git)
- `.env.example` – template for env variables
- `service_account.json` – Google credentials (ignored in git)

---

## ▶️ How to Run
```bash
# 1. Setup virtual env
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt

# 2. Join & build KPI
python scripts/join_and_build_views_pdf.py --data_dir data --out_dir outputs

# 3. Push to Google Sheets
python scripts/push_to_sheet.py --out_dir outputs
