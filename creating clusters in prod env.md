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

