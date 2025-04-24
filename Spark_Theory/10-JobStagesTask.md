## APPLICATION, JOB, STAGE, TASK
# 1. APPLICATION:
- A Spark Application is the complete user program.

- It includes the driver process and a set of executors on the cluster.

- A single application might read data, process it, and write results.

- Example: A PySpark script to clean, transform, and write data to a table.

# 2. JOB:
- A job is triggered each time you call an action on a DataFrame/RDD.

- Actions: .collect(), .count(), .save(), .take(), etc.

- The job is divided into stages by the DAG (Directed Acyclic Graph) scheduler.

- Example: Calling .count() on a DataFrame launches a job.

# 3. STAGE
- A stage is a logical division of a job's computation, corresponding to a sequence of transformations that can be executed without shuffling data across the network.<br>

- Stages are determined by the presence of shuffle operations (e.g., reduceByKey, groupByKey, sortByKey) or data partitioning operations (e.g., repartition, partitionBy).

- Stages are further divided into tasks for actual execution.

# 4. TASK:
- A task is the smallest unit of work in Spark and represents the actual execution of a computation on a single partition of data.
- Each task corresponds to a single partition of an RDD and performs the transformations defined in the stage it belongs to.
- Tasks are executed in parallel across the worker nodes in the Spark cluster.
- Tasks are created for each partition of data in the RDD being operated on within a stage.
