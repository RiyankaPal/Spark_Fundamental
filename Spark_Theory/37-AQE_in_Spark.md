# Adaptive Query Execution (AQE) in Apache Spark


# What is Adaptive Query Execution (AQE)?

**Adaptive Query Execution (AQE)** is one of the most powerful optimization features introduced in **Apache Spark 3.0**.

Normally, Spark creates an execution plan **before** the job starts. This is called a **static execution plan**.

But sometimes Spark does not know the actual size of the data until the job starts running.

AQE solves this problem.

> **Adaptive Query Execution allows Spark to change its execution plan while the job is running based on the actual data statistics.**

Instead of blindly following the original plan, Spark continuously checks the data and improves the execution plan during runtime.

---

# Why was AQE introduced?

Imagine you are planning a road trip.

Before leaving, Google Maps tells you the best route.

But after driving for 20 minutes, an accident happens.

A smart navigation system changes your route.

Spark before AQE was like an old GPS.

It never changed the route.

Spark with AQE is like Google Maps.

It changes the route whenever it finds a better one.

---

# Static Execution vs Adaptive Execution

```text
Without AQE

Query
   │
   ▼
Spark creates execution plan
   │
   ▼
Job starts
   │
   ▼
Spark follows SAME plan till end
```

Even if data becomes smaller or larger...

Even if one partition has huge data...

Even if another join would be faster...

Spark never changes the plan.

---

```text
With AQE

Query
   │
   ▼
Spark creates initial plan
   │
   ▼
Job starts
   │
   ▼
Shuffle completed
   │
   ▼
Spark collects actual statistics
   │
   ▼
Spark changes execution plan
   │
   ▼
Continue execution with better plan
```

---

# Why is AQE needed?

Suppose Spark estimates

```text
Sales table = 1 GB

Customer table = 500 MB
```

Spark decides to perform a **Sort Merge Join**.

But during execution Spark discovers

```text
Sales = 1 GB

Customer = 8 MB
```

Now broadcasting the customer table is much faster.

Without AQE

```text
Still Sort Merge Join
```

With AQE

```text
Switch to Broadcast Join
```

Huge improvement!

---

# When does AQE work?

AQE works **after every shuffle stage**.

This is important.

Spark can only know the exact size of data after shuffle completes.

```text
Read Data
    │
Transformation
    │
Shuffle
    │
Spark now knows

• number of records
• partition size
• actual data size
```

Now AQE decides whether the execution plan should change.

---

# How AQE Works

```text
SQL Query

      │

      ▼

Logical Plan

      │

      ▼

Physical Plan

      │

      ▼

Start Execution

      │

      ▼

Shuffle Completed

      │

      ▼

Collect Runtime Statistics

      │

      ▼

Optimize Again

      │

      ▼

Continue Execution
```

---

# What does AQE optimize?

AQE mainly performs three major optimizations.

```text
                AQE
                 │
      ┌──────────┼───────────┐
      │          │           │
      ▼          ▼           ▼

Coalesce    Join Strategy   Handle
Partitions   Selection      Skew
```

Let's understand each one.

---

# 1. Coalescing Shuffle Partitions

## The Problem

Suppose Spark creates

```text
200 shuffle partitions
```

But the data is very small.

Example

```text
Partition 1 = 2 MB

Partition 2 = 3 MB

Partition 3 = 1 MB

...

Partition 200 = 2 MB
```

Now Spark has to schedule 200 tasks.

Scheduling many tiny tasks adds overhead.

---

## Without AQE

```text
200 partitions

↓

200 tasks

↓

Lots of scheduling overhead
```

---

## With AQE

Spark combines small partitions.

```text
Before

P1 2MB

P2 3MB

P3 2MB

P4 1MB

P5 2MB

↓

After AQE

P1+P2+P3 = 7MB

P4+P5 = 3MB
```

Now maybe only

```text
40 partitions
```

instead of

```text
200 partitions
```

Result

- Fewer tasks
- Less scheduling
- Better CPU utilization
- Faster execution

---

# Example

Without AQE

```text
200 partitions

↓

200 Tasks

↓

Many tasks finish in milliseconds
```

Most time is wasted in task scheduling.

---

With AQE

```text
40 partitions

↓

40 Tasks

↓

Each task performs more useful work
```

---

# 2. Dynamic Join Strategy

One of AQE's best features.

---

## Spark has several join algorithms.

```text
Broadcast Hash Join

Sort Merge Join

Shuffle Hash Join

Nested Loop Join
```

Spark chooses one before execution.

But estimates can be wrong.

AQE can change the join type during execution.

---

## Example

Orders

```text
500 GB
```

Customers

```text
Estimated = 100 MB
```

Spark chooses

```text
Sort Merge Join
```

During execution Spark finds

```text
Customers = 5 MB
```

Now Spark changes

```text
Sort Merge Join

↓

Broadcast Hash Join
```

Broadcasting 5 MB is much faster.

---

## Diagram

Without AQE

```text
Orders

Customers

↓

Sort Merge Join
```

---

With AQE

```text
Orders

Customers (5 MB)

↓

Broadcast Customer

↓

Broadcast Hash Join
```

Much faster.

---

