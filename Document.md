Making a guide readable for GitHub often means using **Markdown effectively**, breaking down complex steps, and emphasizing code blocks for clarity. Here's your provided guide refactored for better GitHub readability:

-----

# End-to-End Azure Data Engineering Project: On-Premises to Power BI

A huge thank you to **Luke J Byrne** (Luke J Byrne Channel) for his invaluable insights and guidance\! You can also refer to his excellent video tutorial on building an end-to-end Azure data engineering project: [End-to-End Azure Data Engineering Project](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3D2e5XQ8K7G5A).

This guide walks you through building a complete data engineering pipeline on **Microsoft Azure**. Our goal is to migrate data from an **on-premises SQL Server** to Azure, transform it, and visualize it in **Power BI**.

-----

## Project Overview

Here's a quick overview of what we'll cover:

  * **Extracting data** from your on-premises SQL database.
  * **Loading raw data** into Azure Data Lake Gen2 (Bronze layer).
  * **Transforming the data** in Azure Databricks (Silver and Gold layers).
  * **Loading the transformed data** into Azure Synapse Analytics for querying.
  * **Visualizing the final data** in Power BI.
  * **Automating the entire pipeline** for scheduled runs.
  * **Implementing security and governance** using Microsoft Entra ID and Azure Key Vault.

-----

### Project Architecture Flow

The data flows logically through several Azure services:

**On-Premises SQL Database** $\\rightarrow$ **Azure Data Factory** (for ingestion) $\\rightarrow$ **Azure Data Lake (Bronze)** $\\rightarrow$ **Azure Databricks** (for transformations across Bronze, Silver, Gold layers) $\\rightarrow$ **Azure Synapse Analytics** (for structured querying) $\\rightarrow$ **Power BI** (for visualization).

**Security** (Microsoft Entra ID, Key Vault) and **governance** are woven throughout the entire pipeline.

-----

## Setting Up Your Environment

Before we dive into the data, let's get your on-premises and Azure environments ready.

-----

### On-Premises Setup (SQL Database)

