# Synapse PySpark

Local development workspace for Azure Synapse notebooks.

The notebooks use customer-supplied configuration values and do not contain environment-specific Synapse workspace or dedicated SQL pool names.

## Project structure

```text
notebooks/
  FisCAL/
    QueryMoviesDB.ipynb
    QueryMoviesDBNative.ipynb
  managed-vnet/
    FisCAL/
      QueryMoviesDB.ipynb
      QueryMoviesDBNative.ipynb
Synapse-PySpark.code-workspace
```

## Open the workspace

Open `Synapse-PySpark.code-workspace` in Visual Studio Code.

## Use the notebooks in another Synapse workspace

The simplest customer workflow is:

1. Download an `.ipynb` file from this repository.
2. In Synapse Studio, open **Develop**, select the add button, and choose **Import**.
3. Import the notebook and attach a supported Spark pool.
4. Set the configuration values at the beginning of the code cell:

   ```python
   workspace_name = "<your-synapse-workspace-name>"
   dedicated_pool = "<your-dedicated-sql-pool-name>"
   schema_name = "dbo"
   table_name = "moviesDB"
   ```

5. Update the selected columns if the target table does not use the sample `moviesDB` schema.
6. Run the notebook.

Importing the `.ipynb` is preferred over copying individual cells because it preserves Markdown, cell boundaries, and notebook metadata.

For the native connector notebook, update:

```python
.option(Constants.SERVER, "<workspace>.sql.azuresynapse.net")
.synapsesql("<dedicated_pool>.<schema>.<table>")
```

For the JDBC notebook, update the server and database in `jdbc_url`, then update the SQL query:

```python
jdbc_url = (
    "jdbc:sqlserver://<workspace>.sql.azuresynapse.net:1433;"
    "database=<dedicated_pool>;"
    "encrypt=true;"
    "trustServerCertificate=false;"
    "hostNameInCertificate=*.sql.azuresynapse.net;"
    "loginTimeout=30;"
)
```

```sql
FROM [<schema>].[<table>]
```

The JDBC notebook uses a Microsoft Entra access token and does not store a username or password.

### Prerequisites

- A supported Synapse Spark runtime
- A dedicated SQL pool containing the referenced table
- Microsoft Entra permission to read the dedicated SQL pool
- Network access from Spark to the dedicated SQL endpoint
- For the native connector, access from Spark to the workspace ADLS staging storage

## Publish a local notebook with Azure CLI

```powershell
az synapse notebook create `
  --workspace-name <your-synapse-workspace-name> `
  --name QueryMoviesDB `
  --file '@notebooks\FisCAL\QueryMoviesDB.ipynb' `
  --folder-path FisCAL `
  --spark-pool-name <your-spark-pool-name> `
  --executor-count 2 `
  --executor-size Small `
  --subscription <your-subscription-id>
```

Both notebooks read up to 100 rows from the configured table.

`QueryMoviesDB.ipynb` uses direct JDBC with Microsoft Entra authentication. `QueryMoviesDBNative.ipynb` uses the native Synapse dedicated SQL pool connector and requires access to the workspace's ADLS staging storage.
