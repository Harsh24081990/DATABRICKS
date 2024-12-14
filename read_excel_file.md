# using spark-excel library
To read an Excel sheet in **Databricks** using **PySpark**, you'll need to use the `spark-excel` library. This library provides the ability to read Excel files (both `.xls` and `.xlsx` formats) into PySpark DataFrames.

- Note : after installing the spark-excel library in Databricks (or any Spark environment), you do not need to explicitly import the library in your PySpark code. The library is automatically made available once it is installed via Maven.

Here’s how you can do it:

### Steps to Read Excel Sheets in Databricks using PySpark:

1. **Install the `spark-excel` Library**:
   You first need to install the `spark-excel` library, which is available through Maven. You can install it by adding the Maven coordinates in the Databricks cluster.

   - Open your Databricks workspace.
   - Go to the **Clusters** tab and select your cluster.
   - Under the **Libraries** tab, click on **Install New**.
   - Choose **Maven** and enter the following Maven coordinates:
     ```
     com.crealytics:spark-excel_2.12:0.13.5
     ```
     This installs the necessary library for reading Excel files.

2. **Read the Excel File in PySpark**:
   After installing the library, you can use the following PySpark code to read an Excel file into a DataFrame.

### Example Code:

```python
from pyspark.sql import SparkSession

# Initialize Spark session (This is usually done automatically in Databricks)
spark = SparkSession.builder.appName("ReadExcel").getOrCreate()

# Define the file path (make sure the file is uploaded to DBFS or is accessible from a mounted path)
excel_file_path = "/dbfs/path/to/your/excel/file.xlsx"

# Read the Excel file into a DataFrame
df = spark.read.format("com.crealytics.spark.excel") \
    .option("sheetName", "Sheet1") \
    .option("useHeader", "true") \
    .option("inferSchema", "true") \
    .load(excel_file_path)

# Show the DataFrame schema and first few rows
df.printSchema()
df.show()
```

### Explanation of the Code:
1. **`spark.read.format("com.crealytics.spark.excel")`**:
   - This specifies that you want to use the `spark-excel` library to read the Excel file.

2. **Options**:
   - **`sheetName`**: Specifies which sheet to read. If you have multiple sheets in your Excel file, you can specify the sheet name you want to load.
   - **`useHeader`**: When set to `true`, this option tells Spark to treat the first row as headers for the columns.
   - **`inferSchema`**: When set to `true`, Spark automatically infers the data types of each column based on the data. This helps avoid reading everything as strings.
   
3. **`load(excel_file_path)`**: Reads the Excel file from the given file path.

4. **`df.printSchema()`**: Displays the schema of the DataFrame so you can verify the data types inferred by Spark.

5. **`df.show()`**: Displays the first few rows of the DataFrame.

### Notes:
- Ensure the file path is correct. If you are using Databricks File System (DBFS), the path should be prefixed with `/dbfs/`, like `/dbfs/mnt/data/excel/file.xlsx`.
- If your Excel file contains multiple sheets and you need to load them all, you can specify the sheet name in the `.option("sheetName", "SheetName")`. If you want to read all sheets, you might need to load them one by one.

### Example of Loading Multiple Sheets:
If you have an Excel file with multiple sheets and want to read all of them, you can read each sheet separately by changing the `sheetName` option:

```python
# Read first sheet
df_sheet1 = spark.read.format("com.crealytics.spark.excel") \
    .option("sheetName", "Sheet1") \
    .option("useHeader", "true") \
    .option("inferSchema", "true") \
    .load(excel_file_path)

# Read second sheet
df_sheet2 = spark.read.format("com.crealytics.spark.excel") \
    .option("sheetName", "Sheet2") \
    .option("useHeader", "true") \
    .option("inferSchema", "true") \
    .load(excel_file_path)

# Show DataFrames
df_sheet1.show()
df_sheet2.show()
```
---------------------------------------------------------
# Alternative Approach (Using Pandas with PySpark):

If you're looking for a simpler way to load Excel files (especially small files), you can use **Pandas** to read the Excel file and then convert it to a Spark DataFrame.

```python
import pandas as pd

# Read the Excel file using pandas
excel_df = pd.read_excel("/dbfs/path/to/your/excel/file.xlsx", sheet_name="Sheet1")

# Convert pandas DataFrame to PySpark DataFrame
spark_df = spark.createDataFrame(excel_df)

# Show the Spark DataFrame
spark_df.show()
```

However, note that this approach might not scale as well as the `spark-excel` approach for large datasets.

---

### Conclusion:

To read Excel files in Databricks using PySpark:
1. Install the `spark-excel` library.
2. Use `spark.read.format("com.crealytics.spark.excel")` to load Excel files as DataFrames.
3. Specify the sheet name, and optionally, `useHeader` and `inferSchema` options to handle the data appropriately.

-------------------------------------------------------
In **Databricks**, **Pandas** is **pre-installed** by default. You do not need to install it manually in most cases, as it is included in the default Python environment for Databricks clusters.

You can verify the installation by simply importing `pandas` and checking the version:

```python
import pandas as pd
print(pd.__version__)
```

If for any reason you need to install or update the **Pandas** library, you can do so by running the following command in a **Databricks notebook** cell:

### To install or update Pandas:

```python
%pip install pandas
```

### Explanation:
- The `%pip` magic command is used in **Databricks** to install Python packages within notebooks. It's similar to running `pip install` in a terminal.
- This installs the latest version of **Pandas**. If you need to specify a specific version, you can do so like this:

```python
%pip install pandas==1.3.0
```

After installation, you can verify the installation by importing Pandas again:

```python
import pandas as pd
print(pd.__version__)
```

### Summary:
- **Pandas** comes pre-installed in **Databricks**, so you usually don’t need to install it.
- If you need to install or update **Pandas**, use the `%pip install pandas` command.
