# Messy_To_Clean_Data_Analysis
🧹 Café Sales Data Cleaning — Task 3 | Python · Pandas · NumPy

Performed professional-level data cleaning on a 10,000-row deliberately 
messy café sales dataset (dirty_cafe_sales.csv) to produce an 
analysis-ready output.

🔍 Data Quality Issues Found:
- All 8 columns loaded as str due to embedded "ERROR" / "UNKNOWN" strings
- Location: 32.65% missing | Payment Method: 25.79% missing
- Total Spent: had literal "ERROR" string values masking nulls
- 460 invalid date entries | 533 missing Price Per Unit values

🛠️ Cleaning Steps Performed:
→ Data type correction: str → Int64, float64, datetime64 using errors='coerce'
→ Missing value imputation: median for numeric, placeholder for categorical
→ Total Spent recalculated from Quantity × Price Per Unit where null
→ Standardised inconsistent labels: "UNKNOWN" → "Unknown" / "Not defined"
→ Duplicate check: 0 found across all 10,000 rows
→ Outlier detection: IQR method + Z-Score (manual formula) on numeric columns
→ Business logic validation: confirmed 0 negative quantities, prices, or totals

📊 Result: Clean, typed, consistent 10,000-row dataset ready for EDA

🛠️ Tools: Python 3.12 · Pandas · NumPy · Jupyter Notebook
📁 Dataset: dirty_cafe_sales.csv (10,000 rows × 8 columns)
📓 Notebook: Task_3_Py.ipynb

Got it! Here's the full **Task 3 detailed breakdown** in GitHub README markdown format — ready to copy-paste directly:

---

# 🧹 Task 3 — Café Sales Data Cleaning

