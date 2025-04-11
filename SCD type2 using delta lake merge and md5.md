---

# 🔁 Complete SCD Type 2 using TEMP VIEWS in Databricks

---

## 🔧 STEP 0: Create Temp Views with `md5_hash` for Source and Target

```sql
-- STEP 0A: Temp view for SourceTable with md5_hash calculated
CREATE OR REPLACE TEMP VIEW SourceWithHash AS
SELECT *,
       md5(concat_ws('||', Col1, Col2)) AS md5_hash
FROM SourceTable;

-- STEP 0B: Temp view for TargetTable with md5_hash calculated (only active records)
CREATE OR REPLACE TEMP VIEW TargetWithHash AS
SELECT *,
       md5(concat_ws('||', Col1, Col2)) AS md5_hash
FROM TargetTable;
```

---

## ✅ STEP 1: Expire old records with changed data

```sql
-- STEP 1: Update TargetTable records where ID matches but data has changed
MERGE INTO TargetTable AS target
USING SourceWithHash AS source
ON target.ID = source.ID AND target.Current = 1
WHEN MATCHED AND md5(concat_ws('||', target.Col1, target.Col2)) != source.md5_hash THEN
  -- (a) Updating existing record in target: same ID, data changed → mark old record as inactive
  UPDATE SET
    target.Current = 0,
    target.EndDate = current_date();
```

---

## ✅ STEP 2: Insert new and changed records

```sql
-- STEP 2: Insert new records and updated versions of changed records
MERGE INTO TargetTable AS target
USING SourceWithHash AS source
ON target.ID = source.ID 
   AND target.Current = 1 
   AND md5(concat_ws('||', target.Col1, target.Col2)) = source.md5_hash
WHEN NOT MATCHED BY TARGET THEN
  -- (b) Inserting updated record: ID matches, but data changed (expired in step 1)
  -- (c) Inserting new record: ID not found in TargetTable
  INSERT (ID, Col1, Col2, StartDate, EndDate, Current, md5_hash)
  VALUES (source.ID, source.Col1, source.Col2, current_date(), NULL, 1, source.md5_hash);
```

---

## 📌 Summary of Logic

| Case                              | What Happens                                                                      |
|-----------------------------------|------------------------------------------------------------------------------------|
| **(a)** ID match + data changed   | Existing row is **expired** (`Current = 0`, `EndDate = current_date()`)          |
| **(b)** ID match + data changed   | New version of the row is **inserted** with `Current = 1`, `StartDate = today`    |
| **(c)** New ID                    | New row is **inserted** with `Current = 1`, `StartDate = today`                   |
| ID match + no change              | **No action** taken — everything is the same                                      |

---

## ✅ Notes & Best Practices

- This approach uses **temporary views**, so no need to alter schema
- Great for ad-hoc jobs, testing, or pipelines where you don’t want to modify tables
- If performance is a concern, consider **materializing `md5_hash` as a persistent column**

---
------------------------------------------------------------------------------

# 💡 Pro Tip for Databricks

If you're running this regularly and `md5_hash` is **always needed**, consider **adding it as a persistent column** to `TargetTable`. That way you don’t need to recalculate it or rely on a temp view every time.

```sql
ALTER TABLE TargetTable ADD COLUMN md5_hash STRING;
```


