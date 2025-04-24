## Spark Context vs Spark Session:
# Spark Context:
SparkContext is the gateway to the Spark cluster. It represents the connection to a Spark cluster and is responsible for coordinating the execution of tasks across it.

# 🧠 Key Responsibilities:
Initializes the Spark application.

Connects to the cluster manager (like YARN, Mesos, or Standalone).

Manages job scheduling and task distribution.

Creates and manages RDDs (Resilient Distributed Datasets)—Spark’s core abstraction for distributed data.
# Spark Session:
With Spark 2.0, SparkSession was introduced to simplify and unify how developers interact with Spark.

🔑 It is now the primary entry point for working with structured data like DataFrames and Datasets, while also encapsulating the functionality of SparkContext. You no longer need to create a separate SparkContext when using SparkSession.
# Benefits:
- Provides a unified API for working with structured data.

- Supports various data sources: CSV, JSON, Parquet, Hive, JDBC, etc.

- Enables SQL queries, DataFrame operations, and Dataset manipulations.

Simplifies the programming model by merging SQLContext, HiveContext, and SparkContext into one.

#  Dataset Concept:
SparkSession also introduces the Dataset API, a distributed collection of data that is strongly typed (in Scala/Java) and combines the benefits of RDDs (type safety) and DataFrames (optimized execution).
# 🔁 How They Relate
Even though you use SparkSession in modern Spark applications, it manages the underlying SparkContext internally. If needed, you can still access it via:
```python
spark.sparkContext
```
So technically, every SparkSession includes a SparkContext, but with extended capabilities.

## Quick Comparison Table

| Feature               | SparkContext                         | SparkSession                                 |
|-----------------------|---------------------------------------|-----------------------------------------------|
| Introduced In         | Spark 1.x                             | Spark 2.0+                                    |
| Entry Point For       | Core Spark functionality, RDDs        | Structured data, SQL, DataFrames, Datasets   |
| Data Abstractions     | RDD                                   | DataFrame, Dataset, SQL                       |
| Data Source Support   | Limited, manual                       | Broad, built-in connectors                    |
| SQL Support           | Needs SQLContext/HiveContext          | Built-in                                      |
| Recommended For       | Low-level transformations, legacy code| Modern structured data applications           |
