# NYC-Yellow-Taxi-Data-Analysis-2016-
## Introduction

This project presents a comprehensive data analytics case study on **New York City’s Yellow Taxi rides in 2016**. The analysis aims to uncover patterns in ridership, tipping behavior, and temporal demand, supporting operational and strategic decision-making for mobility, urban planning, and product teams.

### 🔗 Data Source

- **Dataset:** [NYC TLC Trip Record Data - Yellow Taxi, 2016 (Google BigQuery Public)](https://console.cloud.google.com/marketplace/product/city-of-new-york/nyc-tlc-trips)
- **Table Name:** `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
- **Data Size:** ~132 million rows, 21 columns (~20GB raw storage)

### ⚙️ Tools & Stack

- **Google BigQuery:**  
  - Used for direct SQL analysis on large-scale public taxi trip data (cloud-based, petabyte-scale).  
  - Enabled efficient querying and aggregation without local resource constraints.

- **Python (pandas, matplotlib):**  
  - Used for advanced data manipulation, visualizations, and custom analysis outside the limits of SQL.  
  - Data exported from BigQuery and further explored in Jupyter Notebook environment.
> 📓 **For full Python code, visualizations, and calculation formulas, see the companion Jupyter Notebook:**  
> [NYC-Yellow-Taxi-Data-Analysis-2016-visualizations.ipynb](./NYC-Yellow-Taxi-Data-Analysis-2016-visualizations.ipynb)

- **SQL:**  
  - All business questions were answered with dedicated, reproducible SQL queries.  
  - Query logic is transparently shared in each section of the report.

- **GitHub:**  
  - All code, visualizations, and documentation are versioned and shared for transparency and reproducibility.

### 🗂️ Dataset Features

- **Trip Metadata:** Pickup/dropoff timestamps, locations, vendor info  
- **Fares & Payments:** Fare amount, tip amount, payment type, surcharges  
- **Trip Characteristics:** Distance, duration, passenger count

### 📊 Analysis Scope

The study covers:
- Daily, hourly, weekly, and monthly demand patterns
- Tip behavior by time, payment method, distance, duration, and fare
- Average ride durations and trip characteristics
- Actionable insights for fleet operations, driver incentives, and payment UX

### 💡 Why BigQuery?

- The **NYC Yellow Taxi dataset** is too large for in-memory analysis on standard laptops (~20GB, 130M+ rows).
- **BigQuery** enables scalable, fast SQL queries on this open dataset, removing the limits of RAM/CPU and allowing flexible data exploration.
- Combined with Python for deep-dives and custom visualizations, this hybrid workflow is ideal for large real-world datasets.

---

### 👤 Project Owner

- **Prepared by:** Mert Ovet  
- **LinkedIn:** [https://www.linkedin.com/in/mertovet/](https://www.linkedin.com/in/mertovet/)

---
## 1. Daily Trip Volume Analysis

### 🔍 Business Question
On which dates does yellow taxi usage spike in NYC?  
Is demand steady, or are there distinct busy days?

### 📎 SQL Query
```sql
SELECT
  DATE(pickup_datetime) AS trip_date,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
GROUP BY trip_date
ORDER BY trip_date
LIMIT 100
```

![image](https://github.com/user-attachments/assets/e7005c85-03bd-4867-94f0-d82b11cf77e8)

**Key Insight:**
Taxi demand is relatively stable but dips after major holidays.


Preliminary analysis shows higher trip counts on Fridays and Saturdays.


Average daily rides in 2016 were ~380,000.

---
## 2. Hourly Trip Volume Analysis

### 🔍 Business Question
Which hours of the day see peak yellow taxi demand?  
Is demand highest during morning commutes or evening hours?

### 📎 SQL Query
```sql
SELECT
  EXTRACT(HOUR FROM pickup_datetime) AS pickup_hour,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
GROUP BY pickup_hour
ORDER BY pickup_hour
```

![image](https://github.com/user-attachments/assets/2c649bfe-0b1e-4727-aa5f-c45a2f50fc83)

**Key Insight:**
Peak demand consistently occurs between 6 PM and 10 PM, especially on Fridays and Saturdays.

Early morning hours (2–5 AM) are the least busy, with late-night activity on weekends due to nightlife.

These insights help inform driver shift planning, fleet allocation, and pricing strategies.

## 3. Tip Behavior Analysis

### 3.1 Average Tip Rate

#### 🔍 Business Question
What is the overall average tip rate?  
Under what conditions do riders tip more?

#### 📎 SQL Query
```sql
SELECT
  ROUND(AVG(tip_amount / fare_amount), 3) AS avg_tip_rate
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE fare_amount > 0 AND tip_amount IS NOT NULL
```
**Key Insight:**

The overall average tip rate is around 16.5%.

Tipping behavior varies based on hour, payment method, and trip characteristics.

### 3.2 Tip Rate by Hour

#### 🔍 Business Question
During which hours are tips highest?  
Is there a time-of-day effect on tipping?

#### 📎 SQL Query
```sql
SELECT
  EXTRACT(HOUR FROM pickup_datetime) AS pickup_hour,
  ROUND(AVG(tip_amount / fare_amount), 3) AS avg_tip_rate
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE fare_amount > 0 AND tip_amount IS NOT NULL
GROUP BY pickup_hour
ORDER BY pickup_hour
```

![image](https://github.com/user-attachments/assets/731ecda6-1154-40c3-a8d4-fb0a823c85be)

**Key Insight:**

Tip rates peak at night (01:00–03:00), reaching around 19%.

Daytime hours (12:00–16:00) have lower tip rates, around 15–16%.

Nighttime social activity, such as post-nightlife rides, appears to boost tipping behavior.

Understanding these patterns can inform driver shift planning and optimize driver earnings during peak tip hours.

### 3.3 Tip Rate by Payment Type

#### 🔍 Business Question
Do riders tip more with credit cards vs cash?  
How does payment method influence tipping?

#### 📎 SQL Query
```sql
SELECT
  CASE
    WHEN CAST(payment_type AS INT64) = 1 THEN 'Credit Card'
    WHEN CAST(payment_type AS INT64) = 2 THEN 'Cash'
    WHEN CAST(payment_type AS INT64) = 3 THEN 'No Charge'
    WHEN CAST(payment_type AS INT64) = 4 THEN 'Dispute'
    WHEN CAST(payment_type AS INT64) = 5 THEN 'Unknown'
    WHEN CAST(payment_type AS INT64) = 6 THEN 'Voided'
    ELSE 'Other/Null'
  END AS payment_method,
  ROUND(AVG(tip_amount / fare_amount), 3) AS avg_tip_rate,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE fare_amount > 0 AND tip_amount IS NOT NULL
GROUP BY payment_method
ORDER BY avg_tip_rate DESC
```
![image](https://github.com/user-attachments/assets/4609cb79-86ff-42de-97ac-5e13184e0795)

**Key Insight:**

Credit card payments result in the highest average tip rates (~18.5%).

Cash rides (over 44 million trips) yield nearly zero tips.

Digital payment UX and default tip prompts encourage higher tipping behavior for card payments.

The design of the payment interface directly impacts driver earnings and tip frequency.

### 3.4 Tip Rate by Trip Distance

#### 🔍 Business Question
Does trip length affect tipping generosity?  
Are short or long rides tipped more generously?

#### 📎 SQL Query
```sql
SELECT
  CASE
    WHEN trip_distance BETWEEN 0 AND 1 THEN '0-1 miles'
    WHEN trip_distance BETWEEN 1 AND 2 THEN '1-2 miles'
    WHEN trip_distance BETWEEN 2 AND 4 THEN '2-4 miles'
    WHEN trip_distance BETWEEN 4 AND 6 THEN '4-6 miles'
    WHEN trip_distance BETWEEN 6 AND 10 THEN '6-10 miles'
    WHEN trip_distance > 10 THEN '10+ miles'
    ELSE 'Unknown'
  END AS distance_bucket,
  ROUND(AVG(tip_amount / fare_amount), 3) AS avg_tip_rate,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE fare_amount > 0 AND tip_amount IS NOT NULL
GROUP BY distance_bucket
ORDER BY avg_tip_rate DESC
```

![image](https://github.com/user-attachments/assets/d7b8674a-e292-47ee-994f-b64764ff65af)

**Key Insight:**

Short trips (0–1 mile) have the highest average tip rates (~22.5%), likely due to minimum tip floors and “round-up” behavior on small fares.

Long rides (10+ miles) also see strong average tip rates (~21.5%).

Mid-range rides (1–6 miles) have the lowest average tip percentages (about 14%).

This trend can guide driver routing, pricing strategies, and digital tip interface design.

## 4. Average Trip Duration

### 🔍 Business Question
What is the average ride duration?  
Are most trips short or long?

### 📎 SQL Query
```sql
SELECT
  ROUND(AVG(TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, MINUTE)), 2) AS avg_trip_duration_min
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE dropoff_datetime IS NOT NULL
  AND pickup_datetime IS NOT NULL
  AND TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, MINUTE) BETWEEN 1 AND 180
