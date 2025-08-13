In production, once an **ADF pipeline** triggers a Databricks notebook activity, the cluster configuration comes from **the ADF activity settings itself**, not from a pre-created cluster in Databricks Compute pane.

---

### **How it works**

* In the **ADF "Databricks Notebook" activity**:

  * You select **Cluster Type** → *New job cluster* (prod standard).
  * You define:

    * Node type (e.g., `Standard_E32ds_v5`)
    * Min/max workers (for autoscaling)
    * Spark version
    * Any Spark config overrides
  * These settings are stored **inside the ADF pipeline JSON definition**.
* When ADF triggers:

  1. ADF calls Databricks REST API.
  2. Databricks **reads the cluster config passed in that API call** (from ADF JSON).
  3. Databricks spins up a **job cluster** with those specs.
  4. Job runs, cluster auto-terminates.

---
### Note : 
When the pipeline runs, ADF doesn’t pick an existing cluster from the databricks Compute pane — it passes the ADF notebook activity JSON to Databricks API, which spins up a fresh cluster with these specs.

---
Here’s an example of an **ADF pipeline JSON snippet** for a Databricks notebook activity using a **new job cluster** in production:

```json
{
    "name": "DatabricksNotebookActivity",
    "type": "DatabricksNotebook",
    "dependsOn": [],
    "policy": {
        "timeout": "7.00:00:00",
        "retry": 0,
        "retryIntervalInSeconds": 30,
        "secureOutput": false,
        "secureInput": false
    },
    "typeProperties": {
        "notebookPath": "/ProdJobs/DataProcessing",
        "baseParameters": {
            "inputPath": "abfss://container@storageaccount.dfs.core.windows.net/input",
            "outputPath": "abfss://container@storageaccount.dfs.core.windows.net/output"
        },
        "newCluster": {
            "clusterName": "adf-prod-cluster",
            "sparkVersion": "13.3.x-scala2.12",
            "nodeType": "Standard_E32ds_v5",
            "driverNodeType": "Standard_E32ds_v5",
            "autoscale": {
                "minWorkers": 10,
                "maxWorkers": 20
            },
            "spark_conf": {
                "spark.sql.shuffle.partitions": "2000",
                "spark.dynamicAllocation.enabled": "true"
            },
            "customTags": {
                "Project": "ETL-Prod",
                "Environment": "Production"
            },
            "spark_env_vars": {
                "PYSPARK_PYTHON": "/databricks/python3/bin/python3"
            },
            "init_scripts": [],
            "enableElasticDisk": true
        }
    },
    "linkedServiceName": {
        "referenceName": "AzureDatabricksLinkedService",
        "type": "LinkedServiceReference"
    }
}
```

---

**Where the cluster config lives:**

* Everything inside `"newCluster": { ... }` is **the full cluster definition** that ADF sends to Databricks.
* When the pipeline runs, **ADF doesn’t pick an existing cluster from the Compute pane** — it passes this JSON to Databricks API, which spins up a fresh cluster with these specs.

---

In real production projects, you almost never have **only one cluster** for all pipelines.
Instead, cluster usage depends on **project size, job type, and cost strategy**:

---

# **Common production patterns**

1. **One cluster per pipeline run** (most common for batch ETL)

   * Each ADF pipeline triggers a **job cluster** (new cluster) with specs defined in that pipeline.
   * After the job completes, the cluster **auto-terminates**.
   * Pros: Clean, isolated environment; avoids job interference.
   * Cons: More startup time (\~2–5 mins per run).

2. **Multiple pipelines sharing a pool**

   * Databricks **cluster pools** keep nodes warm.
   * Job clusters for different pipelines are created **from the same pool**, reducing startup cost/time.
   * Still one job cluster per pipeline run, but sharing pool resources.

3. **Shared interactive cluster for multiple jobs** (rare in prod, more in dev/test)

   * A single always-running cluster executes many jobs/pipelines.
   * Used for low-latency streaming jobs or small-scale batch processes.
   * Cons: Resource contention, risk of job conflicts.

4. **Different cluster configs for different workloads**

   * **Heavy ETL jobs** → large, memory-optimized nodes (e.g., `E32ds_v5`).
   * **ML jobs** → GPU clusters.
   * **Small data prep jobs** → smaller, cheaper nodes.

---

in **ADF settings** you don’t directly specify things like *cores per executor* or *RAM per core*.

### How it works in production:

* **ADF “Databricks Notebook” activity → New job cluster**
  You only set high-level **cluster specs**:

  * Node type (decides total RAM & cores per node)
  * Number of workers (or min/max for autoscaling)
  * Driver node type
  * Spark version
  * Optional Spark config overrides

* **Spark parameters** (like executors per node, cores per executor, executor memory)
  You control these **inside the notebook code** or in the `spark_conf` block of the ADF cluster config JSON.

Example in ADF JSON:

```json
"spark_conf": {
    "spark.executor.cores": "8",
    "spark.executor.memory": "52g",
    "spark.executor.instances": "400"
}
```

*(Above is optional — if you don’t set them, Spark auto-allocates based on node type & number of nodes.)*

---

💡 **So:**

* In ADF UI → you choose **hardware capacity** (node type, count).
* In Spark config (in ADF JSON or notebook) → you fine-tune **how those resources are divided into executors/cores/memory**.

---

Here’s a **full ADF Databricks job cluster config JSON** for the **10 TB example** we discussed, including all Spark tuning parameters:

```json
{
  "name": "Process10TBData",
  "type": "DatabricksNotebook",
  "linkedServiceName": {
    "referenceName": "AzureDatabricksLinkedService",
    "type": "LinkedServiceReference"
  },
  "typeProperties": {
    "notebookPath": "/ProdJobs/ETL_10TB",
    "newCluster": {
      "clusterName": "adf-prod-10tb-cluster",
      "sparkVersion": "13.3.x-scala2.12",
      "nodeType": "Standard_E32ds_v5",
      "driverNodeType": "Standard_E32ds_v5",
      "autoscale": {
        "minWorkers": 90,
        "maxWorkers": 100
      },
      "spark_conf": {
        "spark.executor.cores": "8",
        "spark.executor.memory": "52g",
        "spark.executor.instances": "400",
        "spark.sql.shuffle.partitions": "4000",
        "spark.dynamicAllocation.enabled": "true",
        "spark.memory.fraction": "0.8",
        "spark.sql.adaptive.enabled": "true"
      },
      "customTags": {
        "Project": "ETL-Prod",
        "Environment": "Production"
      },
      "spark_env_vars": {
        "PYSPARK_PYTHON": "/databricks/python3/bin/python3"
      },
      "enableElasticDisk": true
    },
    "baseParameters": {
      "inputPath": "abfss://container@storageaccount.dfs.core.windows.net/input",
      "outputPath": "abfss://container@storageaccount.dfs.core.windows.net/output"
    }
  }
}
```

---

### **What this does**

* **Hardware**:

  * 1 driver + 90–100 workers
  * Each node = 256 GB RAM, 32 vCPUs
* **Executors**:

  * 4 per worker → 400 total
  * 8 cores & 52 GB RAM each
* **Spark tuning**:

  * Adaptive execution enabled
  * Large shuffle partitions for 10 TB dataset
  * Memory fraction set to 0.8 for more in-RAM processing

---


