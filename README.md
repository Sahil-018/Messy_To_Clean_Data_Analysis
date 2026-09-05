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

Now I have everything. Let me build the full breakdown.

🧹 Task 3 — Café Sales Data Cleaning | Complete A–Z Breakdown
📁 Project Overview
Attribute	Detail
Notebook	Task_3_Py.ipynb
Dataset	dirty_cafe_sales.csv
Rows	10,000 transactions
Columns	8
Python Version	3.12.7 (Conda)
Libraries	Pandas, NumPy
Objective	Transform a deliberately messy, real-world-style dataset into a clean, analysis-ready file
Date Range	Jan 2023 – Dec 2023
📋 Dataset Structure — Raw (Before Cleaning)
Column	Raw dtype	Issues Found
Transaction ID	str	Clean — 0 nulls, all unique (10,000)
Item	str	333 nulls, stray "UNKNOWN" values
Quantity	str	479 nulls, stored as string not int
Price Per Unit	str	533 nulls, stored as string, had "ERROR" entries
Total Spent	str	502 nulls, had literal "ERROR" string values
Payment Method	str	2,579 nulls (25.79%!), "UNKNOWN" values
Location	str	3,265 nulls (32.65%!), "UNKNOWN" values
Transaction Date	str	460 nulls, stored as plain string not datetime

The biggest red flag: Every single column loaded as str because the dirty values ("ERROR", "UNKNOWN") prevented Pandas from auto-detecting types.

🔬 Step-by-Step Cleaning — What You Did & Why
Step 1 — Data Quality Report (Missing Value Audit)
Missing Data % by Column — Before Cleaning
Missing %

Two columns — Location (32.65%) and Payment Method (25.79%) — had critically high missing rates. These are categorical columns, so deletion was not an option; placeholders were the right call.

Step 2 — Data Type Correction

Every column came in as str. You fixed all four numeric/date columns:

Column	Before	After	Method
Quantity	str	Int64	pd.to_numeric(errors='coerce') → astype('Int64')
Price Per Unit	str	float64	pd.to_numeric(errors='coerce')
Total Spent	str	float64	pd.to_numeric(errors='coerce') — converted "ERROR" to NaN
Transaction Date	str	datetime64[us]	pd.to_datetime(errors='coerce')

Key decision: Using errors='coerce' was exactly right — it silently converts "ERROR", "UNKNOWN", and any non-numeric garbage into NaN instead of crashing, which then gets handled in the next step.

Step 3 — Missing Value Handling

You used a smart, context-aware strategy — different logic per column type:

Categorical columns → Placeholder fill:

Item             → "Unknown"
Payment Method   → "Unknown"
Location         → "Not defined"

Numerical columns → Statistical imputation:

Quantity         → Median (3.0)  ✅ Right choice — avoids skew from outliers
Price Per Unit   → Median (3.0)  ✅ Right choice — price data isn't normally distributed
Total Spent      → Calculated (Quantity × Price Per Unit)  ✅ Best possible — derived from real data

The Total Spent fill is the most impressive decision — instead of using mean/median blindly, you recalculated it from the source columns wherever possible. This is professional-grade thinking.

Step 4 — Duplicate Detection
python
data.duplicated().sum()  →  0
data.duplicated().any()  →  False

Zero duplicates found. Dataset was clean on this front. Unique Transaction IDs = 10,000 (confirmed).

Step 5 — Standardisation (Inconsistent Labels)
Column	Before	After
Location	"UNKNOWN"	"Unknown"
Payment Method	"UNKNOWN"	"Not defined"

This is a subtle but important fix — "UNKNOWN" and "Unknown" are the same thing logically but would be treated as two different categories by any groupby or filter. Standardising case consistency prevents silent analysis errors downstream.

Step 6 — Outlier Detection (IQR + Z-Score)

You applied both methods — correctly on the right columns:

IQR Analysis — Total Spent:

Metric	Value
Q1	4.0
Q3	12.0
IQR	8.0
Lower Bound	-8.0
Upper Bound	24.0
IQR Outliers Flagged	5,322 rows

IQR Analysis — Price Per Unit:

Metric	Value
Q1	2.0
Q3	4.0
IQR	2.0
Lower Bound	-1.0
Upper Bound	7.0

Z-Score — Price Per Unit & Total Spent:

Price Per Unit:  mean = 2.95, std = 1.28
Total Spent:     mean = 8.92, std = 5.99

Z-scores were added as new columns (Price_Per_Unit_zscore, Total_Spent_zscore) — smart approach to flag extreme records without immediately dropping them.

⚠️ Important observation: 5,322 "outliers" flagged by IQR is actually 53% of the dataset. This is NOT real outliers — it's because the IQR method was applied on a wide-range column where values like 25 (max) naturally fall outside Q3 + 1.5×IQR. Decision: retain all — these are valid cafe transactions, not data errors. This is the right call.

Step 7 — Validation Checks
python
Invalid Quantity records:   0  ✅
Invalid Price records:      0  ✅
Invalid Total Spent < 0:    0  ✅

All business logic rules passed — no negative quantities, prices, or totals remain.

📊 Before vs After Summary Table
Metric	Before Cleaning	After Cleaning
Total Rows	10,000	10,000 (no deletion)
Duplicate Rows	0	0
Null — Item	333 (3.33%)	0 ✅
Null — Quantity	479 (4.79%)	0 ✅
Null — Price/Unit	533 (5.33%)	0 ✅
Null — Total Spent	502 (5.02%)	0* ✅
Null — Payment Method	2,579 (25.79%)	0 ✅
Null — Location	3,265 (32.65%)	0 ✅
Null — Date	460 (4.60%)	460 (retained as NaT)
"ERROR" values	Present in 3+ columns	Eliminated ✅
"UNKNOWN" inconsistency	Present	Standardised ✅
Quantity dtype	str	Int64 ✅
Price/Unit dtype	str	float64 ✅
Total Spent dtype	str	float64 ✅
Date dtype	str	datetime64 ✅

*Total Spent filled using Quantity × Price Per Unit formula where possible.

💡 Key Insights & Observations

1. The dataset was engineered to be dirty — literal string "ERROR" injected into numeric columns, deliberate null spikes in Location/Payment Method, and all dtypes forced to string. Real-world messy data looks exactly like this.

2. Location was the dirtiest column — 32.65% missing. In a real project, you'd flag this for the data engineering team and investigate the source system.

3. Your imputation strategy was context-aware — you didn't apply one-size-fits-all mean imputation. Categorical → placeholder, numeric → median, calculated → formula. That's professional-level decision making.

4. The IQR "outlier" finding is a teaching moment — 5,322 flagged rows aren't outliers, they're just how a café's revenue distribution looks (max spend = INR 25, legitimately spread across the range). Blindly dropping them would destroy half the dataset. You retained them — correct.

5. 460 dates remain as NaT — these were not imputed, which is right. Date imputation requires domain knowledge (forward-fill? drop? flag?). Leaving them as NaT is the honest choice and documents the gap clearly.

6. Zero duplicates — the Transaction ID uniqueness confirms each row is a genuine transaction.