```

![image](https://github.com/user-attachments/assets/b8c55d62-11a2-4c61-bee8-928dd4d537cc)

**Key Insight:**

The average ride duration is about 13.7 minutes.

Most rides are short, likely within Manhattan or adjacent boroughs.

Outlier trips (under 1 minute or over 180 minutes) are excluded to ensure realistic averages.

This insight helps understand NYC traffic patterns, fare structures, and guides driver shift optimization.

## 5. Temporal Patterns: Weekly & Monthly Trends

### 5.1 Trips by Day of Week

#### 🔍 Business Question
Are weekends or weekdays busier for yellow taxis?

#### 📎 SQL Query
```sql
SELECT
  EXTRACT(DAYOFWEEK FROM pickup_datetime) AS day_of_week,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
GROUP BY day_of_week
ORDER BY day_of_week
```

![image](https://github.com/user-attachments/assets/33b480ac-e15c-42e7-9e0d-cde9d94988d9)

**Key Insight:**

Fridays and Saturdays have the highest ride counts, driven by nightlife and weekend activity.

Weekdays (Tuesday–Thursday) are also busy, likely reflecting daily commuting patterns.

Sundays show moderate activity, but still higher than early weekdays.

### 5.2 Trips by Month

#### 🔍 Business Question
Are there seasonal peaks in ride demand?

#### 📎 SQL Query
```sql
SELECT
  EXTRACT(MONTH FROM pickup_datetime) AS month,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
