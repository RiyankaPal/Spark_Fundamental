## What is OOM in Spark?
OOM (Out Of Memory) happens when Spark cannot allocate enough memory to execute a task.<br>
JVM throws: java.lang.OutOfMemoryError<br>
Can happen on:<br>
Driver<br>
Executor<br>

## Why do we get Driver OOM?
The Driver is the main process that coordinates the Spark application. It is responsible for:

- Collecting results returned by executors.
- Maintaining metadata about the application, such as the DAG, stages, tasks, and job information.
- Running the SparkContext/SparkSession and coordinating the overall execution of the Spark job.
- Planning and scheduling tasks and sending them to the executors.

**What Causes Driver OOM?**

Driver OOM (Out Of Memory) occurs when the Driver tries to hold more data in its memory than the memory allocated to it.

## Common scenarios:
- Using collect() on large datasets<br>
- Storing large broadcast variables<br>
- Too many partitions/tasks metadata<br>
- Large query plans<br> 

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

## How to Handle / Prevent OOM?<br>

1. Avoid collect()<br>
instead use :<br>
df.show()<br>
df.take()<br>

2. Increase driver memory
```
--driver-memory 4g
```
3. Increase driver overhead 
```
--conf spark.driver.memoryOverhead=1g
```
