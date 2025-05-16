Here's a clear and concise summary of the **key file types and folders** you’ll encounter inside **Databricks** (especially when working with Delta tables), along with their **paths** and **purposes**:

---

## 📁 Summary of Files & Folders in Databricks

### ✅ 1. **User Workspace Storage**

> **Path**: `/Workspace/`

* Contains notebooks, scripts, and folders you create in the UI.
* Not part of DBFS or Delta data storage.

---

### ✅ 2. **DBFS (Databricks File System)** (dbfs:/)

> **Path**: `/dbfs/` or `dbfs:/`

* Unified interface to access files stored in the cluster's storage.
* Accessible via notebook paths or `%fs` commands.

---

### ✅ 3. **Table Storage (Delta or Parquet Tables)**

> **Example Path**: `dbfs:/user/hive/warehouse/my_table/`

Inside this directory, you’ll find:

#### 🟩 `part-*.parquet`

* **Path**: `dbfs:/user/hive/warehouse/my_table/part-00000-*.parquet`
* Actual **data files**, written in **Parquet format**.
* Created on every write (insert/merge).

#### 🟩 `_delta_log/`

* **Path**: `dbfs:/user/hive/warehouse/my_table/_delta_log/`
* Stores Delta Lake’s **transaction logs** and metadata.

Contents include:

| File                        | Purpose                                              |
| --------------------------- | ---------------------------------------------------- |
| `000.json`, `001.json`, ... | Ordered transaction log entries (metadata + actions) |
| `000.crc`, `001.crc`        | Checksum files (validate integrity of JSON logs)     |

---

### ✅ 4. **Optimization Temp Folders**

> **Path**: `dbfs:/user/hive/warehouse/my_table/s3-optimization-*`

| Folder Name         | Purpose                                             |
| ------------------- | --------------------------------------------------- |
| `s3-optimization-0` | Temp output dir for file compaction or optimization |
| `s3-optimization-1` | May store intermediate Parquet files                |
| `s3-optimization-*` | Appear due to auto-optimize, ZORDER, OPTIMIZE, etc. |
| ❗ Misleading Name   | Doesn’t require AWS S3 — appears even in Azure/GCP  |

---

### ✅ 5. **Checkpoints (optional)**

> **Path**: `dbfs:/.../_delta_log/_checkpoints/`

* Used internally to improve performance of reading the Delta log.
* May not appear unless certain workloads (e.g., streaming) are used.

---

## ✅ How to View All Files in a Path

Use this command in a notebook cell:

```python
%fs ls dbfs:/user/hive/warehouse/my_table/
```

Or check:

```sql
DESCRIBE DETAIL my_table;
```

---

## 🔎 Bonus Tip: Table Versions

See Delta table history:

```sql
DESCRIBE HISTORY my_table;
```
## 🧹 VACUUM Deletes:
Old Parquet data files that were:

Overwritten

Deleted

No longer valid due to a MERGE, UPDATE, or DELETE

These files still exist on disk until you run VACUUM — which reclaims that space.

| Command Example                   | What It Does                            |
| --------------------------------- | --------------------------------------- |
| `VACUUM my_table`                 | Deletes files older than 7 days         |
| `VACUUM my_table RETAIN 24 HOURS` | Deletes files older than 1 day          |
| `VACUUM my_table RETAIN 0 HOURS`  | ⚠️ Deletes all unused files immediately |

---
