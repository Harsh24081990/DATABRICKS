---

## ✅ Cluster & Data Size in Databricks Project

---

### **What is the cluster and data size in your project?**

In our project, we use **Azure Databricks** as the core data processing platform. Our **Databricks clusters** are configured to handle both batch and real-time processing of **claims and loan data** for a banking client.

---

### **Cluster Configuration (Databricks)**

- We primarily use **job clusters** (ephemeral, created per job) for production pipelines and **interactive clusters** for development and testing.
- The **default cluster size** is set to **12 nodes**, each with **16 vCPUs and 64 GB RAM**.
- During periods of high workload — such as **month-end processing or heavy data reconciliation** — the cluster is **scaled up to 20 nodes** using **autoscaling**, or manually when we expect peak load.
- We leverage **autoscaling**, **auto-termination**, and **Databricks Runtime 11.3 LTS** for stability and performance.
- **Photon Engine** is currently not enabled, as our workload benefits more from classic Spark-based APIs.

---

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

### **Use Case for Scaling Up the Cluster**

One key scenario is during the **monthly loan settlement and claims reconciliation process**. These jobs aggregate and process transactional claims, calculate interest, apply business logic, and generate compliance reports. During these workloads, we scale the cluster from **12 to 20 nodes** to ensure the jobs complete within defined SLAs.

**Steps followed in Databricks:**

1. **Autoscaling Configuration**: Most job clusters are created with autoscaling enabled (e.g., min 10, max 20 nodes).
2. **Manual Scaling (if needed)**:
   - Go to **Clusters > Edit cluster**
   - Update the **Worker Node Range** (e.g., min 10 → max 20)
   - Click **Confirm and Restart** if it’s an interactive cluster
3. **Run the ETL pipelines**:
   - Pipelines ingest and process claims and loan data
   - Perform cleansing, joins, aggregations, validations
   - Output is written back to Delta or Parquet on ADLS Gen2

---

### **Environments**

- **Production**: 12–20 nodes with autoscaling enabled.
- **Dev/UAT**: 3–5 nodes clusters for lower-cost development and testing.
- Each environment has a **separate Databricks workspace and URL**, enabling isolation and access control.

---

### **Pipeline Summary**

- We run around **60 ADF pipelines**, orchestrating the Databricks jobs.
  - **50 batch pipelines** (daily/monthly)
  - **10 real-time streaming pipelines** (continuous)
- These pipelines use **Azure Data Factory’s Databricks notebook/activity** connector to trigger and monitor the jobs.

---

### 📝 Additional Notes

- RAM (64 GB/node) is used for in-memory processing in Spark.
- Data is read from and written to **external ADLS storage**, separate from compute memory.
- Storage scales independently of the Spark cluster.

---
