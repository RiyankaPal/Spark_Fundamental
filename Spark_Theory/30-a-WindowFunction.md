# Understanding `groupBy()` vs `partitionBy()` in PySpark

When beginners start learning PySpark, one of the most confusing topics is the difference between **`groupBy()`** and **`partitionBy()`**. Although both group data in some way, they serve completely different purposes.

In this article, we'll understand what each function does, when to use it, and why Window Functions use `partitionBy()` instead of `groupBy()`.

---

## Sample Dataset

Let's consider the following employee data.

| Emp ID | Employee | Department | Salary |
|--------|----------|------------|-------:|
| 101 | Alice | IT | 50000 |
| 102 | Bob | IT | 70000 |
| 103 | Charlie | HR | 45000 |
| 104 | David | HR | 60000 |
| 105 | Emma | Sales | 80000 |

---

# What does `groupBy()` do?

The primary purpose of `groupBy()` is to **combine rows into groups** so that you can perform aggregate calculations like:

- `sum()`
- `avg()`
- `count()`
- `max()`
- `min()`

### Example

```python
from pyspark.sql.functions import sum

employee_df.groupBy("Department") \
           .agg(sum("Salary").alias("Total Salary")) \
           .show()
```

### Output

| Department | Total Salary |
|------------|-------------:|
| IT | 120000 |
| HR | 105000 |
| Sales | 80000 |

Notice something important.

The original employee records are **gone**.

Instead of five rows, we now have only three rows—one for each department.

---

## How `groupBy()` Works

```
Original Data

IT
Alice   50000
Bob     70000

HR
Charlie 45000
David   60000

Sales
Emma    80000
```

After `groupBy()`:

```
IT      → 120000
HR      → 105000
Sales   → 80000
```

Spark combines all rows belonging to the same department into a **single output row**.

---

# What is the limitation of `groupBy()`?

Suppose you want to know:

- Each employee's salary
- The total salary of their department

With `groupBy()`, you cannot keep both pieces of information together because it removes the original rows.

For example, this result is **not possible** using only `groupBy()`.

| Employee | Department | Salary | Department Total |
|----------|------------|-------:|-----------------:|
| Alice | IT | 50000 | 120000 |
| Bob | IT | 70000 | 120000 |

This is where **Window Functions** become useful.

---

# What does `partitionBy()` do?

Unlike `groupBy()`, `partitionBy()` **does not reduce the number of rows**.

Instead, it divides the data into logical groups (called **windows**) so that calculations can be performed within each group while keeping every original row.

Think of it as saying:

> "Group the rows temporarily for calculation, but don't remove any rows."

---

## Example

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import sum

window_spec = Window.partitionBy("Department")

employee_df.withColumn(
    "Department Total",
    sum("Salary").over(window_spec)
).show()
```

### Output

| Employee | Department | Salary | Department Total |
|----------|------------|-------:|-----------------:|
| Alice | IT | 50000 | 120000 |
| Bob | IT | 70000 | 120000 |
| Charlie | HR | 45000 | 105000 |
| David | HR | 60000 | 105000 |
| Emma | Sales | 80000 | 80000 |

Notice that **all original rows are still present**.

A new column is simply added.

---

# How `partitionBy()` Works

```
Original Data

Alice     IT      50000
Bob       IT      70000
Charlie   HR      45000
David     HR      60000
Emma      Sales   80000
```

Spark creates logical windows.

```
Window 1

IT
-----------
Alice
Bob
```

```
Window 2

HR
-----------
Charlie
David
```

```
Window 3

Sales
-----------
Emma
```

Calculations happen **inside each window**, but every row remains in the output.

---

# Simple Analogy

Imagine a classroom.

### `groupBy()`

The teacher asks students to stand according to their class and only reports:

```
Class A → 30 students
Class B → 25 students
Class C → 40 students
```

Individual student information disappears.

---

### `partitionBy()`

The teacher still groups students by class, but everyone stays in the classroom.

Now each student can receive information such as:

- Class rank
- Class average
- Highest marks in the class

Every student remains visible.

---

# When should you use `groupBy()`?

Use `groupBy()` when you only need summary information.

Examples:

- Total sales by country
- Average salary by department
- Number of customers in each city
- Maximum order value per customer

---

# When should you use `partitionBy()`?

Use `partitionBy()` inside **Window Functions** when you need calculations while keeping every row.

Examples:

- Employee ranking
- Running totals
- Previous and next values
- Department-wise salary comparison
- Finding top N employees in each department

---

# `groupBy()` vs `partitionBy()`

| Feature | `groupBy()` | `partitionBy()` |
|---------|-------------|-----------------|
| Removes original rows | ✅ Yes | ❌ No |
| Returns aggregated data | ✅ Yes | ❌ No |
| Keeps every row | ❌ No | ✅ Yes |
| Used with Window Functions | ❌ No | ✅ Yes |
| Supports ranking | ❌ No | ✅ Yes |
| Used for summaries | ✅ Yes | ❌ No |

---

# Key Takeaways

- `groupBy()` combines rows and returns one row per group.
- `partitionBy()` creates logical windows without removing rows.
- `groupBy()` is used for aggregation.
- `partitionBy()` is used with Window Functions.
- If you need to keep every row while performing calculations within a group, use `partitionBy()`.

---

## What's Next?

Now that you understand the difference between `groupBy()` and `partitionBy()`, you're ready to learn Window Functions such as:

- `row_number()`
- `rank()`
- `dense_rank()`
- `lag()`
- `lead()`

These functions use `partitionBy()` to perform powerful row-level calculations while preserving the original dataset.