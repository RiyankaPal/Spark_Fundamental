# Apache Spark GroupBy() – Beginner's Guide

## Introduction

When working with data in Apache Spark, one of the most common tasks is **grouping records** and performing calculations such as:

- Total Salary
- Average Salary
- Employee Count
- Maximum Salary
- Minimum Salary

Spark provides the **groupBy()** function to perform these operations efficiently on large datasets.

---

# Sample Dataset

Let's create a simple employee dataset.

```python
emp_data = [
    (1, "Manish", 50000, "IT"),
    (2, "Vikash", 60000, "Sales"),
    (3, "Raushan", 70000, "Marketing"),
    (4, "Mukesh", 80000, "IT"),
    (5, "Pritam", 90000, "Sales"),
    (6, "Nikita", 45000, "Marketing"),
    (7, "Ragini", 55000, "Marketing"),
    (8, "Rakesh", 100000, "IT"),
    (9, "Aditya", 65000, "IT"),
    (10, "Rahul", 50000, "Marketing")
]

columns = ["id", "name", "salary", "dept"]

emp_df = spark.createDataFrame(emp_data, columns)

emp_df.show()
```

Output

|id|name|salary|dept|
|---|---|---:|---|
|1|Manish|50000|IT|
|2|Vikash|60000|Sales|
|3|Raushan|70000|Marketing|
|...|...|...|...|

---

# What is groupBy()?

The **groupBy()** function groups rows having the same value in one or more columns.

After grouping, we can perform aggregate functions like:

- sum()
- avg()
- max()
- min()
- count()

Syntax

```python
df.groupBy("column_name").agg(aggregation_function)
```

---

# Example 1: Sum of Salary Department-wise

```python
from pyspark.sql.functions import sum

emp_df.groupBy("dept") \
      .agg(sum("salary").alias("Total Salary")) \
      .show()
```

Output

|Dept|Total Salary|
|----|-----------:|
|IT|295000|
|Sales|150000|
|Marketing|220000|

---

# Example 2: Count Employees

```python
from pyspark.sql.functions import count

emp_df.groupBy("dept") \
      .agg(count("*").alias("Employee Count")) \
      .show()
```

Output

|Dept|Employee Count|
|----|-------------:|
|IT|4|
|Sales|2|
|Marketing|4|

---

# Example 3: Average Salary

```python
from pyspark.sql.functions import avg

emp_df.groupBy("dept") \
      .agg(avg("salary").alias("Average Salary")) \
      .show()
```

Output

|Dept|Average Salary|
|----|-------------:|
|IT|73750|
|Sales|75000|
|Marketing|55000|

---

# Example 4: Maximum Salary

```python
from pyspark.sql.functions import max

emp_df.groupBy("dept") \
      .agg(max("salary").alias("Highest Salary")) \
      .show()
```

---

# Example 5: Minimum Salary

```python
from pyspark.sql.functions import min

emp_df.groupBy("dept") \
      .agg(min("salary").alias("Lowest Salary")) \
      .show()
```

---

# Multiple Aggregations

We can calculate multiple statistics in one query.

```python
from pyspark.sql.functions import *

emp_df.groupBy("dept") \
      .agg(
          sum("salary").alias("Total Salary"),
          avg("salary").alias("Average Salary"),
          max("salary").alias("Highest Salary"),
          min("salary").alias("Lowest Salary"),
          count("*").alias("Employees")
      ) \
      .show()
```

---

# SQL Approach

First, create a temporary view.

```python
emp_df.createOrReplaceTempView("employee_tbl")
```

Now execute SQL.

```python
spark.sql("""
SELECT
    dept,
    SUM(salary) AS total_salary
FROM employee_tbl
GROUP BY dept
""").show()
```

Output

|Dept|Total Salary|
|----|-----------:|
|IT|295000|
|Sales|150000|
|Marketing|220000|

---

# Common Mistake

Many beginners write:

```python
spark.sql("""
SELECT dept, SUM("salary")
FROM employee_tbl
GROUP BY dept
""").show()
```

❌ This is incorrect.

`"salary"` is treated as a **string literal**, not as the `salary` column.

As a result, Spark cannot calculate the sum and may return **NULL**.

Correct query:

```python
spark.sql("""
SELECT dept,
       SUM(salary)
FROM employee_tbl
GROUP BY dept
""").show()
```

---

# What if Salary is a String?

Check the schema first.

```python
emp_df.printSchema()
```

Example

```
salary: string
```

Convert it to a numeric type before aggregation.

```python
spark.sql("""
SELECT
    dept,
    SUM(CAST(salary AS DOUBLE)) AS total_salary
FROM employee_tbl
GROUP BY dept
""").show()
```

or

```python
emp_df.groupBy("dept") \
      .agg(sum(col("salary").cast("double")))
```

---

# Common Aggregate Functions

|Function|Description|
|---------|-----------|
|sum()|Returns total value|
|avg()|Returns average|
|max()|Returns highest value|
|min()|Returns lowest value|
|count()|Returns number of records|
|countDistinct()|Counts unique values|
|first()|Returns first value|
|last()|Returns last value|

---

# Interview Questions

### Q1. What is the difference between `groupBy()` and `orderBy()`?

- `groupBy()` groups similar records.
- `orderBy()` sorts records.

---

### Q2. Can we perform multiple aggregations?

Yes.

```python
emp_df.groupBy("dept").agg(
    sum("salary"),
    avg("salary"),
    max("salary")
)
```

---

### Q3. Why is `SUM("salary")` returning NULL?

Because `"salary"` is interpreted as a string literal instead of a column name. Use:

```sql
SUM(salary)
```

---

### Q4. Can we use SQL and DataFrame API together?

Yes. Spark supports both approaches, and both produce the same result.

---

# Best Practices

- Always verify the schema using `printSchema()`.
- Ensure numeric columns are of numeric data types before aggregation.
- Use aliases (`alias()`) to make output columns readable.
- Prefer the DataFrame API for better compile-time checks and SQL for readability, depending on your use case.

---

# Summary

In this guide, you learned:

- How to use `groupBy()`
- Sum, Average, Count, Max, and Min aggregations
- Multiple aggregations in one query
- Using SQL with `GROUP BY`
- The common mistake of writing `SUM("salary")`
- How to cast string columns before aggregation

`groupBy()` is one of the most frequently used transformations in Spark and is essential for performing analytics and building data pipelines.