1.  **Get Sample Data:** We'll use Microsoft's **AdventureWorksLT 2022 dataset**, a lightweight data warehouse version with common business data.
2.  **Install SQL Engine:** You'll need **SQL Server Express** (free, lightweight) to host the database.
3.  **Install SQL Management Tool:** Use **SQL Server Management Studio (SSMS)** to interact with and manage your SQL Server instance.
4.  **Restore the Database:**
      * Download the AdventureWorksLT 2022 `.bak` file.
      * Copy it to your SQL Server backups folder (e.g., `C:\Program Files\Microsoft SQL Server\...\`).
      * In SSMS, connect to your SQL Server Express instance, right-click 'Databases', and select 'Restore Database'. Choose the `.bak` file and restore.
5.  **Create a SQL User for Data Factory:** Data Factory needs credentials to access your database.
      * In SSMS, open a new query on `AdventureWorksLT2022`.
      * **Create Login and User:**
        ```sql
        CREATE LOGIN lookto WITH PASSWORD = 'YourSecurePassword123!';
        CREATE USER lookto FOR LOGIN lookto;
        ```
      * **Grant SELECT Access:**
        ```sql
        USE AdventureWorksLT2022;
        GRANT SELECT ON SCHEMA :: SalesLT TO lookto;
        ```
      * **Enable SQL Authentication:** In SSMS, right-click your server instance $\\rightarrow$ 'Properties' $\\rightarrow$ 'Security'. Ensure 'SQL Server and Windows Authentication mode' is selected. You might need to restart SSMS or the SQL Server service.

-----

### Azure Setup

You'll need a **Microsoft Azure account** (a free trial works great).

1.  **Create a Resource Group:**
      * Navigate to Azure portal $\\rightarrow$ Resource Groups $\\rightarrow$ Create.
      * Name it (e.g., `intch-RG`), select your subscription and region (e.g., UK South). This groups all your project resources.
2.  **Create Azure Data Lake Gen2 Storage Account:** This will be your central raw data repository.
      * Go to Storage Accounts $\\rightarrow$ Create.
      * Select your Resource Group, provide a name (e.g., `intchsg`).
      * On the 'Advanced' tab, enable 'Hierarchical namespace' – this is what makes it a **Data Lake Gen2**.
      * After creation, add three containers: **Bronze**, **Silver**, and **Gold** (representing data transformation stages).
3.  **Create Azure Data Factory:** For orchestrating your data pipelines.
      * Go to Data Factory $\\rightarrow$ Create.
      * Select your Resource Group, provide an instance name (e.g., `intch-DF`).
4.  **Create Azure Databricks:** An Apache Spark-based platform for data cleaning and transformation.
      * Go to Databricks $\\rightarrow$ Create.
      * Select your Resource Group, provide a workspace name (e.g., `intch-databricks`).
5.  **Create Azure Synapse Analytics:** For integrated analytics and querying transformed data.
      * Go to Synapse $\\rightarrow$ Create.
      * Select your Resource Group, provide a workspace name (e.g., `intch-synapse`).
      * Choose your Data Lake Gen2 account and create a new file system for Synapse (e.g., `synapse-dfs`).
      * *Note: Register the `Microsoft.Synapse` resource provider if not already done (Subscriptions $\\rightarrow$ Resource Providers).*
6.  **Create Azure Key Vault:** Securely store sensitive information like passwords.
      * Go to Key Vault $\\rightarrow$ Create.
      * Select your Resource Group, provide a name (e.g., `intch-kv`).
      * After creation, go to 'Secrets' and generate/import a secret for your SQL user's password (e.g., `name: sql-password`, `value: YourSecurePassword123!`).

-----

## Ingestion (Azure Data Factory)

Now, let's use Data Factory to pull data from your on-premises SQL database and load it into the Bronze layer of your Data Lake.

1.  **Launch Azure Data Factory Studio:** From your Data Factory resource in the Azure portal, click 'Launch studio'.

-----

### Create Linked Services

These define connections to your data sources.

1.  **Linked Service for On-Prem SQL:**
      * In Data Factory Studio $\\rightarrow$ 'Manage' $\\rightarrow$ 'Linked services' $\\rightarrow$ '+ New'.
      * Select 'SQL Server'. Name it (e.g., `SqlServerOnPremLinkedService`).
      * For 'Connect via integration runtime', choose 'Self-hosted'. You'll need to create and install a **Self-hosted Integration Runtime (SHIR)** on your on-premises machine that can access the SQL server.
      * Enter your SQL Server name and Database name.
      * Choose 'SQL Authentication', enter `lookto` as the Username.
      * For Password, select your Azure Key Vault linked service (create one if you haven't) and choose the `sql-password` secret.
      * Ensure TLS encryption is set to 'Optional'. **Test connection.**
2.  **Linked Service for Azure Data Lake Gen2:**
      * In Data Factory Studio $\\rightarrow$ 'Manage' $\\rightarrow$ 'Linked services' $\\rightarrow$ '+ New'.
      * Select 'Azure Data Lake Storage Gen2'. Name it (e.g., `AzureDataLakeStorageGen2LinkedService`).
      * Use `AutoResolveIntegrationRuntime`.
      * **Recommended:** Authenticate using 'System assigned managed identity' and grant your Data Factory's managed identity **'Storage Blob Data Contributor'** permissions on the storage account in Azure portal.
      * Select your storage account. **Test connection.**
3.  **Linked Service for Azure Key Vault:**
      * In Data Factory Studio $\\rightarrow$ 'Manage' $\\rightarrow$ 'Linked services' $\\rightarrow$ '+ New'.
      * Select 'Azure Key Vault'. Name it (e.g., `AzureKeyVaultLinkedService`).
      * Select your subscription and Key Vault name.
      * Grant your Data Factory's managed identity **'Get'** permissions on secrets in Key Vault (Key Vault $\\rightarrow$ Access control (IAM) $\\rightarrow$ Add role assignment: 'Key Vault Secrets User'). **Test connection.**

-----

### Create Datasets

These represent the data structure within your linked services.

1.  **Source Dataset (On-Prem SQL):**
      * In Data Factory Studio $\\rightarrow$ 'Author' $\\rightarrow$ 'Datasets' $\\rightarrow$ '+ New'.
      * Select 'SQL Server'. Name it (e.g., `SqlServerOnPremDataset`).
      * Select `SqlServerOnPremLinkedService`. Do **NOT** select a specific table name yet.
2.  **Sink Dataset (Data Lake Gen2):**
      * Select 'Azure Data Lake Storage Gen2'. Choose **Parquet** format (optimized for querying).
      * Name it (e.g., `AzureDataLakeStorageGen2Parquet`). Select `AzureDataLakeStorageGen2LinkedService`.
      * Add parameters `schema_name` and `table_name`.
      * In the 'Connection' tab, use dynamic content for the File path:
          * **Directory:** `@concat('bronze/', dataset().schema_name, '/', dataset().table_name)`
          * **File name:** `@concat(dataset().table_name, '.parquet')`

-----

### Create a Pipeline to Copy All Tables

This pipeline will dynamically copy tables.

1.  In Data Factory Studio $\\rightarrow$ 'Author' $\\rightarrow$ 'Pipelines' $\\rightarrow$ '+ New'. Name it (e.g., `CopyAllTablesPipeline`).
2.  **Add a Lookup activity:**
      * Drag 'Lookup' from 'General'. Name it `GetTableNames`.
      * In 'Settings', select `SqlServerOnPremDataset`. Uncheck 'First row only'.
      * Select 'Query' and use this SQL to get schema and table names from `SalesLT` schema:
        ```sql
        SELECT s.name AS schema_name, t.name AS table_name
        FROM sys.tables AS t
        INNER JOIN sys.schemas AS s ON t.schema_id = s.schema_id
        WHERE s.name = 'SalesLT';
        ```
3.  **Add a ForEach activity:**
      * Drag 'ForEach' from 'Iteration & Conditionals'. Connect `GetTableNames` to it.
      * In 'Settings', for 'Items', use dynamic content: `@activity('GetTableNames').output.value`.
4.  **Add a Copy Data activity inside ForEach:**
      * Click the pencil icon on the ForEach activity. Drag 'Copy data'. Name it `CopyData`.
      * **Source Tab:** Select `SqlServerOnPremDataset`.
          * Select 'Query' for source, and use dynamic content: `@concat('SELECT * FROM ', item().schema_name, '.', item().table_name)`.
      * **Sink Tab:** Select `AzureDataLakeStorageGen2Parquet` dataset.
          * Under 'Sink Dataset properties', set `schema_name` to `@item().schema_name` and `table_name` to `@item().table_name`.
5.  **Publish all changes** in Data Factory. **Debug** the pipeline to test if data copies to the Bronze container.

-----

## Transformation (Azure Databricks)

Next, we'll use Databricks to clean and transform data from the Bronze layer to Silver (cleaned, basic transformations) and Gold (aggregated, query-ready) layers.

1.  **Launch Databricks Workspace:** From your Databricks resource in Azure portal, click 'Launch Workspace'.

-----

### Create a Cluster

Databricks needs compute resources.

1.  In Databricks workspace $\\rightarrow$ 'Compute' $\\rightarrow$ 'Create Compute'.
2.  Name it (e.g., `intch-single-node-cluster`).
3.  For trial accounts, use 'Single Node' access mode.
4.  Choose a Databricks Runtime Version.
5.  Configure auto-termination (e.g., 30 minutes) to save costs.
6.  Under 'Advanced Options', enable **'Credential passthrough for user-level data access'**. This simplifies Data Lake access from notebooks.

-----

### Mount Data Lake Storage (or use `abfss://` directly)

1.  Create a new notebook (e.g., `storage_access`) in your workspace (Python). Connect it to your cluster.
2.  With credential passthrough, you can often directly access data using the `abfss://` protocol, which is simpler and more secure than explicit mounts or keys.
      * **Example Path Structure:**
        ```python
        storage_account_name = "your_storage_account_name"
        bronze_path = f"abfss://bronze@{storage_account_name}.dfs.core.windows.net/SalesLT/" # Adjust based on your ADF sink
        silver_path = f"abfss://silver@{storage_account_name}.dfs.core.windows.net/SalesLT/"
        gold_path = f"abfss://gold@{storage_account_name}.dfs.core.windows.net/SalesLT/"
        ```
      * You can list contents using `dbutils.fs.ls(bronze_path)`.

-----

### Perform Transformations (Bronze to Silver Notebook)

1.  Create a new notebook (e.g., `BronzeToSilver`).
2.  **Read data from Bronze** (e.g., `Address` table):
    ```python
    from pyspark.sql.functions import col, date_format

    storage_account_name = "your_storage_account_name"
    table_name = "Address" # Example table
    bronze_file_path = f"abfss://bronze@{storage_account_name}.dfs.core.windows.net/SalesLT/{table_name}/{table_name}.parquet"

    df_bronze = spark.read.parquet(bronze_file_path)
    df_bronze.display()
    ```
3.  **Transformations:** (e.g., standardizing dates)
    ```python
    df_silver = df_bronze.withColumn("ModifiedDate", date_format(col("ModifiedDate").cast("timestamp"), "yyyy-MM-dd"))
    df_silver.display()
    ```
4.  **Write data to Silver:** Use **Delta Lake format** for ACID transactions and versioning.
    ```python
    silver_table_path = f"abfss://silver@{storage_account_name}.dfs.core.windows.net/SalesLT/{table_name}"
    df_silver.write.format("delta").mode("overwrite").save(silver_table_path)
    ```
5.  **Automate:** Adapt this notebook to iterate through all tables dynamically.

-----

### Perform Transformations (Silver to Gold Notebook)

1.  Create another notebook (e.g., `SilverToGold`).
2.  **Read data from Silver:**
    ```python
    df_silver = spark.read.format("delta").load(silver_table_path)
    df_silver.display()
    ```
3.  **Transformations:** (e.g., renaming columns to snake\_case)
    ```python
    import re

    def to_snake_case(name):
      s1 = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
      return re.sub('([a-z0-9])([A-Z])', r'\1_\2', s1).lower()

    def rename_columns_to_snake_case(df):
        old_columns = df.columns
        new_columns = [to_snake_case(col) for col in old_columns]
        renamed_df = df
        for old_name, new_name in zip(old_columns, new_columns):
            renamed_df = renamed_df.withColumnRenamed(old_name, new_name)
        return renamed_df

    df_gold = rename_columns_to_snake_case(df_silver)
    df_gold.display()
    ```
4.  **Write data to Gold:**
    ```python
    gold_table_path = f"abfss://gold@{storage_account_name}.dfs.core.windows.net/SalesLT/{table_name}"
    df_gold.write.format("delta").mode("overwrite").save(gold_table_path)
    ```
5.  **Automate:** Adapt this notebook to process all tables dynamically.

-----

### Integrate Databricks Notebooks into Data Factory

1.  In Data Factory Studio, open your `CopyAllTablesPipeline`.
2.  Add **'Databricks Notebook'** activities. Connect the 'ForEach' (copying to Bronze) to `BronzeToSilver` (on success), and `BronzeToSilver` to `SilverToGold` (on success).
3.  **Configure Notebook activities:**
      * **Linked Service:** Create a new linked service to Databricks. Choose your workspace, select 'Access Token' for authentication.
      * **Generate Databricks Access Token:** In Databricks workspace $\\rightarrow$ User Settings $\\rightarrow$ Developer settings $\\rightarrow$ Access tokens. Generate a new token and store it securely in Azure Key Vault (e.g., `databricks-token`). Grant Data Factory's managed identity **'Get'** permissions on this secret.
      * Point the Data Factory Linked Service for Databricks to this Key Vault secret.
      * **Cluster:** Select your existing cluster.
      * **Settings:** Browse and select the respective notebook file (`BronzeToSilver` or `SilverToGold`).
4.  **Publish** the pipeline in Data Factory. **Debug** to test the full end-to-end process.

-----

## Loading (Azure Synapse Analytics)

Synapse Analytics will create a querying layer over your Gold data in the Data Lake, making it accessible for Power BI.

1.  **Launch Azure Synapse Analytics Studio:** From your Synapse workspace in Azure portal, click 'Open Synapse Studio'.

-----

### Create a SQL Database (Serverless)

1.  In Synapse Studio $\\rightarrow$ 'Data' tab $\\rightarrow$ '+', select 'SQL database', choose 'Serverless'.
2.  Name it (e.g., `GoldDB`).

-----

### Query Data in Data Lake from Synapse

1.  In Synapse Studio $\\rightarrow$ 'Data' tab $\\rightarrow$ 'Linked' $\\rightarrow$ browse to your gold container.
2.  Right-click a table folder (e.g., `Address`) $\\rightarrow$ 'New SQL script' $\\rightarrow$ 'Select TOP 100 rows'. This generates an `OPENROWSET` query for Delta data. Run it to test.

-----

### Create Views for Data

These views provide a persistent, easy-to-query representation for Power BI.

1.  Modify the `OPENROWSET` script to create a view in `GoldDB`. Example for one table:
    ```sql
    USE GoldDB;

    CREATE OR ALTER VIEW SalesLT_AddressView AS
    SELECT
        *
    FROM
        OPENROWSET(
            BULK 'https://your_storage_account_name.dfs.core.windows.net/gold/SalesLT/address/',
            FORMAT = 'DELTA'
        ) AS [result];
    ```
2.  Run this for each table you want to visualize.

-----

### Create a Stored Procedure to Dynamically Create Views

Automate view creation for all tables.

1.  In Synapse Studio $\\rightarrow$ 'Develop' tab $\\rightarrow$ '+', select 'SQL script'.
2.  Enter this stored procedure (replace `your_storage_account_name`):
    ```sql
    USE GoldDB;

    CREATE OR ALTER PROC CreateSQLServerlessView_gold @ViewName NVARCHAR(100)
    AS
    BEGIN
        DECLARE @statement VARCHAR(MAX);
        SET @statement = N'CREATE OR ALTER VIEW ' + @ViewName + ' AS
        SELECT
            *
        FROM
            OPENROWSET(
                BULK ''https://your_storage_account_name.dfs.core.windows.net/gold/SalesLT/' + @ViewName + '/'',
                FORMAT = ''DELTA''
            ) AS [RESULT]';
        EXEC (@statement);
    END;
    ```
3.  **Publish** the script. Run the script to create the stored procedure.

-----

### Create a Synapse Pipeline to Create Views Dynamically

1.  In Synapse Studio $\\rightarrow$ 'Integrate' tab $\\rightarrow$ '+', select 'Pipeline'.
2.  **Add a Get Metadata activity:**
      * Drag 'Get Metadata' from 'General'. Name it `GetGoldTableNames`.
      * In 'Settings', create a Dataset pointing to your Gold container path (e.g., `abfss://gold@your_storage_account_name.dfs.core.windows.net/SalesLT/`). Select 'Binary' format.
      * Under 'Field list', select 'Child items'.
3.  **Add a ForEach activity:**
      * Drag 'ForEach'. Connect `GetGoldTableNames` to it.
      * In 'Settings', for 'Items', use dynamic content: `@activity('GetGoldTableNames').output.childItems`.
4.  **Add a Stored Procedure activity inside ForEach:**
      * Click the pencil on ForEach. Drag 'Stored Procedure' from 'General'.
      * In 'Settings', create a new Linked Service to your Synapse serverless SQL database (`GoldDB`).
      * Authenticate using 'System assigned managed identity' (ensure Synapse workspace's managed identity has permissions on `GoldDB`, e.g., `db_owner` or `db_ddladmin`).
      * Select the stored procedure: `CreateSQLServerlessView_gold`.
      * Set the `@ViewName` parameter using dynamic content: `@item().name`.
5.  **Publish** the pipeline. **Trigger** it to create all views in `GoldDB`.

-----

## Visualization (Power BI)

Finally, we'll connect Power BI to your Synapse SQL database to build an interactive dashboard.

1.  **Get Power BI Desktop:** Download and install Power BI Desktop on a Windows machine. You'll typically need a work/school account to sign in and publish reports.

-----

### Connect to Azure Synapse SQL

1.  Open Power BI Desktop $\\rightarrow$ 'Get Data' $\\rightarrow$ 'More' $\\rightarrow$ 'Azure' $\\rightarrow$ 'Azure Synapse Analytics SQL'.
2.  Enter your **Serverless SQL Endpoint** (from Synapse workspace properties) and **Database Name** (`GoldDB`).
3.  Choose **'Import'** for 'Data Connectivity mode'.
4.  Authenticate using your Microsoft account credentials.

-----

### Load Tables and Views

1.  In the navigator, select the views you created (e.g., `SalesLT_AddressView`). Click 'Load'.

-----

### Manage Relationships

This is crucial for cross-table filtering.

1.  Go to the **Model view** in Power BI Desktop.
2.  Review auto-detected relationships. Delete incorrect ones.
3.  Create relationships by dragging and dropping columns between tables (e.g., `CustomerAddress[address_id]` to `Address[address_id]`). Pay attention to cardinality (One-to-Many, etc.).
      * **Key AdventureWorksLT relationships might include:**
          * `CustomerAddress` (address\_id) $\\rightarrow$ `Address` (address\_id)
          * `CustomerAddress` (customer\_id) $\\rightarrow$ `Customer` (customer\_id)
          * `SalesOrderHeader` (customer\_id) $\\rightarrow$ `Customer` (customer\_id)
          * `SalesOrderDetail` (product\_id) $\\rightarrow$ `Product` (product\_id)
          * `SalesOrderDetail` (sales\_order\_id) $\\rightarrow$ `SalesOrderHeader` (sales\_order\_id)
          * `Product` (product\_category\_id) $\\rightarrow$ `ProductCategory` (product\_category\_id)
          * `Product` (product\_model\_id) $\\rightarrow$ `ProductModel` (product\_model\_id)

-----

### Create Visualizations

1.  Go back to the **Report view**.
2.  Use the 'Visualizations' pane to add charts, cards, and slicers.
3.  **For the project:**
      * **Total Products Sold:** Card visual, count of `ProductID`.
      * **Total Sales Revenue:** Card visual, sum of `LineTotal` from `SalesOrderDetail`.
      * **Gender Split Among Customers:** Donut/Pie chart, use `Title` (e.g., 'Mr.', 'Ms.') from `Customer` table.
      * **Filters:** Add Slicer visuals for 'Product Category Name' and 'Customer Title' (gender).
4.  Arrange and format your visuals to create a user-friendly dashboard.

-----

### Publish to Power BI Service

1.  Save your Power BI Desktop file (`.pbix`).
2.  Click 'Publish' in the 'Home' tab. Select a workspace.
3.  Access your report via `app.powerbi.com` and share it.

-----

## Automation and Security

Finally, let's automate the pipeline for fresh data and secure your resources.

-----

### Automate the Data Factory Pipeline

Schedule your `CopyAllTablesPipeline` to run periodically:

1.  In Data Factory Studio $\\rightarrow$ 'Author' $\\rightarrow$ find your pipeline.
2.  Click 'Add trigger' $\\rightarrow$ 'New/Edit' $\\rightarrow$ '+ New'.
3.  Select 'Schedule' as the type.
4.  Give it a name (e.g., `DailyTrigger`). Configure recurrence (e.g., 'Once a day'), start date/time.
5.  Click 'OK' and then **'Publish'** to activate.
      * *Note: Ensure your on-premises SQL server and SHIR machine are running for successful scheduled runs.*

-----

### Implement Role-Based Access Control (RBAC) with Microsoft Entra ID

RBAC helps manage who has access to what in Azure. You'll primarily assign roles to **Managed Identities** (for Azure services like Data Factory and Synapse) and potentially to specific user groups.

1.  **For Data Factory's Managed Identity:**
      * Go to your Storage Account $\\rightarrow$ 'Access control (IAM)' $\\rightarrow$ 'Add role assignment'.
      * Assign the **'Storage Blob Data Contributor'** role to your Data Factory's system-assigned managed identity.
      * Similarly, grant **'Key Vault Secrets User'** or **'Key Vault Secrets Officer'** role on your Key Vault to your Data Factory's managed identity.
2.  **For Synapse Workspace's Managed Identity:**
      * Grant appropriate permissions (e.g., `db_owner` or `db_ddladmin` for view creation) on your `GoldDB` serverless SQL database to your Synapse workspace's managed identity. This is done via SQL scripts executed in Synapse Studio. Example:
        ```sql
        USE GoldDB;
        CREATE USER [your_synapse_workspace_name] FROM EXTERNAL PROVIDER;
        ALTER ROLE db_ddladmin ADD MEMBER [your_synapse_workspace_name];
        ```
3.  **For Power BI Users:**
      * Users who need to access the Synapse SQL database from Power BI will need appropriate permissions. You can grant them roles like `db_datareader` on `GoldDB` via SQL.
      * For external users, you'll manage sharing through the Power BI Service.

-----

I hope this structured and more markdown-friendly version is perfect for your GitHub repository\! Let me know if you'd like any other adjustments.
