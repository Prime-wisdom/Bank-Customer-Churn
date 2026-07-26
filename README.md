# Bank Customer Churn Analysis — Power BI Dashboard
 
## 📌 Overview
A 4-page Power BI dashboard analyzing customer churn for a bank with 10,000 customers across 3 countries (France, Germany, Spain), built to identify who's leaving, why, and where the bank should focus retention efforts.
 
---
 
## 📂 Dataset
 
**Source:** [Maven Analytics](https://www.mavenanalytics.io/) — "Bank Customer Churn" dataset
 
**Size:** 10,000 rows, 13 columns
 
**Key fields:** CustomerId, Surname, CreditScore, Geography, Gender, Age, Tenure, Balance, NumOfProducts, HasCrCard, IsActiveMember, EstimatedSalary, Exited

 🔗 **[View the Interactive Dashboard](https://rebrand.ly/d1lmvuv)**
---
 
## 🎯 Problem Statement & Objectives
 
A bank noticed a portion of its customers were leaving and wanted to understand the scale and drivers of this churn before deciding where to focus retention efforts.
 
**Objectives:**
1. Quantify overall churn and its business impact (customers lost, balance at risk)
2. Identify which customer segments (geography, age, gender, product usage, tenure, credit score) show the strongest churn signal
3. Build a way to flag *current* customers who match the profile of past churners, so the bank can act proactively rather than only reviewing churn after the fact
---
 
## 🧹 Data Cleaning & Preparation
 
Performed in Power Query using Conditional Columns, then refined further with calculated columns in DAX.
 
**Data quality check:** Power Query's column quality view confirmed 100% valid values across all fields — no nulls or errors found in the raw dataset. No rows were removed; all 10,000 records were retained.
 
**Conditional columns created to convert numeric flags into readable text:**
 
- `HasCrCard` (1/0) → **Has Credit Card** ("Yes"/"No")
- `Exited` (1/0) → **Exited Customer**, later relabeled to **Churned / Retained** for clarity across all visuals and tooltips
- `IsActiveMember` (1/0) → **Active Member**, later relabeled to **Active / Inactive**
**Segmentation columns created for analysis:**
 
- **Age Band** — grouped `Age` into bands (18-30, 31-45, 46-60, 61-75, 75+) to support the age-based churn analysis on Page 2
  *(An earlier, broader "Age Bracket" version — Young / Middle Age / Old — was replaced with this more granular 5-band version for better resolution in the charts)*
- **Product Type** — flagged customers as "Single Product" (NumOfProducts = 1) or "Multi Product" (2+) to test whether product count relates to loyalty
- **Credit Score Band** — grouped `CreditScore` into bands (0-400, 401-650, 651-800, 800+) for the credit score churn analysis
**Surname** was retained in the model (rather than dropped) since it's used for readability in the Page 3 customer watchlist table, where identifying individual flagged customers by name is useful for the retention team.
 
---
 
## 🧠 Thought Process
 
The dataset itself was simple — one table, no time dimension. The real challenge wasn't the data, it was deciding **what questions actually matter to a bank manager**, and structuring the dashboard so each page answers a different layer of that question rather than repeating the same chart with different filters.
 
I broke the analysis into three questions, each becoming its own dashboard page:
 
1. **What is happening?** → Executive Overview (headline numbers, who churned, how much)
2. **Where is it happening?** → Drivers & Patterns (which segments show the strongest churn signal)
3. **Where should the bank act first?** → Risk & Prediction Monitoring (a watchlist, not just a report)
A key decision point: I initially planned the third page around a machine learning prediction model. Since I hadn't built one yet, I reframed it around **rule-based risk flagging** instead — using patterns already surfaced in pages 1 and 2 (inactive + single product + short tenure) to flag current customers who resemble past churners. This kept the dashboard fully explainable to a non-technical audience, and gives me a clear "phase 2" to build toward later.
 
I also added a **Home page** as a 4th page, once I realized a 3-page dashboard with no landing page felt incomplete for anyone opening the file cold.
 
---
 
## 📊 Dashboard Pages & Analysis
 
### Home
![Home Page](Dashboards_PNG/Home_Page.png)
 
Landing page with project title, short intro, and navigation to all 3 analysis pages.
 
---
 
### Page 1: Executive Overview
![Executive Overview](Dashboards_PNG/Executive_Overview.png)
 
**Purpose:** a 10-second glance for someone who wants the headline numbers only — no drill-down, no segmentation, just "what's happening."
 
**Key DAX:**
```DAX
Churn Rate = 
DIVIDE(
    CALCULATE(COUNTROWS('Bank Churn'), 'Bank Churn'[Exited] = "Churned"),
    [Total Customers]
)
 
Customers Churned = 
CALCULATE([Total Customers], 'Bank Churn'[Exited] = "Churned")
 
Total Balance At Risk = 
CALCULATE(
    SUM('Bank Churn'[Balance]),
    'Bank Churn'[Exited] = "Churned"
)
```
 
**Charts:** Churn Rate by Geography, Churn Rate by Active/Inactive status, Customer Outcome (Retained/Churned donut), Total Churned by Age Band, Churn Rate vs. Number of Products.
 
**Findings:**
- Germany's churn rate (32.44%) is almost double France and Spain
- 65.3% of churned customers were already inactive before leaving
- Customers with 3+ products show unexpectedly high churn (82.7%–100%), though on a small base
---
 
### Page 2: Drivers & Patterns
![Drivers & Patterns](Dashboards_PNG/Drivers_&_Patterns.png)
 
**Purpose:** go one layer deeper than Page 1 — isolate which specific factors correlate most strongly with churn, for an analyst/retention-team audience.
 
**Key DAX:**
```DAX
Churn Rate (Female) = 
CALCULATE([Churn Rate], 'Bank Churn'[Gender] = "Female")
 
Churn Rate (Male) = 
CALCULATE([Churn Rate], 'Bank Churn'[Gender] = "Male")
```
 
**Charts:** Churn Rate by Age Band, by Gender, by Credit Score Band, by Tenure (Years), Single vs. Multi-Product Churn.
 
**Findings:**
- Customers aged 46-60 churn at 51.12% — by far the highest of any age band
- Female customers churn more than male (25.07% vs. 16.46%)
- Single-product customers churn more than double the rate of multi-product customers (27.71% vs. 12.77%)
- Churn is highest at signup, drops after year 1, then rises again after year 8 — a U-shaped risk curve across the customer lifecycle
- Below a 400 credit score, churn hits 100% — but this is a very small group (19 customers), flagged as a pattern to watch rather than a confirmed driver
---
 
### Page 3: Risk & Prediction Monitoring
![Risk & Prediction Monitoring](Dashboards_PNG/Risk_&_Prediction_Monitoring.png)
 
**Purpose:** turn the patterns found in Pages 1–2 into an actionable, prioritized list — "who should the bank call first," not just "what happened historically."
 
**Attention Flag logic (rule-based, not ML):**
```DAX
Attention Flag = 
IF(
    'Bank Churn'[Active Member] = "Inactive" &&
    'Bank Churn'[Product Type] = "Single Product" &&
    'Bank Churn'[Tenure] <= 2,
    "High Attention",
    "Normal"
)
 
High Attention Customers = 
CALCULATE([Total Customers], 'Bank Churn'[Attention Flag] = "High Attention")
 
Attention Rate = 
DIVIDE([High Attention Customers], [Total Customers])
 
Balance (High Attention) = 
CALCULATE(
    SUM('Bank Churn'[Balance]),
    'Bank Churn'[Attention Flag] = "High Attention"
)
```
 
**Why rule-based instead of a predictive model:** at this stage I hadn't built a churn-prediction model, and rather than leave the page empty or blocked, I used the strongest confirmed patterns from Pages 1–2 (inactivity, single product, short tenure) as flagging criteria. This keeps the watchlist fully explainable to a non-technical stakeholder — "these customers match known churn patterns" is easy to justify, unlike an opaque model score. It's designed to be swapped for a real ML-based risk score later without changing the dashboard structure.
 
**Findings:**
- 599 customers (≈6% of the base) are flagged High Attention
- That 6% of customers holds $58.45M in balance — disproportionate to their headcount, which justifies prioritizing them
- France has the most flagged customers (302) — though this may partly reflect France having the largest customer base overall, not necessarily higher risk concentration
- Customers aged 46-60 have the highest attention rate (8.20%) of any age band
---
 
## 💡 Overall Recommendation
 
Prioritize outreach to **inactive, single-product customers in Germany**, particularly those **aged 46-60** — this segment carries the highest combined churn risk across every dimension analyzed (geography, activity status, product count, and age), and is small enough to act on directly rather than requiring a mass campaign.
 
**Suggested next steps for the bank:**
1. **Immediate:** contact the 599 flagged "High Attention" customers, starting with the highest-balance accounts
2. **Short-term:** investigate why 3+ product holders are churning at such high rates — this breaks the usual assumption that more products means more loyalty, and deserves root-cause investigation, not just a retention offer
3. **Long-term:** build a proper predictive model to replace the rule-based flag, using this dashboard's confirmed drivers (inactivity, product count, tenure, age, geography) as model features
---
 
## 🛠 Tools
Power BI · Power Query · DAX
 
## 📁 Files
- `BANK CUSTOMER CHURN.pbix` — full Power BI report
---
 ## 📌 Wisdom Analytics — connect with me on [LinkedIn](https://www.linkedin.com/in/chidera-okpala-22417730a/) or [X](https://x.com/PrimeW1sdom).
