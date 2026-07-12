# Pandas: Filtering Data

Filtering allows you to select rows that satisfy one or more conditions.

---

# Basic Filtering

## 1. Filter Rows by One Column Value

Use `loc[]` with a boolean condition.

```python
fortune_tech_df = fortune_df.loc[
    fortune_df["sector"] == "Technology"
]
```

### How it works

```python
fortune_df["sector"] == "Technology"
```

returns a boolean Series:

| sector | Result |
|---------|--------|
| Technology | True |
| Healthcare | False |
| Technology | True |
| Retail | False |

Then:

```python
.loc[boolean_series]
```

keeps only rows where the value is `True`.

---

## 2. Filter Rows and Select Columns Together

```python
fortune_df.loc[
    fortune_df["sector"] == "Technology",
    ["company", "sector"]
]
```

`loc[]` syntax:

```python
.loc[
    row_condition,
    columns_to_keep
]
```

This filters rows first and then returns only the selected columns.

---

## 3. Using Boolean Masks

Store filtering conditions for readability and reuse.

```python
filtering_condition = (
    fortune_df["sector"] == "Technology"
)

fortune_tech_df = fortune_df.loc[
    filtering_condition
]
```

A boolean mask is simply a Pandas Series containing `True` and `False`.

Example:

```
0     True
1    False
2     True
3    False
dtype: bool
```

Passing this mask into `loc[]` selects only rows where the value is `True`.

---

## 4. Partial String Matching

Find rows containing specific text.

```python
food_companies_df = fortune_df.loc[
    fortune_df["sector"].str.contains("Food")
]
```

Examples:

```
Food Production
Food Services
Packaged Food
```

All match because they contain `"Food"`.

### Other Useful String Methods

Starts with:

```python
fortune_df["company"].str.startswith("A")
```

Ends with:

```python
fortune_df["company"].str.endswith("Inc")
```

Contains:

```python
fortune_df["company"].str.contains("Tech")
```

---

## 5. Find Missing Values

```python
missing_employees_df = fortune_df.loc[
    fortune_df["employees"].isna()
]
```

`isna()` returns `True` wherever a value is missing (`NaN`).

---

## 6. Find Non-Missing Values

```python
has_employees_df = fortune_df.loc[
    fortune_df["employees"].notna()
]
```

`notna()` is simply the opposite of `isna()`.

---

# Multiple Filtering

## 1. Multiple Conditions (AND)

Use `&`.

```python
fortune_tech_founders_df = fortune_df.loc[
    (fortune_df["sector"] == "Technology")
    &
    (fortune_df["is_founder_ceo"] == True)
]
```

Both conditions must be `True`.

### Truth Table

| Condition A | Condition B | Result |
|-------------|-------------|--------|
| True | True | True |
| True | False | False |
| False | True | False |
| False | False | False |

---

## 2. Multiple Conditions (OR)

Use `|`.

```python
temp_df = fortune_df.loc[
    (fortune_df["rank_change"] > 10)
    |
    (fortune_df["rank_change"] < -10)
]
```

At least one condition must be `True`.

### Truth Table

| Condition A | Condition B | Result |
|-------------|-------------|--------|
| True | True | True |
| True | False | True |
| False | True | True |
| False | False | False |

---

## 3. Filter Using a List with `isin()`

Check whether values belong to a list.

```python
fortune_df.loc[
    fortune_df["sector"].isin([
        "Technology",
        "Automotive",
        "Energy",
        "Financials"
    ])
]
```

Equivalent to:

```python
sector == "Technology"
or
sector == "Automotive"
or
sector == "Energy"
or
sector == "Financials"
```

---

# Advanced Filtering

## 1. Get Top *k* Rows

Return rows with the largest values.

```python
top_10_profit_df = fortune_df.nlargest(
    10,
    "profit"
)
```

Equivalent to:

```python
sort_values(
    "profit",
    ascending=False
).head(10)
```

but faster.

---

## 2. Get Bottom *k* Rows

```python
temp_df = fortune_df.nsmallest(
    5,
    "employees"
)
```

Equivalent to:

```python
sort_values(
    "employees"
).head(5)
```

---

## 3. Filter Using Existing Boolean Columns

If your dataframe already contains boolean flags, filtering becomes much cleaner.

```python
fortune_top10_df = fortune_df.loc[
    fortune_df["is_top10"]
    &
    (
        fortune_df["is_profitable"]
        |
        fortune_df["is_rank_improved"]
    )
]
```

Much easier to read than repeatedly writing long expressions.

---

## 4. Calculations Inside Filtering

You can perform calculations directly inside conditions.

```python
fortune_high_margin_df = fortune_df.loc[
    (
        fortune_df["rank_2025"] <= 10
    )
    |
    (
        fortune_df["profit"]
        /
        fortune_df["revenue"]
        > 0.25
    )
]
```

This keeps companies that either:

- are ranked in the Top 10, **or**
- have a profit margin greater than 25%.

---

## 5. Negating Conditions (`~`)

`~` means **NOT**.

```python
fortune_low_margin_df = fortune_df.loc[
    ~(
        (fortune_df["rank_2025"] <= 10)
        |
        (
            fortune_df["profit"]
            /
            fortune_df["revenue"]
            > 0.25
        )
    )
]
```

This returns the **complement** of the previous filter.

If the previous filter selected:

```
A
B
C
```

then this filter selects everything **except**:

```
A
B
C
```

---

## 6. Verify Complements

A good sanity check:

```python
len(original_df) + len(complement_df) == len(fortune_df)
```

or

```python
original_count + complement_count == total_count
```

If this is true, your complement filter is working correctly.

---

# Common Boolean Operators

| Operator | Meaning |
|-----------|---------|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |
| `&` | AND |
| `\|` | OR |
| `~` | NOT |

---

# Common Filtering Methods

| Method | Purpose |
|---------|---------|
| `loc[]` | Filter rows and/or columns |
| `isin()` | Check membership in a list |
| `isna()` | Missing values |
| `notna()` | Non-missing values |
| `str.contains()` | Partial string match |
| `str.startswith()` | Prefix match |
| `str.endswith()` | Suffix match |
| `nlargest()` | Largest *k* rows |
| `nsmallest()` | Smallest *k* rows |

---

# Interview Tips

- `loc[]` is the standard way to filter rows in Pandas.
- Filtering conditions produce a **boolean Series** (`True`/`False`), also called a **boolean mask**.
- Use `&` for **AND**, `|` for **OR**, and `~` for **NOT**. Always wrap each condition in parentheses.
- `isin()` is cleaner than writing multiple OR (`|`) conditions.
- `str.contains()`, `startswith()`, and `endswith()` are useful for text filtering.
- `isna()` finds missing values; `notna()` finds non-missing values.
- `nlargest()` and `nsmallest()` are more efficient than sorting the entire DataFrame.
- You can include arithmetic expressions directly inside filtering conditions.
