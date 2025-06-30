## RDD, DATAFRAME AND DATASET:

# WHAT IS Resilient Distributed Dataset(RDD)?
In Apache Spark, a Resilient Distributed Dataset (RDD) is the fundamental data structure and the core abstraction. It's a collection of immutable data elements, partitioned across a cluster of machines, enabling parallel processing. RDDs are fault-tolerant, meaning they can be reconstructed if data is lost due to node failures. 

# Characteristics Of RDD:
- **Immutable:**
Once an RDD is created, it cannot be modified. New RDDs are created by performing transformations on existing RDDs. 
- **Distributed:**
RDDs are automatically partitioned and distributed across the nodes of a Spark cluster, allowing for parallel processing. 
- **Fault-Tolerant:**
Spark keeps track of the lineage of RDDs, allowing it to reconstruct lost partitions if a node fails. 
- **Lazy Evaluation:**
RDD transformations are not executed immediately but are cached until an action is performed, improving efficiency. 
- **Versatile:**
RDDs can hold any type of Python, Java, or Scala objects, including user-defined classes. 
# Common RDD Operations In PySpark:

 - RDD Supports Two Types of operation:
  1. Transformations
  2. Actions

# When to use RDD?

- Full control over data partitioning.
- Low-level transformations without the need for schema enforcement.
- To handle unstructured or semi-structured data.

----
# 2. What Is DataFrame?
A DataFrame is a distributed collection of data organized into named columns. It’s similar to a table in a relational database or a data frame in Python’s Pandas library. DataFrames are optimized for performance and provide a higher-level API for working with structured data, making it easier to handle complex data processing tasks.
# Characteristics Of DataFrame:
- **Optimized for Performance:** DataFrames utilize Spark’s Catalyst optimizer, which improves execution plans for better performance.
- **Schema Support:** DataFrames have an associated schema, making it easy to enforce column names and data types.
- **Integration with Spark SQL:** DataFrames can be queried using SQL syntax, making them accessible to users familiar with SQL.
# Common DataFrame Operations In PySpark:
DataFrames support a wide range of operations, from selecting and filtering data to aggregations and joins.
# When To Use DataFrame:
- Work with structured or semi-structured data.
- Require schema enforcement for tabular data.
- Perform SQL-like queries or aggregations.
---
## Dataset:
A Dataset is a distributed collection of data that provides the benefits of both RDDs and DataFrames. Datasets are primarily available in Scala and Java and offer strong typing, which makes them suitable for complex data workflows that benefit from compile-time type safety. They provide the advantages of RDDs’ functional transformations and the optimizations available in DataFrames.

# Characteristics Of DataFrame:
- Works for all kinds of dataset: Structured, Semi-Structured, Unstructured<br>
- Optimization is applied using the Catalyst Optimizer.<br>
- Compile time safety available as it is strongly typed API.<br>
# When to use RDD?

- Developers using Spark with Scala or Java.
- Applications that benefit from strong typing and type safety.
- Handling structured data with schema enforcement.





