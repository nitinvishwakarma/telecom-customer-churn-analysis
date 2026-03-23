# telecom-customer-churn-analysis
Customer churn analysis using MySQL, AWS S3, AWS RDS — identifying $264,058 revenue at risk

# Telecom Customer Churn Analysis
### Tools: MySQL | AWS S3 | AWS RDS | Power BI

---

## Project Overview

A telecom company was losing nearly 1 in 3 customers with no clear 
understanding of why. I built a complete data analytics pipeline to 
identify who is leaving, why they are leaving, and which current 
customers are most likely to leave next.

---

## Dataset

- **Source:** IBM Telco Customer Churn Dataset — Kaggle
- **Records:** 7,043 customers
- **Columns:** 21
- **Target Column:** Churn (Yes = left, No = stayed)

---

## Project Architecture
```
Raw Dataset (Kaggle)
      ↓
Excel (Data Preparation)
      ↓
AWS S3 (Cloud Storage)
      ↓
AWS RDS + MySQL (Database & Analysis)
      ↓
Power BI (Dashboard) — Coming Soon
```

---

## Data Cleaning Challenges

Real data is never clean. Here is what I fixed:

- Detected hidden carriage return characters using HEX() function
- Removed special characters using REPLACE() and CHAR(13)
- Converted data types for accurate decimal calculations
- Validated data integrity across all 21 columns

---

## SQL Analysis — 5 Queries

### Query 1 — Overall Churn Rate
```sql
SELECT 
    COUNT(*) AS total_customers,
    SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS churned_customers,
    ROUND(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) 
        * 100.0 / COUNT(*), 2) AS churn_rate_percentage
FROM customer_churn;
```
**Result:** 26.54% churn rate — nearly 3x higher than industry standard

---

### Query 2 — Churn by Contract Type
```sql
SELECT 
    Contract,
    COUNT(*) AS total_customers,
    SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS churned_customers,
    ROUND(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) 
        * 100.0 / COUNT(*), 2) AS churn_rate_percentage
FROM customer_churn
GROUP BY Contract
ORDER BY churn_rate_percentage DESC;
```
**Result:** Month-to-month: 42.71% vs Two-year: 2.83% — 15x difference

---

### Query 3 — Churn by Internet Service
```sql
SELECT 
    InternetService,
    COUNT(*) AS total_customers,
    SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS churned_customers,
    ROUND(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) 
        * 100.0 / COUNT(*), 2) AS churn_rate_percentage
FROM customer_churn
GROUP BY InternetService
ORDER BY churn_rate_percentage DESC;
```
**Result:** Fiber optic: 41.89% — most expensive service, highest churn

---

### Query 4 — Churn by Tenure Group
```sql
SELECT 
    CASE 
        WHEN TenureMonths <= 12 THEN '0-12 Months'
        WHEN TenureMonths <= 24 THEN '13-24 Months'
        WHEN TenureMonths <= 48 THEN '25-48 Months'
        ELSE '49-72 Months'
    END AS tenure_group,
    COUNT(*) AS total_customers,
    SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS churned_customers,
    ROUND(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) 
        * 100.0 / COUNT(*), 2) AS churn_rate_percentage
FROM customer_churn
GROUP BY tenure_group
ORDER BY churn_rate_percentage DESC;
```
**Result:** New customers (0-12 months): 47.44% — first year is make or break

---

### Query 5 — High Risk Customer Identification
```sql
SELECT 
    CustomerID,
    TenureMonths,
    Contract,
    InternetService,
    MonthlyCharges,
    CASE 
        WHEN MonthlyCharges > 80 THEN 'High Value'
        WHEN MonthlyCharges > 50 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_value
FROM customer_churn
WHERE Contract = 'Month-to-month'
AND InternetService = 'Fiber optic'
AND TenureMonths <= 12
AND Churn = 'No'
ORDER BY MonthlyCharges DESC;
```
**Result:** 273 customers identified — $264,058 yearly revenue at risk

---

## Key Findings

| Analysis | Finding |
|---|---|
| Overall Churn Rate | 26.54% — nearly 1 in 3 customers leaving |
| Churn by Contract | Month-to-month: 42.71% vs Two-year: 2.83% |
| Churn by Internet | Fiber optic: 41.89% vs No internet: 7.40% |
| Churn by Tenure | New customers (0-12 months): 47.44% churn |
| High Risk Customers | 273 customers — $264,058 yearly revenue at risk |

---

## Business Recommendation

If the company spends $100 per customer on retention offers:
- Cost: 273 × $100 = **$27,300**
- Revenue protected: **$264,058**
- Return on investment: **10x**

---

## Screenshots

### Query 1 — Overall Churn Rate
![Overall Churn Rate](screenshots/churn_rate.jpg)

### Query 3 — Churn by Internet Service
![Churn by Internet Service](screenshots/churn_internet.jpg)

### Query 4 — Churn by Tenure Group
![Churn by Tenure](screenshots/churn_tenure.jpg)

### Query 5 — High Risk Customer Count
![High Risk Customers](screenshots/high_risk.jpg)

### Query 5 — Revenue at Risk
![Revenue at Risk](screenshots/revenue_risk.jpg)
---

## SQL Concepts Used

- GROUP BY for segment analysis
- CASE WHEN for conditional counting and binning
- HEX() and LENGTH() for data quality checks
- REPLACE() and CHAR() for cleaning hidden characters
- Multiple WHERE conditions for high-risk filtering

---

## Part 2 — Coming Soon
Power BI dashboard built on top of this analysis

---

*Project by Nitin | March 2026*
