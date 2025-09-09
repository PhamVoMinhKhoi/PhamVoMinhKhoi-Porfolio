# Báo cáo phân tích – Consumer BI (Power BI)

**Phạm vi:** KPI kinh doanh, phễu Signup→Card, hiệu quả theo kênh, adoption theo tính năng (30 ngày), và cohort/retention.  
**Nguồn số:** đọc trực tiếp từ dashboard (làm tròn theo ảnh bạn gửi).

---

## KPIs & doanh thu: điều gì nổi bật
- **GTV ≈ 347.9k**, **Completed Txns ≈ 8.9k** ⇒ **Avg Txn Value ≈ 39.1**.
- **DAU ≈ 996**, **ARPU ≈ 347.9**. Chuỗi **GTV & DAU giảm dần** theo YearMonth từ 2025 về 2024.
- **Cơ cấu GTV theo product:** *pay* (~130k) & *transfer* (~100k) dẫn dắt; *savings* (~60–70k); *bill_pay* (~30k); *invest/topup* (~20k mỗi loại).

---

## Phễu chuyển đổi và chất lượng kênh
- Tổng phễu: **1,000 Signup → 711 KYC Submit (71%) → 707 KYC Approved (~99%) → 574 Activated (81%) → 305 Carded (53%)**.
- **Nút thắt lớn nhất:** *Activated → Card* (**53%**). Nút thắt thứ hai: *Signup → KYC* (**71%**).
- Theo kênh: *referral* có **Activation Rate cao nhất** (~0.63) nhưng quy mô nhỏ; *campaign* thấp nhất (~0.46).  
  **GTV theo kênh:** *organic* (~165k) > *paid* (~95k) > *referral* (~55k) > *campaign* (~30k).

---

## Tính năng: sử dụng & giá trị trong 30 ngày
- **ARPU 30D:** *pay* (~217) & *transfer* (~197) cao nhất; *savings* (~87) trung bình; *bill_pay* (~59) & *topup* (~26) thấp.
- **GTV 30D:** *pay* (~24.8k), *transfer* (~21.9k), *savings* (~10.4k), *bill_pay* (~7.0k), *topup* (~2.7k).
- **Conversion 30D** (ước lượng): *pay* ~2.6%, *transfer* ~2.4%, *savings* ~1.45%, *bill_pay* ~1.0%, *topup* ~0.6%.
- Hàm ý: *pay/transfer* là trụ cột doanh thu ngắn hạn; *bill_pay/topup* còn dư địa kích hoạt tần suất; *savings* nên có cơ chế tích lũy định kỳ.

---

## Cohort & giữ chân: điều học được
- Phần lớn cohort **M+0: 0.7–0.85**, giảm còn **~0.2–0.3** sau vài tháng ⇒ cần tập trung **30 ngày đầu**.
- **Cohort 2024-11** kém bất thường (M+0 ~0.36) ⇒ nên RCA (chiến dịch/phiên bản app/sự cố?).
- **Cohort size** ổn định ~40–60/tháng; **Cohort GTV** biến động theo mùa & chiến dịch.

---

## Ưu tiên triển khai (tác động × tốc độ)

### P0 – Gỡ nút thắt carding & onboarding
- **Nâng Activated → Card** từ **53% → ≥65%**: rút ngắn flow nhận thẻ, nudge in-app cá nhân hóa, incentive card-present lần 1, theo dõi lý do fail.  
  *Ước lượng tác động:* 574 activated × (65%−53%) ≈ **+68 carded** ⇒ **+23.6k GTV** (ARPU ~348).
- **Tăng Signup → KYC** **71% → ≥80%**: prefill + hướng dẫn realtime + nhắc 30′/24h/72h + hỗ trợ live-chat.  
  *Ước lượng tác động:* +90 KYC ⇒ ~+74 activated ⇒ ~+39 carded ⇒ **+13.6k GTV**.

### P1 – Phân bổ kênh theo chất lượng
- Mở rộng **referral**; tạm siết **campaign** để điều tra target/creative/flow.
- Với **paid**: look-alike theo “khả năng card” (propensity), hành trình hậu cài đặt riêng cho paid users.
- Đưa **GTV per Active theo kênh** thành chỉ số điều hành.

### P2 – Tăng tần suất qua tính năng
- Cross-sell từ *pay/transfer* sang *bill_pay/topup*: deep-link sau giao dịch, bundle “trải nghiệm 3 tính năng/7 ngày”.
- **Savings định kỳ:** nhiệm vụ tuần, streak, thưởng nhỏ để tạo thói quen.

### P3 – Giữ chân 30 ngày đầu
- **Chương trình 30 ngày:** Day 1–3 KYC/Activate → Day 4–10 1st pay → Day 11–20 transfer/savings → Day 21–30 bill/topup (push/email có điều kiện).
- RCA cohort yếu (2024-11) theo kênh/thiết bị/phiên bản/sự cố.

---

## Theo dõi vận hành (nên bổ sung)
- **Activation Rate & Carding Rate by Channel** (daily/weekly).  
- **Time to KYC/Activate (P50/P90)**, bảng lý do fail.  
- **GTV per Active** theo kênh & cohort; readout cho các A/B (nudge card, referral incentive, deep-link).

