# Broadcast Join:
## What Is Broadcast Join?
The smaller dataset is "broadcast" to all executors, where it is built into a hash table.

## How Does Broadcast Join works?
Suppose we have 2 tables one table size is 1 GB and another one 5 MB (Small).Now 1000/128 MB=8 partition.
So in master slave architecture suppose in 3 executor these 8 partition will divide. Now suppose small table one(5mb) is in any one executor so, to join for other executor data needs to move to avoid this much shiffle we will keep this small one in every executor.

Now who send this data? driver right? so driver needs memory so that it can store the file, I f driver has 2 Gb memory anfd file size is 1 GB then only 1 gb memory will left, then out of memory can happen.
now suppose then also it send the file ,data send through n/w, join will slow  & if executor's memory will ful then there will no space for join.So, we need to define the size of table, acording to our cluster.
## Why Do we need Broadcast Join?
To avoid shuffle.
## Difference between broadcast hash join and shuffle hash join?