# 3. Skew Join Optimization

This is probably the most important AQE optimization.

---

# What is Data Skew?

Suppose Spark creates four partitions.

```text
Partition 1 = 100 MB

Partition 2 = 120 MB

Partition 3 = 95 MB

Partition 4 = 7 GB
```

Notice

```text
7 GB
```

One partition is much larger.

This is called **data skew**.

---

## Why is skew bad?

Imagine four workers.

```text
Worker 1

100 MB

Finished

------------

Worker 2

120 MB

Finished

------------

Worker 3

95 MB

Finished

------------

Worker 4

7 GB

Still Running...
```

Three workers sit idle while one worker keeps processing.

Overall job completion waits for the slowest task.

---

## AQE Solution

AQE automatically splits the huge partition.

Instead of

```text
7 GB
```

Spark creates

```text
2 GB

2 GB

1.5 GB

1.5 GB
```

Now multiple executors process them.

```text
Before

7 GB

↓

One Executor
```

After AQE

```text
2 GB

2 GB

1.5 GB

1.5 GB

↓

Four Executors
```

Result

Much faster execution.

---

# AQE Example

Suppose we execute

```python
df1.join(df2, "customer_id")
```

Spark initially decides

```text
Sort Merge Join
```

During execution it discovers

```text
df2 = 8 MB
```

AQE changes to

```text
Broadcast Join
```

No code change required.

Spark does everything automatically.

---

# How to Enable AQE

AQE is enabled by default in modern Spark versions (Spark 3.2+ in most distributions), but you can enable it explicitly.

```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
```

Or when creating a Spark session:

```python
SparkSession.builder \
    .config("spark.sql.adaptive.enabled", "true") \
    .getOrCreate()
```

---

# Important AQE Configurations

### Enable AQE

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

---

### Enable Coalescing Partitions

```python
spark.conf.set(
    "spark.sql.adaptive.coalescePartitions.enabled",
    "true"
)
```

---

### Enable Skew Join Handling

```python
spark.conf.set(
    "spark.sql.adaptive.skewJoin.enabled",
    "true"
)
```

---

### Broadcast Join Threshold

Spark can broadcast tables smaller than the configured threshold.

```python
spark.conf.set(
    "spark.sql.autoBroadcastJoinThreshold",
    "10MB"
)
```

If AQE discovers a table is below this threshold at runtime, it may switch to a Broadcast Hash Join.

---

# Advantages of AQE

- Improves query performance automatically.
- Reduces unnecessary shuffle partitions.
- Chooses better join strategies during execution.
- Handles skewed data automatically.
- Makes better use of cluster resources.
- Requires little or no code change.

---

# Limitations of AQE

- AQE can optimize only after a shuffle stage, because runtime statistics become available then.
- Very small jobs may see little or no benefit.
- AQE cannot fix inefficient application logic (for example, unnecessary wide transformations or poorly written queries).

---

# Real-Life Example

Suppose an e-commerce company joins:

- **Orders** table (500 GB)
- **Customers** table (estimated 200 MB)

Initial plan:

```text
Sort Merge Join
```

During execution, Spark discovers:

```text
Customers = 6 MB
```

AQE changes the plan:

```text
Broadcast Customers

↓

Broadcast Hash Join
```

Later, Spark detects one partition with millions of records due to a very popular customer ID. AQE splits that skewed partition into smaller pieces and processes them in parallel.

Without AQE:
- Slow join
- One executor becomes a bottleneck
- Longer overall runtime

With AQE:
- Faster join
- Better parallelism
- Shorter execution time

---

# AQE Summary

```text
Adaptive Query Execution (AQE)

               │
               ▼
Monitors runtime statistics
               │
               ▼
Optimizes execution plan dynamically
               │
     ┌─────────┼─────────┐
     │         │         │
     ▼         ▼         ▼
Coalesce   Change     Handle
Partitions Join Type   Skew
     │         │         │
     ▼         ▼         ▼
Fewer     Faster     Better
Tasks      Joins     Parallelism
```

---

# Beginner Interview Questions

### 1. What is AQE?

AQE (Adaptive Query Execution) is a Spark optimization feature that modifies the physical execution plan at runtime using actual data statistics collected during execution.

### 2. Why do we need AQE?

Because Spark's initial estimates can be inaccurate. AQE improves performance by adapting the plan based on the real data.

### 3. When does AQE make optimization decisions?

After shuffle stages, when Spark has accurate runtime statistics.

### 4. What are the three main AQE optimizations?

- Coalescing shuffle partitions
- Dynamic join strategy selection
- Skew join optimization

### 5. Can AQE change a Sort Merge Join into a Broadcast Hash Join?

Yes. If Spark discovers at runtime that one side of the join is small enough to broadcast, AQE can change the join strategy automatically.

---

# Key Takeaways

- AQE stands for **Adaptive Query Execution**.
- It was introduced in **Spark 3.0**.
- AQE **changes the execution plan during runtime**, unlike traditional static planning.
- It mainly improves performance through:
  - **Coalescing small shuffle partitions**
  - **Choosing better join strategies**
  - **Handling skewed data automatically**
- AQE is one of the most important Spark SQL performance optimization features and is widely used in production environments.
