# 📘 Apache Spark Memory Management 

Apache Spark uses memory efficiently to process large-scale data.

Each Spark application runs using:

- **Driver** → Controls execution  
- **Executors** → Perform actual work  

Each executor has limited memory, so Spark divides memory into different parts:

- **Execution Memory** → Used for joins, aggregations  
- **Storage Memory** → Used for caching data  
- **User Memory** → Used for user-defined data  

⚠️ If memory is not properly managed → **OOM (Out Of Memory)** error occurs.

---

## 📊 Example Setup

Suppose:

- 1 Driver  
- 3 Executors  

Each Executor has:

- **10 GB memory**  
- **4 cores**

### Memory Configuration

```
spark.executor.memory = 10 GB
spark.executor.memoryOverhead = 10% = 1 GB
```

### 🧠 Total Container Memory

```
11 GB
 ├── 10 GB → JVM Memory
 │      └── ~600–700 MB → PySpark Application
 │
 └── 1 GB → Overhead Memory (Non-JVM)
        ├── ~300–400 MB → Container/System
        └── Remaining → Other processes
```

⚠️ If:
- JVM memory exceeds **10 GB** → OOM  
- Overhead exceeds **1 GB** → OOM  

---

## 🔹 Overhead Memory Usage

- Used for **non-JVM processes**
- ~300–400 MB → Container/System
- ~600–700 MB → PySpark application

---

## 🔍 Breakdown of 10 GB JVM Memory

```
10 GB
 └── Reserved Memory = 300 MB (fixed)
```

### 1. Reserved Memory (300 MB)

- Fixed memory
- Used by Spark internal engine
- Stores Spark internal objects

👉 Minimum executor size:
```
300 MB × 1.5 = 450 MB
```

---

### 2. Spark Memory (40% of 9.7 GB ≈ 3880 MB)

This is divided into:

#### 🟢 Storage Memory

- Stores **cached data**
- Stores **intermediate data (joins, shuffle)**
- Uses **LRU (Least Recently Used)** for eviction

#### 🔵 Execution Memory

- Used during **task execution**
- Stores:
  - Hash tables (for joins/aggregations)
- Short-lived (cleared after operations)
- Can **spill to disk**

---

### 3. User Memory (60% of 9.7 GB ≈ 5820 MB)

- Stores:
  - User-defined data structures
  - Spark metadata
  - UDFs
- Mainly used in **RDD operations**

---

## 🔄 Inside Spark Memory (Unified Model)

```
Spark Memory (≈ 5820 MB)

 ├── Storage Pool → 50% (≈ 2910 MB)
 └── Execution Pool → 50% (≈ 2910 MB)
```

---

## ⚙️ Types of Memory Managers

### 1. Static Memory Manager

- Fixed boundary between:
  - Storage Memory
  - Execution Memory

---

### 2. Unified Memory Manager (Modern Spark)

- Dynamic sharing:
  - Execution can borrow from Storage
  - Storage can borrow from Execution

---

## 🧪 Example: `df.cache()`

```python
df.cache()
```

### Step 1: Storage Memory Fills

```
Storage Memory FULL
        ↓
Execution Memory is FREE
        ↓
Storage borrows from Execution
```

✔ Enabled by Unified Memory Manager

---

### Step 2: Execution Starts

- Executor has **4 cores → 4 parallel tasks**
- Assume **2 tasks are running**

👉 Execution memory starts increasing

---

### Step 3: Execution Needs More Memory

```
Execution needs memory
        ↓
Requests Storage Memory
        ↓
Storage frees space
        ↓
Evicts data using LRU
```

✔ Old cached data is removed

---

### Step 4: Memory Reuse

- Execution uses freed memory  
- Process continues dynamically  

---

## ❌ Final Case: OOM

```
Execution needs memory
        ↓
No data left to evict
        ↓
No memory available
        ↓
OOM ❌
```

---

## ❓ Why OOM Happens Even With Spill?

### Scenario:

- Execution Memory ≈ 2.9 GB  
- Performing:
```python
join(df1, df2)
```

- Data is **skewed**

```
One key → 3 GB data
```

---

### 🚨 Problem

- That **single key must be processed together**
- Cannot be split across tasks

Even if Spark spills to disk:

- It still needs full data for that key in memory

👉 Result:

- Cannot fit in memory  
- Cannot process partially  
- Spill is not enough  
➡️ **OOM occurs**

---

## ⚠️ Why Spill Fails Here

Spill works for:

- Sorting  
- Aggregation (partial processing)

❌ But fails when:

- One partition / one key is too large

---

## ✅ Solutions

### 1. Salting

- Break skewed key into smaller keys  
- Distribute load

---

### 2. Repartitioning

- Redistribute data evenly across partitions  

---

## 🎯 Key Takeaways

- Spark memory is **limited and shared**
- Unified Memory allows **dynamic borrowing**
- OOM often happens due to:
  - Data skew
  - Large partitions
- Spill is helpful, but **not a complete solution**
