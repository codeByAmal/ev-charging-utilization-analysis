# EV Charging Station Utilization & Idle-Time Hogging Analysis

![Power BI Dashboard Preview](dashboard%20preview.png)

## Project Overview
As electric vehicle (EV) adoption grows, drivers frequently face long queues at public and workplace charging hubs. However, charger shortages are often driven by **"idle hogging"** vehicles remaining plugged in long after battery charging finishes—rather than genuine energy demand.

This end-to-end data analytics project analyzes **2,026 real-world physical charging logs** across **51 charging bays** from the **Caltech Adaptive Charging Network (ACN)** in Pasadena, California. Using **Python (Pandas, NumPy)** for edge-case data engineering and **Microsoft Power BI Desktop (DAX)** for executive visualization, this study quantifies post-charge idle waste, isolates peak commuter bottlenecks, and proposes targeted turnover policies.

---

## Key Business Findings
* **44.5% Fleet-Wide Idle Waste:** Across **12,791.2 total connected hours** (`17,924.7 kWh` delivered), vehicles spent **5,687.5 hours** (`168.4` mins/session) sitting idle after charging completed.
* **Morning Commuter Surge (7:00 AM – 9:59 AM):** **41.4% of all arrivals (838 sessions)** occur during the morning commute, peaking at **8:00 AM (416 sessions)** where vehicles stay plugged in for an average of **7.6 hours** (`225.3` idle mins).
* **Mid-Week Demand Peak:** **Wednesday** generates peak session volume (**405 sessions**, `3,430.8 kWh`) and high weekday idle waste (`188.4` mins/session), whereas **Sunday** drops by **66.7%** (`135 sessions`).
* **Driver Hogging Segmentation:** Only **14.86% of sessions (301)** disconnect immediately with zero idle waste, while **51.92% (1,052)** exhibit moderate idle waste (`1–50%`) and **33.22% (673)** suffer severe hogging (`>50%` of connected time spent idle).
* **Spatial Bay Disparities:** Bays such as **`CA-326`**, **`CA-327`**, and **`CA-305`** suffer chronic daytime idle hogging, while **`CA-303`** achieves high session throughput (`85` sessions) with substantially lower idle waste (`27.4%`).

---

## Data Engineering & Edge Cases Handled (Python)
1. **Sparsity Pruning:** Dropped `userInputs` and `userID` (`2,013` of `2,026` rows missing, `>99.3%`).
2. **Conservative Imputation:** Imputed the single missing `doneChargingTime` timestamp with `disconnectTime` to retain full sample integrity (`n = 2,026`) without inflating idle durations.
3. **Timezone & DST Standardization:** Converted UTC timestamps to local Pasadena wall-clock time (`America/Los_Angeles`) and stripped offset metadata prior to export so Power BI preserved exact local arrival hours.
4. **Sensor Clock-Drift Clamping:** Used `np.clip(raw_idle_mins, 0, Total_Connected_Mins)` to eliminate negative idle durations caused by minor hardware clock drift.
5. **Calendar & Behavioral Features:** Engineered `Active_Charging_Mins`, `Idle_Ratio_Pct`, `Hogging_Category`, `Arrival_Hour` (`0–23`), and zero-indexed `Day_Number` (`0 = Monday` to `6 = Sunday`).

---

## Power BI Data Modeling & DAX Measures
* **Schema & Sorting Hygiene:** Enforced explicit decimal types on duration/energy fields, disabled default summation on categorical IDs (`Arrival_Hour`, `Day_Number`, `siteID`, `clusterID`), and sorted `Day_of_Week` by `Day_Number`.
* **Weighted Fleet DAX Measures:** Avoided average-of-averages distortion by computing fleet-wide idle efficiency via weighted ratio aggregation:
  ```dax
  Idle Waste % = DIVIDE(SUM('ACN_Cleaned_Real'[Idle_Mins]), SUM('ACN_Cleaned_Real'[Total_Connected_Mins]), 0)
  ```

---

## Actionable Recommendations
1. **30-Minute Grace Period + Tiered Idle Fees:** Trigger automated mobile notifications upon charge completion and enforce a per-minute idle fee after a **30-minute grace window** on top-hogging stalls (`CA-326`, `CA-327`, `CA-305`).
2. **Mid-Week Commuter Bay Rotation:** Introduce campus valet rotation or afternoon bay-reallocation incentives on **Wednesdays through Fridays** to unlock stalled morning chargers for afternoon arrivals without new hardware expenditure.

---

## Repository Structure
```text
├── ACN_sessions.csv                         # Raw Caltech ACN dataset (2,026 sessions, 13 columns)
├── ACN_Cleaned_Real.csv                     # Cleaned dataset exported for Power BI (2,026 rows, 19 columns)
├── EV_Charging_Analysis.ipynb               # Jupyter Notebook (cleaning, assertions, EDA)
├── EV_Charging_Utilization_Dashboard.pbix   # Interactive Power BI Desktop report (1920x1080)
├── dashboard_preview.png                    # Full-resolution dashboard visual preview
└── README.md                                # Project documentation
```

## Tech Stack
* **Languages & Libraries:** Python, Pandas, NumPy
* **Business Intelligence:** Microsoft Power BI Desktop, DAX
* **Environment:** Jupyter Notebook, Git & GitHub
