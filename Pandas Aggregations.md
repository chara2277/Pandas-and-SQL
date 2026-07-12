# Pandas: Grouping, Aggregation, and Sorting Notes

## 1. Grouped Aggregations (`groupby()` + `agg()`)

Group data by one or more columns and calculate summary statistics.

```python
summary_df = (
    trips_df
    .groupby("ride_type")
    .agg(
        total_fare=("fare", "sum"),
        avg_fare=("fare", "mean"),
        total_trips=("trip_id", "nunique")
    )
    .reset_index()
)
```

### Output

| ride_type | total_fare | avg_fare | total_trips |
|-----------|-----------:|----------:|------------:|
| Economy | 4878.13 | 12.13 | 402 |
| Plus | 5037.14 | 12.72 | 396 |
| Premium | 4821.33 | 12.21 | 395 |
| XL | 5623.63 | 12.58 | 447 |

### Common Aggregation Functions

| Function | Description |
|----------|-------------|
| `sum` | Sum of values |
| `mean` | Average |
| `count` | Counts non-null values |
| `nunique` | Counts unique values |
| `min` | Minimum value |
| `max` | Maximum value |
| `median` | Median |
| `std` | Standard deviation |

---

# 2. Split-Apply-Combine

`groupby()` follows the **Split-Apply-Combine** paradigm.

### Split

Divide the dataframe into groups based on unique values.

```python
trips_df.groupby("ride_type")
```

### Apply

Perform calculations on each group.

```python
.sum()
.mean()
.count()
.nunique()
```

### Combine

Pandas automatically combines the results into a new dataframe.

---

# 3. Value Counts (Frequency Table)

Count occurrences of each unique value.

```python
trips_df["ride_type"].value_counts()
```

Example output:

| ride_type | Count |
|-----------|------:|
| XL | 447 |
| Economy | 402 |
| Plus | 396 |
| Premium | 395 |

### Relative Frequency

Returns proportions instead of counts.

```python
trips_df["ride_type"].value_counts(normalize=True)
```

Example:

| ride_type | Proportion |
|-----------|-----------:|
| XL | 0.273 |
| Economy | 0.245 |
| Plus | 0.242 |
| Premium | 0.240 |

---

# 4. Sorting Values (`sort_values()`)

Sort a dataframe by one or more columns.

```python
summary_df = summary_df.sort_values(
    "total_trips",
    ascending=False
)
```

By default:

```python
ascending=True
```

---

# 5. Grouping and Sorting in One Chain

Method chaining keeps code clean and readable.

```python
summary_df = (
    trips_df
    .groupby("ride_type")
    .agg(
        unique_riders=("rider_id", "nunique")
    )
    .reset_index()
    .sort_values("unique_riders")
)
```

---

# Multiple Column Operations

## 1. Drop Duplicates

Get unique combinations of multiple columns.

```python
df[["location", "category"]].drop_duplicates()
```

---

## 2. Group by Multiple Columns

Group using more than one column.

```python
summary_df = (
    orders_df
    .groupby(["location", "category"])
    .agg(
        order_count=("order_id", "count"),
        user_count=("user_id", "nunique"),
        revenue=("amount", "sum"),
        avg_amount=("amount", "mean")
    )
    .reset_index()
)
```

Example output:

| location | category | order_count | user_count | revenue | avg_amount |
|----------|----------|------------:|-----------:|---------:|-----------:|
| UK | Electronics | 100 | 62 | 24006.25 | 240.06 |
| UK | Fashion | 210 | 124 | 29380.10 | 139.91 |
| UK | Grocery | 10 | 2 | ... | ... |

---

## 3. Order of Grouping Columns Matters

The order of columns passed to `groupby()` determines how the results are organized.

```python
.groupby(["location", "category"])
```

Results are grouped by **location first**, then **category**.

Example:

```
Canada
    Electronics
    Fashion
    Grocery

UK
    Electronics
    Fashion
    Grocery
```

If you reverse the order:

```python
.groupby(["category", "location"])
```

The output becomes:

```
Electronics
    Canada
    UK
    USA

Fashion
    Canada
    UK
    USA
```

The aggregation values remain the same, but the grouping hierarchy and output ordering change.

---

## 4. Sorting by Multiple Columns

Sort using multiple columns with different sort directions.

```python
summary_df = summary_df.sort_values(
    by=["location", "revenue"],
    ascending=[True, False]
)
```

Explanation:

- `by=["location", "revenue"]`
  - Sort by **location** first.
  - Within each location, sort by **revenue**.

- `ascending=[True, False]`
  - `location`: alphabetical (A → Z)
  - `revenue`: highest to lowest

Example:

```
Canada
    15000
    12000
    8000

UK
    24000
    18000
    9000
```

---

# Common Interview Tips

- `groupby()` is used for grouped analysis.
- `agg()` allows multiple aggregations in one operation.
- `count()` counts non-null rows.
- `nunique()` counts distinct values.
- `value_counts()` creates frequency tables.
- `normalize=True` returns percentages/proportions.
- `drop_duplicates()` returns unique rows or unique column combinations.
- `sort_values()` sorts by one or multiple columns.
- `reset_index()` converts grouped indices back into regular columns after `groupby()`.

