# Apache Spark Joins for Beginners

## Introduction

Joins are used to combine data from two DataFrames based on a common column.

This guide explains the joins demonstrated in the notebook.


### Code
```python
from pyspark.sql.types import *
from pyspark.sql.functions import *
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


### Code
```python
customer_data = [(1,'manish','patna',"30-05-2022"),
(2,'vikash','kolkata',"12-03-2023"),
(3,'nikita','delhi',"25-06-2023"),
(4,'rahul','ranchi',"24-03-2023"),
(5,'mahesh','jaipur',"22-03-2023"),
(6,'prantosh','kolkata',"18-10-2022"),
(7,'raman','patna',"30-12-2022"),
(8,'prakash','ranchi',"24-02-2023"),
(9,'ragini','kolkata',"03-03-2023"),
(10,'raushan','jaipur',"05-02-2023")]

customer_schema=['customer_id','customer_name','address','date_of_joining']
customer_df = spark.createDataFrame(data=customer_data,schema=customer_schema)


sales_data = [(1,22,10,"01-06-2022"),
(1,27,5,"03-02-2023"),
(2,5,3,"01-06-2023"),
(5,22,1,"22-03-2023"),
(7,22,4,"03-02-2023"),
(9,5,6,"03-03-2023"),
(2,1,12,"15-06-2023"),
(1,56,2,"25-06-2023"),
(5,12,5,"15-04-2023"),
(11,12,76,"12-03-2023")]

sales_schema=['customer_id','product_id','quantity','date_of_purchase']
sales_df = spark.createDataFrame(data=sales_data,schema=sales_schema)



product_data = [(1, 'fanta',20),
(2, 'dew',22),
(5, 'sprite',40),
(7, 'redbull',100),
(12,'mazza',45),
(22,'coke',27),
(25,'limca',21),
(27,'pepsi',14),
(56,'sting',10)]

product_schema=['id','name','price']
product_df = spark.createDataFrame(data=product_data,schema=product_schema)
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 1

# Inner Join


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"inner")\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"inner")\
    .select(sales_df["customer_id"]).show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"inner")\
    .select(sales_df["product_id"]).sort("product_id").show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


### Code
```python
product_df.show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 2

## left Join


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"left")\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 3

## Right Join


### Code
```python
sales_df.join(product_df,sales_df["product_id"]==product_df["id"],"right")\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 4

## Full Outer Join


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"outer")\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 5

## Left semi Join


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"left_semi")\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 6

## Left Anti Join


### Code
```python
customer_df.join(sales_df,sales_df["customer_id"]==customer_df["customer_id"],"left_anti")\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 7

## Cross Join


### Code
```python
customer_df.crossJoin(sales_df)\
    .show()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


### Code
```python
customer_df.crossJoin(sales_df)\
    .count()
```

**Explanation:** Run the above code in Spark and observe how the join type affects the output.


## Topic 8

##


## Common Join Types

- **Inner Join**: Returns matching rows from both DataFrames.
- **Left Join**: Returns all rows from the left DataFrame and matching rows from the right.
- **Right Join**: Returns all rows from the right DataFrame and matching rows from the left.
- **Full Outer Join**: Returns all rows from both DataFrames.
- **Left Semi Join**: Returns only matching rows from the left DataFrame.
- **Left Anti Join**: Returns only non-matching rows from the left DataFrame.
- **Cross Join**: Produces every possible combination of rows.

## Key Takeaways

- Choose the join type based on the result you need.
- Always identify the common key column.
- Avoid unnecessary cross joins because they create very large outputs.
