# Pandas: Merging DataFrames (Joins)

DataFrame merging combines information from multiple tables using a common column (key).

This is similar to **SQL JOIN operations**.

---

# 1. Inner Join (`merge()`)

An **inner join** returns only rows where the key exists in **both** datasets.

Rows without a matching value are removed.

### Syntax

```python
df1.merge(
    df2,
    on="key_column",
    how="inner"
)
```

`how="inner"` is the default.

---

## Example

### Taxi Owners Table

| vid | owner |
|-----|-------|
| 101 | John |
| 102 | Sarah |
| 103 | Mike |

### Taxi Vehicles Table

| vid | vehicle |
|-----|---------|
| 101 | Toyota |
| 102 | Honda |
| 104 | BMW |

Merge:

```python
taxi_own_veh = taxi_owners.merge(
    taxi_veh,
    on="vid",
    suffixes=("_own", "_veh")
)
```

Result:

| vid | owner | vehicle |
|-----|-------|---------|
| 101 | John | Toyota |
| 102 | Sarah | Honda |

Explanation:

- `vid = 101` exists in both → included.
- `vid = 102` exists in both → included.
- `vid = 103` only in owners → removed.
- `vid = 104` only in vehicles → removed.

---

# Merge Parameters

## `on`

Specifies the column used for matching.

```python
on="vid"
```

Means:

```
taxi_owners.vid == taxi_veh.vid
```

Rows are matched based on this value.

---

## `suffixes`

Used when both DataFrames contain columns with the same name.

Example:

```python
taxi_owners.merge(
    taxi_veh,
    on="vid",
    suffixes=("_own", "_veh")
)
```

Before merge:

### taxi_owners

| vid | name |
|-----|------|
| 101 | John |

### taxi_veh

| vid | name |
|-----|------|
| 101 | Toyota |

After merge:

| vid | name_own | name_veh |
|-----|----------|----------|
| 101 | John | Toyota |

Suffixes make it clear which table each column came from.

---

# 2. One-to-Many Relationships

A **one-to-many relationship** occurs when:

- One value in the first table
- Matches multiple rows in the second table

Example:

A single taxi owner can own multiple vehicles.

---

## Owner Table (One Side)

`taxis_owners`

| owner_id | owner |
|----------|-------|
| 1 | John |
| 2 | Sarah |

Each owner appears once.

---

## Vehicles Table (Many Side)

`taxis_vehicles`

| vehicle_id | owner_id | vehicle |
|------------|----------|---------|
| 101 | 1 | Toyota |
| 102 | 1 | Honda |
| 103 | 2 | BMW |

Owner `1` has two vehicles.

---

## Merge

```python
owner_vehicle_df = owners.merge(
    vehicles,
    on="owner_id",
    how="inner"
)
```

Result:

| owner_id | owner | vehicle |
|----------|-------|---------|
| 1 | John | Toyota |
| 1 | John | Honda |
| 2 | Sarah | BMW |

The row for John is duplicated because there are multiple matching vehicles.

---

# Understanding One-to-Many Expansion

Before merge:

```
Owners

John
Sarah


Vehicles

John → Toyota
John → Honda
Sarah → BMW
```

After merge:

```
John   Toyota
John   Honda
Sarah  BMW
```

The "one" side expands to match the "many" side.

---

# Checking Relationship Types

Pandas can validate relationships using `validate`.

## One-to-One

Each key appears once in both tables.

```python
df1.merge(
    df2,
    on="id",
    validate="one_to_one"
)
```

---

## One-to-Many

First table has unique keys, second can have duplicates.

```python
df1.merge(
    df2,
    on="id",
    validate="one_to_many"
)
```

Example:

```
Customer → Orders
```

One customer can have many orders.

---

## Many-to-One

Reverse relationship:

```python
df1.merge(
    df2,
    on="id",
    validate="many_to_one"
)
```

Example:

```
Many orders → One customer
```

---

## Many-to-Many

Both tables contain duplicate keys.

```python
df1.merge(
    df2,
    on="id",
    validate="many_to_many"
)
```

Example:

```
Students ↔ Courses
```

A student can take many courses, and a course has many students.

---

# Common Join Types

| Join | Description |
|------|-------------|
| Inner Join | Only matching rows |
| Left Join | All rows from left table + matches from right |
| Right Join | All rows from right table + matches from left |
| Outer Join | All rows from both tables |

---

# Interview Tips

- `merge()` is Pandas' equivalent of SQL JOIN.
- `on=` specifies the matching key column.
- Inner joins keep only overlapping keys.
- One-to-many joins duplicate rows from the "one" side.
- Use `suffixes=` when columns have the same name in both tables.
- Use `validate=` to check that the relationship between tables is what you expect.
- Always understand the relationship type:
  - One-to-one
  - One-to-many
  - Many-to-one
  - Many-to-many
