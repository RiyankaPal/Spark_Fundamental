# Broadcast Join:
## What Is Broadcast Join?
A Broadcast Join is a join strategy where the smaller dataset is sent (broadcasted) to all executors, and each executor keeps it in memory as a hash table.

This allows the larger dataset to be processed locally on each executor without needing data movement (shuffle).<br>
## How Does Broadcast Join works?
Suppose:
Table A = 1 GB (large table) <br>
Table B = 5 MB (small table) <br>
Step-by-step: <br>
The large table (1 GB) is split into partitions <br>
Example:<br>
1000 MB / 128 MB ≈ 8 partitions<br>
These partitions are distributed across executors<br>
(e.g., 3 executors → partitions distributed among them)<br>
Instead of moving the large data around:<br>
The small table (5 MB) is copied to every executor<br>
Each executor:
Builds an in-memory hash table from the small dataset
Joins it locally with its partition of the large dataset

👉 Result: No shuffle of the large dataset <br>
Who Sends the Broadcast Data? <br>
The Driver node is responsible for broadcasting the small table to all executors. <br>
⚠️ Important Considerations: <br>
The driver must have enough memory to hold the broadcast data. <br>
Example: <br>
Driver memory = 2 GB <br>
Broadcast table = 1 GB <br>
→ Risk of OutOfMemory (OOM) <br>
Data is sent over the network → large broadcast = slower performance <br>
Executors must also have enough memory to: <br>
Store broadcast data
Perform the join. <br>

## Why Do We Use Broadcast Join?

✅ To avoid shuffle<br>
✅ To improve performance when one dataset is small<br>
✅ To reduce network I/O and disk usage<br>

## When to Use Broadcast Join?
One table is very small (typically < 10 MB by default in Spark)<br>
The other table is large<br>
Enough memory is available on:<br>
Driver<br>
Executors<br>

## Difference between broadcast hash join and shuffle hash join?
## 🔄 Key Differences

| Feature | Broadcast Hash Join | Shuffle Hash Join |
|--------|--------------------|-------------------|
| Data Movement | Small table broadcasted | Both tables shuffled |
| Shuffle | ❌ No | ✅ Yes |
| Performance | Faster | Slower |
| Memory Usage | High (on executors) | Moderate |
| Driver Dependency | Yes | No |
| Best Use Case | One small + one large table | Both tables medium/large |

---

## 📌 Summary

- **Broadcast Hash Join** is best when one dataset is small enough to fit in memory.
- **Shuffle Hash Join** is used when both datasets are too large to broadcast.
