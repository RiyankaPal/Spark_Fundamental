# Handling Corrupted Records in PySpark (Beginner-Friendly Guide)

When reading CSV, JSON, or other files in PySpark, you may encounter **corrupted records**. These are records that don't match the expected schema or have invalid data.

Instead of failing immediately, Spark provides different **read modes** to decide how corrupted records should be handled.

In this guide, we'll learn:

- What are corrupted records?
- Different read modes in PySpark
- How to capture corrupted records
- How to store bad records for later analysis
- Best practices

---

# What is a Corrupted Record?

A corrupted record is a row that cannot be parsed correctly because it doesn't match the expected format or schema.

For example, suppose your schema expects:

| id | name | age | salary |
|----|------|-----|--------|

But your CSV contains:

```text
1,John,25,50000
2,Alice,Thirty,60000
3,Bob,28
4,David,29,abc
```

Problems:

- `"Thirty"` cannot be converted to an Integer.
- Missing salary value.
- `"abc"` cannot be converted to a numeric value.

These rows are considered **corrupted** (depending on the schema and parsing rules).

---

# Read Modes in PySpark

PySpark provides three modes for handling corrupted records:

1. PERMISSIVE (Default)
2. DROPMALFORMED
3. FAILFAST

---

# 1. PERMISSIVE Mode (Default)

This is the default behavior in Spark.

```python
employee_df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "false") \
    .option("mode", "PERMISSIVE") \
    .load("/FileStore/tables/Employee_data.csv")

employee_df.show()
```

### What happens?

- Spark loads every record.
- Invalid fields become `NULL`.
- Processing continues without failing.

### Example

CSV:

```text
id,name,age
1,John,25
2,Alice,Thirty
3,Bob,30
```

Output:

| id | name | age |
|----|------|-----|
| 1 | John | 25 |
| 2 | Alice | NULL |
| 3 | Bob | 30 |

---

## When should you use PERMISSIVE?

Use it when:

- You don't want your job to fail.
- Data quality issues are expected.
- You want to inspect bad records later.

---

# 2. DROPMALFORMED Mode

In this mode, Spark simply removes corrupted rows.

```python
employee_df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "false") \
    .option("mode", "DROPMALFORMED") \
    .load("/FileStore/tables/Employee_data.csv")

employee_df.show()
```

### What happens?

- Valid records are loaded.
- Corrupted rows are skipped.
- No error is thrown.

### Example

Input:

```text
1,John,25
2,Alice,Thirty
3,Bob,30
```

Output:

| id | name | age |
|----|------|-----|
|1|John|25|
|3|Bob|30|

The second row is completely removed.

---

## When should you use DROPMALFORMED?

Use it when:

- Corrupted rows are not important.
- You only need clean data.
- Losing a few bad records is acceptable.

---

# 3. FAILFAST Mode

This is the strictest mode.

```python
employee_df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .option("mode", "FAILFAST") \
    .load("/FileStore/tables/Employee_data.csv")

employee_df.show()
```

### What happens?

- Spark stops immediately after finding the first corrupted record.
- The job fails with an exception.
- No DataFrame is created.

---

## When should you use FAILFAST?

Use it when:

- Data quality is critical.
- No corrupted records are acceptable.
- You want to detect issues as early as possible.

---

# Comparison of Read Modes

| Mode | What Happens? |
|------|---------------|
| PERMISSIVE | Keeps all rows and replaces invalid values with NULL |
| DROPMALFORMED | Removes corrupted rows |
| FAILFAST | Stops execution immediately when a corrupted record is found |

---

# Capturing Corrupted Records

Instead of losing bad data, Spark allows us to store the entire corrupted row in a separate column.

First, create a schema that includes a column for corrupted records.

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType
)

emp_schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", IntegerType(), True),
    StructField("_corrupt_record", StringType(), True)
])
```

Now read the file.

```python
employee_df = spark.read.format("csv") \
    .option("header", "true") \
    .option("mode", "PERMISSIVE") \
    .schema(emp_schema) \
    .load("/FileStore/tables/Employee_data.csv")

employee_df.show(truncate=False)
```

The `_corrupt_record` column stores the original malformed row, making it easier to identify and fix bad data.

---

# Storing Bad Records

Instead of keeping corrupted rows inside the DataFrame, Spark can write them to a separate location using `badRecordsPath`.

```python
employee_df = spark.read.format("csv") \
    .option("header", "true") \
    .schema(emp_schema) \
    .option("badRecordsPath", "/FileStore/tables/bad_Records_Employee") \
    .load("/FileStore/tables/Employee_data.csv")
```

Spark automatically creates a folder containing information about corrupted records.

---

# Reading Bad Records

Since Spark stores bad records in JSON format, you can read them like any other JSON file.

```python
bad_data_df = spark.read.format("json") \
    .load("/FileStore/tables/bad_Records_Employee/.../bad_records/")

bad_data_df.show(truncate=False)
```

This helps in:

- Finding data quality issues
- Debugging pipelines
- Fixing source data
- Auditing bad records

---

# Interview Questions

### What is a corrupted record?

A record that cannot be parsed correctly according to the expected schema.

---

### What is the default read mode in Spark?

**PERMISSIVE**

---

### Which mode removes corrupted rows?

**DROPMALFORMED**

---

### Which mode immediately fails the job?

**FAILFAST**

---

### What is `badRecordsPath`?

It specifies the directory where Spark stores corrupted records for later analysis.

---

### What is `_corrupt_record`?

A special column that stores the original malformed record when using a schema designed to capture corrupted data.

---

# Best Practices

- ✅ Use **PERMISSIVE** while developing pipelines.
- ✅ Use **FAILFAST** when data quality is critical.
- ✅ Store bad records using `badRecordsPath`.
- ✅ Review corrupted records regularly.
- ✅ Define an explicit schema instead of relying on schema inference.

---

# Summary

- Corrupted records are rows that don't match the expected schema.
- PySpark provides three read modes:
  - **PERMISSIVE** → Keeps records and replaces invalid values with `NULL`.
  - **DROPMALFORMED** → Skips corrupted rows.
  - **FAILFAST** → Stops processing immediately.
- Use `_corrupt_record` to capture malformed rows.
- Use `badRecordsPath` to store bad records for debugging and auditing.
- Choosing the right read mode depends on your application's data quality requirements.

---

## Happy Learning! 🚀

If you found this guide helpful, consider giving the repository a ⭐ and following for more beginner-friendly PySpark content.