# Pandas: Conditional Transformations & Conditional Aggregations

## 1. Conditional Transformation with `np.where()`

Use `np.where()` to assign values based on a condition.

```python
flights_df["distance_km"] = np.where(
    flights_df["distance_unit"] == "miles",
    flights_df["distance"] * 1.60934,
    flights_df["distance"]
)
```

### Syntax

```python
np.where(condition, value_if_true, value_if_false)
```

### Arguments

- **Condition**: Boolean expression evaluated for every row.
- **Value if True**: Assigned when the condition is `True`.
- **Value if False**: Assigned when the condition is `False`.

Example:

```python
np.where(
    flights_df["distance_unit"] == "miles",
    flights_df["distance"] * 1.60934,
    flights_df["distance"]
)
```

- If unit is `"miles"` → convert to kilometers.
- Otherwise → keep the original value.

---

# 2. Multiple Conditional Transformations with `np.select()`

Use `np.select()` when there are multiple conditions.

```python
conditions = [
    flights_df["delay"] <= -5,
    flights_df["delay"] < 5,
    flights_df["delay"] >= 5
]

outcomes = [
    "Early",
    "On time",
    "Late"
]

flights_df["delay_status"] = np.select(
    conditions,
    outcomes,
    default="Unknown"
)
```

### Syntax

```python
np.select(
    conditions,
    outcomes,
    default
)
```

### Components

- **conditions**
  - List of boolean expressions.
  - One condition for each possible category.

- **outcomes**
  - List of values returned when the corresponding condition is `True`.

- **default**
  - Returned if none of the conditions match.

### How it works

For every row, `np.select()` checks conditions **in order**.

- First `True` condition is selected.
- Remaining conditions are ignored.

Example:

```
Delay = -8
Condition 1 → True
Result = "Early"

Delay = 2
Condition 1 → False
Condition 2 → True
Result = "On time"

Delay = 12
Condition 3 → True
Result = "Late"
```

---

# Conditional Aggregation

## 1. Filter Before Grouping

Filter rows first, then perform aggregation.

```python
flight_df = flights_df.loc[
    flights_df["flight_status"] == "arrived"
]

flight_df = (
    flight_df
    .groupby("airline")
    .agg(
        average_delay=("delay", "mean")
    )
    .reset_index()
)
```

Only flights with status `"arrived"` are included in the aggregation.

---

## 2. Using `.query()` After Aggregation

Useful for filtering aggregated results while method chaining.

```python
arrived_flights_df = flights_df[
    flights_df["flight_status"] == "arrived"
]

airline_summary_df = (
    arrived_flights_df
    .groupby("airline")
    .agg(
        avg_delay=("delay", "mean")
    )
    .query("avg_delay > 5")
)
```

Here:

- Calculate average delay for each airline.
- Keep only airlines with an average delay greater than 5 minutes.

---

## 3. Conditional Aggregation with `lambda`

Calculate statistics only for rows satisfying a condition.

```python
airline_summary_df = (
    arrived_flights_df
    .groupby("airline")
    .agg(
        avg_delay=("delay", "mean"),
        avg_delay_delayed=(
            "delay",
            lambda x: x[x > 5].mean()
        )
    )
)
```

Equivalent function:

```python
def avg_delay_delayed(x):
    return x[x > 5].mean()
```

### What is `x`?

Inside `groupby().agg()`, pandas passes **one grouped column** to the function.

For example, if grouping by airline:

```
Airline A

delay
-----
3
7
12
-2
```

Then:

```python
x
```

is

```python
0     3
1     7
2    12
3    -2
dtype: int64
```

It is a **Pandas Series**, **not** a DataFrame.

Then:

```python
x[x > 5]
```

returns

```
7
12
```

and

```python
.mean()
```

returns

```
9.5
```

---

# 4. Conditional Aggregation with `apply()`

Use `apply()` when calculations involve multiple columns.

