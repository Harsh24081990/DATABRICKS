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
