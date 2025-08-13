### **Data Size**

- We handle approximately **100 TB of structured data**, which includes:
  - **Loan application data**
  - **Claims data**
  - **User interaction logs**
  - **Transactional records**
- The data is stored in **Azure Data Lake Storage Gen2 (ADLS Gen2)**.
- We process about:
  - **10 TB of batch data per day**
  - **1–2 TB of streaming data per day**, handled through structured streaming jobs in Databricks

---

### **Pipeline Summary**

- We run around **80 ADF pipelines** on daily basis, orchestrating the Databricks jobs.
  - **70 batch pipelines** (daily/monthly)
  - **10 real-time streaming pipelines** (continuous)
- These pipelines use **Azure Data Factory’s Databricks notebook/activity** connector to trigger and monitor the jobs.

---

# Sample cluster configuration for processing 10 TB of data (in approximately 2 hours):-

Here’s the **10 TB** Databricks (Azure) setup summary (based on `Standard_E32ds_v5`: 256 GB RAM, 32 vCPUs per node):

* **Number of clusters:** 1 (job cluster)
* **Master (driver) nodes:** 1
* **Worker nodes:** 100
* **Executors per worker:** 4
* **Cores per executor:** 8
* **RAM per core (executor-level):** \~6.5 GB (≈52 GB ÷ 8)
* **CPU cores per core:** 1 vCPU per core
* **Total cluster RAM (driver + workers):** \~25.9 TB (≈101 × 256 GB)
* **Total cluster CPU cores (driver + workers):** 3,232 vCPUs

  * **Task-capable cores (executors only):** 3,200 (400 executors × 8 cores)

------

# In general more time is allowd to process all 10 TB of data in 1 day. So we can say following in interview:-

- we generally use standard E series E32 machines (nodes) which have 256 GB RAM and 32 CPU cores.
- we have such 40 (workder) nodes in our cluster. with autoscaling capacity between 30 to 40 nodes.

If further asked, then we can say:-

- each of the worker nodes have 4 executors running.
- each executor have 4 cores.

----