```python
airline_summary_df = (
    arrived_flights_df
    .groupby("airline")
    .apply(
        lambda x: pd.Series({
            "avg_delay": x["delay"].mean(),
            "avg_delay_delayed":
                x.loc[x["delay"] > 5, "delay"].mean(),
            "total_passengers":
                x["passengers"].sum(),
            "passengers_delayed":
                x.loc[
                    x["delay"] > 5,
                    "passengers"
                ].sum()
        }),
        include_groups=False
    )
)
```

### How `apply()` Works

Unlike `agg()`, `apply()` receives the **entire grouped DataFrame**.

Example group:

| flight_id | delay | passengers |
|-----------|------:|-----------:|
| 1 | 3 | 120 |
| 2 | 10 | 180 |
| 3 | 7 | 160 |

Inside the lambda:

```python
x
```

is the complete DataFrame above.

Therefore you can access multiple columns:

```python
x["delay"]

x["passengers"]

x.loc[x["delay"] > 5, "passengers"]
```

### Pattern

```python
groupby(...)
.apply(
    lambda x: pd.Series({
        ...
    }),
    include_groups=False
)
```

- `apply()` passes the full grouped DataFrame.
- `lambda x` receives one grouped DataFrame at a time.
- `pd.Series()` creates the output columns.
- `include_groups=False` prevents grouping columns from being included in `x`.

---

# 5. Arithmetic Inside `apply()`

You can combine filtering and arithmetic.

```python
airline_summary_df = (
    flights_df
    .groupby("airline")
    .apply(
        lambda x: pd.Series({
            "total_flights":
                x["flight_id"].count(),

            "cancellation_rate":
                100 *
                x.loc[
                    x["flight_status"] == "canceled",
                    "flight_status"
                ].count()
                /
                x["flight_status"].count()
        }),
        include_groups=False
    )
)
```

Cancellation rate formula:

```
Cancellation Rate

=
(Number of canceled flights
 /
Total flights)

× 100
```

---

# 6. Putting Everything Together

Complex grouped summaries often combine multiple conditional calculations.

```python
flight_df = (
    flight_df
    .groupby("airline")
    .apply(
        lambda x: pd.Series({
            "flights":
                x["flight_id"].count(),

            "flights_longhaul":
                x.loc[
                    x["distance"] > 3000,
                    "flight_id"
                ].count(),

            "average_fuel_efficiency":
                x["fuel_efficiency"].mean(),

            "average_fuel_efficiency_longhaul":
                x.loc[
                    x["distance"] > 3000,
                    "fuel_efficiency"
                ].mean()
        }),
        include_groups=False
    )
    .query("flights > 50")
)
```

This pipeline performs:

1. Group by airline.
2. Count total flights.
3. Count long-haul flights (`distance > 3000`).
4. Compute average fuel efficiency.
5. Compute average fuel efficiency for only long-haul flights.
6. Keep airlines with more than 50 flights.

---

# `agg()` vs `apply()`

| Feature | `agg()` | `apply()` |
|---------|---------|-----------|
| Receives | One grouped **Series** | Entire grouped **DataFrame** |
| Access multiple columns | ❌ No | ✅ Yes |
| Faster | ✅ Yes | ❌ Usually slower |
| Best for | Standard aggregations | Complex custom logic |
| Conditional calculations on one column | ✅ Yes | ✅ Yes |
| Conditional calculations across multiple columns | ❌ Difficult | ✅ Easy |

---

# Interview Tips

- Use **`np.where()`** for two-way conditional assignments.
- Use **`np.select()`** for multiple conditions.
- Filter rows before grouping when appropriate.
- Use **`.query()`** to filter aggregated results while chaining.
- Inside **`agg()`**, the lambda receives a **Series**.
- Inside **`apply()`**, the lambda receives the **entire grouped DataFrame**.
- Use **`apply()`** when calculations depend on multiple columns.
- `include_groups=False` avoids including grouping columns inside the grouped DataFrame and prevents future pandas warnings.
