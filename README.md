# PhamVoMinhKhoi-Porfolio
## About me
About Me  Hi! I’m Khoi Pham, passionate about Data Science &amp; Business Analysis. I love working with data to uncover insights and solve real-world problems in logistics, finance, and business operations. this is my portfolio. I love investigating different types of data, discovering insights, and representing it with beautiful visuals. I have background in logistics, financial and business operation analysis

You can see more infomation in my **CV**

This repository was created to showcase my analytical and technical skills (Excel, Python, SQL, Power BI, PowerPoint, and others).
## Contents
* [About me](#about-me)
* [Portfolio Projects](#portfolio-projects)
* [Study projects](#study-projects)
* [Certificates](#certificates)
* [Contacts](#contacts)

## Portfolio Projects
### [ETL Logistics Dashboard](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/tree/main/ETL_Logistic)<br>
**Description**: The project’s goal is to build a logistics analytics pipeline from raw CSV datasets (Customer, Employee, Shipment, Payment, Membership, Status). The workflow covers the full ETL cycle: data cleaning and joins in Python, exporting fact & KPI views, pushing transformed data to Google Sheets via Service Account, and building interactive dashboards in Looker Studio for business insights.<br>
**Code**: [Python ETL Scripts (join_and_build_views.py, push_to_sheet.py)](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/tree/main/ETL_Logistic/script)<br>
**Dashboard**: [Looker Studio Dashboard (Overview, SLA Monitoring, Membership & Customer Insights)](https://lookerstudio.google.com/reporting/ea209ef8-9b0b-4c58-8d86-0868a1b27f44)<br>
**Original dataset**: [Custom logistics dataset (Orders, Customers, Shipments, Payments, Memberships)](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/tree/main/ETL_Logistic/data_raw)<br>
**Skills**: ETL design, data modeling, data visualization, dashboarding, analytical thinking<br>
**Hard skills**: Python (Pandas, Pathlib, dotenv), Google Sheets API (gspread, Service Accounts), GitHub, Looker Studio<br>
**Results**: Delivered a fully automated pipeline (raw CSV → Google Sheets → Dashboard).<br>
**Overview Page**: Showed order volume, cancellations, revenue concentration, and delivery performance.<br>
**SLA Monitoring Page**: Identified severe SLA compliance issues (late vs on-time), weekday delivery imbalances.<br>
**Membership & Customer Page**: Revealed 90%+ revenue dependency on one membership segment and universal SLA failure across tiers.<br>
Insights provide actionable recommendations on reducing cancellations, balancing capacity, SLA tiering, and customer diversification.<br>

### [Pooling Prediction Pipeline] 
**Description:** The project’s goal is to build a predictive pipeline for logistics pooling using Random Forest Classification and Bin-wise XGB Regression. The workflow covers the full ML cycle: feature engineering, preprocessing, classification, regression, evaluation, and insights generation for operational optimization.<br>
**Code:** Python ML Scripts (RandomForest + XGB, preprocessing, evaluation)<br>
**Dataset:** Custom logistics dataset (Orders, Service, Region, Distance, Fees, Pooling times)<br>
**Skills:** Machine learning pipeline design, feature engineering, data visualization, operational analytics<br>
**Hard skills:** Python (Pandas, scikit-learn, XGBoost), feature engineering, preprocessing (OHE, StandardScaler), data visualization (Matplotlib, Seaborn)<br>
**Results:** Built a 2-step pipeline:<br>
**Classification:** Predicted order labels (INTERNAL, POOL, CANCELLED).<br>
**Regression:** Predicted first pooling times within 15-min bins to reduce outliers.<br>
Delivered realistic outputs reflecting business logic and imbalance handling.<br>
**Classification Page:** Showed probability of orders being INTERNAL vs CANCELLED, improving resource allocation.<br>
**Pooling Time Page:** Identified groups of orders with fast (0–15 mins) vs slow (>60 mins) pooling.<br>
**Insights:** Revealed common characteristics of slow vs fast pooling orders, including distance, demand, COD, service type, and region.<br>
Insights provide actionable recommendations on anticipating cancellations, rebalancing pooling configuration, and improving driver acceptance rates.<br>


## Study Projects
## Certificates
* [MOS - Microsoft Office Specialist](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/tree/main/Certification/MOS) - Microsoft - 2023
* [Google Data Analytics Certificate](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/blob/main/Certification/Google%20Data%20Analytics.pdf) - Coursera - Google, 2024
* [Google Advanced Data Analytics](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/blob/main/Certification/Google%20Advanced%20Data%20Analytics.pdf) - Coursera - Google, 2024
* [Data Visualization and Communication with Tableau](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/blob/main/Certification/Data%20Visualization%20and%20Communication%20with%20Tableau.pdf) - Coursera - Duke University, 2024
* [Mastering Data Analysis in Excel](https://github.com/PhamVoMinhKhoi/PhamVoMinhKhoi-Porfolio/blob/main/Certification/Mastering%20Data%20Analysis%20in%20Excel.pdf)  - Coursera - Duke University, 2024
## Contacts
* Linkedin: [https://www.linkedin.com/in/pavelliaoshka](https://www.linkedin.com/in/khoi-pham-vo-minh/)
* Mobile: 0848883817
* Email: pvmkhoi14042003@gmail.com
* Telegram: @minhkhoi1404
