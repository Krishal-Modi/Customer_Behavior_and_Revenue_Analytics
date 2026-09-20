<div align="center">

# 🛍️ Customer Behavior & Revenue Analytics

### End-to-End Retail Analytics: Python → PostgreSQL → Power BI

![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white)

*Turning 3,900 retail transactions into revenue, loyalty, and marketing insights.*

</div>

---

## 📑 Table of Contents

- [Business Problem](#-business-problem)
- [Project Highlights](#-project-highlights)
- [Tech Stack](#%EF%B8%8F-tech-stack)
- [Workflow](#-workflow)
- [Dataset](#-dataset)
- [Data Preparation (Python)](#-data-preparation-python)
- [SQL Analysis](#-sql-analysis)
- [Power BI Dashboard](#-power-bi-dashboard)
- [Key Insights & Recommendations](#-key-insights--recommendations)
- [Repository Structure](#-repository-structure)
- [How to Run](#-how-to-run)
- [Skills Demonstrated](#-skills-demonstrated)
- [Author](#-author)

---

## 🎯 Business Problem

A leading retail company has noticed shifting purchase patterns across demographics, product categories, and sales channels. Management wants to know which factors, such as **discounts, reviews, seasons, shipping, and subscriptions**, actually drive consumer decisions and repeat purchases.

> **Core question:** *How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?*

---

## ✨ Project Highlights

- 🧹 **Cleaned and engineered** a 3,900-record dataset in Python (missing-value imputation, feature engineering, schema standardization)
- 🗄️ **Loaded data into PostgreSQL** programmatically using SQLAlchemy
- 🔍 **Answered 10 business questions** with SQL using CTEs, window functions, subqueries, and conditional aggregation
- 📊 **Built an interactive Power BI dashboard** with KPI cards, drill-down charts, and four dynamic slicers
- 💡 **Translated findings into actionable recommendations** for marketing, pricing, and loyalty strategy

---

## 🛠️ Tech Stack

| Layer | Tools |
|---|---|
| **Language** | Python 3, SQL |
| **Data Processing** | Pandas, NumPy |
| **Database** | PostgreSQL (MySQL and SQL Server connection code also included) |
| **DB Connectivity** | SQLAlchemy, psycopg2 |
| **Visualization** | Power BI Desktop, DAX |
| **Environment** | Jupyter Notebook, pgAdmin, Anaconda |
| **Version Control** | Git, GitHub |

---

## 🔄 Workflow

```
 Raw CSV Data
      │
      ▼
┌───────────────────┐
│  Python (Pandas)  │  → Clean, impute, engineer features
└───────────────────┘
      │
      ▼
┌───────────────────┐
│    PostgreSQL     │  → Load via SQLAlchemy, query with SQL
└───────────────────┘
      │
      ▼
┌───────────────────┐
│     Power BI      │  → Interactive dashboard & insights
└───────────────────┘
```

---

## 📂 Dataset

**Source:** Customer shopping behavior dataset · **Size:** 3,900 rows × 18 columns

| Column | Description |
|---|---|
| `customer_id` | Unique customer identifier |
| `age`, `gender` | Customer demographics |
| `item_purchased`, `category` | Product purchased (25 items across 4 categories) |
| `purchase_amount` | Transaction value in USD |
| `location` | US state (50 unique) |
| `size`, `color`, `season` | Product attributes and seasonality |
| `review_rating` | Customer rating (2.5 to 5.0) |
| `subscription_status` | Subscriber or non-subscriber |
| `shipping_type` | Standard, Express, Free, Store Pickup, etc. |
| `discount_applied` | Whether a discount was used |
| `previous_purchases` | Number of prior purchases (loyalty proxy) |
| `payment_method` | Preferred payment type |
| `frequency_of_purchases` | How often the customer buys |

---

## 🧹 Data Preparation (Python)

All steps are in [`Customer_Shopping_Behavior_Analysis.ipynb`](Customer_Shopping_Behavior_Analysis.ipynb).

| Step | Action | Why |
|---|---|---|
| 1 | Loaded data and ran `.info()`, `.describe()` | Understand structure and distributions |
| 2 | Found **37 missing** values in `Review Rating` | Only column with nulls |
| 3 | Imputed with **median rating per product category** | More accurate than a global median; robust to outliers |
| 4 | Standardized column names to `snake_case` | SQL-friendly, consistent naming |
| 5 | Engineered **`age_group`** using quartiles (`pd.qcut`) | Segment into Young Adult, Adult, Middle-aged, Senior with balanced sizes |
| 6 | Engineered **`purchase_frequency_days`** | Converted text frequencies to numeric days for analysis |
| 7 | Verified `discount_applied` = `promo_code_used` for **all rows** and dropped the duplicate column | Removed redundant feature |
| 8 | Loaded cleaned data into PostgreSQL with SQLAlchemy | Enables SQL analysis and Power BI connection |

```python
# Example: category-aware imputation
df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(
    lambda x: x.fillna(x.median())
)
```

---

## 🔍 SQL Analysis

All queries are in [`customer_behavior_sql_queries.sql`](customer_behavior_sql_queries.sql).

| # | Business Question | Techniques |
|---|---|---|
| Q1 | Revenue by gender | `GROUP BY`, `SUM` |
| Q2 | Discount users who still spent above average | Subquery |
| Q3 | Top 5 products by average review rating | `AVG`, `ROUND`, `LIMIT` |
| Q4 | Standard vs Express shipping spend | Filtering, `AVG` |
| Q5 | Do subscribers spend more? | Multi-metric aggregation |
| Q6 | Top 5 products by discount rate | Conditional aggregation (`CASE WHEN`) |
| Q7 | Segment customers: New / Returning / Loyal | CTE, `CASE` |
| Q8 | Top 3 products within each category | CTE, `ROW_NUMBER() OVER (PARTITION BY ...)` |
| Q9 | Are repeat buyers more likely to subscribe? | Filtering, grouping |
| Q10 | Revenue contribution by age group | `GROUP BY`, `ORDER BY` |

<details>
<summary><b>📌 Sample query: Top 3 products per category (click to expand)</b></summary>

```sql
WITH item_counts AS (
    SELECT category,
           item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (
               PARTITION BY category
               ORDER BY COUNT(customer_id) DESC
           ) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```
</details>

---

## 📊 Power BI Dashboard

File: [`customer_behavior_dashboard.pbix`](customer_behavior_dashboard.pbix), connected live to the PostgreSQL `customer` table.

**KPI Cards**
- Total number of customers
- Average purchase amount
- Average review rating

**Visuals**
- 🍩 % of customers by subscription status (donut)
- 📊 Revenue by category and sales by category (column charts)
- 📊 Revenue by age group and sales by age group (bar charts)

**Interactive Slicers**
- Subscription status · Gender · Category · Shipping type

---

## 💡 Key Insights & Recommendations

> ⚠️ **Before publishing:** run your queries and replace the bracketed placeholders below with your real numbers. Recruiters look for specific figures.

### Insights

1. **Revenue by gender:** [Male/Female] customers generate [X]% of total revenue.
2. **Subscribers:** Subscribers spend [more/less/similar] on average ($[X] vs $[Y]) than non-subscribers.
3. **Shipping:** [Express/Standard] shipping is associated with a higher average order value ($[X] vs $[Y]).
4. **Loyalty:** [X]% of customers are classified as Loyal (more than 10 previous purchases).
5. **Age groups:** [Age group] contributes the highest revenue at [X]%.
6. **Discounts:** [Product] has the highest discount rate at [X]%.

### Recommendations

| Finding | Recommended Action |
|---|---|
| Discount-heavy products | Review pricing and margin; avoid training customers to wait for sales |
| High-value shipping tier | Promote it at checkout for larger baskets |
| Low subscription conversion among repeat buyers | Target loyal customers with membership offers |
| Top-revenue age group | Focus campaigns and product mix on this segment |
| Top-rated products | Feature in marketing and cross-sell bundles |

---

## 📁 Repository Structure

```
Customer_Behavior_and_Revenue_Analytics/
│
├── Customer_Shopping_Behavior_Analysis.ipynb   # Data cleaning, feature engineering, SQL load
├── customer_behavior_sql_queries.sql           # 10 business-question queries
├── customer_behavior_dashboard.pbix            # Power BI dashboard
├── customer_shopping_behavior.csv              # Raw dataset
└── README.md
```

---

## 🚀 How to Run

**1. Clone the repository**
```bash
git clone https://github.com/Krishal-Modi/Customer_Behavior_and_Revenue_Analytics.git
cd Customer_Behavior_and_Revenue_Analytics
```

**2. Install dependencies**
```bash
pip install pandas numpy sqlalchemy psycopg2-binary jupyter
```

**3. Create a PostgreSQL database**
```sql
CREATE DATABASE customer_behavior;
```

**4. Run the notebook**

Open `Customer_Shopping_Behavior_Analysis.ipynb`, update the connection details with **your own credentials**, and run all cells. This cleans the data and loads it into a table named `customer`.

**5. Run the SQL queries**

Open `customer_behavior_sql_queries.sql` in pgAdmin (or any SQL client) and execute the queries.

**6. Open the dashboard**

Open `customer_behavior_dashboard.pbix` in Power BI Desktop and point the data source to your PostgreSQL database.

> 🔐 **Security note:** never commit real database passwords. Use environment variables or a `.env` file.

---

## 🧠 Skills Demonstrated

`Data Cleaning` · `Feature Engineering` · `Exploratory Data Analysis` · `Advanced SQL (CTEs, Window Functions)` · `Customer Segmentation` · `KPI Design` · `Dashboard Development` · `Data Storytelling` · `Business Recommendations`

---

## 👤 Author

**Krishal Modi**
Master of Applied Computing, University of Windsor

[![GitHub](https://img.shields.io/badge/GitHub-Krishal--Modi-181717?style=flat&logo=github)](https://github.com/Krishal-Modi)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/)

---

