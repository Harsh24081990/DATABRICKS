To access **Databricks cluster details using Azure CLI**, you need to interact with the **Databricks REST API**, since Azure CLI doesn’t directly expose Databricks cluster commands.

---

### ✅ Steps to Access Cluster Details via Azure CLI:

#### 1. **Get a Databricks PAT (Personal Access Token)**

From Databricks UI:
→ **User Settings** → **Access Tokens** → Generate token.

---

#### 2. **Set required variables**

```bash
# Replace with your actual values
DATABRICKS_HOST="https://<databricks-instance>.azuredatabricks.net"
DATABRICKS_TOKEN="your_pat_token"
```

---

#### 3. **Get cluster list**

```bash
curl -X GET "$DATABRICKS_HOST/api/2.0/clusters/list" \
     -H "Authorization: Bearer $DATABRICKS_TOKEN"
```

---

#### 4. **Get details of a specific cluster**

```bash
curl -X GET "$DATABRICKS_HOST/api/2.0/clusters/get?cluster_id=<your_cluster_id>" \
     -H "Authorization: Bearer $DATABRICKS_TOKEN"
```

---

### 🔁 Using Databricks CLI (Alternative)

1. Install & configure Databricks CLI:

```bash
pip install databricks-cli
databricks configure --token
```

2. Then run:

```bash
databricks clusters list
databricks clusters get --cluster-id <cluster-id>
```

Let me know if you want to script this or automate using Azure DevOps or ADF.
