# Understanding `lead()` and `lag()` in PySpark Window Functions

When working with real-world data, we often need to compare the current row with the **previous** or **next** row.

For example:

- What was yesterday's sales?
- How much has an employee's salary increased compared to the previous employee?
- What is the next transaction date?
- Compare today's stock price with yesterday's price.

These types of problems can be solved easily using **`lag()`** and **`lead()`** Window Functions in PySpark.

In this article, we'll learn both functions with simple examples.

---

# Sample Dataset

Suppose we have the following employee data.

| Emp ID | Employee | Department | Salary |
|--------|----------|------------|-------:|
|101|Alice|IT|50000|
|102|Bob|IT|65000|
|103|Charlie|IT|80000|
|104|David|Sales|45000|
|105|Emma|Sales|70000|

We will use this dataset throughout the article.

---

# Before Using `lead()` and `lag()`

Both functions work with **Window Specifications**.

First, create a window.

```python
from pyspark.sql.window import Window

window_spec = Window.partitionBy("Department") \
                    .orderBy("Salary")
```

This means:

- Divide employees department-wise.
- Sort employees by salary inside each department.

For the IT department, Spark sees the data like this:

| Employee | Salary |
|----------|-------:|
|Alice|50000|
|Bob|65000|
|Charlie|80000|

---

# What is `lag()`?

`lag()` returns the value from a **previous row**.

Think of it as asking:

> "What was the value in the row just before this one?"

---

## Syntax

```python
lag(column_name, offset)
```

- **column_name** → Column whose previous value is required.
- **offset** → Number of rows to go back.

---

## Example

```python
from pyspark.sql.functions import lag

employee_df.withColumn(
    "Previous Salary",
    lag("Salary", 1).over(window_spec)
).show()
```

---

## Output

| Employee | Salary | Previous Salary |
|----------|-------:|----------------:|
|Alice|50000|NULL|
|Bob|65000|50000|
|Charlie|80000|65000|

---

## How does `lag()` work?

```
Salary

50000
65000
80000
```

Spark shifts the values **downward**.

```
Current Salary    Previous Salary

50000             NULL
65000             50000
80000             65000
```

The first row has no previous row, so Spark returns **NULL**.

---

# Changing the Offset

Suppose we want the salary from **two rows earlier**.

```python
lag("Salary", 2)
```

Output

| Salary | Previous Salary |
|-------:|----------------:|
|50000|NULL|
|65000|NULL|
|80000|50000|

The offset determines how many rows Spark moves backward.

---

# What is `lead()`?

`lead()` is exactly the opposite of `lag()`.

Instead of returning the previous value, it returns the **next row's value**.

Think of it as asking:

> "What comes after the current row?"

---

## Syntax

```python
lead(column_name, offset)
```

---

## Example

```python
from pyspark.sql.functions import lead

employee_df.withColumn(
    "Next Salary",
    lead("Salary", 1).over(window_spec)
).show()
```

---

## Output

| Employee | Salary | Next Salary |
|----------|-------:|------------:|
|Alice|50000|65000|
|Bob|65000|80000|
|Charlie|80000|NULL|

---

## How does `lead()` work?

```
Salary

50000
65000
80000
```

Spark shifts values **upward**.

```
Current Salary    Next Salary

50000             65000
65000             80000
80000             NULL
```

Since the last row has no next row, Spark returns **NULL**.

---

# Using Default Values

Instead of returning NULL, we can provide a default value.

### Example

```python
lag("Salary", 1, 0)
```

Output

| Salary | Previous Salary |
|-------:|----------------:|
|50000|0|
|65000|50000|
|80000|65000|

Similarly,

```python
lead("Salary", 1, 0)
```

returns **0** for the last row instead of NULL.

---

# `lag()` vs `lead()`

| Function | Returns |
|----------|---------|
|`lag()`|Previous row value|
|`lead()`|Next row value|

---

# Visual Difference

Suppose salaries are:

```
50000
65000
80000
90000
```

### `lag()`

```
Current    Previous

50000      NULL
65000      50000
80000      65000
90000      80000
```

### `lead()`

```
Current    Next

50000      65000
65000      80000
80000      90000
90000      NULL
```

---

# Real-World Use Cases

`lag()` is commonly used for:

- Comparing today's sales with yesterday's sales
- Salary comparison
- Detecting changes in records
- Previous transaction amount
- Calculating profit or loss

`lead()` is commonly used for:

- Finding the next transaction
- Predicting upcoming events
- Calculating time gaps
- Identifying the next purchase date
- Comparing current and future values

---

# Common Mistakes

### Forgetting `orderBy()`

`lag()` and `lead()` depend on row order.

Without `orderBy()`, Spark has no defined sequence, and the results may be unpredictable.

Always specify an ordering column.

```python
Window.partitionBy("Department").orderBy("Salary")
```

---

### Using the Wrong Offset

```python
lag("Salary", 1)
```

means **one row back**.

```python
lag("Salary", 2)
```

means **two rows back**.

The same applies to `lead()`.

---

# Interview Questions

### Q1. What is the difference between `lag()` and `lead()`?

- `lag()` returns the previous row's value.
- `lead()` returns the next row's value.

---

### Q2. Why do `lag()` and `lead()` require `orderBy()`?

Because Spark must know the sequence of rows before determining which row is previous or next.

---

### Q3. What happens if there is no previous or next row?

Spark returns **NULL** unless a default value is specified.

---

### Q4. Can `lag()` and `lead()` be used without Window Functions?

No.

Both functions are Window Functions and must be used with `.over(window_spec)`.

---

# Key Takeaways

- `lag()` retrieves values from previous rows.
- `lead()` retrieves values from upcoming rows.
- Both functions work inside a Window Specification.
- Always use `orderBy()` to define row order.
- Use the `offset` parameter to control how many rows to move.
- You can replace `NULL` values with a default value.

---

## Conclusion

`lead()` and `lag()` are two of the most useful Window Functions in PySpark. They make it easy to compare rows without writing complex joins or subqueries.

Once you understand these functions, you'll be able to solve many real-world data engineering problems such as trend analysis, change detection, transaction comparison, and time-series analysis with just a few lines of code.