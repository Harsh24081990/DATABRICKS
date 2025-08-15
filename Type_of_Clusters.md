**In Spark (open-source Apache Spark)** — broadly 3 types based on how they run:

1. **Standalone Cluster**

   * Spark’s built-in cluster manager.
   * Simple, no external dependency.

2. **YARN Cluster**

   * Uses Hadoop YARN as the resource manager.
   * Common in on-prem and Hadoop environments.

3. **Mesos / Kubernetes Cluster**

   * Mesos (less common now) or Kubernetes for containerized deployments.

---

**In Databricks** — clusters are classified differently:

1. **Interactive Clusters** (a.k.a. All-purpose clusters)

   * Created manually from the **Compute** pane.
   * Used for developing, debugging, running ad-hoc queries, notebooks.
   * Stay alive until you terminate them.

2. **Job Clusters**

   * Created automatically by a **job** (e.g., scheduled or triggered from ADF).
   * Spins up before the job, shuts down after completion.
   * Config defined in the job JSON.

3. **High Concurrency Clusters**

   * Special type of interactive cluster allowing multiple users/notebooks to share resources efficiently.
   * Often used for BI tools like Power BI + Databricks SQL Analytics.

---

Here’s the mapping between **Databricks cluster types** and the **underlying Spark cluster managers** they use internally:

| **Databricks Cluster Type**   | **Purpose**                               | **Underlying Spark Cluster Manager**                                                 | **Notes**                                                |
| ----------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| **Interactive (All-purpose)** | Development, exploration, debugging       | **Databricks’ own cluster manager** (built on top of Apache Spark’s Standalone mode) | Managed by Databricks; you don’t see YARN/Mesos.         |
| **Job Cluster**               | Automated jobs (ADF, Databricks Jobs API) | **Databricks’ own cluster manager**                                                  | Created per job run; automatically terminated after run. |
| **High Concurrency Cluster**  | Multi-user, BI tool connections           | **Databricks’ own cluster manager**                                                  | Uses fine-grained resource sharing + security isolation. |

💡 **Key point:**
Even though Spark itself supports **Standalone**, **YARN**, **Kubernetes**, and **Mesos**, in **Azure Databricks** you don’t directly choose these — you always get **Databricks’ managed Standalone-like cluster manager**, fully abstracted from you.

