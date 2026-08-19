# Spark Deployment Mode and Edge Node 

## Understanding Deployment Mode

When you run a Spark application, there are two important questions:

1.  Where does the Spark application run?
2.  Where does the Driver program run?

Deployment mode answers the second question.

There are two deployment modes:

-   Client Mode
-   Cluster Mode

> **Remember:** Deployment mode decides **only where the Driver runs**,
> not the Executors.

------------------------------------------------------------------------

# Spark Architecture

``` text
                    Spark Application

                  +----------------+
                  |     Driver     |
                  +----------------+
                     /    |     \
                    /     |      \
             Task 1   Task 2   Task 3
                 |        |        |
          +----------+ +----------+ +----------+
          | Executor | | Executor | | Executor |
          +----------+ +----------+ +----------+
```

The Driver creates tasks and sends them to Executors.

## Driver

The Driver is the brain of a Spark application.

Responsibilities:

-   Creates SparkSession
-   Reads your code
-   Creates the DAG
-   Divides jobs into stages
-   Divides stages into tasks
-   Sends tasks to Executors
-   Collects results

## Executors

Executors are worker processes.

Responsibilities:

-   Execute tasks
-   Store data in memory
-   Return results to the Driver

------------------------------------------------------------------------

# Client Mode

In Client Mode, the Driver runs on the same machine where you execute
`spark-submit`.

``` bash
spark-submit \
--master yarn \
--deploy-mode client \
app.py
```

``` text
Laptop / Edge Node
+----------------+
|     Driver     |
+----------------+
        |
------------------------------
 Executor   Executor   Executor
```

## Workflow

1.  Execute `spark-submit`.
2.  Driver starts on your machine.
3.  Driver requests resources.
4.  Cluster starts Executors.
5.  Driver sends tasks to Executors.

### Advantages

-   Easy debugging
-   Local Driver logs
-   Best for development

### Disadvantages

-   If the client machine disconnects, the application stops.

------------------------------------------------------------------------

# Cluster Mode

In Cluster Mode, the Driver runs inside the cluster.

``` bash
spark-submit \
--master yarn \
--deploy-mode cluster \
app.py
```

``` text
Laptop / Edge Node
       |
   spark-submit
       |
------------------------------
 Driver
   |
 Executor  Executor  Executor
```

## Workflow

1.  Submit the application.
2.  Cluster creates the Driver.
3.  Driver requests Executors.
4.  Executors process data.
5.  Driver collects results.

### Advantages

-   Production ready
-   Client can disconnect
-   Better reliability

### Disadvantages

-   Driver logs remain in the cluster.
-   Debugging is harder.

------------------------------------------------------------------------

# Client vs Cluster

  Feature           Client Mode                  Cluster Mode
  ----------------- ---------------------------- --------------
  Driver Location   Client machine / Edge Node   Cluster
  Executors         Cluster                      Cluster
  Best For          Development                  Production
  Debugging         Easy                         Moderate
  Reliability       Lower                        Higher

------------------------------------------------------------------------

# What is an Edge Node?

An **Edge Node** is a gateway machine outside the Hadoop/Spark cluster
that users connect to for submitting jobs.

``` text
Developers
     |
+-------------+
|  Edge Node  |
+-------------+
      |
-----------------------------
 Hadoop / Spark Cluster
```

## Uses

-   Write Spark code
-   Run `spark-submit`
-   Access HDFS
-   Run Hive commands
-   Schedule jobs
-   Debug applications

> Edge Nodes generally do **not** execute Spark tasks.

------------------------------------------------------------------------

# Edge Node in Client Mode

``` text
Developer
    |
 SSH
    |
+-------------+
| Edge Node   |
| Driver      |
+-------------+
      |
--------------------------
Executor Executor Executor
```

------------------------------------------------------------------------

# Edge Node in Cluster Mode

``` text
Developer
    |
 SSH
    |
+-------------+
| Edge Node   |
+-------------+
      |
 spark-submit
      |
--------------------------
Driver
Executor Executor Executor
```

------------------------------------------------------------------------

# Interview Questions

### Does deployment mode decide where Executors run?

No. Deployment mode only decides where the Driver runs.

### Difference between Client and Cluster mode?

-   Client: Driver runs on the client machine.
-   Cluster: Driver runs inside the cluster.

### What is an Edge Node?

A gateway machine used to access the cluster and submit Spark jobs.

------------------------------------------------------------------------

# Quick Revision

-   Client Mode → Driver on client machine / Edge Node.
-   Cluster Mode → Driver inside cluster.
-   Edge Node → Gateway used to submit Spark jobs.
