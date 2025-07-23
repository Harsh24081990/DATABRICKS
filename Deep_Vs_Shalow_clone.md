In **Databricks**, cloning a table means creating a copy of an existing table using Delta Lake. There are two types:

---

### 🔹 Shallow Clone

* **Syntax**:

  ```sql
  CREATE OR REPLACE TABLE new_table SHALLOW CLONE source_table;
  ```

* **Behavior**:

  * Copies only **metadata**, not the data files.
  * Points to the same underlying data files as the original table.
  * Fast and space-efficient.
  * Useful for testing, backups, or short-lived copies.

* ✅ Changes in source **data files** reflect in the clone.

---

### 🔸 Deep Clone

* **Syntax**:

  ```sql
  CREATE OR REPLACE TABLE new_table DEEP CLONE source_table;
  ```

* **Behavior**:

  * Copies both **metadata and data files**.
  * Creates a **fully independent** table.
  * Takes more time and storage.
  * Useful for permanent backup or creating a stable copy.

* ❌ Changes in source don’t affect the clone and vice versa.

---

