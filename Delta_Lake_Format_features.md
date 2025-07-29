### ✅ Key **Features of Delta Lake** in Databricks:

---

### 🔹 1. **ACID Transactions**

* Ensures **atomicity, consistency, isolation, durability**
* Enables safe concurrent reads/writes

---

### 🔹 2. **Schema Enforcement**

* Prevents bad or unexpected data from being written
* Columns/types must match the defined schema

---

### 🔹 3. **Schema Evolution**

* Allows **automatic handling of schema changes** (e.g., adding columns) using options like `mergeSchema = true`

---

### 🔹 4. **Time Travel**

* Query data **as of a specific timestamp or version**

```sql
SELECT * FROM table TIMESTAMP AS OF '2025-07-28 00:00:00'
```

---

### 🔹 5. **Data Versioning**

* Delta automatically keeps versions of data files
* Helps with auditing, debugging, and rollback

---

### 🔹 6. **Efficient Upserts (MERGE)**

* Perform `MERGE INTO` to handle **insert/update/delete** efficiently

---

### 🔹 7. **Scalable Metadata Handling**

* Supports millions of files with **fast queries** (unlike traditional Hive tables)

---

### 🔹 8. **Streaming + Batch Support**

* Works with **Structured Streaming** for real-time pipelines
* Seamlessly switches between batch and streaming

---

### 🔹 9. **Z-Ordering**

* Improves performance by **clustering data** based on columns
* Speeds up selective queries (filter pushdowns)

---

### 🔹 10. **Optimize and Vacuum**

* `OPTIMIZE` = compact small files
* `VACUUM` = clean up old, unused data files

---

These features make Delta Lake ideal for **reliable, performant, large-scale data pipelines** in Databricks.
