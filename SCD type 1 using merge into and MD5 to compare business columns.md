Got it! Here's a **Databricks-ready SCD Type 1 implementation** using Delta Lake's `MERGE INTO` that:

> ✅ Compares **key business columns** (e.g., `Name`, `City`) to determine if a record is new or updated  
> ✅ **Overwrites** data in the target table (no history preserved)  
> ✅ Uses **Delta Lake merge** efficiently  

---

## 🧱 Scenario

We are tracking customer data:

- **Business key**: `CustomerID`  
- **Business columns to compare**: `Name`, `City`  
- **Goal**: If `CustomerID` exists and `Name` or `City` has changed → **overwrite it**  
- If it doesn't exist → **insert it**

---

## ✅ Step 1: Create Sample Source Data

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import lit

# Sample source data (latest snapshot)
source_data = [
    (1, "Alice", "New York"),         # No change
    (2, "Bob", "San Diego"),          # City changed
    (3, "Charlie", "Chicago")         # New customer
]

source_df = spark.createDataFrame(source_data, ["CustomerID", "Name", "City"])
source_df.createOrReplaceTempView("source")
```

---

## ✅ Step 2: Create Initial Target Table

```python
# Initial target (existing historical data)
target_data = [
    (1, "Alice", "New York"),
    (2, "Bob", "Los Angeles")  # Will be updated
]

target_df = spark.createDataFrame(target_data, ["CustomerID", "Name", "City"])
target_df.write.format("delta").mode("overwrite").saveAsTable("customer_target")
```

---

## ✅ Step 3: SCD Type 1 `MERGE INTO` with Business Column Comparison

```sql
MERGE INTO customer_target AS tgt
USING (
    SELECT *,
           md5(concat_ws('||', Name, City)) AS hash_key
    FROM source
) AS src
ON tgt.CustomerID = src.CustomerID

WHEN MATCHED AND md5(concat_ws('||', tgt.Name, tgt.City)) != src.hash_key THEN
  UPDATE SET
    tgt.Name = src.Name,
    tgt.City = src.City

WHEN NOT MATCHED THEN
  INSERT (CustomerID, Name, City)
  VALUES (src.CustomerID, src.Name, src.City);
```

---

### 🔍 Step 4: View the Final Table

```sql
SELECT * FROM customer_target ORDER BY CustomerID;
```

---

## 📝 Example Output:

| CustomerID | Name    | City       |
|------------|---------|------------|
| 1          | Alice   | New York   | ✅ Unchanged  
| 2          | Bob     | San Diego  | ✅ Updated  
| 3          | Charlie | Chicago    | ✅ Inserted  

---

## ✅ Benefits of This Pattern

- Uses **`md5()` hash** for efficient column comparison  
- Avoids unnecessary updates (only changes when business columns change)  
- Keeps logic **tight and readable**, perfect for pipelines in production  

---

Let me know if you want to include **timestamps, metadata (e.g., UpdatedBy)**, or use **PySpark-only (no SQL)** version!
