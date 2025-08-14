# 1️⃣ Co-partition & Co-locate Data (Bucketing)
- Bucket both tables on the join key with the same number of buckets.
- Store as Delta/Parquet with BUCKET BY and same sort order.
- Spark will skip shuffle if bucket/partition layout matches.

## Bucketing on an existing Databricks Delta table
-- Suppose we already have a Delta table in Bronze

```
CREATE TABLE bronze.customers
USING delta
LOCATION 'abfss://datalake/bronze/customers';

-- Create a bucketed copy in Silver
CREATE TABLE silver.customers_bucketed
USING delta
CLUSTERED BY (customer_id) INTO 200 BUCKETS
AS SELECT * FROM bronze.customers;
```
## Bucketing on external table/file directly
### Example 1. Read from Oracle via JDBC
```
df = (spark.read.format("jdbc")
      .option("url", "jdbc:oracle:thin:@host:port/service")
      .option("dbtable", "CUSTOMERS")
      .option("user", "...")
      .option("password", "...")
      .load())

# Save as bucketed Delta table
(df.write.format("delta")
    .bucketBy(200, "customer_id")
    .saveAsTable("silver.customers_bucketed"))
```
--------
-----------------

# Example: Use Bucketing for joining 2 bigger tables to avoid shuffling.
```
-- Bucketed Delta table
CREATE TABLE customers_bucketed
USING delta
CLUSTERED BY (customer_id) INTO 200 BUCKETS
AS SELECT * FROM customers_source;

CREATE TABLE transactions_bucketed
USING delta
CLUSTERED BY (customer_id) INTO 200 BUCKETS
AS SELECT * FROM transactions_source;

-- Join without shuffle if bucket metadata matches
SELECT *
FROM customers_bucketed c
JOIN transactions_bucketed t
ON c.customer_id = t.customer_id;
```
## Cautions:-
- Both tables must be bucketed on the same column(s).
- Same number of buckets.
- Joins must be on exactly the bucketed column(s).
- Spark still sometimes shuffles if it can’t guarantee bucket alignment (e.g., mergeSchema enabled, extra filters before join).
- If data is updated frequently, bucketing can be tricky because INSERT/UPDATE/MERGE can break bucket alignment unless you re-bucket during write.
  

Spark still sometimes shuffles if it can’t guarantee bucket alignment (e.g., mergeSchema enabled, extra filters before join).
------------------------------------

# 2️⃣ Partition Pruning on Join Key (Partitioning)
- Partition both datasets by the join key in storage (ADLS/Delta partitioning).
- Physically split data into separate directories by partition column so Spark only reads relevant directories when filtering/joining.
- When reading, Spark only reads the required partitions → less shuffle.
- (Only helps if join key has low/medium cardinality.)

## Partitioning on an existing Databricks Delta table.
```
-- Customers table partitioned by region
CREATE TABLE customers_partitioned
USING delta
PARTITIONED BY (region)
AS SELECT * FROM customers_source;

-- Transactions table partitioned by region
CREATE TABLE transactions_partitioned
USING delta
PARTITIONED BY (region)
AS SELECT * FROM transactions_source;
```
```
--Join with pruning:
df = spark.table("customers_partitioned").join(
    spark.table("transactions_partitioned"),
    "region"
)
```

## Partitioning on external table/file directly

### Example 1 – From external Parquet/CSV in ADLS
```
# Read from external storage
df_customers = (spark.read
    .format("parquet")
    .load("abfss://raw@datalake/path/customers/"))

df_transactions = (spark.read
    .format("csv")
    .option("header", True)
    .load("abfss://raw@datalake/path/transactions/"))

# Write as partitioned Delta tables
(df_customers.write
    .format("delta")
    .partitionBy("region")
    .mode("overwrite")
    .saveAsTable("silver.customers_partitioned"))

(df_transactions.write
    .format("delta")
    .partitionBy("region")
    .mode("overwrite")
    .saveAsTable("silver.transactions_partitioned"))
```

### Example 2 – From external JDBC source (e.g., Oracle)
```
# Customers from Oracle
df_customers = (spark.read.format("jdbc")
    .option("url", "jdbc:oracle:thin:@host:port/service")
    .option("dbtable", "CUSTOMERS")
    .option("user", "my_user")
    .option("password", "my_pass")
    .load())

# Transactions from Oracle
df_transactions = (spark.read.format("jdbc")
    .option("url", "jdbc:oracle:thin:@host:port/service")
    .option("dbtable", "TRANSACTIONS")
    .option("user", "my_user")
    .option("password", "my_pass")
    .load())

# Write directly to partitioned Delta tables
(df_customers.write
    .format("delta")
    .partitionBy("region")
    .mode("overwrite")
    .saveAsTable("silver.customers_partitioned"))

(df_transactions.write
    .format("delta")
    .partitionBy("region")
    .mode("overwrite")
    .saveAsTable("silver.transactions_partitioned"))
```
