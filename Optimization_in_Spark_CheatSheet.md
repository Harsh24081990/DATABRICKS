
## **1️⃣ Manual (Code / SQL Command Level)** optimization techniques:-

You explicitly write code or SQL commands to trigger them:

* **Partitioning** → at table creation or write time (`PARTITIONED BY` in SQL, or `.partitionBy()` in DataFrameWriter).
* **Bucketing** → at table creation time (`CLUSTERED BY ... INTO n BUCKETS`).
* **repartition() / coalesce()** → DataFrame methods to control number of partitions.
* **cache() / persist()** → explicitly called on DataFrame.
* **checkpointing** → explicitly called in code with `df.checkpoint()`.
* **broadcast join** → explicitly with `broadcast(df2)` or hint (`.hint("broadcast")`).
* **sort merge join** → can be *forced* by sorting + config, but normally chosen by optimizer; manual forcing requires explicit sort or setting join hints.
* **OPTIMIZE** → Databricks SQL command for file compaction.
* **OPTIMIZE ... ZORDER BY** → Databricks SQL command for data skipping optimization.
* **filter pushdown** → writing filters in query so Spark pushes them to the source (manual in query design).

---

## **2️⃣ Configuration-Level (spark.conf / SparkSession settings)** optimization techniques:-

You set parameters and Spark automatically applies the optimization at runtime:

* **spark.sql.autoBroadcastJoinThreshold** → controls when broadcast joins are auto-applied.
* **spark.sql.shuffle.partitions** → controls number of shuffle partitions (affects SMJ, hash join, aggregations).
* **spark.sql.join.preferSortMergeJoin** → preference for SMJ over shuffle hash join.
* **spark.sql.files.maxPartitionBytes** → controls input split sizes.
* **spark.sql.adaptive.enabled** → enables Adaptive Query Execution (AQE).
* **spark.sql.adaptive.skewJoin.enabled** → handles skew in join keys.
* **spark.databricks.delta.optimizeWrite.enabled** / **autoCompact.enabled** → automatic small file compaction for Delta tables.
* **spark.sql.files.ignoreCorruptFiles** / **spark.sql.files.ignoreMissingFiles** → improve read stability (not exactly performance, but tuning).

---

Here’s the **Spark Optimization Quick Reference Sheet** you can keep:

---

| **Workload Type**                      | **Manual Actions**                                                                               | **Config Settings**                                                                                                                                                                          |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Batch ETL (Large Datasets)**         | Partitioning (by filter col), ZORDER (Delta), OPTIMIZE after load, Repartition before write      | `spark.sql.shuffle.partitions` (\~2–3× cores), `spark.databricks.delta.optimizeWrite.enabled=true`                                                                                           |
| **Joins (Large-Large / Large-Small)**  | Broadcast join (small side), Bucketing on join col (recurring joins), Ensure sorted data for SMJ | `spark.sql.autoBroadcastJoinThreshold` (increase if memory allows), `spark.sql.adaptive.enabled=true`, `spark.sql.adaptive.skewJoin.enabled=true`, `spark.sql.join.preferSortMergeJoin=true` |
| **Streaming Pipelines**                | Checkpointing, Repartition for balanced batches, Cache() only if reused, Watermarking            | `spark.sql.shuffle.partitions` (lower for small batches)                                                                                                                                     |
| **Heavy Analytics / Repeated Queries** | Cache/Persist intermediate results, OPTIMIZE + ZORDER for query speed                            | `spark.sql.adaptive.enabled=true`, `spark.sql.files.maxPartitionBytes` (tune for scan parallelism)                                                                                           |
| **Small File Problem Fix**             | Repartition before write, OPTIMIZE after load                                                    | `spark.databricks.delta.autoCompact.enabled=true`, `spark.databricks.delta.optimizeWrite.enabled=true`                                                                                       |

---

