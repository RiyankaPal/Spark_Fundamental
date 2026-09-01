# Reading JSON Files in Apache Spark: A Practical Guide with Examples

JSON is one of the most widely used data formats for storing and
exchanging data. In real-world data engineering projects, JSON files
often come in different structures---line-delimited JSON, multiline
JSON, nested JSON, and sometimes even corrupted JSON files.

In this article, we'll explore how Apache Spark reads different types of
JSON files and how various Spark options help handle these scenarios
efficiently.

## 1. Reading a Line-Delimited JSON File

A line-delimited JSON file contains one JSON object per line.

### Sample Data

``` json
{"name":"Manish","age":20,"salary":20000}
{"name":"Nikita","age":25,"salary":21000}
{"name":"Pritam","age":16,"salary":22000}
{"name":"Prantosh","age":35,"salary":25000}
{"name":"Vikash","age":67,"salary":40000}
```

### Spark Code

``` python
spark.read.format("json") \
    .option("inferSchema","True") \
    .option("mode","PERMISSIVE") \
    .load("/FileStore/tables/name_data.json") \
    .show()
```

### Explanation

-   `format("json")` tells Spark to read JSON files.
-   `inferSchema=True` automatically detects column data types.
-   `mode="PERMISSIVE"` handles malformed records gracefully.

Spark automatically converts JSON keys into DataFrame columns and loads
the data efficiently.

## 2. Handling JSON Files with Additional Fields

When records have different fields, Spark automatically creates missing
columns and fills unavailable values with `null`.

``` json
{"name":"Vikash","age":67,"salary":40000,"gender":"M"}
```

Use the same reader configuration.

## 3. Reading Multiline JSON Files

Use:

``` python
.option("multiline","True")
```

when the JSON file is stored as a formatted JSON array spanning multiple
lines.

## 4. Line-Delimited JSON vs Multiline JSON


| Line-Delimited JSON | Multiline JSON |
|---|---|
| One record per line | Entire JSON document |
| Better parallelism | More parsing overhead |
| Faster for big data processing | Slower for large datasets |

**Recommendation:** Prefer line-delimited JSON (JSONL/NDJSON) for
distributed workloads.

## 5. Reading Incorrect Multiline JSON

Malformed multiline JSON may be treated as corrupt or parsed
unexpectedly. Always validate JSON before production ingestion.

## 6. Handling Corrupted JSON Files

When reading JSON files, Spark can handle corrupted or malformed records using the `PERMISSIVE` mode.

```python
.option("mode", "PERMISSIVE")
```

With `PERMISSIVE` mode, Spark:

- Loads valid records successfully.
- Stores invalid or corrupted records in the `_corrupt_record` column.
- Prevents the pipeline from failing because of corrupted records.

### Example

```python
df = spark.read \
    .option("mode", "PERMISSIVE") \
    .json("input.json")
```

This allows the pipeline to continue processing valid records while keeping track of corrupted records separately.

## 7. Reading Nested JSON Files

Use:

``` python
spark.read.format("json") \
    .option("inferSchema","True") \
    .option("mode","PERMISSIVE") \
    .load("/FileStore/tables/resturant_json_data.json") \
    .printSchema()
```

`printSchema()` displays nested structs, arrays, and data types.

## Conclusion

Apache Spark provides flexible support for line-delimited, multiline,
nested, and corrupted JSON files, making it well suited for real-world
data engineering pipelines.

## Key Takeaways

-   ✅ Prefer line-delimited JSON for better performance.
-   ✅ Enable `multiline=True` for formatted JSON arrays.
-   ✅ Use `PERMISSIVE` mode to handle malformed records.
-   ✅ Use `printSchema()` to inspect nested JSON.
-   ✅ Use `inferSchema=True` when exploring new datasets.
---
## Happy Learning! 🚀

If you found this guide helpful, consider giving the repository a ⭐ and following for more beginner-friendly PySpark content.
