# Reading CSV Files in PySpark: A Beginner's Guide

If you're starting your journey with **PySpark**, one of the first
things you'll learn is how to read data into a DataFrame. CSV
(Comma-Separated Values) is one of the most common file formats used in
data engineering, so learning to read CSV files is an essential skill.

## What You'll Learn

-   Read a CSV file in PySpark
-   Understand `spark.read`
-   Use `header`, `inferSchema`, and `mode`
-   Print the schema
-   Follow best practices

------------------------------------------------------------------------

# Basic Syntax

``` python
spark.read \
    .format("csv") \
    .option(...) \
    .load("file_path")
```

  Method       Purpose
  ------------ -----------------------------------------
  `format()`   Specifies the file format.<br>
  `option()`   Adds configuration options.<br>
  `schema()`   Defines the schema manually (optional).<br>
  `load()`     Loads the file from the given path.<br>

> **Note:** If `format()` is not specified, Spark assumes the file is in
> **Parquet** format.

------------------------------------------------------------------------

# Reading a CSV File

``` python
FLIGHT_DF = spark.read \
    .format("csv") \
    .option("header","false") \
    .option("inferSchema","false") \
    .option("mode","FAILFAST") \
    .load("/FileStore/tables/2010_summary.csv")
```

Display the first five rows:

``` python
FLIGHT_DF.show(5)
```

## Understanding `header`

`header` tells Spark whether the first row contains column names.

### `header = false`

Spark treats the first row as data and creates columns like `_c0`,
`_c1`, `_c2`.

### `header = true`

Spark uses the first row as column names, making the DataFrame easier to
understand.

## Understanding `inferSchema`

By default (`inferSchema=false`), Spark reads every column as a
**string**.

``` python
.option("inferSchema","false")
```

If you use:

``` python
.option("inferSchema","true")
```

Spark automatically detects data types such as `integer`, `double`, and
`boolean`.

Check the schema:

``` python
FLIGHT_DF.printSchema()
```

## Understanding `mode`

### PERMISSIVE (Default)

-   Continues reading even if bad records exist.
-   Invalid values become `null`.

### FAILFAST

-   Stops immediately when a malformed record is found.

### DROPMALFORMED

-   Skips malformed records and loads only valid rows.

## Best Practices

-   Use `header=True` when the file has column names.
-   Use `inferSchema=True` for learning and exploration.
-   Define the schema manually in production for better performance.
-   Always verify the schema using `printSchema()`.
-   Choose the appropriate read mode based on data quality.

## Conclusion

Reading CSV files is one of the most important PySpark skills.
Understanding options like `header`, `inferSchema`, and `mode` helps you
load data correctly and avoid common mistakes. As you gain experience,
you'll be able to read large datasets confidently and efficiently.

---
## Happy Learning! 🚀

If you found this guide helpful, consider giving the repository a ⭐ and following for more beginner-friendly PySpark content.