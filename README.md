# Banking Risk Analytics — Integrated Project
### SQL · Python EDA · Power BI Dashboard
#### Big 4 Analyst Standard | Finance Capstone

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Solution Architecture](#3-solution-architecture)
4. [Dataset](#4-dataset)
5. [Component 1 — SQL](#5-component-1--sql)
6. [Component 2 — Python EDA](#6-component-2--python-eda)
7. [Component 3 — Power BI Dashboard](#7-component-3--power-bi-dashboard)
8. [Key Findings](#8-key-findings)
9. [Repository Structure](#9-repository-structure)
10. [How to Run](#10-how-to-run)
11. [Future Work](#11-future-work)

---

## 1. Project Overview

This is a full end-to-end integrated banking analytics project spanning three tools and three analytical layers. The project follows Big 4 consulting standards — raw data is first explored in SQL, then subjected to deep statistical analysis in Python, and finally delivered as a 9-page interactive Power BI dashboard covering Risk, Growth, and Profitability.

The three components are not independent — they are sequential stages of the same analytical pipeline, each feeding into the next.

```
SQL (Setup & Exploration) → Python EDA (Statistical Analysis) → Power BI (Interactive Dashboard)
```

---

## 2. Problem Statement

Banks face a fundamental tension between growth and risk. The more aggressively a bank lends, the higher its revenue potential — but also its exposure to default. The core business question this project addresses is:

> *How do you use client data to identify who is likely to repay a loan, which segments carry the most risk, where growth is coming from, and which clients are most profitable — all at the same time?*

Traditional approaches rely on manual assessment, which is slow, inconsistent, and unable to process portfolios of thousands of clients. This project replaces that approach with a fully data-driven analytics pipeline.

---

## 3. Solution Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Raw Data Source                   │
│            Banking.csv / Banking01.xlsx             │
│        3,000 clients · 25 columns · 1999–2021      │
└──────────────────────┬──────────────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
         ▼             ▼             ▼
  ┌────────────┐ ┌──────────┐ ┌────────────────┐
  │    SQL     │ │  Python  │ │   Power BI     │
  │            │ │   EDA    │ │   Dashboard    │
  │ - Database │ │          │ │                │
  │   setup    │ │ - Profil │ │ - Power Query  │
  │ - Table    │ │ - Income │ │ - DAX Measures │
  │   explore  │ │   Band   │ │ - 9 Pages      │
  │ - Initial  │ │ - Bivar  │ │ - Risk         │
  │   queries  │ │   analysis│ │ - Growth      │
  │            │ │ - Correl │ │ - Profitability│
  │            │ │   matrix │ │                │
  └────────────┘ └──────────┘ └────────────────┘
         │             │             │
         └─────────────┴─────────────┘
                       │
                       ▼
          ┌────────────────────────┐
          │   Business Insights    │
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
| Date Range | 1999 – 2021 (joining dates) |
| Nulls | 0 (100% complete) |
| Age Range | 17 – 85 years |
| Unique Occupations | 195 |

### Column Reference

| Column | Type | Description |
|--------|------|-------------|
| Client ID | Text | Unique client identifier |
| Name | Text | Client full name |
| Age | Integer | Client age |
| Joined Bank | Date | Date client joined |
| Nationality | Text | American / African / Asian / European / Australian |
| Occupation | Text | 195 unique occupations |
| Fee Structure | Text | High / Mid / Low |
| Loyalty Classification | Text | Jade / Gold / Silver / Platinum |
| Estimated Income | Decimal | Annual estimated income |
| Superannuation Savings | Decimal | Retirement savings balance |
| Amount of Credit Cards | Integer | Number of credit cards held |
| Credit Card Balance | Decimal | Total CC outstanding balance |
| Bank Loans | Decimal | Outstanding bank loan balance |
| Bank Deposits | Decimal | Total bank deposit balance |
| Checking Accounts | Decimal | Checking account balance |
| Saving Accounts | Decimal | Savings account balance |
| Foreign Currency Account | Decimal | FX account balance |
| Business Lending | Decimal | Commercial/business loan balance |
| Properties Owned | Integer | Number of properties owned |
| Risk Weighting | Integer | Internal risk score |
| BRId | Integer | Banking Relationship ID (1–4) |
| GenderId | Integer | Gender ID (1=Male, 2=Female) |
| IAId | Integer | Investment Advisor ID (1–22) |

### ID Mappings

| Column | Values |
|--------|--------|
| BRId | 1 = Premium · 2 = Business · 3 = Personal · 4 = SME |
| GenderId | 1 = Male · 2 = Female |
| Fee Structure | High = 0.05 fee · Mid = 0.03 fee · Low = 0.01 fee |
| Loyalty | Jade · Gold · Silver · Platinum |

---

## 5. Component 1 — SQL

**File:** `Banking_analysis_sql.sql`  
**Tool:** MySQL  
**Purpose:** Database environment setup and initial data exploration

### What Was Done

The SQL component establishes the analytical database environment and performs initial data connectivity checks before the data is passed to Python and Power BI for deeper analysis.

```sql
-- Database creation
CREATE DATABASE banking_case;

-- Set active database
USE banking_case;

-- Verify tables loaded
SHOW TABLES;

-- Initial full table scan
SELECT * FROM customer;

-- Environment verification
SHOW VARIABLES WHERE Variable_name = 'hostname';
SELECT current_user();
```

### What This Establishes
- Isolated database environment (`banking_case`) for the project
- Confirms data has loaded correctly into the target table
- Verifies the connection context before running analytical queries
- Environment metadata capture (hostname, user) for reproducibility documentation

> **Note:** The SQL component covers database setup and initial exploration. Deep analytical querying is handled in the Python EDA component and Power BI DAX layer, which are better suited to iterative visual analysis on this dataset size.

---

## 6. Component 2 — Python EDA

**File:** `Banking_EDA_Case_Project.ipynb`  
**Tool:** Python (pandas, matplotlib, seaborn, numpy) — Google Colab  
**Purpose:** Statistical exploration and pattern identification before dashboard development

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
df.head(5)      # Preview
df.shape        # (3000, 25)
df.info()       # Data types and null check
df.describe()   # Descriptive statistics
```

### Step 1 — Feature Engineering: Income Band

Income binned into three segments — definition consistent with Power BI dashboard:

```python
bins = [0, 100000, 300000, float('inf')]
labels = ['Low', 'Mid', 'High']
df['Income Band'] = pd.cut(df['Estimated Income'], bins=bins, labels=labels, right=False)
```

| Band | Income Range | Clients |
|------|-------------|---------|
| Low | < $100,000 | 287 |
| Mid | $100,000 – $300,000 | 1,835 |
| High | > $300,000 | 878 |

### Step 2 — Univariate Analysis: Categorical Columns

Value counts for all categorical variables including BRId, GenderId, Nationality, Fee Structure, Loyalty Classification, Income Band.

**Key findings:**
- BRId 3 (Personal) is the dominant banking relationship segment
- Gender split is near-equal across the portfolio
- Fee Structure High is the most common tier
- Loyalty Classification Jade holds the majority of clients

### Step 3 — Bivariate Analysis: Gender Hue

Countplots for all categorical columns segmented by Gender using `hue='GenderId'`.

**Purpose:** Identify whether gender influences distribution across BR type, loyalty tier, fee structure, and income band — foundational for the Gender slicer in Power BI.

### Step 4 — Bivariate Analysis: Nationality Hue

Countplots segmented by Nationality using `hue='Nationality'`.

**Purpose:** Identify whether nationality drives differences in fee structure, loyalty, and BR type — informs Loan and Deposit by Nationality visuals.

### Step 5 — Univariate Analysis: Numerical Distributions

Histplots with KDE for all numerical columns in a 4×3 subplot grid.

**Key observations:**
- Bank Loans and Business Lending are right-skewed — most clients cluster at lower balances with a long tail of high-value clients
- Estimated Income shows a broad distribution centred around $150K–$200K
- Credit Card Balance is heavily concentrated at low values
- Checking and Saving Accounts show similar distributions — they move together

### Step 6 — Correlation Matrix

```python
numerical_cols = ['Age', 'Estimated Income', 'Superannuation Savings',
                  'Credit Card Balance', 'Bank Loans', 'Bank Deposits',
                  'Checking Accounts', 'Saving Accounts',
                  'Foreign Currency Account', 'Business Lending',
                  'Properties Owned']

correlation_matrix = df[numerical_cols].corr()
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', fmt=".2f")
```

**Key correlations:**
- Bank Deposits ↔ Checking Accounts — strong positive correlation
- Business Lending ↔ Bank Loans — moderate positive (dual exposure clients)
- Age ↔ Superannuation — positive (lifecycle savings confirmed)
- Credit Card Balance ↔ all others — weak (CC debt is independent of wealth profile)

### Step 7 — Regression Analysis: Key Variable Pairs

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

| Pair | Business Rationale |
|------|-------------------|
| Bank Deposits vs Saving Accounts | Tests whether total deposits are driven by savings balances |
| Checking vs Saving Accounts | Tests whether clients balance liquidity vs savings |
| Checking vs Foreign Currency | Identifies internationally active high-value clients |
| Age vs Superannuation | Validates lifecycle savings hypothesis |
| Income vs Checking Accounts | Tests whether income drives transactional activity |
| Bank Loans vs CC Balance | Combined debt burden — dual borrower risk indicator |
| Business Lending vs Bank Loans | Commercial + personal dual exposure = highest risk segment |

---

## 7. Component 3 — Power BI Dashboard

**Tool:** Power BI Desktop (Free)  
**Pages:** 9  
**Analytical pillars:** Risk · Growth · Profitability

---

### Phase 1 — Data Loading & Audit
Loaded via **Get Data → Text/CSV**. Data types, ID columns, and date formats inspected before loading.

---

### Phase 2 — Power Query Transformations

**Derived columns created:**

| Column | Logic |
|--------|-------|
| Age Band | 18-30 / 31-45 / 46-60 / 61+ |
| Income Band | Low (<100K) / Mid (<300K) / High |
| Processing Fees | High=0.05 / Mid=0.03 / Low=0.01 |
| Engagement Days | Days from Joined Bank to today |
| Engagement Timeframe | <1yr / 1-5yr / 5-10yr / 10+yr |
| Year | Extracted from Joined Bank |
| Gender | Mapped from GenderId |
| Banking Relationship | Mapped from BRId |

**Columns removed:** Location ID · Banking Contact · Risk Weighting

---

### Phase 3 — Data Modelling

**Date Table** built in DAX (1995–2021), marked as official date table.  
**Relationship:** Date Table[Date] → Banking[Joined Bank] | One to Many | Single filter direction

---

### Phase 4 — DAX Measures (Full Library)

**Base:** Total Clients · Bank Loan Amount · Business Lending Amount · Credit Cards Balance · Total Bank Deposit · Total Checking Accounts · Total Saving Account · Foreign Currency Amount · Engagement Length · Total CC Amount

**Composite:** Total Loan · Total Deposit · Total Fees (SUMX)

**Ratios:** Avg Loan Per Client · Avg Deposit Per Client · Loan to Deposit Ratio · Avg Fee Per Client · Revenue per Loan · Loan Concentration % · Fee Concentration %

**Time Intelligence:** YoY Loan Growth · YoY Deposit Growth · YoY Client Growth · Cumulative Clients · Growth Rate %

**Risk:** High Risk Loan Clients · Overleveraged Clients · Credit Risk Ratio · High Loan Low Income Count · High Income Clients · Avg Loan by Income Band

---

### Phase 5 — Dashboard Pages (9 Pages)

| Page | Purpose | Key Visuals |
|------|---------|-------------|
| 1 — Home | Executive overview | KPI cards · Client acquisition line · BR donut · Loyalty donut · Nationality bar |
| 2 — Loan Analysis | Loan portfolio breakdown | Loan by BR · by Nationality · by Income Band · by Occupation · Loan trend |
| 3 — Deposit Analysis | Deposit portfolio breakdown | Deposit by BR · Gender · Fee×BR stacked · Nationality · Occupation · Income Band |
| 4 — Deposit Analysis 2 | Extended deposit analysis | Loyalty×BR stacked · Deposit vs Loan trend · Age×Gender · Engagement Timeframe |
| 5 — Risk Analysis | Risk identification | Loan concentration · LDR by BR · CC Treemap · High Loan Low Income · Risk by engagement |
| 6 — Growth Analysis | Portfolio growth trends | Cumulative area · YoY dual axis · Nationality stacked · Age×Income band |
| 7 — Growth Analysis 2 | Loyalty & demographic growth | Income Band over time · Engagement×Gender · Loyalty stacked area |
| 8 — Revenue & Profitability | Fee revenue and margins | Fee by BR×Loyalty · Fee by Income×Fee Structure · Fee trend · Avg fee by Loyalty · Fee by Nationality |
| 9 — Summary | Full executive snapshot | 12 KPI cards · Loyalty clustered column · Portfolio donut · Full trend line |

---

### Phase 6 — Formatting & Navigation
> ⚠️ **Status: Pending**  
> Blue theme · Navigation buttons · Card formatting · Chart titles · Top N filters

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
| High Risk Loan Clients | 1,197 (39.9%) |
| Overleveraged Clients | 1,496 (49.9%) |

### Risk
- Loan to Deposit Ratio of **1.16** — bank is lending more than it holds in deposits
- Nearly **50% of clients are overleveraged** — loan balances exceed deposit balances
- High Loan Low Income segment is the most vulnerable cohort for default

### Growth
- Client acquisition accelerated from **2019 onwards** — peak year 2020 with 248 new clients
- Personal banking clients form the largest segment at **1,352 clients**
- Near-equal gender split confirmed by both Python EDA and Power BI

### Profitability
- High fee structure clients generate disproportionately more revenue per loan
- Fee structure is the **primary profitability lever** available to the bank
- Total fees of $158M represent a **3.6% yield** on total loan exposure

### Python EDA Correlations
- Bank Deposits and Checking Accounts are strongly correlated — deposit products move together
- Business Lending and Bank Loans show moderate correlation — commercial borrowers carry dual exposure
- Age and Superannuation are positively correlated — lifecycle savings behaviour confirmed
- Credit Card Balance is weakly correlated with all variables — CC debt is independent of wealth profile

---

## 9. Repository Structure

```
Banking-Risk-Analytics/
│
├── Banking.csv                                  # Raw dataset
├── Banking_EDA_Case_Project.ipynb               # Python EDA notebook
├── Banking_analysis_sql.sql                     # SQL setup and exploration
├── Banking_Report.docx                          # Project report
├── Banking_Solution_Dashboard_Project.docx      # Full solution documentation
├── Banking.pptx                                 # Presentation slides
└── README.md                                    # This file
```

> ⚠️ **Missing:** Power BI `.pbix` dashboard file — add to repository to complete the deliverable set.

---

## 10. How to Run

### SQL
```sql
-- Run in MySQL Workbench
-- Import Banking.csv as table into banking_case database
CREATE DATABASE banking_case;
USE banking_case;
```

### Python EDA
```bash
# Google Colab (original environment)
# Upload Banking01.xlsx and run Banking_EDA_Case_Project.ipynb

# Local Jupyter
pip install pandas matplotlib seaborn numpy openpyxl
jupyter notebook Banking_EDA_Case_Project.ipynb
```

### Power BI
1. Open Power BI Desktop
2. Get Data → Text/CSV → load `Banking.csv`
3. Transform Data → apply all Power Query transformations
4. Close & Apply
5. Build Date Table and relationship
6. Build all DAX measures
7. Build 9 dashboard pages

> **Note:** YoY measures require a year selected via the Date Table slicer. This is expected — without year context there is no prior year to compare against.

---

## 11. Future Work

| Priority | Item |
|----------|------|
| 🔴 High | Add `.pbix` file to repository |
| 🔴 High | Complete EDA insights cell (Cell 23 is empty) |
| 🔴 High | Expand SQL with aggregation and segmentation queries |
| 🔴 High | Complete Power BI formatting phase |
| 🟡 Medium | Row Level Security in Power BI |
| 🟡 Medium | Publish to Power BI Service with scheduled refresh |
| 🟡 Medium | Python default prediction model (logistic regression on loan repayment) |
| 🟢 Low | Add interest rate data for Net Interest Margin analysis |
| 🟢 Low | Industry benchmark comparisons for Loan to Deposit Ratio |

---

*Banking Risk Analytics — Integrated Project | SQL · Python · Power BI |  
