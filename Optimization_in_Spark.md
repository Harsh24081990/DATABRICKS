# Code level optimization :-

- Partitioning
- Bucketing
- repartition() / coalesce()
- Cache() and Persist()
- Checkpointing
- broadcast join
- sort merge join
- OPTIMIZE
- OPTIMIZE..ZORDER BY

-------------------------
### Partitioning :- 
- Partitioning – Physically splits data into separate folders based on a column’s value (e.g., region=US/, region=IN/) so queries can skip entire folders (partition pruning).
- Best for exact match filters or partition pruning.
- Data skipping is coarse-grained (entire folder).
- should only be used if the cardinality of the columns is low/medium. (i.e. Less number of distinct values. Ex. Country, Region, gender) (Lot of repetative/duplicate similar values)
- benificial in joins also if both the tables are partitioned in the similar columns. 

-----------------------------

### Bucketing :- 
- Pre-splits data into fixed number of “buckets” on join key so Spark can match buckets without shuffling.
- Very useful while joining big-big tables, Can avoid shuffling completely if both sides have the same bucket counts and keys.
- Must only be used if the cardinality of the column is HIGH. (i.e. Large number of distinct/unique values. Ex. ID, name)
  
-----------------------------

### Z-Ordering :-
- Available in detla table format only.
- Reorders data within files (using multi-column sort) so related rows are stored close together physically, improving selective query performance without creating new folders.
- Best for filter range queries or filtering on multiple columns.
- Data skipping is fine-grained (inside files using min/max stats).
- Can be used on partitioned as well as non partitioned data.
- in Databricks, Z-Ordering is always executed through the **OPTIMIZE** command.
  - OPTIMIZE alone → compacts small files into fewer large files.
  - OPTIMIZE ... ZORDER BY → compacts and reorders data files using Z-Order curve on specified columns for faster filtering.

-----------------------------

### Sort Merge join:-
In Spark, Sort Merge Join (SMJ) is a join algorithm where Spark sorts both datasets on the join key, then merges them like a zipper to match rows.
It’s used for large datasets when broadcast join isn’t possible.
Sort Merge Join (SMJ) in Spark is automatic — Spark will choose it when:
- Join type is inner, left outer, right outer, or full outer.
- Both sides of the join are large (not broadcastable).
- Data is already sorted and partitioned on the join keys (or Spark will sort them at runtime).
```
spark.conf.set("spark.sql.join.preferSortMergeJoin", "true")  # default is true
```
Sort Merge Join in Spark does not mandatorily require partitioned tables.
Partitioning just makes it more efficient because data for the same join key is already colocated, reducing shuffle.

-----------------

# Configuration level optimization :-

# Broadcast join threshold (in bytes) – default 10MB
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)

# Number of shuffle partitions (affects joins, aggregations, etc.)
spark.conf.set("spark.sql.shuffle.partitions", 200)

# Prefer Sort Merge Join over Shuffle Hash Join
spark.conf.set("spark.sql.join.preferSortMergeJoin", True)

# Max partition size for reading files (in bytes) – default 128MB
spark.conf.set("spark.sql.files.maxPartitionBytes", 134217728)

# Enable Adaptive Query Execution (AQE)
spark.conf.set("spark.sql.adaptive.enabled", True)

# Enable skew join handling under AQE
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", True)

# Enable Delta optimize write
spark.conf.set("spark.databricks.delta.optimizeWrite.enabled", True)

# Enable Delta auto compaction
spark.conf.set("spark.databricks.delta.autoCompact.enabled", True)

# Ignore corrupt files
spark.conf.set("spark.sql.files.ignoreCorruptFiles", True)

# Ignore missing files
spark.conf.set("spark.sql.files.ignoreMissingFiles", True)

------------
