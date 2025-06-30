## TRANSFORMATION AND ACTION

**Spark**, or more specifically RDD, supports 2 kinds of operations:  
**Transformations** and **Actions**.

---

## WHAT IS TRANSFORMATION

**Functions that return another dataset/RDD from existing dataset/RDD after performing some transformation.**
Ex: Suppose we have a dataset of student now we want t0 select the student whose age is <18: this filter out process is transformation.

---

## WHAT IS ACTIONS

Any operation that does not return a dataset/RDD. Evaluation is executed when an action is taken. Actions trigger the scheduler, which builds a Directed Acyclic Graph (DAG) as a plan of execution.
Ex: .show(), .count()

- Since Spark follows lazy evaluation, no transformation is actually computed until an action is encountered.
- When an action is encountered, Spark creates a new job.

---

## TYPES OF TRANSFORMATIONS

Transformations are of 2 types:

- **Narrow dependency**
- **Wide dependency**

---

### Narrow Dependency Transformation

A narrow transformation is a type of transformation where each input partition contributes to only one output partition. It does not require data shuffling or movement across partitions. Narrow transformations are executed within a single stage without requiring data exchange between nodes.
**Examples:** `map()`, `filter()`, `union()`,select

---

### Wide Dependency Transformation

A wide transformation is a type of transformation where each input partition can contribute to multiple output partitions. It involves shuffling and data movement across partitions, often resulting in a stage boundary in Spark. Wide transformations require coordination and data exchange across nodes.  
**Examples:** `groupByKey()`, `join()`, `repartition()`,distinct etc.

