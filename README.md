# Banking Risk Analytics — Integrated Project
### SQL · Python EDA · Power BI Dashboard

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Business Questions](#2-business-questions)
3. [Solution Architecture](#3-solution-architecture)
4. [Dataset](#4-dataset)
5. [Component 1 — SQL](#5-component-1--sql)
6. [Component 2 — Python EDA](#6-component-2--python-eda)
7. [Component 3 — Power BI Dashboard](#7-component-3--power-bi-dashboard)
8. [Key Findings](#8-key-findings)
9. [Repository Structure](#9-repository-structure)
10. [How to Run](#10-how-to-run)

---

## 1. Project Overview

This project looks at a banking portfolio of **3,000 clients** using three tools:

- **MySQL** for database setup and initial exploration
- **Python** for exploratory data analysis
- **Power BI** for reporting and interactive analysis

The main goal was to understand the portfolio from three angles:

**risk, growth and profitability.**

The workflow was:

```text
SQL → Python EDA → Power BI
```

Each stage had a different purpose. SQL was used to set up and inspect the data, Python was used to explore distributions and relationships, and Power BI was used to bring the results together into a 9-page dashboard.

## Dashboard Preview

### Banking Portfolio Executive Overview

![Banking Portfolio Executive Overview](screenshot/01.home.png)

---

## 2. Business Questions

The project was built around a few practical questions:

- Which client groups hold the largest loan exposure?
- Which clients may be carrying more debt than their deposits can support?
- Which customer segments are driving portfolio growth?
- How are deposits and lending distributed across banking relationships?
- Which client groups generate the most fee revenue?
- How do income, age, banking relationship and loyalty relate to financial behaviour?
- Are some customer groups showing signs of higher credit risk?

The aim was not to predict default, but to use the available portfolio data to identify areas that may deserve closer review.

---

## 3. Solution Architecture

```text
┌─────────────────────────────────────────────────────┐
│                   Raw Data Source                   │
│            Banking.csv / Banking01.xlsx             │
│        3,000 clients · 25 columns · 1999–2021      │
└──────────────────────┬──────────────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
         ▼             ▼             ▼
  ┌────────────┐ ┌────────────┐ ┌────────────────┐
  │    SQL     │ │   Python   │ │    Power BI    │
  │            │ │    EDA     │ │    Dashboard   │
  │ Database   │ │ Profiling  │ │ Power Query    │
  │ Setup      │ │ Income     │ │ DAX Measures   │
  │ Initial    │ │ Bands      │ │ 9 Pages        │
  │ Checks     │ │ Correlation│ │ Risk           │
  │            │ │ Analysis   │ │ Growth         │
  │            │ │            │ │ Profitability  │
  └────────────┘ └────────────┘ └────────────────┘
         │             │             │
         └─────────────┴─────────────┘
                       │
                       ▼
          ┌────────────────────────┐
          │   Business Analysis    │
          │  Risk · Growth · P&L   │
          └────────────────────────┘
```

---

## 4. Dataset

| Attribute | Detail |
|-----------|--------|
| File | `Banking.csv` / `Banking01.xlsx` |
| Rows | 3,000 clients |
| Columns | 25 raw variables |
| Date Range | 1999 – 2021 joining dates |
| Nulls | 0 |
| Age Range | 17 – 85 years |
| Unique Occupations | 195 |

### Column Reference

| Column | Type | Description |
|--------|------|-------------|
| Client ID | Text | Unique client identifier |
| Name | Text | Client full name |
| Age | Integer | Client age |
| Joined Bank | Date | Date client joined |
| Nationality | Text | Client nationality group |
| Occupation | Text | Client occupation |
| Fee Structure | Text | High / Mid / Low |
| Loyalty Classification | Text | Jade / Gold / Silver / Platinum |
| Estimated Income | Decimal | Estimated annual income |
| Superannuation Savings | Decimal | Retirement savings balance |
| Amount of Credit Cards | Integer | Number of credit cards held |
| Credit Card Balance | Decimal | Credit card balance |
| Bank Loans | Decimal | Outstanding bank loan balance |
| Bank Deposits | Decimal | Total bank deposit balance |
| Checking Accounts | Decimal | Checking account balance |
| Saving Accounts | Decimal | Savings account balance |
| Foreign Currency Account | Decimal | Foreign currency account balance |
| Business Lending | Decimal | Business lending balance |
| Properties Owned | Integer | Number of properties owned |
| Risk Weighting | Integer | Internal risk indicator |
| BRId | Integer | Banking Relationship ID |
| GenderId | Integer | Gender ID |
| IAId | Integer | Investment Advisor ID |

### ID Mappings

| Column | Values |
|--------|--------|
| BRId | 1 = Premium · 2 = Business · 3 = Personal · 4 = SME |
| GenderId | 1 = Male · 2 = Female |
| Fee Structure | High = 0.05 · Mid = 0.03 · Low = 0.01 |
| Loyalty | Jade · Gold · Silver · Platinum |

---

## 5. Component 1 — SQL

**File:** `Banking_analysis_sql.sql`  
**Tool:** MySQL  
**Purpose:** Database setup and initial data checks

### What Was Done

The SQL stage was used to create the database environment and confirm that the data loaded correctly before moving into Python and Power BI.

```sql
CREATE DATABASE banking_case;

USE banking_case;

SHOW TABLES;

SELECT * FROM customer;

SHOW VARIABLES WHERE Variable_name = 'hostname';

SELECT current_user();
```

### What This Stage Covers

- creates a separate project database
- checks that the customer table loaded correctly
- verifies the active database environment
- performs an initial review of the imported data
- records basic environment information for reproducibility

The deeper analytical work in this project was carried out in Python and Power BI.

---

## 6. Component 2 — Python EDA

**File:** `Banking_EDA_Case_Project.ipynb`  
**Tool:** Python  
**Libraries:** pandas, matplotlib, seaborn, numpy

The Python stage was used to understand the dataset before building the dashboard.

### Libraries Used

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
```

### Data Loading & Profiling

```python
df = pd.read_excel('/content/Banking01.xlsx')

df.head(5)
df.shape
df.info()
df.describe()
```

This confirmed a dataset containing **3,000 rows and 25 columns** with no missing values.

---

### Step 1 — Income Bands

Estimated income was grouped into three broad segments:

```python
bins = [0, 100000, 300000, float('inf')]
labels = ['Low', 'Mid', 'High']

df['Income Band'] = pd.cut(
    df['Estimated Income'],
    bins=bins,
    labels=labels,
    right=False
)
```

| Band | Income Range | Clients |
|------|-------------|---------|
| Low | < $100,000 | 287 |
| Mid | $100,000 – $300,000 | 1,835 |
| High | > $300,000 | 878 |

These bands were later reused in Power BI.

---

### Step 2 — Categorical Analysis

I reviewed the main categorical fields, including:

- Banking Relationship
- Gender
- Nationality
- Fee Structure
- Loyalty Classification
- Income Band

Some early observations were:

- Personal Banking was the largest relationship group
- Gender was close to evenly split
- High Fee Structure was the most common fee tier
- Jade was the largest loyalty group

---

### Step 3 — Gender Comparison

Categorical distributions were also compared by gender.

The purpose was to check whether banking relationship, loyalty, fee structure or income bands differed noticeably between male and female clients.

This later supported the gender filters used in Power BI.

---

### Step 4 — Nationality Comparison

The same approach was used to compare categories across nationality groups.

This helped identify differences in:

- banking relationship
- loyalty
- fee structure
- income band

These fields were later used in the loan and deposit dashboard pages.

---

### Step 5 — Numerical Distributions

Numerical variables were reviewed using histograms and KDE plots.

Main observations included:

- Bank Loans and Business Lending were right-skewed
- most clients had smaller balances, with a smaller group of very large balances
- Estimated Income was spread across a broad range
- Credit Card Balance was concentrated toward lower values
- Checking and Saving Accounts had similar distribution shapes

---

### Step 6 — Correlation Analysis

```python
numerical_cols = [
    'Age',
    'Estimated Income',
    'Superannuation Savings',
    'Credit Card Balance',
    'Bank Loans',
    'Bank Deposits',
    'Checking Accounts',
    'Saving Accounts',
    'Foreign Currency Account',
    'Business Lending',
    'Properties Owned'
]

correlation_matrix = df[numerical_cols].corr()

sns.heatmap(
    correlation_matrix,
    annot=True,
    cmap='coolwarm',
    fmt=".2f"
)
```

### Main Relationships Observed

- Bank Deposits and Checking Accounts showed a strong positive relationship
- Business Lending and Bank Loans showed a moderate positive relationship
- Age and Superannuation Savings were positively related
- Credit Card Balance showed relatively weak relationships with most other financial variables

These relationships were used as starting points for further analysis rather than treated as proof of causation.

---

### Step 7 — Variable Pair Analysis

Several variable pairs were selected for closer inspection:

```python
pairs_to_plot = [
    ('Bank Deposits', 'Saving Accounts'),
    ('Checking Accounts', 'Saving Accounts'),
    ('Checking Accounts', 'Foreign Currency Account'),
    ('Age', 'Superannuation Savings'),
    ('Estimated Income', 'Checking Accounts'),
    ('Bank Loans', 'Credit Card Balance'),
    ('Business Lending', 'Bank Loans'),
]
```

| Pair | Reason for Checking |
|------|---------------------|
| Bank Deposits vs Saving Accounts | See how much savings balances contribute to deposits |
| Checking vs Saving Accounts | Compare short-term liquidity with savings balances |
| Checking vs Foreign Currency | Look for clients with broader banking activity |
| Age vs Superannuation | Check whether retirement savings generally rise with age |
| Income vs Checking Accounts | Compare income with transactional balances |
| Bank Loans vs CC Balance | Look at combined debt exposure |
| Business Lending vs Bank Loans | Identify clients carrying both business and personal lending |

---

## 7. Component 3 — Power BI Dashboard

**Tool:** Power BI Desktop  
**Pages:** 9  
**Main areas:** Risk · Growth · Profitability

---

### Phase 1 — Data Loading

The CSV file was loaded into Power BI and checked before modelling.

The main checks included:

- data types
- ID columns
- dates
- numerical fields
- categorical fields

---

### Phase 2 — Power Query

Several useful columns were created before loading the model.

| Column | Logic |
|--------|-------|
| Age Band | 18–30 / 31–45 / 46–60 / 61+ |
| Income Band | Low / Mid / High |
| Processing Fees | High = 0.05 / Mid = 0.03 / Low = 0.01 |
| Engagement Days | Days since joining |
| Engagement Timeframe | <1yr / 1–5yr / 5–10yr / 10+yr |
| Year | Extracted from Joined Bank |
| Gender | Mapped from GenderId |
| Banking Relationship | Mapped from BRId |

**Columns removed:** Location ID · Banking Contact · Risk Weighting

---

### Phase 3 — Data Model

A Date Table covering 1995–2021 was created in DAX and marked as the official date table.

The main relationship was:

```text
Date Table[Date] → Banking[Joined Bank]
```

Relationship type:

```text
One-to-Many
Single Filter Direction
```

---

### Phase 4 — DAX Measures

The report uses DAX measures covering portfolio size, deposits, lending, growth, fees and risk.

### Base Measures

- Total Clients
- Bank Loan Amount
- Business Lending Amount
- Credit Cards Balance
- Total Bank Deposit
- Total Checking Accounts
- Total Saving Account
- Foreign Currency Amount
- Engagement Length
- Total CC Amount

### Composite Measures

- Total Loan
- Total Deposit
- Total Fees

### Ratios

- Avg Loan Per Client
- Avg Deposit Per Client
- Loan to Deposit Ratio
- Avg Fee Per Client
- Revenue per Loan
- Loan Concentration %
- Fee Concentration %

### Time Intelligence

- YoY Loan Growth
- YoY Deposit Growth
- YoY Client Growth
- Cumulative Clients
- Growth Rate %

### Risk Measures

- High Risk Loan Clients
- Overleveraged Clients
- Credit Risk Ratio
- High Loan Low Income Count
- High Income Clients
- Avg Loan by Income Band

---

### Phase 5 — Dashboard Pages

| Page | Purpose | Key Visuals |
|------|---------|-------------|
| 1 — Home | Portfolio overview | KPI cards · Client acquisition · Banking Relationship · Loyalty · Nationality |
| 2 — Loan Analysis | Loan portfolio breakdown | Loan by relationship · nationality · income · occupation · trend |
| 3 — Deposit Analysis | Deposit portfolio | Deposit by relationship · gender · fee structure · nationality · occupation |
| 4 — Deposit Analysis 2 | Extended deposit analysis | Loyalty · loan vs deposit · age · engagement |
| 5 — Risk Analysis | Risk indicators | Loan concentration · LDR · CC balances · High Loan Low Income · engagement |
| 6 — Growth Analysis | Portfolio growth | Cumulative clients · YoY growth · nationality · age · income |
| 7 — Growth Analysis 2 | Demographic growth | Income band · gender · loyalty |
| 8 — Revenue & Profitability | Fee analysis | Fees by relationship · loyalty · income · nationality · trend |
| 9 — Summary | Executive summary | KPI cards · loyalty · portfolio mix · trend |

---

## Dashboard Gallery

### 1. Executive Banking Portfolio Overview

![Executive Banking Portfolio Overview](screenshot/01.home.png)

### 2. Loan Portfolio Analysis

![Loan Portfolio Analysis](screenshot/02.loan_analysis.png)

### 3. Deposit Portfolio Analysis

![Deposit Portfolio Analysis](screenshot/03.deposit_analysis.png)

### 4. Extended Deposit Analysis

![Extended Deposit Analysis](screenshot/04.deposit_analysis_2.png)

### 5. Credit Risk Analysis

![Credit Risk Analysis](<screenshot/05.Risk Analysis.png>)

### 6. Portfolio Growth Analysis

![Portfolio Growth Analysis](<screenshot/06.Growth Analysis.png>)

### 7. Demographic Growth Analysis

![Demographic Growth Analysis](<screenshot/07.Growth Analysis 2.png>)

### 8. Revenue & Profitability Analysis

![Revenue & Profitability Analysis](<screenshot/08.Revenue and profitability.png>)

### 9. Executive Summary Dashboard

![Executive Summary Dashboard](screenshot/09.Summary.png)

---

## Power BI Data Model

![Power BI Data Model](screenshot/10_powerbi_data_model.png.png)

---

## 8. Key Findings

### Portfolio Overview

| Metric | Value |
|--------|-------|
| Total Clients | 3,000 |
| Total Loan Exposure | $4.38 billion |
| Total Deposits | $3.77 billion |
| Business Lending | $2.60 billion |
| Total Fee Revenue | $158.19 million |
| Avg Fee Per Client | $52,730 |
| Loan to Deposit Ratio | 1.16 |
| High Risk Loan Clients | 1,197 |
| Overleveraged Clients | 1,496 |

---

### Risk

The portfolio has a **Loan-to-Deposit Ratio of 1.16**, meaning total lending is higher than total deposits in the dataset.

Around **1,496 clients** have loan balances above their deposit balances under the rule used in this project.

The **High Loan / Low Income** segment is another group worth monitoring because these clients combine larger borrowing with lower estimated income.

These are portfolio screening indicators rather than predictions of default.

---

### Growth

Client acquisition increased strongly toward the later years of the dataset.

The largest year for new clients was **2020**, with 248 additions.

Personal Banking was the largest relationship group, with **1,352 clients**.

The gender split was close to even in both the Python analysis and the Power BI dashboard.

---

### Profitability

Fee revenue was strongly influenced by the client's assigned fee structure.

Clients in the High fee tier generated more fee revenue by design because the percentage charged to them was higher.

Across the dataset, total calculated fees were approximately **$158.19M**.

The dashboard allows fee performance to be compared by:

- banking relationship
- loyalty level
- income band
- fee structure
- nationality

---

### Python EDA Findings

- Bank Deposits and Checking Accounts showed a strong positive relationship
- Business Lending and Bank Loans showed a moderate positive relationship
- Age and Superannuation Savings were positively related, which is consistent with older clients having larger retirement balances
- Credit Card Balance had relatively weak relationships with most other financial variables

These patterns describe relationships in the dataset and should not be interpreted as proof of cause and effect.

---

## Main Takeaway

The project shows that banking performance cannot be understood from a single metric.

A client may:

- hold large deposits but also large loans
- generate strong fee revenue but carry significant exposure
- belong to a fast-growing customer group without necessarily being low risk

Looking at **risk, growth and profitability together** gives a much more useful picture of the portfolio.

The Power BI report was built to let users move between these different views rather than relying only on overall totals.

---

## 9. Repository Structure

```text
Banking-Risk-Analytics/
│
├── Banking.csv
├── Banking_EDA_Case_Project.ipynb
├── Banking_analysis_sql.sql
├── Banking_Report.docx
├── Banking_Solution_Dashboard_Project.docx
├── Banking.pptx
└── README.md
```

---

## 10. How to Run

### SQL

Run the following in MySQL Workbench:

```sql
CREATE DATABASE banking_case;

USE banking_case;
```

Import `Banking.csv` into the database and use the included SQL file for the setup and exploration steps.

---

### Python EDA

For Google Colab:

```text
Upload Banking01.xlsx
Open Banking_EDA_Case_Project.ipynb
Run the notebook from top to bottom
```

For local Jupyter:

```bash
pip install pandas matplotlib seaborn numpy openpyxl

jupyter notebook Banking_EDA_Case_Project.ipynb
```

---

### Power BI

1. Open Power BI Desktop.
2. Select **Get Data → Text/CSV**.
3. Load `Banking.csv`.
4. Open **Transform Data**.
5. Apply the Power Query transformations.
6. Create the Date Table and relationship.
7. Add the DAX measures.
8. Build or review the nine dashboard pages.

> **Note:** Year-over-year measures require year context from the Date Table. Without a selected year, there is no prior-year period to compare against.

---

## Author

**Shah Tahsin**  
Business Data Analyst | SQL · Python · Power BI · Financial & Risk Analytics

[GitHub](https://github.com/shababtahsin)
