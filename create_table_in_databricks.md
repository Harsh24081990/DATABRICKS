# Create table using SQL

In Databricks, when you create a table using the `CREATE TABLE` statement, the default behavior depends on whether you specify the `LOCATION` clause:

- **Internal (Managed) Table**: If you create a table without specifying a `LOCATION`, it will be created as an internal (managed) table. Databricks manages the storage location of the data, and if you drop the table, the data will also be deleted.

  ```sql
  CREATE TABLE my_internal_table (
      id INT,
      name STRING
  );
  ```

- **External Table**: If you specify a `LOCATION`, the table will be created as an external table. In this case, Databricks does not manage the data; instead, it references the data in the specified location. If you drop the external table, the data remains intact in the specified location.

  ```sql
  CREATE TABLE my_external_table (
      id INT,
      name STRING
  )
  LOCATION '/mnt/mydata/external_table';
  ```
### Summary
- **Default Behavior**: `CREATE TABLE` without `LOCATION` creates an internal (managed) table.
- **External Table**: Use `LOCATION` to create an external table.

###==========================================================================

# Create table using pyspark
When creating tables in Databricks using PySpark syntax, the default behavior regarding internal (managed) vs. external tables is similar to the SQL approach:
### 1. **Creating an Internal (Managed) Table**
If you create a table without specifying a `path`, it will be created as an internal (managed) table. The data is managed by Databricks, and if you drop the table, the data will also be deleted.

```python
# Create a Spark DataFrame
data = [(1, "Alice"), (2, "Bob")]
df = spark.createDataFrame(data, ["id", "name"])

# Write the DataFrame to a managed table
df.write.format("delta").saveAsTable("my_internal_table")
```
### 2. **Creating an External Table**
If you want to create an external table, you need to specify the `path` where the data will be stored. This way, the table references the data at the specified location, and dropping the table does not delete the underlying data.
```python
# Create a Spark DataFrame
data = [(1, "Alice"), (2, "Bob")]
df = spark.createDataFrame(data, ["id", "name"])

# Specify a path for the external table
external_path = "/mnt/mydata/external_table"

# Write the DataFrame to an external table
df.write.format("delta").mode("overwrite").save(external_path)

# Register the external table
spark.sql(f"CREATE TABLE my_external_table USING DELTA LOCATION '{external_path}'")
```
### Summary

- **Internal (Managed) Table**: Create using `saveAsTable()` without specifying a path.
- **External Table**: Create by saving data to a specific path and registering it using SQL.
