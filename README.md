# Retail Sales Data Pipeline & Executive Dashboard

A comprehensive data analysis project simulating a mid-size retail chain 
environment. Built from synthetic data generation through ETL, relational 
database design, multi-tool analysis, and automated executive reporting.

---

## Project Overview

| Phase | Component | Tools | Skills |
|-------|-----------|-------|--------|
| **Data Engineering** | Synthetic data generation, ETL pipeline | Python, pandas | Data modeling, validation, automation |
| **Database Design** | Star schema, relational integrity | SQLite | Schema design, foreign keys, indexing |
| **Business Analysis** | KPIs, trends, segmentation | Excel, SQL, Python | Pivot tables, window functions, RFM |
| **Data Science** | Forecasting, customer segmentation | Python, scikit-learn | Linear regression, quintile scoring |
| **Visualization** | Executive dashboard automation | matplotlib | GridSpec, multi-panel layouts |

---

## Business Scenario

You are the sole data analyst at a retail chain with **4 stores**, **3 product 
categories**, and **2 years of transactional data**. The CEO requires:

1. Quarterly performance dashboard with KPIs
2. Store and category growth analysis (quarter-over-quarter)
3. Next-quarter revenue forecast
4. Customer segmentation for targeted marketing

---

## Dataset Architecture

| File | Records | Content | Role |
|------|---------|---------|------|
| `sales_transactions.csv` | 100,000 | Individual transactions with discounts | **Fact table** |
| `store_locations.csv` | 4 | Store metadata, region, manager | **Dimension** |
| `product_master.csv` | 200 | Product details, category, cost, launch date | **Dimension** |
| `customers.csv` | 5,000 | Demographics, signup date, segment | **Dimension** |

**Synthetic data features:**
- Seasonality (Nov-Dec and Jun-Jul spikes)
- Price-quantity correlation (cheaper items = higher quantities)
- Realistic discount patterns (10% of transactions)
- Segment-based customer distributions

---

## Repository Structure
<img width="688" height="371" alt="image" src="https://github.com/user-attachments/assets/beb1f883-0904-49e6-98c6-7c710811a47f" />
<img width="545" height="421" alt="image" src="https://github.com/user-attachments/assets/fb865196-7710-4268-8ab6-b24bab5efbdd" />

---

## Key Analyses & Findings

### 1. Quarterly Performance (KPI Dashboard)
| Metric | Calculation | Tool |
|--------|-------------|------|
| Total Revenue | `SUM(revenue)` | SQL + Excel |
| Total Transactions | `COUNT(*)` | SQL |
| Average Order Value | `AVG(revenue)` | SQL |
| QoQ Growth | `(Q2 - Q1) / Q1` with `LAG()` | SQL window functions |

### 2. Store & Category Growth
- **Window function:** `LAG(revenue) OVER (PARTITION BY region, category ORDER BY yr, qtr)`
- **Critical lesson:** Calculate `LAG()` in CTE before filtering with `WHERE`

### 3. Next-Quarter Forecast
- **Method:** Linear regression on monthly revenue trend
- **Python:** `sklearn.linear_model.LinearRegression`
- **Assumption:** Linear trend continues (documented for stakeholders)

### 4. RFM Customer Segmentation

| Segment | R Score | F Score | Action |
|---------|---------|---------|--------|
| Champions | ≥4 | ≥4 | Reward, upsell |
| Loyal Customers | ≥3 | ≥3 | Retain, grow |
| New Customers | ≥4 | ≤2 | Onboard, engage |
| At Risk | ≤2 | ≥3 | Win-back campaign |
| Lost | Low | Low | Re-activation or ignore |

---

## Technical Highlights

### ETL Pipeline (Python)
```python
def load_and_validate(file_path, required_columns):
    """Load CSV, standardize columns, check nulls, validate schema."""
    df = pd.read_csv(file_path)
    df.columns = df.columns.str.strip().str.lower().str.replace(' ', '_')
    # ... validation logic
    return df
```
```sql
CREATE TABLE sales (
    transaction_id INTEGER PRIMARY KEY,
    date TEXT,
    store_id INTEGER,
    product_id INTEGER,
    customer_id INTEGER,
    quantity INTEGER,
    unit_price REAL,
    discount_pct REAL,
    revenue REAL,
    FOREIGN KEY (store_id) REFERENCES stores(store_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```
Quarter-over-Quarter Growth (SQLite Window Functions)
WITH quarterly AS (
    SELECT 
        strftime('%Y', date) AS yr,
        CASE 
            WHEN CAST(strftime('%m', date) AS INT) BETWEEN 1 AND 3 THEN 1
            WHEN CAST(strftime('%m', date) AS INT) BETWEEN 4 AND 6 THEN 2
            WHEN CAST(strftime('%m', date) AS INT) BETWEEN 7 AND 9 THEN 3
            ELSE 4
        END AS qtr,
        region,
        category,
        SUM(revenue) AS revenue
    FROM sales s
    JOIN stores st ON s.store_id = st.store_id
    JOIN products p ON s.product_id = p.product_id
    GROUP BY yr, qtr, region, category
),
with_lag AS (
    SELECT 
        yr, qtr, region, category, revenue,
        LAG(revenue) OVER (PARTITION BY region, category ORDER BY yr, qtr) AS prev_quarter
    FROM quarterly
)
SELECT 
    region, category, revenue, prev_quarter,
    ROUND((revenue - prev_quarter) * 100.0 / prev_quarter, 2) AS growth_pct
FROM with_lag
WHERE yr = '2024' AND qtr = 2;
<img width="684" height="405" alt="image" src="https://github.com/user-attachments/assets/43efc55b-6c28-4a97-a833-90758f950f63" />
<img width="755" height="354" alt="image" src="https://github.com/user-attachments/assets/c4d32037-fb89-480d-9d88-7d69b6d28649" />
4. Run Analysis
Excel: Open excel/executive_summary.xlsx
SQL: Execute queries 03–05 in SQLite
Python: Run notebooks/analysis.ipynb or scripts/executive_dashboard.py
<img width="789" height="380" alt="image" src="https://github.com/user-attachments/assets/873b4a40-ec61-44e0-a68c-43d96b2960af" />
Future Enhancements
[ ] Power BI/Tableau live dashboard connection
[ ] ARIMA or Prophet forecasting (replace linear regression)
[ ] Automated email report distribution (Python smtplib)
[ ] A/B test framework for retention offers
[ ] Migrate from SQLite to PostgreSQL for production scale
About This Project
```
```
Built as part of a project-based learning approach to data analytics.
Same scenario analyzed across Excel, SQL, and Python to understand
when each tool adds value and how they hand off in a real workflow.
Connect: [LinkedIn](https://www.linkedin.com/in/marshal-favour/?lipi=urn%3Ali%3Apage%3Ad_flagship3_feed%3BlfOFjvuoQ4W2y1UVtIhGTQ%3D%3D)
```
```
License
MIT License — feel free to fork, modify, and build upon this structure
for your own learning or portfolio.



