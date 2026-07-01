## What is OOM in Spark?
OOM (Out Of Memory) happens when Spark cannot allocate enough memory to execute a task.
JVM throws: java.lang.OutOfMemoryError
Can happen on:
Driver
Executor

## Why do we get Driver OOM?
The driver is responsible for:

Collecting results
Maintaining metadata (DAG, stages, tasks)
Running SparkContext

Driver OOM occurs when it tries to hold more data than its allocated memory.

## Common scenarios:
Using collect() on large datasets
Storing large broadcast variables
Too many partitions/tasks metadata
Large query plans

## What is Driver Overhead Memory?

In Spark, memory is divided into:

Driver Memory (spark.driver.memory) :<br>
JVM heap memory<br>
Driver Overhead Memory (spark.driver.memoryOverhead)
Off-heap memory used for:<br>
JVM overhead <br>
Python processes (PySpark)<br>
Native memory<br>
Garbage collection overhead<br>

Default:

Usually 10% of driver memory (minimum ~384MB)

## Common Reasons for Driver OOM ?
1. Using collect()
``` 
df.collect()
```
2. Pulls entire dataset to driver
Biggest cause of OOM

Using toPandas()
```
df.toPandas()
```
Loads full dataset into driver memory (very risky)

3. Large Broadcast Variables
```
broadcast(df)
```
 If broadcast data is too large → driver crash

## How to Handle / Prevent OOM?
1. Avoid collect()
instead use 
df.show()
df.take()
2.Increase driver memory
```
--driver-memory 4g
```
3. Increase driver overhead 
```
--conf spark.driver.memoryOverhead=1g
```
