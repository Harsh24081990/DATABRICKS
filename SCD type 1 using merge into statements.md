Sure! Here’s a **simple and complete Databricks example** of how to implement **SCD Type 1 (Slowly Changing Dimension Type 1)** using **Delta Lake `MERGE INTO`** statements.

---

### 🧠 **What is SCD Type 1?**

SCD Type 1 **overwrites** old data with the latest data—**no history is preserved**. It’s used when tracking changes isn't important.

---

### ✅ **Step-by-Step: Implementing SCD Type 1 in Databricks with Delta**

---

### 🔹 1. **Create Sample Source Data (Latest Updates)**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import lit

# Sample source data
source_data = [
    (1, "Alice", "New York"),          # No change
    (2, "Bob", "San Diego"),           # City changed
    (3, "Charlie", "Chicago")          # New record
]

source_df = spark.createDataFrame(source_data, ["ID", "Name", "City"])
source_df.createOrReplaceTempView("source")
```

---

### 🔹 2. **Create Initial Target Table**

```python
# Initial target table with existing records
target_data = [
    (1, "Alice", "New York"),
    (2, "Bob", "Los Angeles")
]

target_df = spark.createDataFrame(target_data, ["ID", "Name", "City"])

# Save as a Delta table
target_df.write.format("delta").mode("overwrite").saveAsTable("target")
```

---

### 🔁 3. **Perform SCD Type 1 Merge (Overwrite Existing Data)**

```sql
MERGE INTO target AS tgt
USING source AS src
ON tgt.ID = src.ID

-- If record exists and has changes, update
WHEN MATCHED THEN
  UPDATE SET tgt.Name = src.Name,
             tgt.City = src.City

-- If record doesn’t exist, insert it
WHEN NOT MATCHED THEN
  INSERT (ID, Name, City)
  VALUES (src.ID, src.Name, src.City);
```

---

### 🔍 4. **View Final Result**

```sql
SELECT * FROM target ORDER BY ID;
```

---

### ✅ Output After Merge

| ID | Name    | City       |
|----|---------|------------|
| 1  | Alice   | New York   | ✅ No change  
| 2  | Bob     | San Diego  | ✅ City updated (overwrite)
| 3  | Charlie | Chicago    | ✅ New record inserted

---

### 💡 Notes:
- No `IsCurrent`, `StartDate`, `EndDate`—because history isn’t stored in Type 1.
- Great for attributes like `Email`, `Phone`, or `Name` when historical tracking isn’t needed.

---

Let me know if you'd like this wrapped in a **Databricks notebook**, or need a version using **PySpark instead of SQL**!
