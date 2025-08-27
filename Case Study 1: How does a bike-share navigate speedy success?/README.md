# Cyclistic Bike-Share Case Study (April 2025)

This project analyzes Divvy/Cyclistic bike-share trip data (April 2025) to explore how **annual members** and **casual riders** use the service differently. The case study follows the Google Data Analytics Capstone guidelines: *Ask, Prepare, Process, Analyze, Share, Act.*

---

## 1. Business Task
Identify usage differences between members and casual riders, and generate insights to support marketing strategies that convert **casual riders into annual members**.

---

## 2. Data Source
- File used: `202504-divvy-tripdata.csv` (April 2025 monthly dataset).
- Publicly available Divvy/Cyclistic data (structured schema with ride_id, rideable_type, timestamps, stations, lat/lng, and rider type).

---

## 3. Data Preparation & Cleaning
- Parsed timestamps (`started_at`, `ended_at`) and removed invalid or missing values.
- Dropped rides with non-positive durations and capped outliers (>24h).
- Derived new features:
  - **ride_length_min**
  - **day_of_week**, **hour**, **weekend indicator**
  - **distance_km** using the Haversine formula (filtered out >100 km).
- Standardized rider type: `member_casual` → `rider_type` (values: *member*, *casual*).

---

## 4. Analysis Conducted
Key comparisons were made between members and casuals:

1. **Volume by day of week and hour**  
   - Members: commute-oriented patterns (weekday peaks, morning/evening rush hours).  
   - Casuals: weekend-oriented, midday/afternoon peaks.

2. **Ride length**  
   - Casuals ride longer on average → leisure behavior.  
   - Members ride shorter trips → commuting.

3. **Rideable type mix**  
   - Members: higher usage of classic bikes.  
   - Casuals: relatively more e-bikes/scooters.

4. **Station popularity**  
   - Members start more often at transit-adjacent stations.  
   - Casuals start more often at attraction/waterfront stations.

---

## 5. Results & Outputs
Generated outputs include:

- **Tables**: ride counts by day/hour, average duration, rideable mix, top stations.  
- **Visualizations**: bar/line charts for usage by day, hour, ride length, and rideable type.  
- **Summary statistics**: by rider type (rides, avg/median duration, distance).

All outputs are saved under `/mnt/data/cyclistic_outputs/`.

---

## 6. Insights & Recommendations

- **Members = commuters**: highlight reliability and cost savings.  
- **Casuals = leisure riders**: target weekend/afternoon users with short-term passes.  
- **E-bike opportunity**: trial memberships with e-bike perks can attract casual riders.

**Top 3 recommendations:**  
1. Weekend Explorer Pass → credit toward annual membership.  
2. Commute Guarantee: capped pricing + dock availability for members.  
3. E-bike Onboarding: first-month fee waiver for casuals upgrading to members.

---

## 7. Limitations & Next Steps
- Only one month (April 2025) analyzed → no seasonality captured.  
- For stronger insights, extend analysis to a **12-month dataset**.  
- Future work: segmentation by weather, region, or trip purpose.

---

## 8. Tools Used
- **Python** (pandas, numpy, matplotlib)  
- **Outputs**: CSV summary tables, PNG figures, Markdown report

---

## Reproducibility
Run the included Python script with the dataset to regenerate all results.  
Adjust the file path to include multiple months for extended analysis.

