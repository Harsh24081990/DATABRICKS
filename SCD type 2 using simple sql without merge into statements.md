Sure! Here's a **complete example** of implementing **SCD Type 2 in Databricks using Delta Lake, but *without using `MERGE INTO`***—just plain **SQL statements**.

This approach uses standard SQL logic like `JOIN`, `UNION`, and `INSERT`/`UPDATE`. It gives you more control and works well for understanding the internals of SCD2.

---

### 🧱 **Assumptions**
You have:
- A **source table**: `SourceTable` (latest data).
- A **target table**: `TargetTable` (historical SCD2 table).
- Columns: `ID`, `Name`, `City`, and metadata fields: `StartDate`, `EndDate`, `IsCurrent`.

---

### 🗃️ **Step-by-Step SQL-Based SCD Type 2 Logic**

```sql
-- 1. Create SourceWithHash (with MD5 hash for change detection)
CREATE OR REPLACE TEMP VIEW SourceWithHash AS
SELECT *,
       md5(concat_ws('||', ID, Name, City)) AS md5_hash
FROM SourceTable;

-- 2. Create TargetWithHash (only current active records)
CREATE OR REPLACE TEMP VIEW TargetWithHash AS
SELECT *,
       md5(concat_ws('||', ID, Name, City)) AS md5_hash
FROM TargetTable
WHERE IsCurrent = 1;

-- 3. Identify changed records (existing records with updated values)
CREATE OR REPLACE TEMP VIEW ChangedRecords AS
SELECT t.ID
FROM SourceWithHash s
JOIN TargetWithHash t
ON s.ID = t.ID
WHERE s.md5_hash != t.md5_hash;

-- 4. Mark old records as expired (IsCurrent = 0, EndDate = today)
UPDATE TargetTable
SET IsCurrent = 0,
    EndDate = current_date()
WHERE ID IN (SELECT ID FROM ChangedRecords)
  AND IsCurrent = 1;

-- 5. Identify new inserts (new IDs or changed ones)
CREATE OR REPLACE TEMP VIEW NewOrChangedRecords AS
SELECT s.ID, s.Name, s.City,
       current_date() AS StartDate,
       NULL AS EndDate,
       1 AS IsCurrent
FROM SourceWithHash s
LEFT JOIN TargetWithHash t
  ON s.ID = t.ID
WHERE t.ID IS NULL
   OR s.md5_hash != t.md5_hash;

-- 6. Insert new records (both new and changed ones)
INSERT INTO TargetTable (ID, Name, City, StartDate, EndDate, IsCurrent)
SELECT * FROM NewOrChangedRecords;
```

---

### 📘 **Sample Table Structure for Reference**

```sql
CREATE TABLE IF NOT EXISTS TargetTable (
  ID INT,
  Name STRING,
  City STRING,
  StartDate DATE,
  EndDate DATE,
  IsCurrent INT
) USING DELTA;
```

> 💡 Tip: Replace `USING DELTA` with your preferred format (e.g., `PARQUET`, `DELTA`) if not using Delta Lake.

---

### 🧠 Optional Enhancements

- **Add surrogate key**: If needed for joins or analytics.
- **Partition by IsCurrent or StartDate**: For performance.
- **ZORDER BY (ID)**: If using Delta Lake for faster reads.

---

Let me know if you want a version using **PySpark DataFrames** instead of SQL or with additional tracking fields like `UpdatedBy`, `ChangeReason`, etc.
