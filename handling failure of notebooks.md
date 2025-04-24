A common and effective approach to handle failure scenarios, restart pipelines from the failed point, and support incremental loading in Azure Data Factory (ADF) using Azure Databricks (ADB) notebooks is using a Metadata Table (Control Table).

Metadata Table Approach

- 1. Control Table Design

Create a metadata/control table in a database (SQL DB, Synapse, etc.) with columns like:

PipelineName

TableName

LastLoadedTimestamp or LastLoadedID

Status (e.g., Success, Failed, InProgress)

NotebookPath

#### These values are dynamically passed from ADF using parameters, or retrieved/set inside Databricks notebooks via widgets or runtime logic. Nothing is "auto-filled" — you define them as part of your pipeline orchestration.

- 2. Incremental Load Logic

In the ADB notebook:

Read the LastLoadedTimestamp from the control table.

Load only data greater than that timestamp.

After successful processing, update the LastLoadedTimestamp in the control table.


- 3. Failure Handling in ADF

Use Try-Catch in the notebook to catch exceptions and update the control table status to "Failed".

In ADF, check this status (using Lookup + If Condition) to determine if the load should proceed or be retried.


4. Restart from Failed Point

When re-running the pipeline, it reads the LastLoadedTimestamp from the control table.

Since data is filtered based on this value, it resumes from where it failed, without reloading all data.


