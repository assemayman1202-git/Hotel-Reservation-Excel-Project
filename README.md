# 🏨 Hotel Reservations Dashboard — Tourism Project

> A full end-to-end business intelligence project built in Microsoft Excel using **Power Query**, **Power Pivot**, and **DAX** — transforming raw hotel reservation data into an interactive 5-page dashboard with deep insights into cancellations, pricing, seasonality, customer behavior, and market segmentation.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Dataset Description](#-dataset-description)
- [Data Model](#-data-model-star-schema)
- [ETL Process — Power Query](#-etl-process--power-query)
- [DAX Measures](#-dax-measures)
- [Dashboard Pages & Analysis](#-dashboard-pages--analysis)
- [Key Insights](#-key-insights)
- [Recommendations](#-recommendations)
- [Tools & Technologies](#-tools--technologies)
- [How to Use](#-how-to-use)
- [File Structure](#-file-structure)

---

## 📖 Project Overview

This project analyzes **36,238 hotel reservations** spanning 2017 and 2018. The goal is to provide hotel management with a data-driven understanding of what drives cancellations, how pricing varies across segments and room types, when demand peaks, and who their customers really are.

The entire pipeline — from raw data to interactive dashboard — was built inside Microsoft Excel using:
- **Power Query** for data ingestion and transformation
- **Power Pivot** for building a relational star schema
- **DAX** for calculated KPIs and measures
- **PivotCharts + Slicers** for interactive visualization

**Key business outcomes:**
- Identified the primary drivers of the 33% cancellation rate
- Uncovered pricing inefficiencies across channels and room types
- Quantified the loyalty gap between new and returning guests
- Provided segment-level profitability signals

---

## 📦 Dataset Description

The raw data contains one record per hotel reservation and includes the following fields:

| Field | Description |
|---|---|
| Lead Time | Number of days between booking date and arrival date |
| AVG Price | Average price per room per night |
| Parking Space | Whether the guest requested a parking space (0/1) |
| Repeated Guest | Whether the guest has stayed before (0/1) |
| No. of Previous Cancellations | Count of past cancellations by this guest |
| No. of Previous Bookings | Count of prior bookings by this guest |
| No. of Special Requests | Number of special requests made |
| Room Type | Categorical room type (1–7) |
| Meal Type | Meal plan selected |
| Market Segment | Booking channel (Online, Offline, Corporate, Complementary, Aviation) |
| Booking Status | Whether the booking was canceled or not |
| Date | Arrival date (used to build the date dimension) |

---

## 🗄️ Data Model (Star Schema)

The data is modeled as a classic **Star Schema** in Power Pivot, with one central fact table connected to five dimension tables via surrogate keys.

![Data Model](images/data_model.png)

```
Dim_date ──────────────────────────────┐
Dim_Meal_Type ─────────────────────────┤
Dim_Room ──────────────────────────────┼──── Fact_Reservations
Dim_Market_Seg ────────────────────────┤
Dim_Booking_Status ────────────────────┘
```

### Tables

**Fact_Reservations** (many side)
- Contains all transactional measures: lead time, price, parking, special requests, repeated guest flag, previous cancellations, previous bookings
- Foreign keys: `Date_key`, `Room_Type_SK`, `Meal_Type_SK`, `Market_Segment_SK`, `Booking_status_SK`

**Dim_date** — Date, Year, Quarter, Month, Day Name, Date_key + calculated columns: Total Booking, Total Cancellation

**Dim_Room** — Room_Type_SK → Room Type

**Dim_Meal_Type** — Meal_Type_SK → Meal Type

**Dim_Market_Seg** — Market_Segment_SK → market segment

**Dim_Booking_Status** — Booking_status_SK → booking status

### Why Star Schema?
- Enables fast aggregations via PivotTables
- Clean separation between descriptive attributes (dimensions) and numeric facts
- Makes DAX measures simpler and more readable
- Allows slicers on dimension tables to filter the fact table across all visuals simultaneously

---

## 🔄 ETL Process — Power Query

Before loading data into the model, Power Query was used to clean and prepare each table:

1. **Remove duplicates** — ensured one record per reservation
2. **Handle nulls** — blank values in meal type and special requests filled or flagged
3. **Data type enforcement** — dates parsed correctly, numeric fields validated
4. **Surrogate key generation** — integer keys added to dimension tables for clean relationships
5. **Dimension table extraction** — unique values from segment, room type, meal type, and booking status were extracted into separate dimension tables
6. **Date table creation** — a dedicated date dimension was generated to support time intelligence

---

## 📐 DAX Measures

Key calculated measures used across the dashboards:

```dax
-- Total Bookings
Total Bookings = COUNTROWS(Fact_Reservations)

-- Total Revenue
Total Revenue = SUMX(Fact_Reservations, Fact_Reservations[AVG Price])

-- Cancellation Rate
Cancellation Rate =
DIVIDE(
    CALCULATE(COUNTROWS(Fact_Reservations), Dim_Booking_Status[booking status] = "Canceled"),
    COUNTROWS(Fact_Reservations)
)

-- Average Lead Time
AVG Lead Time = AVERAGE(Fact_Reservations[lead Time])

-- Repeated Guest Rate
Repeated Guest Rate =
DIVIDE(
    CALCULATE(COUNTROWS(Fact_Reservations), Fact_Reservations[repeated guest] = 1),
    COUNTROWS(Fact_Reservations)
)

-- Average Price Per Room
AVG Price Per Room = AVERAGE(Fact_Reservations[AVG Price])
```

---

## 📊 Dashboard Pages & Analysis

All 5 pages share the same **Year** (2017 / 2018) and **Month** (1–12) slicers, so every visual updates together when a filter is applied.

---

### Page 1 — Cancellations

![Cancellations Dashboard](images/cancellations.png)

**Purpose:** Understand what drives the 33% cancellation rate and which segments/room types are most at risk.

**Lead Time vs Cancellation Rate (Scatter Plot)**
There is a clear positive correlation — as lead time increases, cancellation rate rises sharply. Bookings made more than 200 days in advance approach 80–100% cancellation rates. This suggests that very early bookings are speculative rather than committed.

**Cancellation Rate by Room Type**
Room Type 6: ~45% | Room Type 4 & 2: ~35% | Room Type 7: lowest.
Higher-priced room types carry more risk, possibly because guests keep their options open longer.

**Meal Plan Associated with Cancellations**
"Not Selected" leads at 45.6%, followed by Meal Plan 2 (33.1%) and Meal Plan 1 (20%).
Guests who commit to a meal plan are less likely to cancel — meal selection acts as a commitment signal.

**Special Requests & Cancellation**
Guests with 0 special requests cancel at ~41%. Each additional special request reduces cancellation significantly. This is one of the strongest behavioral signals of booking commitment.

---

### Page 2 — Pricing

![Pricing Dashboard](images/pricing.png)

**Purpose:** Analyze how room price varies across room types, booking channels, time of year, and cancellation status.

**Avg Price by Room Type**
Room Type 6: ~$175 | Room Type 7: ~$145 | Room Type 1–3: $55–$85.
There is a wide price range across room types. Premium rooms (6, 7) command significantly higher rates.

**Average Room Prices by Month**
Prices peak between April and September, aligning with high-demand travel seasons. January and December are the lowest-price months, likely reflecting low occupancy periods.

**Canceled vs Non-Canceled Booking Prices**
Non-canceled: $110.61 avg | Canceled: $99.94 avg.
Guests who cancel tend to book cheaper rooms — or guests with lower price sensitivity are more committed to their stays.

**Online vs Offline Booking Prices**
Online: $112.26 | Offline: $91.64 | Corporate: ~$85.
Online channel generates the highest average price per room. Corporate accounts are negotiated at lower rates.

**Children and Room Price**
As the number of children in a booking increases, the average room price rises — families book larger, more expensive rooms.

---

### Page 3 — Seasonality

![Seasonality Dashboard](images/seasonality.png)

**Purpose:** Understand how bookings and cancellations vary over time.

**Total Bookings by Year**
2017: 6,514 | 2018: 29,724.
The data represents only partial 2017 coverage, but 2018 saw a massive volume increase reflecting strong business growth.

**Monthly Booking Volume**
Volume rises steadily from January, peaks around September–October, then drops sharply in November–December. This suggests a strong autumn travel season.

**Cancellation Rate by Month**
January: 2% (lowest) | July: 45% (highest).
Summer months, despite being the highest-demand period, also carry the highest cancellation rates — creating significant uncertainty in Q3 revenue forecasting.

**Weekend vs Weekday Nights**
Weekday nights: 79,876 | Weekend nights: 29,370.
The hotel is primarily used for business/midweek stays. Weekend occupancy is significantly lower and represents a clear growth opportunity.

---

### Page 4 — Customers

![Customers Dashboard](images/customers.png)

**Purpose:** Profile guests to understand retention, preferences, and behavior.

**New vs Repeated Guests**
New: 35,312 (97.4%) | Repeated: 926 (2.6%).
The hotel is almost entirely reliant on acquiring new customers. Virtually no retention strategy is working.

**Cancellation Rate by Guest Type**
New guests cancel at 34% | Repeated guests cancel at just 2%.
This is the single most dramatic finding in the dataset. Returning guests are 17x less likely to cancel.

**Meal Plan Preference**
"Not Selected": 27,802 | Meal Plan 1: 5,129 | Meal Plan 2: 3,302 | Meal Plan 3: 5.
The overwhelming majority of guests do not select a meal plan upfront — both a revenue opportunity and a cancellation risk signal.

**Room Type Preference**
Room Type 1 dominates with 28,105 reservations. All other room types (2–7) total only 8,133 combined.

---

### Page 5 — Market Segment

![Market Segment Dashboard](images/segment.png)

**Purpose:** Compare performance, pricing, and behavior across booking channels.

**Booking Volume by Segment**
Online: 23,194 (64%) | Offline: 10,518 (29%) | Corporate: 2,011 | Complementary: 390 | Aviation: 125.

**Cancellation Rate by Segment**
Aviation: 37% | Online: 30% | Offline: 30% | Corporate: 11%.
Corporate bookings are by far the most reliable. Aviation is the riskiest.

**Average Lead Time by Segment**
Offline: ~125 days | Online: ~85 days | Corporate: ~25 days.
Corporate guests book close to arrival and almost never cancel — the ideal booking profile.

**Online vs Corporate Booking Prices**
Online: ~$112 | Corporate: ~$85.
Online generates more revenue per booking despite the higher cancellation rate.

---

## 💡 Key Insights

1. **Lead time is the #1 cancellation predictor.** Bookings made more than 3 months in advance are significantly more likely to cancel.
2. **New guests cancel at 34% — repeat guests at 2%.** Returning guests are 17x less likely to cancel.
3. **Special requests = commitment.** Guests who make special requests almost never cancel. This is a strong behavioral signal.
4. **Corporate is the most reliable segment.** Lower price, but near-zero cancellations and low lead time make corporate the most predictable revenue stream.
5. **Summer brings volume AND volatility.** July peaks in both bookings and cancellations (45%).
6. **97.4% of guests are first-time visitors.** There is no meaningful retention engine in place.
7. **Weekends are underutilized.** Weekday nights are 2.7x higher than weekend nights.
8. **Meal plan selection predicts loyalty.** Guests who choose a meal plan cancel far less.

---

## ✅ Recommendations

### 🏢 Business (Management)

**1. Re-Confirmation System for Long Lead Bookings**
Any booking made more than 90 days in advance should trigger an automated re-confirmation at the 30-day mark. Guests who don't respond can be offered an incentive or moved to a waitlist, recovering inventory early.

**2. Build a Loyalty Program**
With only 2.6% repeat guests, there is massive untapped retention value. A simple tiered loyalty program could shift the repeat rate meaningfully — and given repeat guests cancel at 2%, every converted repeat guest directly reduces overall cancellation rate.

**3. Introduce Non-Refundable Rate Tiers**
Offer a discounted non-refundable rate for early bookers alongside a flexible rate. This directly addresses the high cancellation rate in the long lead-time segment.

**4. Promote Weekend Packages**
Weekend occupancy is 63% lower than weekday. Targeted weekend deals (leisure packages, spa bundles, family packages) could significantly increase revenue without cannibalizing existing demand.

**5. Bundle Meal Plans into Early Booking Incentives**
Since meal plan selection correlates with lower cancellations, offering a complimentary breakfast with early bookings encourages commitment and increases per-stay revenue.

**6. Grow the Corporate Segment**
Corporate bookings have an 11% cancellation rate (vs 30% online) and predictable close-in booking behavior. Investing in corporate sales would diversify revenue away from the volatile online channel.

---

### 💻 Technical (Data Analyst)

**1. Build a Cancellation Probability Score**
Using lead time, market segment, meal plan selection, number of special requests, and repeated guest flag — a logistic regression or decision tree model could score each booking at creation time for cancellation risk.

**2. Add a Revenue at Risk Metric**

```dax
-- Revenue at Risk (simplified)
Revenue at Risk =
SUMX(
    FILTER(Fact_Reservations, Fact_Reservations[lead Time] > 90),
    Fact_Reservations[AVG Price] * 0.33
)
```

**3. Implement Time Intelligence Measures**

```dax
-- YoY Booking Growth
YoY Bookings Growth =
DIVIDE(
    [Total Bookings] - CALCULATE([Total Bookings], SAMEPERIODLASTYEAR(Dim_date[Date])),
    CALCULATE([Total Bookings], SAMEPERIODLASTYEAR(Dim_date[Date]))
)
```

**4. Add a Customer Dimension**
Currently there is no customer dimension — each booking is independent. Adding a `Dim_Customer` table with a customer ID would enable proper lifetime value analysis and retention cohorts.

**5. Add Forecast Sheets**
Using the monthly booking volume data, Excel's `FORECAST.ETS` function can project future demand and give management a forward-looking view alongside the historical dashboard.

**6. Migrate to Power BI**
The star schema built here would migrate directly to Power BI with minimal changes — unlocking automated refresh, row-level security, and more advanced visuals like decomposition trees and natural language Q&A.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Microsoft Excel | Primary development environment |
| Power Query | ETL — data ingestion, cleaning, transformation |
| Power Pivot | In-memory data model, star schema, relationships |
| DAX | Calculated measures and KPIs |
| PivotTables | Dynamic aggregations feeding all chart visuals |
| PivotCharts | Bar, line, scatter, donut, and pie visualizations |
| Slicers | Interactive cross-page filtering (Year, Month) |

---

## ▶️ How to Use

1. Open `Tourism_Project.xlsx` in Microsoft Excel (2016 or later)
2. Enable editing and content if prompted
3. Navigate between dashboard pages using the left sidebar buttons
4. Use the **Year slicer** (2017 / 2018) and **Month slicer** to filter all visuals simultaneously
5. Click **Control Insights** on any page to see top-level KPI context
6. All charts are PivotCharts — they update automatically when slicers change

---

## 📁 File Structure

```
Tourism_Project.xlsx
├── Data Tables
│   ├── Fact_Reservations       # Core transaction data
│   ├── Dim_date                # Date dimension
│   ├── Dim_Room                # Room type dimension
│   ├── Dim_Meal_Type           # Meal plan dimension
│   ├── Dim_Market_Seg          # Market segment dimension
│   └── Dim_Booking_Status      # Booking status dimension
│
└── Dashboard Pages
    ├── Cancellations           # Cancellation drivers & patterns
    ├── Pricing                 # Revenue & pricing analysis
    ├── Seasonality             # Time-based demand trends
    ├── Customers               # Guest behavior & profiles
    └── Market Segment          # Channel-level performance
```

---

> **Note:** This project demonstrates end-to-end BI development in Excel — from raw data ingestion through Power Query, to a fully relational star schema in Power Pivot, to an interactive multi-page dashboard with DAX-powered KPIs and business recommendations.
