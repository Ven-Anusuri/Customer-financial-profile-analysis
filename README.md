# 💰 Bank Revenue Intelligence — Cross-Sell & Lead Prioritization Engine

[![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Tableau](https://img.shields.io/badge/Tableau-E97627?style=for-the-badge&logo=tableau&logoColor=white)](https://public.tableau.com/app/profile/ven.anusuri)

## 📊 Live Dashboard

🔗 [View on Tableau Public](https://public.tableau.com/app/profile/ven.anusuri/viz/CustomerFinancialProfileAnalysisDashboard/CustomerFinancialProfileAnalysisDashboard?publish=yes)

---

## 📌 Project Overview

This project transforms the financial profiles of **45,211 bank customers** into a working revenue intelligence system — identifying **~$8M in cross-sell opportunity** across product segments, scoring every customer for outreach priority, and assigning a data-driven Next Best Product recommendation.

Built from a former TD Bank financial advisor's perspective, this mirrors the actual workflow a bank analytics team would run before launching a cross-sell or retention campaign: segment the book, size the revenue gap, prioritize the leads, and tell the advisor exactly what to offer.

**Products tracked:** Housing Loan (Mortgage) · Personal Loan

**Revenue model assumptions** *(conservative retail banking benchmarks)*:
- Mortgage net interest contribution: ~$2,400/yr per customer
- Personal loan net interest: ~$800/yr per customer
- Conversion rates: 10% (zero-product) · 12% (loan cross-sell) · 8% (mortgage cross-sell)

---

## 🎯 Business Questions Answered

| Layer | Question |
|-------|----------|
| **Who are our customers?** | Which age and job segments hold the most wealth? |
| **What do they hold?** | Which segments have the lowest product penetration rate? |
| **What's the revenue gap?** | How much annual revenue is sitting uncaptured in each segment? |
| **What do we sell them?** | What is the Next Best Product for each customer type? |
| **Who do we call first?** | Which customers score HOT vs. WARM vs. COLD for outreach? |

---

## 💡 Key Revenue Insights

- **~$8M total cross-sell opportunity** exists across the 45,211-customer book at conservative conversion rates
- **Management segment alone represents $1.83M** — 4,206 customers with zero products and an avg balance of $1,766
- **Blue-collar is the hidden mortgage cross-sell play** — 5,890 mortgage-only customers eligible for personal loan, worth ~$565K at 12% conversion
- **Retired segment has a 69.1% zero-product rate** despite the highest avg balance among mid-tier segments ($1,977) — strongest Term Deposit / GIC opportunity
- **Students show 72.5% zero-product rate** — lowest absolute revenue but critical for long-term customer lifetime value
- **Premium customers (balance >$20K, 192 total, zero defaulters)** score HOT on lead tier — wealth management and bundled products next
- **3,743 customers with negative balances** are flagged and excluded from cross-sell targeting — routed to credit recovery

---

## 📈 Dashboard & Query Architecture

### Layer 1 — Customer Profiling (Q1 & Q2)

**Q1 — Age Segmentation**
Maps life-stage distribution: balance accumulation, mortgage penetration, and default concentration by age band. Feeds the NBP age-gate logic in Q7.

**Q2 — Education × Job Balance Profile**
Identifies the top 20 highest-avg-balance education/job combinations. Tertiary-educated management and retired segments lead — primary targets for higher-margin products.

### Layer 2 — Product Holding Analysis (Q3 & Q4)

**Q3 — Product Holding Breakdown by Job**
Counts No-Products / Mortgage-Only / Loan-Only / Both per segment. The raw input used to calculate PPR and revenue gaps in Q5–Q6.

**Q4 — Customer Value Tier Segmentation**
Classifies all customers into Premium / High / Mid / Low / Negative balance tiers. Defaulter concentration is highest in Negative (-$317 avg) — zero overlap with cross-sell lists.

### Layer 3 — Revenue Intelligence (Q5–Q8) ★ New

**Q5 — Product Penetration Rate (PPR)**
The #1 KPI in retail banking. Calculates avg products per customer by segment vs. the 2-product ceiling, and ranks segments by their PPR gap — the bigger the gap, the larger the untapped opportunity.

| Segment | PPR | Gap | Zero-Product % |
|---------|-----|-----|----------------|
| unknown | 0.10 | 1.90 | 90.5% |
| student | 0.28 | 1.72 | 72.5% |
| retired | 0.35 | 1.65 | 69.1% |
| unemployed | 0.50 | 1.50 | 54.6% |
| management | 0.63 | 1.37 | 44.7% |
| blue-collar | 0.90 | 1.10 | 21.9% |

**Q6 — Revenue Opportunity Sizing**
Converts product gaps into dollars. Three opportunity pools per segment: zero-product customers, mortgage-only customers eligible for personal loan, and loan-only customers eligible for mortgage. Conservative conversion rates applied throughout.

| Segment | Total Opportunity |
|---------|-------------------|
| management | $1,829,376 |
| technician | $1,361,280 |
| blue-collar | $1,350,912 |
| admin. | $826,624 |
| services | $617,376 |
| retired | $571,104 |
| **All segments** | **~$8,000,000** |

**Q7 — Next Best Product (NBP) Assignment**
Rule-based product recommendation engine. Logic matrix: `life stage (age) + current products held + balance threshold`. Defaulters excluded. Output drives advisor talking points and CRM campaign targeting.

Sample NBP assignments:
- Age 31–45, zero products, balance ≥ $1,000 → **Mortgage — First Home / Upgrade**
- Age 31–45, mortgage only, balance ≥ $2,000 → **Personal Loan — Creditworthy Mortgage Holder**
- Age >55, any product state → **Term Deposit / GIC — Capital Preservation**
- Both products held → **Term Deposit / Investment — Fully Leveraged, Build Savings**

**Q8 — Priority Lead Scoring & Tier Classification**
Composite 0–100 score per customer assigned to HOT / WARM / COLD outreach tiers. Score breakdown:

| Component | Max Points | Logic |
|-----------|-----------|-------|
| Balance tier | 40 | Premium=40, High=30, Mid=20, Low=10, Negative=0 |
| Product gap | 30 | No products=30, One product=15, Both=5 |
| Life stage | 20 | Ages 26–55=20, 56–65=15, 18–25=10, 65+=8 |
| No default | 10 | Clean record=10, Has default=0 |

- **HOT (≥70):** Priority phone/advisor outreach — schedule within 30 days
- **WARM (45–69):** Digital campaign — email/app notification sequence
- **COLD (<45):** Nurture/monitor — quarterly touchpoint or life event trigger

---

## 🛠️ SQL Queries

### Q1 — Customer Segmentation by Age Group

```sql
SELECT
    CASE
        WHEN age BETWEEN 18 AND 25 THEN '18-25 Young Adults'
        WHEN age BETWEEN 26 AND 35 THEN '26-35 Early Career'
        WHEN age BETWEEN 36 AND 45 THEN '36-45 Mid Career'
        WHEN age BETWEEN 46 AND 55 THEN '46-55 Peak Earners'
        WHEN age BETWEEN 56 AND 65 THEN '56-65 Pre Retirement'
        ELSE '65+ Retirement'
    END AS Age_Group,
    COUNT(*) AS Total_Customers,
    ROUND(AVG(balance), 2) AS Avg_Balance,
    SUM(CASE WHEN housing = 'yes' THEN 1 ELSE 0 END) AS Has_Mortgage,
    SUM(CASE WHEN loan = 'yes' THEN 1 ELSE 0 END) AS Has_Personal_Loan,
    SUM(CASE WHEN defaulter = 'yes' THEN 1 ELSE 0 END) AS Defaulters
FROM Bank_Customer_Data
GROUP BY Age_Group
ORDER BY MIN(age);
```

### Q5 — Product Penetration Rate (PPR) by Job Segment ★ New

```sql
SELECT
    job,
    COUNT(*) AS Total_Customers,
    SUM(CASE WHEN housing = 'yes' THEN 1 ELSE 0 END) AS Customers_With_Mortgage,
    SUM(CASE WHEN loan = 'yes' THEN 1 ELSE 0 END) AS Customers_With_Loan,
    SUM(CASE WHEN housing = 'no' AND loan = 'no' THEN 1 ELSE 0 END) AS Customers_No_Products,
    ROUND(
        (SUM(CASE WHEN housing = 'yes' THEN 1 ELSE 0 END) +
         SUM(CASE WHEN loan = 'yes' THEN 1 ELSE 0 END)) * 1.0 / COUNT(*),
    2) AS PPR_Avg_Products_Per_Customer,
    ROUND(
        2.0 - (SUM(CASE WHEN housing = 'yes' THEN 1 ELSE 0 END) +
               SUM(CASE WHEN loan = 'yes' THEN 1 ELSE 0 END)) * 1.0 / COUNT(*),
    2) AS PPR_Gap_To_Ceiling,
    ROUND(
        SUM(CASE WHEN housing = 'no' AND loan = 'no' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),
    1) AS Pct_Zero_Products
FROM Bank_Customer_Data
GROUP BY job
ORDER BY Customers_No_Products DESC;
```

### Q6 — Revenue Opportunity Sizing by Segment ★ New

```sql
SELECT
    job,
    COUNT(*) AS Total_Customers,
    SUM(CASE WHEN housing = 'no' AND loan = 'no' THEN 1 ELSE 0 END) AS No_Product_Customers,
    SUM(CASE WHEN housing = 'yes' AND loan = 'no' THEN 1 ELSE 0 END) AS Mortgage_Only_Customers,
    SUM(CASE WHEN housing = 'no' AND loan = 'yes' THEN 1 ELSE 0 END) AS Loan_Only_Customers,
    ROUND(SUM(CASE WHEN housing = 'no' AND loan = 'no' THEN 1 ELSE 0 END) * 0.10 * 3200, 0) AS Est_Rev_No_Products,
    ROUND(SUM(CASE WHEN housing = 'yes' AND loan = 'no' THEN 1 ELSE 0 END) * 0.12 * 800, 0) AS Est_Rev_Loan_CrossSell,
    ROUND(SUM(CASE WHEN housing = 'no' AND loan = 'yes' THEN 1 ELSE 0 END) * 0.08 * 2400, 0) AS Est_Rev_Mortgage_CrossSell,
    ROUND(
        (SUM(CASE WHEN housing = 'no' AND loan = 'no' THEN 1 ELSE 0 END) * 0.10 * 3200) +
        (SUM(CASE WHEN housing = 'yes' AND loan = 'no' THEN 1 ELSE 0 END) * 0.12 * 800) +
        (SUM(CASE WHEN housing = 'no' AND loan = 'yes' THEN 1 ELSE 0 END) * 0.08 * 2400),
    0) AS Total_Revenue_Opportunity
FROM Bank_Customer_Data
GROUP BY job
ORDER BY Total_Revenue_Opportunity DESC;
```

### Q7 — Next Best Product (NBP) Assignment ★ New

```sql
SELECT
    CASE
        WHEN age BETWEEN 18 AND 25 THEN '18-25 Young Adults'
        WHEN age BETWEEN 26 AND 35 THEN '26-35 Early Career'
        WHEN age BETWEEN 36 AND 45 THEN '36-45 Mid Career'
        WHEN age BETWEEN 46 AND 55 THEN '46-55 Peak Earners'
        WHEN age BETWEEN 56 AND 65 THEN '56-65 Pre Retirement'
        ELSE '65+ Retirement'
    END AS Age_Group,
    CASE
        WHEN housing = 'no' AND loan = 'no' AND age <= 30 THEN 'Personal Loan — Credit Building'
        WHEN housing = 'no' AND loan = 'no' AND age BETWEEN 31 AND 45 AND balance >= 1000 THEN 'Mortgage — First Home / Upgrade'
        WHEN housing = 'no' AND loan = 'no' AND age BETWEEN 46 AND 55 THEN 'Mortgage — Investment Property'
        WHEN housing = 'no' AND loan = 'no' AND age > 55 THEN 'Term Deposit / GIC — Capital Preservation'
        WHEN housing = 'yes' AND loan = 'no' AND balance >= 2000 THEN 'Personal Loan — Creditworthy Mortgage Holder'
        WHEN housing = 'no' AND loan = 'yes' AND age BETWEEN 26 AND 55 THEN 'Mortgage — Loan Holder Ready for Home Product'
        WHEN housing = 'yes' AND loan = 'yes' THEN 'Term Deposit / Investment — Fully Leveraged, Build Savings'
        ELSE 'Financial Review — Advisor Assessment Needed'
    END AS Next_Best_Product,
    COUNT(*) AS Customer_Count,
    ROUND(AVG(balance), 2) AS Avg_Balance
FROM Bank_Customer_Data
WHERE defaulter = 'no'
GROUP BY Age_Group, Next_Best_Product
ORDER BY Customer_Count DESC;
```

### Q8 — Priority Lead Scoring & Tier Classification ★ New

```sql
SELECT
    job,
    SUM(CASE WHEN (
        CASE WHEN balance > 20000 THEN 40 WHEN balance BETWEEN 5001 AND 20000 THEN 30
             WHEN balance BETWEEN 1001 AND 5000 THEN 20 WHEN balance BETWEEN 0 AND 1000 THEN 10 ELSE 0 END +
        CASE WHEN housing = 'no' AND loan = 'no' THEN 30
             WHEN housing = 'yes' AND loan = 'no' THEN 15
             WHEN housing = 'no' AND loan = 'yes' THEN 15 ELSE 5 END +
        CASE WHEN age BETWEEN 26 AND 55 THEN 20 WHEN age BETWEEN 56 AND 65 THEN 15
             WHEN age BETWEEN 18 AND 25 THEN 10 ELSE 8 END +
        CASE WHEN defaulter = 'no' THEN 10 ELSE 0 END
    ) >= 70 THEN 1 ELSE 0 END) AS HOT_Leads,
    SUM(CASE WHEN (...score...) BETWEEN 45 AND 69 THEN 1 ELSE 0 END) AS WARM_Leads,
    SUM(CASE WHEN (...score...) < 45 THEN 1 ELSE 0 END) AS COLD_Leads,
    COUNT(*) AS Total_Customers
FROM Bank_Customer_Data
GROUP BY job
ORDER BY HOT_Leads DESC;
-- Full query with complete score expression in bank_revenue_intelligence.sql
```

*See `bank_revenue_intelligence.sql` for the complete Q8 query with full score expressions.*

---

## 🛠️ Tools & Technologies

- **SQL (SQLite / DB Browser for SQLite)** — Segmentation, PPR, revenue sizing, NBP logic, lead scoring
- **Tableau Public** — Dashboard design and visualization
- **Kaggle** — Source dataset (Bank Customer Dataset)

---

## 📂 Data Source

- **Dataset:** [Bank Customer Dataset](https://www.kaggle.com/datasets/megasatish/bank-customer-dataset) — Kaggle (Megasatish)
- **Records:** 45,211 customers
- **Features:** Age, Job, Education, Balance, Housing Loan, Personal Loan, Defaulter status

---

## 🗂️ Repository Structure

```
📁 customer-financial-profile-analysis/
├── 📄 README.md
├── 📄 bank_revenue_intelligence.sql        ← All 8 queries (clean, readable)
├── 📄 Customer_profiles_quires.sqbpro      ← DB Browser project file
├── Q1_age-segmentation.csv                 ← Life-stage profiling output
├── Q2_education_job_balances.csv           ← High-value segment mapping output
├── Q3_Crosssell_opertunities.csv           ← Product holding breakdown output
├── Q4_customer_segments.csv               ← Balance tier classification output
├── Q5_product_penetration_rate.csv        ← PPR by job segment ★ New
└── Q6_revenue_opportunity_sizing.csv      ← Revenue gap in dollars ★ New
```

---

## 🔮 Next Steps / Future Enhancements

- **Expand product suite** — Add credit cards, GICs, investment accounts, insurance to build a true 6-product PPR model
- **ML propensity scoring** — Replace rule-based NBP with logistic regression or XGBoost trained on historical conversion data
- **Time-series analysis** — Track balance changes quarter-over-quarter to catch wealth-building inflection points
- **Geographic segmentation** — Regional cross-sell rate variation for branch-level targeting
- **Campaign ROI tracking** — Close the loop: measure actual conversion rates vs. modelled assumptions per lead tier

---

## 👤 About Me

**Ven Anusuri** | Financial Advisor → Data Analyst

- 3+ years experience as a Financial Advisor at **TD Bank**
- Certifications: CSC, PFSA, FP-1, BCO, Google Data Analytics, Bloomberg Market Concepts
- Building a 10-project data analytics portfolio bridging finance domain expertise with data skills

🔗 [Tableau Public Profile](https://public.tableau.com/app/profile/ven.anusuri)  
🔗 [Project 1 — Canadian Big 5 Banks Stock Dashboard](https://public.tableau.com/app/profile/ven.anusuri/viz/CanadianBig5Banks-StockPerformanceDashboard/Dashboard1)

---

*Project 2 of 10 — Financial Data Analytics Portfolio*
