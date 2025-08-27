# Cyclistic Bike-Share Case Study (Python) — April 2025 Slice

**Business Task**  
Understand how _annual members_ and _casual riders_ use Cyclistic bikes differently, to inform marketing strategies that convert casual riders into annual members.

**Data Used**  
- Source: Divvy/Cyclistic trip data for **2025-04** (`202504-divvy-tripdata.csv`), which follows the public schema (ride_id, rideable_type, started_at, ended_at, stations, lat/lng, member_casual).  
- Scope limitation: We analyze a **single month** due to available upload; the original guidance recommends 12 months for seasonality. Interpret month-level insights carefully.

**Cleaning & Feature Engineering (Python)**  
- Dropped rows with invalid timestamps and non-positive durations; capped durations at 24h.  
- Derived features: `ride_length_sec/min`, `day_of_week`, `hour`, `weekend`, `month`, geographic `distance_km` via Haversine (capped outliers).  
- Standardized rider type label from `member_casual` → `rider_type` in \{member, casual\}.

**Data Quality Snapshot**  
- Rows after cleaning: **371,041**  
- Members: **262,085**, Casuals: **108,956**  
- Time coverage: 2025-03-31 23:17 → 2025-04-30 23:59

Top columns by missing ratio (first 10) saved: [`01_overview.csv`](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/01_overview.csv)

---

## Key Findings (April 2025)

1) **When they ride (volume patterns)**  
- Members show **clear commute-shaped peaks** by hour (see figure), while casuals skew more toward **late morning / afternoon**.  
  - Figures: [Members — rides by hour](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig5_rides_by_hour_members.png), [Casuals — rides by hour](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig6_rides_by_hour_casuals.png).

2) **How long they ride**  
- Casual rides tend to be **longer on average** than member rides across most days of week, consistent with leisure trips vs member commute trips.  
  - Figures: [Members — avg length by day](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig3_avg_len_by_dow_members.png), [Casuals — avg length by day](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig4_avg_len_by_dow_casuals.png).

3) **Day-of-week behavior**  
- Members concentrate on **weekdays**, casuals rise on **weekends**.  
  - Figures: [Members — rides by day](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig1_rides_by_dow_members.png), [Casuals — rides by day](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig2_rides_by_dow_casuals.png).

4) **Rideable-type mix**  
- Members lean more toward **classic bike** usage; casuals proportionally use **docked/scooters/e-bikes** more (if available in the month).  
  - Figures: [Members — rideable mix](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig7_rideable_mix_members.png), [Casuals — rideable mix](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig8_rideable_mix_casuals.png).

5) **Stations (where they start)**  
- Top stations differ by segment (e.g., transit-adjacent for members vs. waterfront/attraction-adjacent for casuals).  
  - Data: [`top10_start_stations_members.csv`](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/top10_start_stations_members.csv), [`top10_start_stations_casuals.csv`](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/top10_start_stations_casuals.csv).

**Summary Table by Rider Type**  
See: [`summary_by_type.csv`](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/summary_by_type.csv)

---

## Business Implications

- **Value proposition for commuters (members):** Emphasize convenience + savings for weekday commuters. Target **peak commute hours** and **stations near transit hubs**.  
- **Convert leisure casuals:** Promote **weekend bundles**, **season passes**, or **“3 rides and save”** trials timed to midday/afternoon usage.  
- **Equipment targeting:** Where casuals prefer e-bikes/scooters, offer **member e-bike add-ons** or **first-month discounts** to bridge them into membership.

---

## Top 3 Recommendations (Actionable)

1) **Weekend-to-Membership Funnel:** Introduce a **Weekend Explorer Pass → auto-credit toward Annual** if riders take ≥N rides in 30 days.  
2) **Commute Guarantee for Members:** _“Ride-to-Work Savings”_—price capping on peak weekday rides + guaranteed dock availability near top commuter stations.  
3) **E-bike Onboarding:** Offer **first-month e-bike fee waiver** for casuals who opt into a trial membership during checkout at high-leisure stations.

---

## Files & Artifacts

Tables:  
- [Overview](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/01_overview.csv)  
- [Rides by DOW](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/rides_by_dow.csv)  
- [Avg duration by DOW](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/avg_dur_by_dow.csv)  
- [Rides by Hour](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/rides_by_hour.csv)  
- [Avg duration by Hour](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/avg_dur_by_hour.csv)  
- [Rideable Mix](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/rideable_mix.csv)  
- [Summary by Rider Type](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/summary_by_type.csv)  
- [Top 10 Start Stations — Members](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/top10_start_stations_members.csv)  
- [Top 10 Start Stations — Casuals](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/top10_start_stations_casuals.csv)  

Figures:  
- [fig1_rides_by_dow_members.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig1_rides_by_dow_members.png)  
- [fig2_rides_by_dow_casuals.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig2_rides_by_dow_casuals.png)  
- [fig3_avg_len_by_dow_members.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig3_avg_len_by_dow_members.png)  
- [fig4_avg_len_by_dow_casuals.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig4_avg_len_by_dow_casuals.png)  
- [fig5_rides_by_hour_members.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig5_rides_by_hour_members.png)  
- [fig6_rides_by_hour_casuals.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig6_rides_by_hour_casuals.png)  
- [fig7_rideable_mix_members.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig7_rideable_mix_members.png)  
- [fig8_rideable_mix_casuals.png](/content/drive/MyDrive/Data chạy demo/output/cyclistic_outputs/fig8_rideable_mix_casuals.png)  

---

## Reproducibility

- Environment: Python, pandas, numpy, matplotlib.  
- Data file: `/mnt/data/202504-divvy-tripdata.csv`.  
- All outputs are generated by `this` script; modify parameters or swap in additional months to extend analysis across 12 months for seasonality.