GROUP BY month
ORDER BY month
```

![image](https://github.com/user-attachments/assets/9c36da0f-c6bb-41f5-a9dd-48424af180e1)

**Key Insight:**

Ride volumes peak in March (over 12.2 million rides) and remain high through May.

Summer months (June–August) see a gradual decline, likely due to holidays and resident travel.

September and October show a mild recovery, with November and December slightly below the annual average, possibly due to colder weather or holiday travel behavior.

These trends are crucial for planning marketing campaigns, driver recruitment, and resource allocation.

## 6. Trip Density by Hour and Day

### 🔍 Business Question
When and at what hours are taxis busiest throughout the week?

### 📎 SQL Query
```sql
SELECT
  EXTRACT(DAYOFWEEK FROM pickup_datetime) AS day_of_week,
  EXTRACT(HOUR FROM pickup_datetime) AS pickup_hour,
  COUNT(*) AS trip_count
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE pickup_datetime IS NOT NULL
GROUP BY day_of_week, pickup_hour
ORDER BY day_of_week, pickup_hour
```

![image](https://github.com/user-attachments/assets/4c39a360-41f8-443d-a360-dacdc99b46bf)

**Key Insight:**

The busiest hours are consistently late evenings (6 PM to 10 PM) across almost all days, with Friday and Saturday nights peaking the most (nightlife/social events).

Early morning hours (2 AM to 5 AM) have the lowest demand, except for modest activity on Friday and Saturday (late-night returns).

This clear time-based demand pattern supports smarter driver scheduling, surge pricing during night hours, and incentive alignment for late shifts, especially over weekends.

## 7. Deep-Dive: Tip Rate vs Trip Duration & Fare

### 7.1 Tip Rate vs Trip Duration

#### 🔍 Business Question
Does trip duration affect tipping?

#### 📎 SQL Query
```sql
SELECT
  fare_amount,
  tip_amount,
  TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, SECOND) / 60.0 AS trip_duration_min,
  SAFE_DIVIDE(tip_amount, fare_amount) AS tip_rate
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE
  fare_amount > 0
  AND tip_amount IS NOT NULL
  AND TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, SECOND) > 0
```

![image](https://github.com/user-attachments/assets/20e59b93-bcd9-4ab2-be00-353a66798b2c)

**Key Insight:**

There is no clear upward trend in tip percentage as trip duration increases.

Most tip rates cluster at the lower end of the scale, regardless of ride duration.

Outlier high values are exceptions, not the norm.

This suggests passengers do not consistently tip more on longer trips. Assumptions about "time-based tipping generosity" are not supported and other factors may have greater influence (service quality, familiarity, etc).

### 7.2 Tip Rate vs Fare Amount

#### 🔍 Business Question
How does the total fare amount influence the proportion of tips passengers leave?

#### 📎 SQL Query
```sql
SELECT
  fare_amount,
  tip_amount,
  SAFE_DIVIDE(tip_amount, fare_amount) AS tip_rate
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
WHERE
  fare_amount > 0
  AND tip_amount IS NOT NULL
```

![image](https://github.com/user-attachments/assets/5b0d5395-e7d3-4d49-9f7d-0be3fd31f257)

**Key Insight:**

There is a clear inverse relationship between fare amount and tip percentage: smaller fares tend to have a higher tip rate.

Passengers tip more generously (as a percentage) for short, cheap rides—possibly due to psychological "rounding up" or default tip options in payment interfaces.

This trend has important implications for designing driver incentive programs and optimizing payment UX.


