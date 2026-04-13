# 05 — Pandas Fundamentals: Data Wrangling

**Course:** Python for AI & Data  
**Unit:** 2 — Cleaning and Transforming Data with Python Tools  
**Session Time:** 60 minutes

---

## Prerequisites

- Unit 1 complete: Python fundamentals, functions, file I/O
- `pandas` installed in your virtual environment (`pip install -r requirements.txt`)

---

## Table of Contents

1. [Module Overview](#module-overview)
2. [Learning Objectives](#learning-objectives)
3. [Session Breakdown](#session-breakdown)
4. [Introducing Pandas and DataFrames](#introducing-pandas-and-dataframes)
5. [Loading and Inspecting Data](#loading-and-inspecting-data)
6. [Handling Missing Values and Duplicates](#handling-missing-values-and-duplicates)
7. [Transforming and Standardising Data](#transforming-and-standardising-data)
8. [Saving Clean Data](#saving-clean-data)
9. [Wrap-Up](#wrap-up)
10. [Resources](#resources)

---

## Module Overview

In Unit 1 you built the Python foundations: variables, logic, functions, data structures, and file I/O. You loaded BeanScene's menu data from a CSV and processed it using lists of dictionaries.

Unit 2 begins now — and the first thing it teaches is that real-world data is almost never clean.

**PaperLane** is a small stationery and office supplies shop with two sales channels — online and in-store — across four regions. Their sales team has been logging orders manually in spreadsheets for years. As a result, the data has missing values, duplicate rows, inconsistent capitalisation, and date formatting errors.

In this module you'll learn how **Pandas** — Python's core library for tabular data — gives you the tools to turn that messy raw data into a clean, analysis-ready dataset.

---

## Learning Objectives

By the end of this module, you'll be able to:

- Load a CSV file into a Pandas DataFrame using `pd.read_csv()`
- Inspect a DataFrame's structure, types, and data quality
- Identify and handle missing values using `dropna()`, `fillna()`, and `isna()`
- Identify and remove duplicate rows using `drop_duplicates()`
- Standardise text columns using string methods
- Convert data types including dates using `pd.to_datetime()`
- Create derived columns using vectorised operations
- Filter rows using boolean conditions
- Save a cleaned DataFrame to CSV using `to_csv()`

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Introducing Pandas and DataFrames | 5 min |
| 2 | Loading and Inspecting Data | 20 min |
| 3 | Handling Missing Values and Duplicates | 15 min |
| 4 | Transforming and Standardising Data | 15 min |
| 5 | Wrap-Up | 5 min |

---

## Introducing Pandas and DataFrames

In Unit 1 you represented tabular data as a list of dictionaries — each dict was one row, each key was a column. Pandas formalises this into a **DataFrame**: a two-dimensional, labelled table that you can filter, transform, aggregate, and export.

```python
import pandas as pd

# A DataFrame is essentially a table
data = {
    "product": ["Notebook", "Pen", "Backpack"],
    "price":   [8.99, 1.25, 39.00],
    "units":   [42, 130, 17],
}
df = pd.DataFrame(data)
print(df)
```

```
    product  price  units
0  Notebook   8.99     42
1       Pen   1.25    130
2  Backpack  39.00     17
```

**What to notice:**
- Each column is a named series of values
- Each row has a numeric index starting from 0
- Columns can have different types — `price` is float, `units` is int, `product` is object (string)

Pandas is the foundation of nearly every data analytics and ML workflow in Python. Pandas DataFrames are also the input format expected by Matplotlib, Seaborn, scikit-learn, and virtually every other library you'll use in this course.

---

## Loading and Inspecting Data

The demo data for this session is in `data/raw/paperlane_orders.csv`. This is a real-looking but deliberately messy dataset — exactly the kind you'll encounter in practice.

### Loading a CSV

```python
from pathlib import Path
import pandas as pd

df = pd.read_csv(Path("data") / "raw" / "paperlane_orders.csv")
df.head()
```

`head()` shows the first 5 rows — always the first thing you run after loading.

---

### Understanding the Shape

```python
print(df.shape)      # (rows, columns)
print(df.columns.tolist())
```

---

### Inspecting Types and Nulls

```python
df.info()
```

`info()` is your single most useful inspection tool. It shows:
- Column names and positions
- Number of non-null values per column (immediately reveals missing data)
- Data types (reveals type problems)

**What to look for in the output:**
- Any column where non-null count < total rows → missing values
- Dates stored as `object` → need conversion
- Numbers stored as `object` → likely a formatting issue in the source file

---

### Summary Statistics

```python
df.describe()            # numeric columns only
df.describe(include="all")  # all columns including text
```

`describe()` shows min, max, mean, std for numerics and top/freq for text columns. Glance at it to spot outliers and unexpected values.

---

### Targeted Quality Checks

```python
# Missing values per column
print(df.isna().sum())

# Duplicate rows
print(df.duplicated().sum())

# Unique values in a categorical column — spot inconsistencies
print(df["status"].value_counts())
print(df["region"].value_counts())
```

**Demo: run these on the PaperLane data and narrate what you find:**
- `status` has mixed casing: `Complete`, `complete`, `COMPLETE`
- `region` has a leading space in some values: `" West"`
- `unit_price` has a missing value
- `region` has a missing value
- One row is duplicated
- One date uses a different format: `01/09/2026` instead of `2026-01-09`

> **Professional habit:** Always inspect before you modify. Understand the problems fully before applying any fix.

---

## Handling Missing Values and Duplicates

### Removing Duplicates

```python
print("Before:", df.shape)
df = df.drop_duplicates()
print("After:", df.shape)
```

`drop_duplicates()` removes rows that are identical across all columns. Always print row counts before and after to confirm the expected number were removed.

---

### Checking Missing Values

```python
df.isna().sum()
```

Before deciding what to do with missing values, ask: **why is it missing?**

- A missing `unit_price` on a returned order might be intentional
- A missing `region` might be a data entry error
- A missing value in a critical column needed for analysis might require dropping the row

---

### Strategies for Missing Values

```python
# Strategy 1: Drop rows where a critical column is null
df = df.dropna(subset=["unit_price"])

# Strategy 2: Fill with a fixed value
df["region"] = df["region"].fillna("Unknown")

# Strategy 3: Fill numeric columns with the median
df["unit_price"] = df["unit_price"].fillna(df["unit_price"].median())
```

> **There is no single correct strategy.** The right choice depends on context — what the column represents, how the data will be used, and what a stakeholder would expect. Always document your decision.

---

## Transforming and Standardising Data

### Standardising Text Columns

```python
# Strip leading/trailing whitespace
df["region"] = df["region"].str.strip()

# Normalise casing
df["status"] = df["status"].str.strip().str.title()

# Verify
print(df["status"].value_counts())
print(df["region"].value_counts())
```

After these steps, `"complete"`, `"COMPLETE"`, and `"Complete"` all become `"Complete"`.

---

### Converting Dates

```python
df["order_date"] = pd.to_datetime(df["order_date"], dayfirst=False)
print(df["order_date"].dtype)    # datetime64[ns]
```

`pd.to_datetime()` parses multiple date formats automatically. Once converted, you can extract year, month, day:

```python
df["order_month"] = df["order_date"].dt.month
df["order_day_of_week"] = df["order_date"].dt.day_name()
```

---

### Creating Derived Columns

```python
# Revenue = quantity × unit_price
df["revenue"] = df["quantity"] * df["unit_price"]
df[["product", "quantity", "unit_price", "revenue"]].head()
```

This is a **vectorised operation** — Pandas applies the multiplication to the entire column at once, without any loop. This is both faster and more readable than writing a loop manually.

---

### Filtering Rows

```python
# Exclude returned orders for revenue analysis
df_complete = df[df["status"] == "Complete"].copy()
print(f"Complete orders: {len(df_complete)} of {len(df)} total")
```

`.copy()` creates an independent copy — always use it when creating a filtered subset you plan to modify. Without it, you may get a `SettingWithCopyWarning`.

---

## Saving Clean Data

```python
from pathlib import Path

PROCESSED = Path("data") / "processed"
PROCESSED.mkdir(parents=True, exist_ok=True)

df.to_csv(PROCESSED / "paperlane_orders_clean.csv", index=False)
print("Saved")
```

`index=False` prevents Pandas from writing the row numbers as a column — almost always what you want.

---

## Wrap-Up

Reflect on these questions before moving into the lab:

- Why is `df.info()` usually more useful than `df.describe()` as a first inspection step?
- What is the difference between `dropna()` and `fillna()`?
- Why do you need `.copy()` when creating a filtered subset?
- What does `df["status"].str.title()` do?

---

## Data Reference

The demo data for this session is in `data/raw/paperlane_orders.csv`. It contains the following deliberate data quality issues for demonstration purposes:

| Issue | Column | Detail |
|-------|--------|--------|
| Mixed casing | `status` | `Complete`, `complete`, `COMPLETE` |
| Leading whitespace | `region` | `" West"` |
| Missing value | `unit_price` | One row |
| Missing value | `region` | One row |
| Duplicate row | All columns | `order_id` 1007 appears twice |
| Non-standard date | `order_date` | `01/09/2026` instead of `2026-01-09` |

---

## Resources

- **Pandas documentation:** https://pandas.pydata.org/docs/
- **Pandas user guide — missing data:** https://pandas.pydata.org/docs/user_guide/missing_data.html
- **Real Python — Pandas DataFrame:** https://realpython.com/pandas-dataframe/
- **Real Python — Cleaning data with Pandas:** https://realpython.com/python-data-cleaning-numpy-pandas/
