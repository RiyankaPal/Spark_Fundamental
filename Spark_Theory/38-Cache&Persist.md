# Cache and Persist in Spark (Part 1)

## Introduction
Cache and Persist are Spark performance optimization techniques that prevent recomputation.

## Why Needed
When the same DataFrame is used for multiple actions, Spark recomputes the entire lineage every time unless cached.

```python
df = spark.read.csv("employees.csv", header=True)
filtered_df = df.filter(df.salary > 50000)

filtered_df.show()
filtered_df.count()
filtered_df.write.parquet("/output")
```

Without cache:

```text
Read CSV
  ↓
Filter
  ↓
Execute Action
```

This happens for every action.

## Cache

```python
filtered_df.cache()
```

The first action computes and stores the DataFrame in memory.

Future actions reuse the cached result.

### Tea Analogy

```text
Without Cache:
Make tea every time.

With Cache:
Make tea once
↓
Store in flask
↓
Serve everyone
```

## Spark Without Cache

```text
CSV
 ↓
Read
 ↓
Filter
 ↓
Action

(repeated)
```

## Spark With Cache

```text
CSV
 ↓
Read
 ↓
Filter
 ↓
Cache in Memory
 ├─ show()
 ├─ count()
 └─ write()
```
## Persist

```python
from pyspark import StorageLevel
df.persist(StorageLevel.MEMORY_ONLY)
```

Persist lets you choose storage level.

## Cache vs Persist

| Cache | Persist |
|---|---|
| Memory by default | Memory/Disk/Both |
| Simple | Flexible |

## Storage Levels

### MEMORY_ONLY

```python
df.persist(StorageLevel.MEMORY_ONLY)
```

### MEMORY_AND_DISK

```python
df.persist(StorageLevel.MEMORY_AND_DISK)
```

### DISK_ONLY

```python
df.persist(StorageLevel.DISK_ONLY)
```

### MEMORY_ONLY_SER / MEMORY_AND_DISK_SER

Serialized storage (mainly Scala/Java).

## Lazy Evaluation

```python
df.cache()
```

Nothing happens until:

```python
df.show()
```

Then Spark computes and caches the DataFrame.

## Unpersist

```python
df.unpersist()
```

## Check Cache

```python
df.is_cached
```

## Spark UI

Use the **Storage** tab to monitor cached DataFrames.

## When to Cache

- Reused DataFrames
- Expensive transformations
- Machine learning
- Interactive analysis

## When Not to Cache

- Used only once
- Small datasets
- Low memory

## Best Practices

- Cache only reused datasets.
- Use persist() when custom storage is required.
- Call unpersist().
- Monitor Spark UI.

## Summary

| Feature | Cache | Persist |
|---|---|---|
| Default | Memory | User-defined |
| Disk support | No | Yes |

## Key Takeaways

- Cache avoids recomputation.
- Persist provides storage flexibility.
- Both are lazy and take effect only after an action.
