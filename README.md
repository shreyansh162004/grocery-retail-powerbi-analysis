# 📊 End-to-End Finance Data Analysis Project: SQL, Python, Excel & Power BI

A complete data analysis project on financial transaction data. Raw data is first **cleaned and validated using Excel, SQL and Python (Pandas)**, then analyzed and turned into interactive executive dashboards in **Power BI** with dynamic metric switching, Year-over-Year (YoY) analysis, dynamic visual headers and drill-through operational reporting.

<p align="center">
  <a href="images/dashboard-overview.png">
    <img src="images/dashboard-overview.png" alt="Dashboard Preview" width="100%">
  </a>
</p>

**Data flow:** Raw CSV → Excel (inspection) → SQL (validation & analysis) → Python/Pandas (cleaning & EDA) → Power Query → Power BI (model, DAX, dashboards)

---

## 📑 Table of Contents

- [Project Overview](#project-overview)
- [Business Requirements](#business-requirements)
- [KPI Requirements](#kpi-requirements)
- [Chart Requirements](#chart-requirements)
- [Data Analysis Workflow](#data-analysis-workflow)
- [Data Cleaning & Preparation](#data-cleaning-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Key Features](#key-features)
- [Data Model & Architecture](#data-model-architecture)
- [DAX Calculations](#dax-calculations)
- [Dashboard Pages](#dashboard-pages)
- [Screenshots](#screenshots)
- [Key Insights](#key-insights)
- [Tech Stack](#tech-stack)
- [Folder Structure](#folder-structure)
- [What I Learned](#what-i-learned)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

<a id="project-overview"></a>

## 📌 Project Overview

Financial reporting often needs the same slices (Month, State, Customer Segment, Transaction Type) viewed across several metrics such as Total Amount, Fees, Tax and Transaction Volume. Building a separate chart for every metric clutters a report and makes it harder to use.

This project covers the **full analytics cycle**: understanding the data, cleaning and validating it with Excel, SQL and Python, analyzing it, and presenting the results in Power BI. **Field Parameters** let one dropdown switch the metric across all visuals, and **DAX time-intelligence functions** track year-over-year trends across the key KPIs.

---

<a id="business-requirements"></a>

## 🎯 Business Requirements

**Project:** Finance Analysis Project

A financial organization wants an interactive **Finance Analytics Dashboard in Power BI** to monitor and analyze overall financial transactions, customer behavior, fees, taxes and transaction performance across different business segments and regions.

### Challenges Faced by Management

The management team struggles to track:

- Overall transaction growth and financial performance
- Monthly trends in transaction amounts
- Successful vs. failed transactions
- Customer segment contribution
- State-wise financial performance
- Transaction type profitability
- Gender-based customer analysis
- Year-over-Year (YoY) performance changes

### Objective

Provide a centralized analytical solution that helps stakeholders:

- Monitor KPIs in real time
- Identify high-performing customer segments and states
- Analyze transaction patterns and trends
- Track operational fees and taxes
- Understand customer demographics
- Improve financial decision-making and business strategy

### Dynamic Filters

Users can filter the data dynamically by:

- **Year**
- **Dynamic Measure**
- **Occupation**
- **Category**

---

<a id="kpi-requirements"></a>

## 📈 KPI Requirements

| # | KPI | Description |
|---|---|---|
| 1 | **Total Amount** | Total transaction amount processed, with YoY growth comparison |
| 2 | **Total Transactions** | Total number of transactions, tracking yearly volume changes |
| 3 | **Average Transaction Value** | Average amount per transaction |
| 4 | **Total Fees** | Total fees collected from transactions |
| 5 | **Total Tax** | Total tax generated from all transactions |

---

<a id="chart-requirements"></a>

## 📊 Chart Requirements

| # | Chart | Type | Objective |
|---|---|---|---|
| 1 | **Total Amount by Month** | Line / Area Chart | Analyze monthly transaction trends and identify seasonal spikes or drops |
| 2 | **Total Amount by Transaction Status** | Donut Chart | Compare Success, Failed and Pending amounts to measure operational efficiency and success rate |
| 3 | **Total Amount by Customer Segment** | Horizontal Bar Chart | Analyze contribution of Retail, Premium, SME, Corporate and Wealth segments to find the most valuable groups |
| 4 | **Total Amount by State** | Horizontal Bar Chart | Compare state-wise amounts to identify top-performing regions |
| 5 | **Transaction Type Analysis** | Matrix / Heatmap Table | Show Amount, Fees, Tax and Transaction Count by type to understand profitability by category |
| 6 | **Total Amount by Gender** | Donut Chart | Analyze Male vs. Female contribution to understand demographic participation |

**Transaction types covered:** Bill Payment, Card Payment, Deposit, Fee Charge, Interest Credit, Investment, Loan EMI, Refund, Transfer, Withdrawal.

### Dashboard 2: Detailed Grid View & Drill-Down

A detailed grid view with drill-down to the underlying transaction records, so users can inspect and export row-level data behind any chart.

---

<a id="data-analysis-workflow"></a>

## 🔄 Data Analysis Workflow

| Step | Phase | Tools | What was done |
|---|---|---|---|
| 1 | **Data Understanding** | Excel | Opened the raw file, reviewed columns, data types, ranges and value lists |
| 2 | **Data Cleaning** | Excel, SQL, Python | Removed duplicates, fixed data types and dates, standardized text, handled missing values |
| 3 | **Data Validation** | SQL, Python | Checked business rules (for example tax vs. fees, invalid amounts, valid status values) |
| 4 | **Exploratory Analysis** | SQL, Python | Explored trends, segments, regions and transaction types before building visuals |
| 5 | **Data Modeling** | Power Query, Power BI | Loaded cleaned data, built a star schema and a date table |
| 6 | **Measures & Dashboards** | DAX, Power BI | Built KPIs, YoY measures, Field Parameters and two dashboards |
| 7 | **Insights** | Power BI | Interpreted the results into business insights |

---

<a id="data-cleaning-preparation"></a>

## 🧹 Data Cleaning & Preparation

Raw data was cleaned **before** analysis so that every number on the dashboard can be trusted.

**Dataset columns:** `transaction_id`, `transaction_date`, `customer_name`, `transaction_type`, `transaction_status`, `gender`, `customer_segment`, `state`, `city`, `occupation`, `merchant_category`, `amount`, `fee_amount`, `tax_amount`

**Source:** [dataset source, e.g. Kaggle link or provided in tutorial]

### 1️⃣ Excel: First Inspection

- Reviewed columns, data types and value ranges
- Used **Remove Duplicates** on `transaction_id`
- Used **TRIM / PROPER** to fix extra spaces and inconsistent text
- Used **Filter** and **Conditional Formatting** to spot blanks and outliers
- Checked that dates were in one consistent format

### 2️⃣ SQL: Validation and Cleaning Queries

```sql
-- Duplicate transaction IDs
SELECT transaction_id, COUNT(*) AS cnt
FROM finance_transaction
GROUP BY transaction_id
HAVING COUNT(*) > 1;

-- Missing values in key columns
SELECT
    SUM(CASE WHEN transaction_date   IS NULL THEN 1 ELSE 0 END) AS null_date,
    SUM(CASE WHEN amount             IS NULL THEN 1 ELSE 0 END) AS null_amount,
    SUM(CASE WHEN transaction_status IS NULL THEN 1 ELSE 0 END) AS null_status
FROM finance_transaction;

-- Invalid amounts
SELECT * FROM finance_transaction WHERE amount <= 0;

-- Valid category values (spot spelling and case issues)
SELECT DISTINCT transaction_status FROM finance_transaction;
SELECT DISTINCT customer_segment   FROM finance_transaction;

-- Standardize text
UPDATE finance_transaction
SET transaction_status = TRIM(transaction_status),
    transaction_type   = TRIM(transaction_type),
    state              = TRIM(state),
    city               = TRIM(city);
```

### 3️⃣ Python (Pandas): Cleaning and Validation

```python
import pandas as pd

df = pd.read_csv("data/raw/finance_transaction.csv")

# Inspect
df.info()
print(df.isnull().sum())
print("Duplicates:", df.duplicated(subset="transaction_id").sum())

# Remove duplicates
df = df.drop_duplicates(subset="transaction_id")

# Fix data types
df["transaction_date"] = pd.to_datetime(df["transaction_date"], errors="coerce")

# Standardize text columns
text_cols = ["transaction_type", "transaction_status", "gender",
             "customer_segment", "state", "city", "occupation", "merchant_category"]
for col in text_cols:
    df[col] = df[col].astype(str).str.strip().str.title()

# Handle missing values
df[["fee_amount", "tax_amount"]] = df[["fee_amount", "tax_amount"]].fillna(0)
df = df.dropna(subset=["transaction_id", "transaction_date", "amount"])

# Validation: tax should be about 18% of fees
diff = (df["tax_amount"] - df["fee_amount"] * 0.18).abs()
print("Rows where tax != 18% of fees:", (diff > 0.05).sum())

# Save cleaned data for Power BI
df.to_csv("data/cleaned/finance_transaction_cleaned.csv", index=False)
```

### 4️⃣ Cleaning Summary

| Check | Method | Outcome |
|---|---|---|
| Duplicate transactions | Excel, SQL, Pandas | [rows removed] |
| Missing values | SQL, Pandas | [what was handled] |
| Date format and type | Excel, Pandas | Converted to a proper date type |
| Text inconsistencies | Excel, SQL, Pandas | Trimmed and standardized |
| Invalid amounts | SQL, Pandas | [rows reviewed or removed] |
| Tax vs. fees rule | Pandas | Tax equals 18% of fees |
| **Rows before / after** | | [raw rows] → [cleaned rows] |

The cleaned file was then loaded into Power BI through **Power Query**.

---

<a id="exploratory-data-analysis"></a>

## 🔍 Exploratory Data Analysis

Before building the dashboards, the cleaned data was explored with SQL and Python to answer the key business questions.

```sql
-- Transaction success rate
SELECT transaction_status,
       COUNT(*) AS transactions,
       ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) AS pct
FROM finance_transaction
GROUP BY transaction_status;

-- Top customer segments by amount
SELECT customer_segment, SUM(amount) AS total_amount
FROM finance_transaction
GROUP BY customer_segment
ORDER BY total_amount DESC;

-- Monthly trend
SELECT EXTRACT(YEAR FROM transaction_date)  AS yr,
       EXTRACT(MONTH FROM transaction_date) AS mth,
       SUM(amount) AS total_amount
FROM finance_transaction
GROUP BY yr, mth
ORDER BY yr, mth;

-- Fees and tax by transaction type
SELECT transaction_type,
       SUM(amount)     AS total_amount,
       SUM(fee_amount) AS total_fees,
       SUM(tax_amount) AS total_tax,
       COUNT(*)        AS transactions
FROM finance_transaction
GROUP BY transaction_type
ORDER BY total_amount DESC;
```

```python
# Quick EDA in Pandas
print(df.groupby("customer_segment")["amount"].sum().sort_values(ascending=False))
print(df.groupby("state")["amount"].sum().sort_values(ascending=False).head(10))
print(df["transaction_status"].value_counts(normalize=True) * 100)
```

---

<a id="key-features"></a>

## ✨ Key Features & Capabilities

1. **Dynamic Metric Switching (Field Parameters)**
   Switch between **Total Amount**, **Total Fees**, **Total Tax** and **Total Transactions** across all visuals with a single slicer.

2. **Time Intelligence & YoY Tracking**
   KPI cards show current values with YoY % growth and absolute variance versus the previous year (`SAMEPERIODLASTYEAR`).

3. **Dynamic KPI Headers & Context Labels**
   DAX picks up the selected metric and filter context (for example, `Total Amount 2025`) and shows it in visual titles and cards.

4. **Transaction Status & Demographics Breakdown**
   Distribution of transaction status (Success, Pending, Failed), gender split, customer segments and location-wise rankings.

5. **Operational Drill-Through & Data Export**
   Right-click a chart element (such as Pending status or a specific month) and drill through to a detailed transaction page to review row-level records and export them for audit or resolution.

6. **Custom UI/UX & Navigation**
   Custom card backgrounds, icons, synced slicers and page navigation buttons (Overview vs. Transactions).

---

<a id="data-model-architecture"></a>

## 📐 Data Model & Architecture

The model follows a **Star Schema** for fast DAX queries and reliable time intelligence:

<p align="center">
  <img src="images/data-model-star-schema.svg" alt="Star Schema Data Model" width="90%">
</p>

| Table | Type | Purpose |
|---|---|---|
| `finance_transaction` | Fact | `transaction_id`, `transaction_date`, `customer_name`, `transaction_type`, `transaction_status`, `gender`, `customer_segment`, `state`, `city`, `occupation`, `merchant_category`, `amount`, `fee_amount`, `tax_amount` |
| `calendar_table` | Dimension | Continuous dates, year, month name and `month_number` for correct Jan–Dec sorting |
| `Dynamic Metric` | Parameter | Field Parameter table used to switch metrics dynamically |

**Relationship:** `calendar_table[date]` → `finance_transaction[transaction_date]` (one-to-many). `calendar_table` is marked as the Date Table.

---

<a id="dax-calculations"></a>

## 🧮 DAX Calculations & Formulas

### 1. Total Transactions

```dax
total transactions =
DISTINCTCOUNT('finance_transaction'[transaction_id])
```

### 2. Previous Year Transactions

```dax
previous year transaction =
CALCULATE(
    [total transactions],
    SAMEPERIODLASTYEAR('calendar_table'[date])
)
```

### 3. YoY Transaction Growth %

```dax
year on year transactions % =
DIVIDE(
    [total transactions] - [previous year transaction],
    [previous year transaction],
    0
)
```

### 4. Average Transaction Value

```dax
average transaction value =
AVERAGE('finance_transaction'[amount])
```

### 5. Total Fees

```dax
total fee =
SUM('finance_transaction'[fee_amount])
```

### 6. Total Tax

```dax
total tax =
SUM('finance_transaction'[tax_amount])
```

### 7. Month Number (for sorting)

```dax
month number =
MONTH('calendar_table'[date])
```

### 8. Dynamic Title Context

```dax
dynamic item =
SWITCH(
    'Dynamic Metric'[Dynamic Metric Order],
    0, "Total Amount",
    1, "Total Fees",
    2, "Total Tax",
    3, "Total Transactions",
    "Other"
)
```

---

<a id="dashboard-pages"></a>

## 🖥️ Dashboard Layout & Visualizations

### Dashboard 1: Overview Analysis

- **Filters:** Year, Dynamic Measure, Occupation, Category
- **KPI Cards:** Total Amount, Total Transactions, Average Transaction Value, Total Fees, Total Tax, each with YoY variance and vs. previous year
- **Dynamic Trend (Area Chart):** Monthly trend driven by the metric dropdown
- **Transaction Status (Donut Chart):** Success vs. Pending vs. Failed
- **Customer Segment (Horizontal Bar Chart):** Performance across segments
- **City-wise Performance (Bar Chart):** Top cities ranked by the selected metric (state-level detail is available in the Transactions grid)
- **Transaction Type Matrix:** Breakdown by type with dynamic gradient formatting
- **Gender Split (Donut Chart):** Male vs. Female contribution

### Dashboard 2: Detailed Grid View & Drill-Down (Operational Transactions)

- Line-item table with `transaction_date`, `transaction_id`, `Customer name`, `transaction_status`, `transaction_type`, `gender`, `customer_segment`, `state`, Total Amount, Total fees and Total tax
- Year selector and Dynamic Measure dropdown, with the same KPI cards on top
- Drill-through enabled for quick inspection, with data export

---

<a id="screenshots"></a>

## 📸 Screenshots

### Overview Page

<p align="center">
  <a href="images/dashboard-overview.png">
    <img src="images/dashboard-overview.png" alt="Overview Dashboard" width="100%">
  </a>
</p>

🔗 **PNG link:** [images/dashboard-overview.png](images/dashboard-overview.png)

### Drill-Through Transactions Page

<p align="center">
  <a href="images/drill-through.png">
    <img src="images/drill-through.png" alt="Drill Through Transactions Page" width="100%">
  </a>
</p>

🔗 **PNG link:** [images/drill-through.png](images/drill-through.png)

---

<a id="key-insights"></a>

## 💡 Key Insights

- In 2023 the dashboard shows **₹137.07M** processed across **15K transactions**, an average of about **₹9.12K per transaction**
- Fees collected were **₹216.56K**, roughly **0.16%** of the total transaction amount
- Tax collected was **₹38.98K**, which is exactly **18% of total fees**, consistent with a GST-style tax applied on fees
- [Add 1–2 insights from the charts, e.g. top customer segment, top city, or peak month]

---

<a id="tech-stack"></a>

## 🛠️ Tech Stack & Tools Used

| Category | Tools |
|---|---|
| Data Cleaning | **Excel**, **SQL**, **Python (Pandas)** |
| Analysis | **SQL**, **Python (Pandas)** |
| ETL | **Power Query (M)** |
| Visualization & Modeling | **Power BI Desktop**, **DAX** |
| Version Control | **Git & GitHub** |

---

<a id="folder-structure"></a>

## 📁 Folder Structure

```text
finance-data-analysis-project/
│
├── data/
│   ├── raw/
│   │   ├── customers.csv
│   │   └── finance_transaction.csv
│   └── cleaned/
│       ├── customers.csv
│       └── finance_transaction.csv
│
├── powerbi/
│   └── Finance_Dashboard.pbix
│
├── images/
│   ├── dashboard-overview.png
│   ├── drill-through.png
│   └── data-model-star-schema.svg
│
└── README.md
```

---

<a id="what-i-learned"></a>

## 📚 What I Learned

- **Cleaning and validating** raw data using Excel, SQL and Python before analysis
- Writing SQL queries for duplicates, nulls, aggregation and window functions
- Using **Pandas** for cleaning, type conversion and exploratory analysis
- Building dynamic visuals with **Field Parameters**
- Writing **time-intelligence DAX** (`SAMEPERIODLASTYEAR`, `CALCULATE`, `DIVIDE`)
- Designing a **Star Schema** with a proper date table
- Setting up **drill-through** pages and exporting data for operational use
- [One challenge you faced and how you solved it]

---

<a id="future-improvements"></a>

## 🚀 Future Improvements

- Add forecasting for transaction amount
- Connect a live data source (SQL Database) instead of CSV
- Publish to Power BI Service with scheduled refresh
- Add row-level security by region or segment

---

<a id="author"></a>

## 👤 Author

**Shreyansh Burman**
Aspiring Data Analyst | B.Tech CSE (Computer Science & Design), GGITS Jabalpur

📧 Email: [shreyanshburman10@gmail.com](mailto:shreyanshburman10@gmail.com)

⭐ If you found this project useful, please give it a star!
