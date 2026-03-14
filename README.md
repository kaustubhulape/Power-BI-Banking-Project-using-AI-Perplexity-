# 🏦 Power BI Banking Dashboard

A complete end-to-end **Banking Analytics project** built with SQL Server and Power BI. This project demonstrates real-world data engineering practices — including intentional dirty data, multi-format date normalization, referential integrity challenges, and a fully interactive Power BI dashboard with DAX-powered KPIs. Raw data is collected using AI tool (Perplexity.ai)

---

## 📁 Project Structure

```
Power-BI-Banking/
│
├── SQL_Query_1.txt          # Database creation, schema definition & synthetic data generation
├── SQL_Query_2.txt          # Data cleaning — date normalization across all tables
├── SQL_Query_3.txt          # Final combined dataset via LEFT JOINs
│
├── Power_BI_Banking.pbix    # Power BI Dashboard file
│
└── KPI Reference/
    ├── KPI-Description-Visual-DAXCalculatedColumnExample_Page_1.csv
    └── KPI-Description-Visual-DAXCalculatedColumnExample_Page_2.csv
```

---

## 🗄️ Database: `Power_BI2`

### Tables

| Table | Description |
|---|---|
| `Customers` | 5 customer records with intentional data quality issues (missing emails, inconsistent name casing, mixed DOB formats) |
| `Accounts` | 5 accounts with outlier balances, inconsistent type casing, mixed date formats, and an orphaned `CustomerID` |
| `Transactions` | 10,000 synthetic rows with mixed date formats, negative outliers, NULL descriptions, and orphaned `AccountID` references |

### Intentional Data Issues (for demo/learning)
- Mixed date formats: `yyyy-MM-dd`, `dd-MM-yyyy`, `yyyy/MM/dd`, `MM/dd/yyyy`
- Inconsistent text casing: `'SAVINGS'`, `'current'`, `'Savings'`
- NULL values in `Email`, `Address`, `Description`
- Orphaned foreign keys (e.g., `CustomerID = 99` in Accounts, `AccountID = 9999` in Transactions)
- Negative account balances and transaction amount outliers

---

## 🧹 Data Cleaning (SQL Query 2)

All date fields across `Customers`, `Accounts`, and `Transactions` are normalized to a unified `MM/dd/yyyy` format using `TRY_CONVERT` with multiple SQL Server style codes:

| Style Code | Format Handled |
|---|---|
| 101 | `MM/DD/YYYY` |
| 23 | `YYYY-MM-DD` |
| 111 | `YYYY/MM/DD` |
| 105 | `DD-MM-YYYY` |
| 103 | `DD/MM/YYYY` |

Unrecognized formats are left unchanged (fail-safe).

---

## 🔗 Combined Dataset (SQL Query 3)

A denormalized view `CombinedBankingDataset` is created by LEFT JOINing all three tables:

```
Transactions → Accounts (on AccountID) → Customers (on CustomerID)
```

This single flat table is the **data source for the Power BI dashboard**.

---

## 📊 Power BI Dashboard KPIs

### Page 1 KPIs

| KPI | Visual | Description |
|---|---|---|
| Transactions by Type | Pie / Stacked Column | Volume/proportion of Credit vs Debit |
| Monthly Transaction Amount | Line / Area Chart | Total transaction value per month |
| Top N Customers by Transaction Value | Bar Chart (Top N filter) | Highest-value customers |
| Average Account Balance | Column Chart / Gauge | Mean balance per account |
| Total Balance by Account Type | Clustered Bar / Column | Aggregated balance per account type |
| Inactive Accounts (Last 90 Days) | Clustered Bar / Table | Accounts with no recent transactions |

### Page 2 KPIs

| KPI | Visual | Description |
|---|---|---|
| Customer Gender Distribution | Donut Chart | Proportion of customers by gender |
| Accounts by Account Type | Clustered Bar / Treemap | Count of accounts per type |
| Transaction Volume Trend | Line / Area Chart | Monthly transaction count over time |
| Customer Location Distribution | Filled Map / Column | Geographic spread of customers |

---

## ⚙️ DAX Measures (Examples)

```dax
-- Monthly Transaction Amount
Monthly Transaction Amount =
CALCULATE(
    SUM(CombinedBankingDataset[Amount]),
    ALLEXCEPT(CombinedBankingDataset, CombinedBankingDataset[TransactionDate].[Month])
)

-- Average Account Balance
Average Balance = AVERAGE(CombinedBankingDataset[Balance])

-- Total Balance by Account Type
Total Balance = SUM(CombinedBankingDataset[Balance])

-- Inactive Accounts (Last 90 Days)
Inactive Accounts =
CALCULATE(
    DISTINCTCOUNT(CombinedBankingDataset[Account_AccountID]),
    FILTER(
        VALUES(CombinedBankingDataset[Account_AccountID]),
        CALCULATE(MAX(CombinedBankingDataset[TransactionDate])) < TODAY() - 90
    )
)
```

---

## 🚀 How to Run

### Prerequisites
- SQL Server (2016 or later recommended)
- SQL Server Management Studio (SSMS)
- Power BI Desktop

### Steps

1. **Set up the database**
   - Open and run `SQL_Query_1.txt` in SSMS
   - This creates the `Power_BI2` database, all tables, and inserts synthetic data

2. **Clean the data**
   - Run `SQL_Query_2.txt` to normalize all date fields

3. **Build the combined dataset**
   - Run `SQL_Query_3.txt` to create the `CombinedBankingDataset` table

4. **Open Power BI**
   - Open `Power_BI_Banking.pbix` in Power BI Desktop
   - Update the data source connection to point to your SQL Server instance
   - Refresh the data

---

## 🎯 Learning Objectives

- Handling dirty/real-world data in SQL Server
- Multi-format date parsing and normalization with `TRY_CONVERT`
- Building denormalized datasets for BI tools
- Creating DAX measures and calculated columns in Power BI
- Designing banking KPI dashboards from scratch

---

## 📌 Notes

- The project uses `LEFT JOIN` intentionally to preserve all transactions even when account or customer data is missing — reflecting real-world data pipeline design
- Orphaned records and NULLs are preserved to demonstrate how Power BI handles incomplete data
- The `CombinedBankingDataset` table is designed as a snapshot — for production use, consider views or incremental refresh

---

## 🛠️ Tech Stack

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=flat&logo=microsoft-sql-server&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)

---
