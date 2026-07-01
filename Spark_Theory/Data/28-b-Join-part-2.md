# Join

## What Is Shuffle in Spark?

Shuffle in Apache Spark means data movement.
When Spark needs to group, join, or aggregate data, it may have to move records across different executors/partitions so that related data ends up together.

- Example:
Suppose we have Two Data Frame:<br>

**DF1 = customer**

| customer_id | customer_name |
|-------------|---------------|
| 101         | Alice         |
| 102         | Bob           |
| 103         | Charlie       |
| 104         | David         |
| 105         | Eva           |

**DF2 = sales**

| customer_id | sales |
|-------------|-------|
| 101         | 500   |
| 102         | 200   |
| 103         | 700   |
| 104         | 300   |
| 105         | 100   |

🔹 Before Shuffle (Data Scattered Across Executors)

Imagine these are split across 4 partitions (2 executors):

Executor 1
   DF1 P1 → (101, Alice), (102, Bob)
   DF2 P1 → (101, 500)

   DF1 P2 → (103, Charlie)
   DF2 P2 → (104, 300)

Executor 2
   DF1 P3 → (104, David)
   DF2 P3 → (102, 200)

   DF1 P4 → (105, Eva)
   DF2 P4 → (103, 700), (105, 100)

   Here, notice:

Customer 101 (Alice) and sales 500 are already together in Executor1.

Customer 102 (Bob) is in Executor1, but sales 200 is in Executor2 → they’re apart.

Customer 103 (Charlie) is in Executor1, but sales 700 is in Executor2 → apart.

So Spark cannot join directly.<br>
🔹 Shuffle Step

Spark reshuffles both DataFrames based on the join key customer_id:

All rows with the same customer_id go to the same new partition.
🔹 After Shuffle (Data Co-located)

Now the data looks like:<br>
Executor 1 <br>
   P1’ → (101, Alice) + (101, 500) <br>
   P2’ → (102, Bob)   + (102, 200) <br>

Executor 2 <br>
   P3’ → (103, Charlie) + (103, 700) <br>
   P4’ → (104, David)   + (104, 300), (105, Eva) + (105, 100)<br>

🔹 Now Join Happens Locally

After shuffle, Spark can safely do the join inside each executor:<br>
Result: <br>
| customer_id | customer_name | sales |
|-------------|---------------|-------|
| 101         | Alice         | 500   |
| 102         | Bob           | 200   |
| 103         | Charlie       | 700   |
| 104         | David         | 300   |
| 105         | Eva           | 100   |

⚡ Key Point:
Shuffle ensures that the customer row and the sales row with the same customer_id end up on the same executor. Without shuffle, the join wouldn’t work because data is scattered.

## When does shuffle happen?**

Shuffle happens in operations like:

- groupByKey()

- reduceByKey()

- join()

- distinct()

- repartition()

## What Are The Join Strategies?

**1. Broadcast Hash Join (BHJ) :**

- How it works: The smaller dataset is "broadcast" to all executors, where it is built into a hash table. <br>
- When to use: Ideal when one of the datasets is small enough to fit into memory on each executor, avoiding shuffles on the larger dataset. <br>
- Configuration: The spark.sql.autoBroadcastJoinThreshold setting determines the maximum size of a DataFrame that can be broadcast. <br>

**2. Shuffle Hash Join (SHJ) :**
- How it works: Both large datasets are shuffled and partitioned by the join key. A hash table is built locally on each partition, and then the join occurs. 
- When to use: When one side is large but still manageable for building a local hash table, and Broadcast Hash Join isn't feasible. 

**3. Sort Merge Join (SMJ) :**
- How it works: Both datasets are shuffled, sorted by the join key, and then merged on the executors. 
- When to use: The default for large datasets when a broadcast isn't possible. It's robust and scalable, handling large tables and various join types (including outer joins). 

**4. Broadcast Nested Loop Join (BNLJ) :**
- How it works: A variation of the broadcast join that is used for non-equi-join conditions. It broadcasts one relation and then performs a nested loop join with the other relation, which is a more expensive process. 
- When to use: Used when there is no equi-condition to join on, and other strategies are not possible. 

**5. Cartesian Product Join (CPJ) :**
- How it works: Combines every row from the first DataFrame with every row from the second DataFrame. 
- When to use: Reserved for when you intend to join every possible pair of rows. It can also be used as a fallback when an equi-condition is missing, though it is generally slow. 






