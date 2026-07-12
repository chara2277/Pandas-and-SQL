# Pandas: Creating & Transforming Columns

Column transformations allow you to create new features by performing calculations on existing columns.

---

# 1. Create a New Column

Create a new column using arithmetic operations on existing columns.

```python
df["new_column"] = (
    df["col1"] * df["col2"]
    - df["col3"]
)
```

Pandas performs the calculation **row by row**.

Example:

| col1 | col2 | col3 | new_column |
|-----:|-----:|-----:|-----------:|
| 5 | 2 | 3 | 7 |
| 10 | 4 | 8 | 32 |
| 7 | 3 | 5 | 16 |

Calculation:

```
Row 1

(5 × 2) - 3 = 7

Row 2

(10 × 4) - 8 = 32

Row 3

(7 × 3) - 5 = 16
```

No loops are needed because Pandas performs **vectorized operations**, applying the calculation to every row automatically.

---

# 2. Calculating Ratios or Percentages

Create percentage-based metrics using arithmetic.

```python
campaign_performance_df["pcvr"] = (
    100 *
    (
        campaign_performance_df["paid_orders"]
        /
        campaign_performance_df["paid_sessions"]
    )
)
```

Example:

| paid_orders | paid_sessions | PCVR |
|------------:|--------------:|-----:|
| 40 | 200 | 20% |
| 25 | 100 | 25% |
| 18 | 60 | 30% |

Formula:

```
PCVR

=
(Paid Orders
 /
Paid Sessions)

× 100
```

This is commonly used to calculate:

- Conversion rate
- Profit margin
- Click-through rate (CTR)
- Return rate
- Percentage contribution

---

# Complex Transformations

## 1. Percentage Contribution

Calculate each row's contribution to the total.

```python
category_sales_df["margin_pct"] = (
    100 *
    category_sales_df["margin"]
    /
    category_sales_df["margin"].sum()
)
```

Suppose:

| Category | Margin |
|-----------|-------:|
| Electronics | 500 |
| Grocery | 300 |
| Fashion | 200 |

Total margin:

```
500 + 300 + 200 = 1000
```

New column:

| Category | Margin | Margin % |
|-----------|-------:|---------:|
| Electronics | 500 | 50 |
| Grocery | 300 | 30 |
| Fashion | 200 | 20 |

Each row is divided by the **column total**.

---

# 2. Rounding Values

Round numerical values to a fixed number of decimal places.

```python
df["column"] = round(
    df["column"],
    2
)
```

or equivalently,

```python
df["column"] = df["column"].round(2)
```

Example:

| Original | Rounded (2 dp) |
|----------:|---------------:|
| 12.34567 | 12.35 |
| 9.87654 | 9.88 |
| 4.11111 | 4.11 |

The second argument specifies the number of decimal places.

```
round(value, decimals)
```

Examples:

```python
round(12.3456, 0)   # 12

round(12.3456, 1)   # 12.3

round(12.3456, 2)   # 12.35

round(12.3456, 3)   # 12.346
```

---

# Common Arithmetic Operators

| Operator | Description |
|-----------|-------------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `//` | Floor division |
| `%` | Modulus (remainder) |
| `**` | Exponentiation |

These operators work **element-wise** on Pandas columns.

Example:

```python
df["profit"] = (
    df["revenue"]
    -
    df["cost"]
)
```

Every row computes:

```
profit = revenue - cost
```

---

# Vectorized Operations

Most arithmetic in Pandas is **vectorized**, meaning operations are automatically applied to every row.

Instead of writing:

```python
for row in df:
    ...
```

you simply write:

```python
df["profit"] = (
    df["revenue"]
    -
    df["cost"]
)
```

This is:

- shorter
- easier to read
- significantly faster

---

# Interview Tips

- Arithmetic operations on Pandas columns are **vectorized** and performed element-wise.
- New columns are created using simple assignment: `df["new_col"] = ...`.
- Ratios and percentages are commonly calculated using division and multiplication by 100.
- `sum()` is often used to calculate percentage contributions to a total.
- Use `round()` or `.round()` to control decimal precision for reporting.
- Prefer vectorized operations over Python loops for better performance and cleaner code.