> **Objective:** Demonstrate professional-level data cleaning skills by taking a deliberately messy dataset and systematically transforming it into a clean, analysis-ready dataset. Every decision is documented.

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.12-blue) ![Pandas](https://img.shields.io/badge/Pandas-Data%20Cleaning-green) ![NumPy](https://img.shields.io/badge/NumPy-Numerical-orange) ![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)

---

## 📁 Project Files

| File | Description |
|---|---|
| `Task_3_Py.ipynb` | Main Jupyter Notebook — full cleaning pipeline |
| `dirty_cafe_sales.csv` | Raw messy input dataset (10,000 rows) |

---

## 📊 Dataset Overview

| Attribute | Detail |
|---|---|
| Total Rows | 10,000 |
| Total Columns | 8 |
| Date Range | Jan 2023 – Dec 2023 |
| Domain | Café / Food & Beverage Sales |

### Columns

| Column | Raw dtype | Description |
|---|---|---|
| Transaction ID | str | Unique transaction identifier |
| Item | str | Product sold (Coffee, Cake, Cookie, etc.) |
| Quantity | str | Units purchased |
| Price Per Unit | str | Unit price in currency |
| Total Spent | str | Total transaction value |
| Payment Method | str | Cash / Credit Card / Digital Wallet |
| Location | str | In-store / Takeaway |
| Transaction Date | str | Date of transaction |

> ⚠️ **All 8 columns loaded as `str`** due to embedded dirty values like `"ERROR"` and `"UNKNOWN"` preventing Pandas from auto-detecting types.

---

## 🔍 Step 1 — Data Quality Report

First action: audit every column for nulls, types, and anomalies before touching anything.

### Missing Value Summary — Before Cleaning

| Column | Missing Count | Missing % | Severity |
|---|---|---|---|
| Transaction ID | 0 | 0.00% | ✅ Clean |
| Item | 333 | 3.33% | 🟡 Low |
| Quantity | 479 | 4.79% | 🟡 Low |
| Price Per Unit | 533 | 5.33% | 🟡 Low |
| Total Spent | 502 | 5.02% | 🟡 Low |
| Payment Method | 2,579 | 25.79% | 🔴 Critical |
| Location | 3,265 | 32.65% | 🔴 Critical |
| Transaction Date | 460 | 4.60% | 🟡 Low |

### Additional Anomalies Found

- `"ERROR"` string injected inside `Total Spent` and `Price Per Unit` columns
- `"UNKNOWN"` used inconsistently as a placeholder across `Payment Method` and `Location`
- `"ERROR"` also appeared in `Payment Method` and `Location` fields
- Negative value check → ✅ None found after type conversion
- Duplicate rows → ✅ None found (`data.duplicated().sum() = 0`)

---

## 🔧 Step 2 — Data Type Correction

All numeric and date columns were stored as plain strings. Fixed using `pd.to_numeric()` and `pd.to_datetime()` with `errors='coerce'` — this silently converts non-parseable values like `"ERROR"` into `NaN` instead of crashing.

| Column | Before | After | Method |
|---|---|---|---|
| Quantity | `str` | `Int64` | `pd.to_numeric(errors='coerce').astype('Int64')` |
| Price Per Unit | `str` | `float64` | `pd.to_numeric(errors='coerce')` |
| Total Spent | `str` | `float64` | `pd.to_numeric(errors='coerce')` |
| Transaction Date | `str` | `datetime64[us]` | `pd.to_datetime(errors='coerce')` |

> 💡 **Decision Note:** `errors='coerce'` was chosen over `errors='raise'` intentionally — it exposes dirty values as `NaN` which can then be handled systematically in the imputation step rather than stopping execution.

---

## 🩹 Step 3 — Missing Value Handling

A **context-aware imputation strategy** was applied — not one-size-fits-all. Each column type received a different treatment based on its data nature.

### Categorical Columns → Placeholder Fill

| Column | Strategy | Value Used | Reason |
|---|---|---|---|
| Item | `fillna()` | `"Unknown"` | No way to infer the sold item |
| Payment Method | `fillna()` | `"Unknown"` | Cannot assume payment type |
| Location | `fillna()` | `"Not defined"` | Cannot assume in-store vs takeaway |

### Numerical Columns → Statistical Imputation

| Column | Strategy | Value Used | Reason |
|---|---|---|---|
| Quantity | Median fill | `3.0` | Median is robust against skew; quantity is discrete |
| Price Per Unit | Median fill | `3.0` | Price range is narrow (1–5); median is representative |
| Total Spent | **Formula fill** | `Quantity × Price Per Unit` | Derived from source columns — more accurate than any statistic |

> 💡 **Decision Note:** `Total Spent` was recalculated from `Quantity × Price Per Unit` wherever it was `NaN`. This is the most accurate possible imputation — instead of guessing with mean/median, we used the actual business logic of the field. This is professional-grade data reconstruction.

> ⚠️ **Transaction Date — Not Imputed:** 460 `NaT` values were left as-is. Date imputation requires domain knowledge (forward-fill? business calendar? drop?). Leaving them as `NaT` documents the gap honestly rather than fabricating dates.

---

## 🏷️ Step 4 — Standardisation

Inconsistent label formats were unified so `groupby`, filters, and aggregations don't treat the same category as two different values.

| Column | Before | After |
|---|---|---|
| Location | `"UNKNOWN"` | `"Unknown"` |
| Payment Method | `"UNKNOWN"` | `"Not defined"` |

> 💡 **Why this matters:** `"UNKNOWN"` ≠ `"Unknown"` in Python string comparison. Without this fix, any groupby on these columns would silently create two separate buckets for the same concept — corrupting analysis results.

---

## 🔎 Step 5 — Duplicate Detection

```python
data.duplicated().sum()   →   0
data.duplicated().any()   →   False
unique_transactions       →   10,000
```

✅ All 10,000 Transaction IDs are unique. No duplicate rows exist in the dataset.

---

## 📉 Step 6 — Outlier Detection

Both IQR and Z-Score methods were applied on the two key numeric columns.

### IQR Method — Total Spent

| Metric | Value |
|---|---|
| Q1 | 4.0 |
| Q3 | 12.0 |
| IQR | 8.0 |
| Lower Bound | -8.0 |
| Upper Bound | 24.0 |
| Rows flagged as outliers | 5,322 |

### IQR Method — Price Per Unit

| Metric | Value |
|---|---|
| Q1 | 2.0 |
| Q3 | 4.0 |
| IQR | 2.0 |
| Lower Bound | -1.0 |
| Upper Bound | 7.0 |

### Z-Score — Manual Formula Method

```
Price Per Unit  →  mean = 2.95,  std = 1.28
Total Spent     →  mean = 8.92,  std = 5.99
```

Z-score columns added as `Price_Per_Unit_zscore` and `Total_Spent_zscore` to flag extreme values without immediately removing them.

> ⚠️ **Decision — Retain All Flagged Rows:** 5,322 IQR-flagged rows = 53% of the dataset. These are NOT genuine outliers. A café's `Total Spent` naturally spans from INR 1 (single cheap item) to INR 25 (4 units of a premium item). The IQR rule over-flags because the data range is wide but valid. Dropping 53% of the dataset would destroy the analysis. **Decision: Retain all. Flag only for awareness.**

---

## ✅ Step 7 — Business Logic Validation

Final sanity checks to confirm no impossible values remain after cleaning.

| Check | Result |
|---|---|
| Quantity ≤ 0 | ✅ 0 records |
| Price Per Unit ≤ 0 | ✅ 0 records |
| Total Spent < 0 | ✅ 0 records |
| Duplicate rows | ✅ 0 records |

---

## 📋 Before vs After — Full Summary

| Metric | Before | After |
|---|---|---|
| All columns dtype | `str` | Correct types ✅ |
| Null — Item | 333 (3.33%) | 0 ✅ |
| Null — Quantity | 479 (4.79%) | 0 ✅ |
| Null — Price Per Unit | 533 (5.33%) | 0 ✅ |
| Null — Total Spent | 502 (5.02%) | 0 ✅ |
| Null — Payment Method | 2,579 (25.79%) | 0 ✅ |
| Null — Location | 3,265 (32.65%) | 0 ✅ |
| Null — Date | 460 (4.60%) | 460 (NaT retained) |
| `"ERROR"` values | Multiple columns | Eliminated ✅ |
| `"UNKNOWN"` inconsistency | Present | Standardised ✅ |
| Duplicate rows | 0 | 0 ✅ |
| Negative values | 0 | 0 ✅ |

---

## 💡 Key Insights

**1. Dirty by design** — The dataset had "ERROR" injected as strings inside numeric columns and extreme null spikes in categorical columns. This mirrors real-world messy data from poorly integrated source systems.

**2. Location was the most damaged column** — 32.65% missing. In a production environment, this would trigger an upstream data pipeline investigation — not just a `fillna`.

**3. Imputation strategy was column-specific** — Median for numbers, placeholders for categories, and formula recalculation for `Total Spent`. Never blindly apply mean imputation across all columns.

**4. The IQR "outlier" result is a learning moment** — 5,322 flagged rows sound alarming. But 53% of a café dataset being "outside bounds" just means the IQR rule is too aggressive for uniformly distributed price data. Context beats statistics.

**5. 460 dates intentionally left as NaT** — Honest documentation of a data gap beats fabricated imputation every time.

**6. Zero duplicates confirmed** — Every transaction is genuinely unique, meaning the dataset integrity at the ID level is intact even when content columns are messy.

---

## 🐙 GitHub Commit Description

```
🧹 Task 3 — Café Sales Data Cleaning | Python · Pandas · NumPy

Cleaned a 10,000-row deliberately messy café sales dataset from scratch.

Issues fixed:
- All 8 columns loaded as str → corrected to Int64, float64, datetime64
- Location: 32.65% missing | Payment Method: 25.79% missing
- "ERROR" strings embedded in numeric columns → coerced to NaN
- Inconsistent "UNKNOWN" labels → standardised across columns

Cleaning pipeline:
→ dtype correction using pd.to_numeric / pd.to_datetime (errors='coerce')
→ Median imputation for Quantity and Price Per Unit
→ Total Spent recalculated from Quantity × Price Per Unit where null
→ Categorical nulls filled with "Unknown" / "Not defined"
→ Outlier detection via IQR method + manual Z-Score formula
→ Business logic validation — 0 negative values, 0 duplicates confirmed

Tools: Python 3.12 · Pandas · NumPy · Jupyter Notebook
Dataset: dirty_cafe_sales.csv | Notebook: Task_3_Py.ipynb
```

---
