# Apache Spark Union vs UnionAll vs UnionByName: A Complete Beginner’s Guide

When working with multiple datasets in Apache Spark, one of the most common requirements is combining data from different DataFrames. Spark provides several ways to achieve this, including **union()**, **unionAll()**, and **unionByName()**.

Although these operations appear similar, they behave differently in specific scenarios such as duplicate records, column order mismatches, and schema differences.

In this article, we'll explore each of them with practical examples.

---

# What is Union in Spark?

The `union()` operation combines rows from two DataFrames having the same schema.

Let's create two DataFrames:

```python
data = [
    (10, 'Anil', 50000, 18),
    (11, 'Vikas', 75000, 16),
    (12, 'Nisha', 40000, 18)
]

schema = ['id', 'name', 'sal', 'mngr_id']

manager_df = spark.createDataFrame(data, schema)

data1 = [
    (19, 'Sohan', 50000, 18),
    (20, 'Sima', 75000, 17)
]

manager_df1 = spark.createDataFrame(data1, schema)
```

Apply Union:

```python
manager_df.union(manager_df1).show()
```

Output:

```text
+---+-----+-----+-------+
|id |name |sal  |mngr_id|
+---+-----+-----+-------+
|10 |Anil |50000|18     |
|11 |Vikas|75000|16     |
|12 |Nisha|40000|18     |
|19 |Sohan|50000|18     |
|20 |Sima |75000|17     |
+---+-----+-----+-------+
```

Spark simply appends rows from the second DataFrame to the first.

---

# Does Spark Union Remove Duplicates?

A common misconception is that `union()` removes duplicates.

It does **not**.

Consider a DataFrame containing duplicate rows:

```python
duplicate_manager_df.union(manager_df1).count()
```

Even if duplicate records exist, Spark DataFrame `union()` keeps all rows.

If you want unique records, use:

```python
df1.union(df2).distinct()
```

---

# Union vs UnionAll in Spark DataFrames

Many developers coming from SQL expect different behavior between `union()` and `unionAll()`.

In Spark DataFrames:

```python
df1.union(df2)
```

and

```python
df1.unionAll(df2)
```

produce the same result.

Example:

```python
duplicate_manager_df.union(manager_df1).count()

duplicate_manager_df.unionAll(manager_df1).count()
```

Both return the same count because neither operation removes duplicates.

### Key Point

For DataFrames:

- `union()` = keeps duplicates
- `unionAll()` = keeps duplicates
- Both behave identically

In modern Spark versions, `unionAll()` is simply an alias of `union()`.

---

# Union vs Union All in Spark SQL

Things become different when using SQL.

Create temporary views:

```python
manager_df1.createOrReplaceTempView("manager_df1_tbl")
duplicate_manager_df.createOrReplaceTempView("duplicate_manager_df_tbl")
```

### SQL UNION

```sql
SELECT * FROM manager_df1_tbl
UNION
SELECT * FROM duplicate_manager_df_tbl
```

Here SQL removes duplicate rows automatically.

### SQL UNION ALL

```sql
SELECT * FROM manager_df1_tbl
UNION ALL
SELECT * FROM duplicate_manager_df_tbl
```

This keeps all rows including duplicates.

### Difference

| Operation | Removes Duplicates |
|------------|-------------------|
| SQL UNION | Yes |
| SQL UNION ALL | No |
| DataFrame union() | No |
| DataFrame unionAll() | No |

This is one of the most important interview questions in Spark.

---

# What Happens If Column Order Is Different?

Suppose two DataFrames have the same columns but in a different order.

### DataFrame 1

```text
[id, name, sal, mngr_id]
```

### DataFrame 2

```text
[id, sal, mngr_id, name]
```

```python
manager_df1.union(Wrong_manager_df).show()
```

Spark matches columns based on position, not column names.

As a result:

```text
id -> id
name -> sal
sal -> mngr_id
mngr_id -> name
```

Data becomes incorrect.

This is a very common real-world issue.

---

# Introducing UnionByName

To solve column-order problems, Spark provides `unionByName()`.

```python
manager_df1.unionByName(Wrong_manager_df).show()
```

Now Spark matches columns using column names instead of positions.

Result:

```text
id matched with id
name matched with name
sal matched with sal
mngr_id matched with mngr_id
```

Data remains accurate regardless of column order.

### Recommendation

Whenever schemas contain the same columns but order may vary, prefer:

```python
unionByName()
```

over

```python
union()
```

---

# What If Number of Columns Is Different?

Consider:

```text
[id, sal, mngr_id, name, bonus]
```

and

```text
[id, name, sal, mngr_id]
```

Now one DataFrame contains an additional column (`bonus`).

Attempting:

```python
Wrong_manager_df.union(manager_df1)
```

results in an error because both DataFrames must have the same number of columns.

Typical error:

```text
Union can only be performed on tables with the same number of columns.
```

---

# How to Handle Different Column Counts?

Select only matching columns:

```python
Wrong_manager_df.select(
    'id',
    'sal',
    'mngr_id',
    'Name'
).union(manager_df1)
```

This ensures both DataFrames have identical structures before performing the union.

---

# Limitation of UnionByName

Consider:

```python
Wrong_schema2 = [
    'id',
    'sal',
    'mngr_id',
    'Nam'
]
```

Notice:

```text
Nam
```

instead of

```text
Name
```

Now:

```python
Wrong_manager_df.unionByName(Wrong_manager_df2)
```

fails because Spark cannot find a matching column.

`unionByName()` only works when column names match exactly.

Even a small spelling mistake causes failure.

### Best Practice

Before performing union operations:

- Standardize column names
- Remove extra spaces
- Follow naming conventions
- Validate schemas

---

# Best Practices for Using Union Operations

### Use `union()` when

- Column order is identical
- Schemas are identical
- Performance is critical

### Use `unionByName()` when

- Column order may differ
- Data comes from different sources
- Schema consistency cannot be guaranteed

### Use `distinct()` when

- Duplicate removal is required

Example:

```python
df1.union(df2).distinct()
```

---

# Final Thoughts

Understanding the difference between `union()`, `unionAll()`, and `unionByName()` can save hours of debugging in Spark projects.

Remember these golden rules:

✅ DataFrame `union()` keeps duplicates

✅ DataFrame `unionAll()` behaves the same as `union()`

✅ SQL `UNION` removes duplicates

✅ SQL `UNION ALL` keeps duplicates

✅ `union()` matches columns by position

✅ `unionByName()` matches columns by name

✅ Column counts must match for union operations

Mastering these concepts will help you build reliable ETL pipelines and avoid data quality issues in production environments.