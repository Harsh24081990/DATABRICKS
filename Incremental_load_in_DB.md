### Example 1: Incremental Load Based on Timestamp with High Watermark Table

#### Scenario:
- **`source_table`**: The table containing new incoming data.
- **`target_table`**: The table where data needs to be loaded.
- **`highwatermark_table`**: A table used to store the **last processed timestamp** to track which data has already been loaded.

#### Steps:

1. **Track the last load timestamp**: This value is stored in the **highwatermark_table**.
2. **Fetch new records** from the `source_table` where the `last_updated` timestamp is greater than the last processed timestamp from the **highwatermark_table**.
3. **Insert the new data** into the `target_table`.
4. **Update the high watermark** after each successful load.

#### Example SQL Code:

```sql
-- Step 1: Get the last processed timestamp from the highwatermark table
-- Fetch the last processed timestamp from the highwatermark_table
SELECT MAX(last_processed_timestamp) AS last_load_timestamp
FROM highwatermark_table;

-- Step 2: Extract the new or modified records from the source table
WITH incremental_data AS (
    SELECT *
    FROM source_table
    WHERE last_updated > (SELECT MAX(last_processed_timestamp) FROM highwatermark_table)
)

-- Step 3: Insert the new or modified records into the target table
INSERT INTO target_table
SELECT *
FROM incremental_data;

-- Step 4: Update the high watermark with the timestamp of the latest record loaded
UPDATE highwatermark_table
SET last_processed_timestamp = (SELECT MAX(last_updated) FROM incremental_data);
```

#### Explanation:
- **Step 1**: The last processed timestamp is retrieved from the **`highwatermark_table`**.
- **Step 2**: A subquery is used to get the **last processed timestamp** and filter the `source_table` for records where `last_updated` is greater than the stored timestamp. This eliminates the need to hardcode a date.
- **Step 3**: New or updated records are inserted into the `target_table`.
- **Step 4**: The **`highwatermark_table`** is updated with the latest timestamp from the processed data.

---

### Example 2: Incremental Load Based on Unique Identifier (e.g., `id`) with High Watermark Table

#### Scenario:
- **`source_table`**: The source table containing new or modified records.
- **`target_table`**: The destination table where data needs to be loaded.
- **`highwatermark_table`**: A table used to track the highest `id` of the last successfully processed record.

#### Steps:

1. **Track the last processed `id`**: This value is stored in the **highwatermark_table**.
2. **Extract new records** from the `source_table` where the `id` is greater than the last processed `id` from the **highwatermark_table**.
3. **Insert the new records** into the `target_table`.
4. **Update the high watermark** to reflect the latest `id` processed.

#### Example SQL Code:

```sql
-- Step 1: Get the last processed ID from the highwatermark table
-- Fetch the last processed ID from the highwatermark_table
SELECT MAX(last_processed_id) AS last_max_id
FROM highwatermark_table;

-- Step 2: Extract the new records from the source table where the ID is greater than the last processed ID
WITH incremental_data AS (
    SELECT *
    FROM source_table
    WHERE id > (SELECT MAX(last_processed_id) FROM highwatermark_table)
)

-- Step 3: Insert the new records into the target table
INSERT INTO target_table
SELECT *
FROM incremental_data;

-- Step 4: Update the high watermark with the maximum ID of the newly loaded records
UPDATE highwatermark_table
SET last_processed_id = (SELECT MAX(id) FROM incremental_data);
```

#### Explanation:
- **Step 1**: The **last processed `id`** is retrieved from the **`highwatermark_table`**.
- **Step 2**: A subquery is used to fetch the **maximum processed `id`** from the **`highwatermark_table`**. This value is then used to filter records from `source_table` where the `id` is greater than this last processed `id`. This eliminates the need to hardcode an `id`.
- **Step 3**: The new or updated records are inserted into the `target_table`.
- **Step 4**: After the load, the **highwatermark_table** is updated with the highest `id` from the processed data.

---

### Example 3: Using Delta Lake for Incremental Load with `MERGE` (Upsert)

If you are using **Delta Lake** (Databricks' version of Apache Spark's table format), you can perform an incremental load more efficiently using the **Delta `MERGE`** (also known as **Upsert**) operation. This allows you to update or insert data based on a condition.

#### Example SQL Code:

```sql
-- Step 1: Perform incremental load using Delta Merge
MERGE INTO target_table AS target
USING source_table AS source
ON target.id = source.id  -- Match based on unique identifier
WHEN MATCHED AND source.last_updated > target.last_updated THEN
    UPDATE SET target.data = source.data, target.last_updated = source.last_updated
WHEN NOT MATCHED THEN
    INSERT (id, data, last_updated) VALUES (source.id, source.data, source.last_updated);
```

#### Explanation:
- The **`MERGE`** statement combines **insert** and **update** operations into a single query.
- It updates records in the `target_table` where the `id` matches and the `last_updated` in the `source_table` is newer than the `target_table`. It inserts records into the `target_table` when no match is found.

---
### Summary:
- In **Example 1** and **Example 2**, you use a **highwatermark table** to store the **last processed timestamp or ID**. This allows you to dynamically fetch new data from the `source_table` without hardcoding values.
- In **Example 3**, the **Delta `MERGE`** operation offers a more efficient way to perform incremental loads by updating or inserting data based on a unique identifier and change logic (e.g., `last_updated` timestamp).
