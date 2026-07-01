# Understanding Joins in Apache Spark (Beginner-Friendly Guide)

When working with Apache Spark, you'll often combine data from multiple
DataFrames. This process is called a **join**.

One common question beginners ask is:

> **How can Spark join data that is stored across multiple machines?**

The answer is **Shuffle**.

------------------------------------------------------------------------

# What is Shuffle in Spark?

**Shuffle** is the process of **moving data across partitions and
executors** so that rows with the same key are placed together.

Spark stores data in multiple partitions distributed across executors.
Before performing operations like **join**, **groupBy**, or
**aggregation**, Spark may need to redistribute the data.

Think of shuffle as:

> **Rearranging data so related records are in the same place.**

## Example

### Customer DataFrame

| customer_id | customer_name |
|-------------|---------------|
| 101         | Alice         |
| 102         | Bob           |
| 103         | Charlie       |
| 104         | David         |
| 105         | Eva           |

### Sales DataFrame

| customer_id | sales |
|-------------|-------|
| 101         | 500   |
| 102         | 200   |
| 103         | 700   |
| 104         | 300   |
| 105         | 100   |

``` python
customer.join(sales, "customer_id")
```

## Before Shuffle

    Executor 1
    Customer
    101 Alice
    102 Bob
    103 Charlie

    Sales
    101 500
    104 300

    Executor 2
    Customer
    104 David
    105 Eva

    Sales
    102 200
    103 700
    105 100

Spark cannot directly join rows whose matching keys are on different
executors.

## Shuffle

Spark redistributes both DataFrames using the join key (`customer_id`)
so that all rows with the same key end up in the same partition.

## After Shuffle

    Executor 1
    101 Alice      101 500
    102 Bob        102 200

    Executor 2
    103 Charlie    103 700
    104 David      104 300
    105 Eva        105 100

Now Spark performs the join locally.

## Result

 | customer_id |  customer_name |    sales|
 |-------------| ---------------| --------|
 |101          | Alice          |     500 |
 |102          | Bob            |     200 |
 |103          | Charlie        |     700 |
 |104          | David          |     300 |
 |105          | Eva            |     100 |

## Why is Shuffle Expensive?

-   Network data transfer
-   Disk I/O
-   Sorting
-   Repartitioning

## When Does Shuffle Happen?

-   `join()`
-   `groupBy()`
-   `groupByKey()`
-   `reduceByKey()`
-   `distinct()`
-   `repartition()`
-   `orderBy()`
-   `sort()`

# Join Strategies

## 1. Broadcast Hash Join (BHJ)

-   Broadcasts the small table to every executor.
-   No shuffle for the large table.
-   Best when one table is small enough to fit into executor memory.
-   Controlled by `spark.sql.autoBroadcastJoinThreshold`.

## 2. Shuffle Hash Join (SHJ)

-   Shuffles both DataFrames.
-   Builds a hash table locally in each partition.
-   Best for medium-sized datasets.

## 3. Sort Merge Join (SMJ)

-   Default strategy for large joins.
-   Shuffles both DataFrames.
-   Sorts by join key.
-   Merges sorted partitions.
-   Supports large datasets and all common join types.

## 4. Broadcast Nested Loop Join (BNLJ)

-   Used mainly for non-equi joins.
-   Broadcasts one table.
-   Compares rows using a nested loop.
-   More expensive than hash joins.

## 5. Cartesian Product (Cross Join)

-   Every row from the first table is paired with every row from the
    second table.
-   Produces `Rows(A) × Rows(B)` output rows.
-   Should only be used when intentionally required.

# Summary

  Join Strategy                Shuffle     Best For
  ---------------------------- ----------- ----------------------------------
  Broadcast Hash Join          No          Small lookup table + large table
  Shuffle Hash Join            Yes         Medium datasets
  Sort Merge Join              Yes         Large datasets
  Broadcast Nested Loop Join   Sometimes   Non-equi joins
  Cartesian Product            Depends     Cross joins

# Key Takeaways

-   Shuffle moves data so matching keys are together.
-   Broadcast Hash Join is usually the fastest when one table is small.
-   Sort Merge Join is Spark's default strategy for large joins.
-   Shuffle is expensive because it involves network transfer, disk I/O,
    and sorting.
