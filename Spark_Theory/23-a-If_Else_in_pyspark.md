# Mastering Conditional Statements in PySpark: Using `when()`, `otherwise()`, and SQL `CASE WHEN`

If you've worked with SQL before, you're probably familiar with the
`CASE WHEN` statement. In PySpark, the equivalent functionality is
achieved using the `when()` and `otherwise()` functions.

## Why Do We Need Conditional Logic?

Conditional transformations are used to:

-   Categorize customers or employees
-   Handle null values
-   Assign grades or labels
-   Apply business rules
-   Create derived columns

## Import Required Libraries

``` python
from pyspark.sql.functions import col, when, lit
from pyspark.sql.types import StructType, StructField
from pyspark.sql.types import IntegerType, StringType
```

## Create Sample DataFrame

``` python
employee_schema = StructType([
    StructField("Id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", IntegerType(), True),
    StructField("address", StringType(), True),
    StructField("gender", StringType(), True)
])
```

## Basic `when()` Example

``` python
Employee_df.withColumn(
    "adult",
    when(col("age") < 18, "No")
    .when(col("age") > 18, "Yes")
    .otherwise("NoValue")
).show()
```

### Explanation

-   Age \< 18 → No
-   Age \> 18 → Yes
-   Otherwise → NoValue

------------------------------------------------------------------------

## Handling Null Values

``` python
Employee_df.withColumn(
    "age",
    when(col("age").isNull(), lit(19))
    .otherwise(col("age"))
)
```

`lit()` is used to insert a constant value into a DataFrame.

------------------------------------------------------------------------

## Multiple Conditions

``` python
Employee_df.withColumn(
    "age_wise",
    when((col("age") > 0) & (col("age") < 18), "Minor")
    .when((col("age") > 18) & (col("age") < 30), "Mid")
    .otherwise("Major")
)
```

------------------------------------------------------------------------

## Logical Operators

  Operator   Meaning
  ---------- ---------
  `&`        AND
  `|`        OR
  `~`        NOT

Example:

``` python
when(
    (col("salary") > 50000) &
    (col("age") > 25),
    "Eligible"
)
```

------------------------------------------------------------------------

## SQL Equivalent

``` sql
SELECT *,
CASE
    WHEN age < 18 THEN 'Minor'
    WHEN age > 18 THEN 'Major'
    ELSE 'NoValue'
END AS adult
FROM employee_tbl;
```

------------------------------------------------------------------------

## Best Practices

-   Always use `otherwise()`
-   Handle null values before applying conditions
-   Write readable chained conditions
-   Use meaningful column names

------------------------------------------------------------------------

## Conclusion

The `when()` and `otherwise()` functions are essential for implementing
business logic in PySpark. Whether you're categorizing data, handling
missing values, or translating SQL `CASE WHEN` statements into the
DataFrame API, mastering these functions will make your ETL pipelines
more robust and maintainable.
