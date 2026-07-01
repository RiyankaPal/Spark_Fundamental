## Aggregate Function

- Count :
The count function in Spark can act as both an action and a transformation, depending on how it is used.


✅ **df.count()**
- This is an **action** in Spark.

- It **returns an integer** (the total row count) directly to the driver..

- Execution is triggered immediately → Spark runs the full DAG **(Application → Job → Stages → Tasks)**.

✅ **df.select(count("col"))**

- This is a **transformation.**

- It only builds a new DataFrame plan with the aggregation (count("col")).

- Spark does not execute it right away (lazy evaluation).

- To trigger execution, you must apply an action, e.g.: <br>
  - show()
  - .collect()
  - .write...

👉 **Key Difference:**

- df.count() → **Action**, returns a number.

- df.select(count("col")) → **Transformation**, produces a new DataFrame but needs an action to run.

