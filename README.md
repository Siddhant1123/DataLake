# DataLake
# Data Ingestion, Transformation, and Storage in Azure

This project focuses on ingesting, transforming, and storing data within **Azure Data Lake Storage (ADLS) Gen2**, leveraging **Azure Data Factory (ADF)**, **Azure Databricks**, and various **Azure services** for security, performance optimization, and scalability. The data is sourced from an **HTTP API** and **Microsoft SQL Server** and goes through a series of steps to clean, transform, and stage the data in ADLS for further use.

## Prerequisites:

- **Azure Data Factory** (ADF) for orchestrating data pipelines.
- **Azure Data Lake Storage Gen2** for data storage.
- **Azure Key Vault** for securely managing credentials and secrets.
- **Azure Databricks** for data processing.
- **GitHub** for version control and managing the source code.

---

## Steps Taken:

### 1. Setting Up Azure Data Factory (ADF)

#### 1.1 Linking ADF to GitHub
- **GitHub Repository**: Created a repository called **DataLake** for version control. This repository is connected to ADF, and the **Dev branch** is used for pipeline development.
- **Version Control**: This ensures that all changes to the pipeline are versioned and can be tracked in the GitHub repository.

#### 1.2 Creating Linked Services
- **HTTP API to ADLS Gen2**:
  - Created a linked service to fetch data from the **HTTP API** (https://jsonplaceholder.typicode.com/users).
  - Stored the data in the **raw-api** container of ADLS Gen2 in **Parquet** format.
  - **Azure Key Vault** was used for securely managing the **storage account key**.

- **Microsoft SQL Server to ADLS Gen2**:
  - Established a linked service to connect to the **SQL Server** using a **self-hosted integration runtime**.
  - Transferred data to the **raw-sql** container in ADLS Gen2 in **CSV** format.

#### 1.3 Creating Pipelines in ADF
- **Pipeline 1**: **HTTP API to ADLS Gen2**
  - Used the **Copy Data activity** to fetch data from the HTTP API and save it in **Parquet** format in the **raw-api** container.

- **Pipeline 2**: **SQL Server to ADLS Gen2**
  - Used the **Copy Data activity** to transfer data from **SQL Server** to the **raw-sql** container in **CSV** format.

---

### 2. Pipeline Triggers and Branching from Dev to QA

#### 2.1 Triggers for Data Pipelines
- **Scheduled Triggers**: 
  - Used **time-based triggers** to initiate data pipelines for periodic data ingestion.
  - A trigger was set for **Pipeline 1** to pull the HTTP API data at a scheduled time.
  - A trigger for **Pipeline 2** to pull data from SQL Server on a regular basis.

- **Manual Triggers**: 
  - For cases where immediate data ingestion is needed, manual triggers were employed to kick off the pipeline.
  
- **Trigger Dependencies**: 
  - **Trigger 1** (HTTP API to ADLS) must complete before **Trigger 2** (SQL Server to ADLS) can begin, ensuring data is ingested in the correct order.
  
#### 2.2 Branching from Dev to QA
- **Development (Dev) Branch**:
  - Created pipelines in the **Dev** environment to validate the end-to-end data flow from ingestion to storage.
  - All new changes and features were tested in the **Dev branch** first.
  
- **Quality Assurance (QA) Branch**:
  - Once the pipelines were successfully tested in **Dev**, a **pull request** (PR) was initiated to merge the Dev branch into the **QA branch**.
  - After approval, pipelines were deployed to the **QA environment** for further validation.

- **Version Control in GitHub**: 
  - Ensured that both **Dev** and **QA branches** are synchronized for consistent pipeline deployment across environments.

---

### 3. Data Processing and Transformation in Databricks

#### 3.1 Setting Up Databricks Workspace
- Mounted the **raw-api** and **raw-sql** containers from ADLS Gen2 to Databricks using **secret scopes** for secure access to storage account keys.
- Used the following Databricks code to mount the storage accounts:

   ```python
  dbutils.fs.mount(
      source="wasbs://raw-api@casestudy1new.blob.core.windows.net",
      mount_point = "/mnt/raw-api",
      extra_configs={"fs.azure.account.key.casestudy1new.blob.core.windows.net": dbutils.secrets.get(scope = "casestudy", key = "storage")}
  )

#### 3.2 Reading Data from Parquet and CSV
- Read data from `raw-api` (Parquet format) and `raw-sql` (CSV format) using Spark in Databricks:

```python
df_parquet = spark.read.parquet("/mnt/raw-api/users.parquet")
df_csv = spark.read.csv("/mnt/raw-sql/dbo.football.csv", header=True, inferSchema=True)
