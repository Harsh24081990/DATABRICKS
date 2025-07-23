## What is **Schema Evolution** in Delta Lake?

**Schema evolution** means **automatically adapting the table's schema** when new columns or data types appear in your source data.

> Think of it as: “Hey Delta Table, update yourself if new columns show up in the incoming data!”

---

## 🧠 Why Does This Matter?

In real-world pipelines (like ETL or SCD Type 2), the source data can change over time:

- A new column (e.g., `Col3`) is added
- A column's type changes slightly (e.g., `int` → `long`)
- You restructure your source schema

If you **don’t handle schema evolution**, your merge will **fail** with an error like:
```
Cannot write to 'TargetTable' because schema mismatch detected
```

---

## ✅ Delta Lake to the Rescue: `mergeSchema = true`

Delta Lake supports **schema evolution during `MERGE INTO`**, but only if you **explicitly allow it**.

### 👇 How to enable schema evolution in SQL:

```sql
-- Set schema auto-merge ON (session level)
SET spark.databricks.delta.schema.autoMerge.enabled = true;
```

Then your `MERGE INTO` statement will automatically evolve the schema if needed — for example, to accommodate a new column.

---

### 🧪 Example Scenario

Let’s say your source table now includes a new column `Col3`.

If your target table (`TargetTable`) does **not** have `Col3`, but you try this:

```sql
MERGE INTO TargetTable AS target
USING SourceTable AS source
ON target.ID = source.ID
WHEN NOT MATCHED THEN
  INSERT (ID, Col1, Col2, Col3)
  VALUES (source.ID, source.Col1, source.Col2, source.Col3);
```

👉 Without schema evolution, this will **fail**.

👉 With schema evolution (`SET spark.databricks.delta.schema.autoMerge.enabled = true`), Delta will **add `Col3` to TargetTable automatically**.

---

## 💡 Use Case with SCD Type 2

In SCD Type 2 logic:
- If your `SourceTable` gets new business columns (e.g., `Col3`)
- And you don’t want to manually alter the schema of `TargetTable`
- Schema evolution helps ensure your pipeline **keeps working without manual intervention**

---

## ⚠️ When to Be Careful

While convenient, schema evolution:
- Can accidentally let **undesired schema changes** sneak into production
- Should be **monitored or logged** in regulated environments
- Can be **restricted** in enterprise environments for governance reasons

---

## ✅ TL;DR

| Term             | Meaning                                                                 |
|------------------|-------------------------------------------------------------------------|
| **Schema Evolution** | Auto-update Delta Table schema when incoming data has new columns      |
| **Why use it?**       | Avoid breaking merges/inserts when source schema changes over time   |
| **How to enable?**    | `SET spark.databricks.delta.schema.autoMerge.enabled = true`         |
| **Use case?**         | Useful for SCD Type 2 pipelines when business data grows over time   |

---

# ⭐ Schema Evolution Vs Schema Inference
## (mergeSchema = true Vs inferSchema = true)

**🔹 mergeSchema = true**
- Used With: write operations (e.g. df.write.option("mergeSchema", "true")...)
- Enables schema evolution during write
- If new columns exist in incoming DataFrame, they are added to the existing Delta table schema
✅ Use when you want the table schema to adapt automatically during writes (e.g., new columns in new batch)

**🔸 inferSchema = true**
- Used With: read operations (e.g. spark.read.option("inferSchema", "true"))
- Tells Spark to infer the schema from data (e.g., when reading CSV, JSON, etc.)
- Spark scans some data and guesses data types.
✅ Use when reading files without predefined schema
❌ Don’t confuse with schema evolution — This (schema inference) is only during read.

------

# ⭐ Schema Evolution Vs Schema Enforcement

**✅ Schema Evolution**
- Allows table schema to change over time (e.g., add new columns)
- Enabled using `**mergeSchema = true**` or explicitly via ALTER TABLE
- Supported in Delta Lake, not in plain Parquet.
- Use case: Ingesting semi-structured or changing data sources (e.g., event logs)

**❌ Schema Enforcement**
- Prevents writing data that does not match the existing schema.
- Delta Lake will throw an error if columns/types mismatch.
- Ensures data quality.
- Use case: Critical production pipelines where schema stability is required.

-----------
