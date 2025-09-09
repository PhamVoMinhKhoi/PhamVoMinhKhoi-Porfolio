# Data Dictionary (Synthetic Fintech Dataset)

## customers.csv
- customer_id, signup_date
- kyc_submit_date, kyc_approved_date, activate_date, card_activate_date (nullable)
- channel {organic, paid, referral, campaign}
- region {HCMC, Hanoi, DaNang, CanTho, HaiPhong}
- age_group {18-24, 25-34, 35-44, 45-54, 55+}
- gender {M, F, Other}
- risk_segment {low, medium, high}

## app_events.csv
- event_id, customer_id, event_time, event_name
- channel, device, session_id

## transactions.csv
- txn_id, customer_id, txn_time, product, amount, fee, merchant_category, card_present, status, region, channel

## date_dim.csv
- date, year, month, day, quarter, week, month_name, is_weekend