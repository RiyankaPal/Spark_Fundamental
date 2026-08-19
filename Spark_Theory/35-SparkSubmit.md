# Spark Submit 

## What is `spark-submit`?

`spark-submit` is the command-line tool used to run a Spark application.

It submits your Spark application (Python, Scala, or Java) to a Spark cluster or to your local machine.

Without `spark-submit`, your Spark application will not execute in a Spark environment.

---

## Basic Syntax

```bash
spark-submit [options] application_file.py
```

### Example

```bash
spark-submit word_count.py
```

---

## General Syntax

```bash
spark-submit \
    --master <master-url> \
    --deploy-mode <client|cluster> \
    --name <application-name> \
    --driver-memory <memory> \
    --executor-memory <memory> \
    --num-executors <number> \
    application.py
```

---

# Common Options

## 1. `--master`

### Purpose

Specifies where the Spark application should run.

### Syntax

```bash
--master <master-url>
```

### Examples

Run on local machine using one core:

```bash
spark-submit --master local app.py
```

Run on local machine using four cores:

```bash
spark-submit --master local[4] app.py
```

Run on local machine using all available CPU cores:

```bash
spark-submit --master local[*] app.py
```

### Common Values

| Value | Meaning |
|------|---------|
| `local` | One CPU core |
| `local[2]` | Two CPU cores |
| `local[4]` | Four CPU cores |
| `local[*]` | All available CPU cores |
| `yarn` | Hadoop YARN cluster |
| `k8s://...` | Kubernetes cluster |

---

## 2. `--deploy-mode`

### Purpose

Determines where the Driver program runs.

### Syntax

```bash
--deploy-mode client
```

or

```bash
--deploy-mode cluster
```

### Client Mode

- Driver runs on your local machine.
- Easy to debug.
- Mostly used while learning.

Example:

```bash
spark-submit \
    --master local[*] \
    --deploy-mode client \
    app.py
```

### Cluster Mode

- Driver runs inside the Spark cluster.
- Used for production jobs.

---

## 3. `--name`

### Purpose

Assigns a name to your Spark application.

### Syntax

```bash
--name MyApplication
```

### Example

```bash
spark-submit \
    --name SalesAnalysis \
    app.py
```

The name appears in the Spark UI.

---

## 4. `--driver-memory`

### Purpose

Allocates memory to the Driver process.

### Example

```bash
spark-submit \
    --driver-memory 2G \
    app.py
```

Common values:

- 1G
- 2G
- 4G
- 8G

---

## 5. `--executor-memory`

### Purpose

Allocates memory to each Executor.

### Example

```bash
spark-submit \
    --executor-memory 4G \
    app.py
```

If Spark creates four executors and each has 4 GB memory,

Total executor memory = 16 GB

---

## 6. `--num-executors`

### Purpose

Specifies the number of executors Spark should create.

Example

```bash
spark-submit \
    --num-executors 4 \
    app.py
```

Spark will create four executors.

---

# Complete Example

```bash
spark-submit \
    --master local[*] \
    --deploy-mode client \
    --name EmployeeAnalysis \
    --driver-memory 2G \
    --executor-memory 4G \
    employee_analysis.py
```

---

# What Happens Internally?

When you execute

```bash
spark-submit app.py
```

Spark performs the following steps:

1. Starts the Driver program.
2. Creates a SparkSession.
3. Requests resources.
4. Starts Executors.
5. Divides the job into stages.
6. Divides stages into tasks.
7. Sends tasks to Executors.
8. Executors process the data.
9. Results are returned to the Driver.
10. Driver displays or saves the output.

## When Should You Use It?

-   Running PySpark scripts
-   Running Spark SQL jobs
-   Scheduling ETL pipelines
-   Production batch jobs

---

# Client Mode vs Cluster Mode

| Feature | Client Mode | Cluster Mode |
|---------|-------------|--------------|
| Driver Location | Local Machine | Spark Cluster |
| Best For | Development | Production |
| Debugging | Easy | More Difficult |
| Performance | Good | Better |

---

# Frequently Used Commands

Run normally:

```bash
spark-submit app.py
```

Run locally:

```bash
spark-submit --master local[*] app.py
```

Give application name:

```bash
spark-submit \
    --name MyApp \
    app.py
```

Increase Driver memory:

```bash
spark-submit \
    --driver-memory 4G \
    app.py
```

Increase Executor memory:

```bash
spark-submit \
    --executor-memory 8G \
    app.py
```

Create four executors:

```bash
spark-submit \
    --num-executors 4 \
    app.py
```

---

# Tips for Beginners

- Always start with `local[*]` while learning.
- Use **client** mode for development.
- Give meaningful application names.
- Increase memory only when required.
- Read Spark error messages carefully—they often identify the failing stage.

---

# Summary

- `spark-submit` is the standard command used to execute Spark applications.
- It can run applications locally or on a cluster.
- Important options include:
  - `--master`
  - `--deploy-mode`
  - `--name`
  - `--driver-memory`
  - `--executor-memory`
  - `--num-executors`
- Learn to use these options before moving to production Spark clusters.

---

# Quick Revision

- `spark-submit` → Runs a Spark application.
- `--master` → Specifies where Spark runs.
- `--deploy-mode` → Specifies where the Driver runs.
- `--name` → Sets the application name.
- `--driver-memory` → Memory allocated to the Driver.
- `--executor-memory` → Memory allocated to each Executor.
- `--num-executors` → Number of Executors to create.



























































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































