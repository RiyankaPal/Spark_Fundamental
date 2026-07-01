# Dataframe Transformation:

## Introduction
PySpark DataFrames are the foundation of data processing in Apache Spark. Understanding schemas, file reading, column operations, filtering, and SQL queries is essential for every Data Engineer.

## 1. Creating a Schema in PySpark

```python
from pyspark.sql.types import StructType, StructField
from pyspark.sql.types import IntegerType, StringType, DoubleType

employee_schema = StructType([
    StructField("Id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", DoubleType(), True),
    StructField("address", StringType(), True)
])
```

### Benefits
- Better performance
- Data validation
- Prevents incorrect datatype inference

## 2. Reading CSV Files with Schema

```python
Employee_df = spark.read.option("header", True) \
    .schema(employee_schema) \
    .csv("employee_write_data.csv")
```

## 3. Selecting Columns

```python
Employee_df.select("name").show()
Employee_df.select("name", "age", "Id").show()
```

## 4. Using expr()

```python
Employee_df.select(expr("id + 5")).show()
```

## 5. Spark SQL

```python
Employee_df.createOrReplaceTempView("employee_tbl")
spark.sql("SELECT * FROM employee_tbl").show()
```

## 6. Aliases

```python
Employee_df.select(col("id").alias("employee_id"), "name").show()
```

## 7. Filtering Records

```python
Employee_df.filter(col("salary") > 150000).show()
Employee_df.where(col("salary") > 150000).show()
```

## 8. Adding Literal Values

```python
Employee_df.select("*", lit("Kumar").alias("Last_Name")).show()
```

## 9. Adding New Columns

```python
Employee_df.withColumn("Sur_name", lit("singh")).show()
```

## 10. Renaming Columns

```python
Employee_df.withColumnRenamed("id", "Employee_Id").show()
```

## 11. Casting Data Types

```python
Employee_df.withColumn("id", col("id").cast("string"))
```

## 12. Dropping Columns

```python
Employee_df.drop("id", col("name")).show()
```

## Conclusion

These DataFrame operations form the foundation of PySpark development and are frequently used in real-world data engineering projects.
