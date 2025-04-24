## ✅ Step 1: Register App in **Entra ID** / Create an Azure AD (Entra ID) Service Principal
[ Note : A Service Principal is the instance of that app created in specific tenant (directory) ]

#### Register App:-
	- Go to **Microsoft Entra ID** → **App registrations** → Click **+ New registration**
	- → Give it a name (e.g., `databricks_sp_app`) → Click **Register**

#### Create Client secret (secret value for app/SP):-
	- Go to `databricks_sp_app` → **Certificates & secrets** → Click **New client secret**
	- Save the **secret value** generated in a notepad. 
   
#### Note down the following:-
   - **Application (client) ID** [username] --> This is the unique identifier for your registered app / Service Principal.
   - **Directory (tenant) ID** [directory]--> This is the unique identifier of your Microsoft Entra ID.
   - **Secret value (Client secret)** [password]--> This is the password created for the registered app / Service Principal.

---

## ✅ Step 2: **Assign required Storage Role to App / SP** in IAM of storage account. 
[Assign required Permissions to App or service principal (via IAM) to be able to access ADLS]

1. Go to your **Azure Data Lake Storage** account in the portal.
2. Navigate to **Access Control (IAM)** → **Add role assignment**. Give below details:-
   - **Role**: Choose `Storage Blob Data Reader` OR `Storage Blob Data Contributor`
   - **Scope**: Choose **Storage account level** (Recommended) OR specific **container level** (if restrictive access needed)
   - **Assign access to**: **User, group, or service principal**
   - Search & select our SP: `databricks_sp_app`
3. Save

---

## ✅ Step 3: **Setup Azure Key Vault**

### 🔹 **1. Add Secrets to Azure Key Vault**
- Go to your **Key Vault** → **Secrets** → click on **+ Generate/Import** to create new secrets. 
   - `client-id` → paste **Application (Client) ID**
   - `tenant-id` → paste **Directory (Tenant) ID**
   - `client-secret` → paste the **secret value**
   - `storage-account-name` → (optional) save your ADLS account name

**Make note of following Key vault properties**
- Key Vault DNS Name (Vault URI)
https://<your-keyvault-name>.vault.azure.net/ : This tells Databricks where the Key Vault is.
- Key Vault Resource ID


### 🔹 **2. Give Databricks workspace, Access to Azure Key Vault**
- Go to your **Key Vault** → **Access Policies**
	- Click **+ Add Access Policy**
	- Select **Secret Permissions**: `Get`
	- Assign principal: Search and select your **Databricks Workspace's Managed Identity** (Ex. my-databricks-ws-name)
	- Click **Save**

##### NOTE : 
- We register the Databricks app in Entra ID to access ADLS securely.
- We give our Databricks workspace, access to Azure Key Vault, so it can retrieve that app's credentials in a secure, automated way. 
---

## ✅ Step 4: **Create Secret Scope in Databricks**
- You create a secret scope in Databricks using the UI: In Azure databricks URL, at the end type --> #secreats/createScope.
- Give below details:- 

	- Scope name: e.g., my_databricks_secret_scope
	- Backed by: Azure Key Vault
	- Provide **`DNS name`** (Vault URI) Vault **`Resource ID`** at the time of scope creation.

 ![image](https://github.com/user-attachments/assets/88de3d40-d81c-4f46-8e7c-49cfe166a832)


#### Now, Databricks can securely access secrets stored in Azure Key Vault. In the notebook, you just use:
```
dbutils.secrets.get(scope="my_databricks_secret_scope", key="client-id")
```
---

## ✅ Step 5: **Create ADLS Mount Point in Databricks Notebook** to be able to access ADLS from databricks notebook**

```
configs = {
  "fs.azure.account.auth.type": "OAuth",
  "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
  "fs.azure.account.oauth2.client.id": dbutils.secrets.get(scope="my_databricks_secret_scope", key="client-id"),
  "fs.azure.account.oauth2.client.secret": dbutils.secrets.get(scope="my_databricks_secret_scope", key="client-secret"),
  "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<tenant-id>/oauth2/token"
}

dbutils.fs.mount(
  source = "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/",
  mount_point = "/mnt/<your-mount-name>",
  extra_configs = configs
)
```
> Replace `<container-name>`, `<storage-account-name>`, and `<your-mount-name>` accordingly.
-------------

### ✅ Now You Can:

- Access data via: `/mnt/myadlsmount`
- Rotate secrets via Key Vault without touching Databricks
- Stay compliant with enterprise-grade security

Ex. 
list all the files in ADLS location:-
```
dbutils.fs.ls("/mnt/<your-mount-name>")
```
Read a CSV/Parquet/JSON File present at ADLS Using Spark
```
df = spark.read.parquet("/mnt/<your-mount-name>/sales/data.parquet")
df.printSchema()
df.show()
```
-------------------

