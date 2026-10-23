# AI Predicts What Happens Next: Customer Churn Data Preparation

A Python/pandas notebook that turns raw retail order data into a customer-level dataset (RFM features plus a churn label) ready for machine learning.

> **Status:** data preparation is complete. No predictive model has been trained yet (see [Roadmap](#roadmap)).

## Contents

- [Overview](#overview)
- [Repository structure](#repository-structure)
- [Dataset](#dataset)
- [Getting started](#getting-started)
- [What the notebook does](#what-the-notebook-does)
- [Results](#results)
- [Known data issues](#known-data-issues)
- [Roadmap](#roadmap)

## Overview

The goal is to predict which customers will stop buying so the business can target retention offers. The notebook:

1. Loads and inspects 2,000 order lines (416 customers).
2. Cleans dates and revenue values.
3. Aggregates orders into one row per customer.
4. Builds RFM features: **Recency**, **Frequency**, **Monetary**.
5. Defines churn in two ways: a simple rule, and a leakage-safe time-split.

## Repository structure

```
.
├── README.md
├── data/
│   └── MASTERCLASS 1 - Clean Data.csv
├── notebooks/
│   └── MC3_AI_Predicts_What_Happens_Next.ipynb
└── reports/
    └── Project Report (Word or PDF export)
```

The notebook currently reads the CSV from `/content/MASTERCLASS 1 - Clean Data.csv` (the Google Colab upload folder). If you use the layout above, change `file_path` in the first code cell to `../data/MASTERCLASS 1 - Clean Data.csv`.

## Dataset

One row per order line, 10 columns.

| Column | Description |
|---|---|
| `Order_ID` | Order identifier |
| `Order_Date` | Order date, day/month/year text |
| `Customer_ID` | Customer identifier (416 unique) |
| `Unnamed: 3` | New vs Repeat Customer flag (the CSV header is missing) |
| `Product` | Product name (24 products) |
| `Category` | Electronics, Apparel, Furniture, Home, Beauty |
| `Region` | Seven cities plus `UNKNOWN` |
| `Quantity` | Units ordered (0 to 6) |
| `Revenue` | Order revenue, stored as text with commas (e.g. `32,876`) |
| `Profit` | Order profit |

## Getting started

### Option A: Google Colab

1. Open the notebook in [Colab](https://colab.research.google.com/).
2. Upload `MASTERCLASS 1 - Clean Data.csv` to the session storage.
3. Run all cells.

### Option B: Run locally

Requirements: Python 3.9+, `pandas`, and Jupyter.

```bash
pip install pandas jupyter
jupyter notebook notebooks/MC3_AI_Predicts_What_Happens_Next.ipynb
```

The notebook uses `display()`, which is built into Jupyter and Colab. If you convert it to a plain script, replace `display(...)` with `print(...)`.

## What the notebook does

| Step | Description |
|---|---|
| 1. Load | Read the CSV; show head, columns, shape, dtypes |
| 2. Missing values | Only `Product` has nulls (7) |
| 3. Clean types | Parse `Order_Date` with `dayfirst=True`; convert `Revenue` to numeric |
| 4. Customer table | Group by `Customer_ID`: order count, total revenue, average order value, first and last purchase |
| 5. RFM | Recency = days from reference date to last purchase; Frequency = order count; Monetary = total revenue |
| 6. Churn v1 (rule) | Churned if Recency ≥ 180, Frequency < 5 and Monetary < 100,000 |
| 7. Churn v2 (time-split) | Cutoff 2026-06-01. Features from earlier orders only; churned if no purchase on or after the cutoff |

**Use v2 for modelling.** In v1 the label is computed directly from Recency, Frequency and Monetary, so a model using those columns would simply re-learn the rule (target leakage). In v2 the label describes behaviour after the features were measured.

## Results

| Approach | Customers | Churned | Not churned | Churn rate |
|---|---|---|---|---|
| v1: rule on current RFM | 416 | 99 | 317 | 23.8% |
| v2: time-split (leakage-safe) | 378 | 135 | 243 | 35.7% |

The final v2 table has one row per customer and these columns: `Customer_ID`, `Recency`, `Frequency`, `Monetary`, `Avg_Order_Value`, `Churn_Status`.

The 38 customers whose first order is on or after 1 June are excluded from v2 because they have no history to build features from.

## Known data issues

Despite the file name, the data is not fully clean. Fix these before modelling:

- **23 duplicate rows** (same `Order_ID`). They inflate `Monetary`; `Frequency` is unaffected because it counts unique order IDs.
- **Suspicious dates.** Orders run 1 Jan to 20 Aug 2026, plus five rows dated 1 Oct (four) and 1 Dec (one), likely day/month swaps. The notebook uses the latest date (1 Dec) as the v1 reference date, which adds about 103 days to every customer's recency.
- **Zero-value orders:** 23 with `Quantity = 0`, 27 with `Revenue = 0`, and 27 where `Profit > Revenue`.
- **7 missing `Product` values** and **16 `Region = UNKNOWN`** rows.
- **Outliers.** Order 1882 (customer C223) is about 2.68 million revenue, over 4x the next largest. Customer C000 has 26 orders and about 1.9 million revenue.
- **Unlabelled column** named `Unnamed: 3` by pandas.

## Roadmap

- [ ] Remove duplicates; decide how to treat zero-value orders
- [ ] Verify the five late-dated rows and set the true reference date
- [ ] Add features: category mix, margin, region, days between orders, new/repeat flag
- [ ] Train baseline (logistic regression), then random forest / gradient boosting
- [ ] Evaluate with stratified split, cross-validation, precision, recall, F1 and ROC-AUC
- [ ] Produce a ranked list of at-risk customers

## License

No license specified. Add one before sharing publicly.
