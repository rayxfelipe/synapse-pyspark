# Synapse PySpark

Local development workspace for Azure Synapse notebooks.

## Included environments

| Workspace | Dedicated SQL pool | Spark pool | Notebook path |
|---|---|---|---|
| `synw-demoworkspace-001` | `sqldedpool1` | `sparkpool1` | `notebooks/FisCAL` |
| `raysynapsemgdvnet` | `dedpoolinmgdvnet` | `FiscalSpark35` | `notebooks/raysynapsemgdvnet/FisCAL` |

Both workspaces use the `FisCAL` Synapse folder. The managed-VNet workspace also has a second equivalent Spark 3.5 pool named `FiscalSpark35B`.

## Project structure

```text
notebooks/
  FisCAL/
    QueryMoviesDB.ipynb
    QueryMoviesDBNative.ipynb
  raysynapsemgdvnet/
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
4. Replace the sample workspace, database, schema, table, and selected column names with values from the target environment.
5. Run the notebook.

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
  --workspace-name synw-demoworkspace-001 `
  --name QueryMoviesDB `
  --file '@notebooks\FisCAL\QueryMoviesDB.ipynb' `
  --folder-path FisCAL `
  --spark-pool-name sparkpool1 `
  --executor-count 2 `
  --executor-size Small `
  --subscription 499bc654-f84c-46c2-952c-b30be508f78c
```

Both notebooks read up to 100 rows from `sqldedpool1.dbo.moviesDB`.

`QueryMoviesDB.ipynb` uses direct JDBC with Microsoft Entra authentication. `QueryMoviesDBNative.ipynb` uses the native Synapse dedicated SQL pool connector and requires access to the workspace's ADLS staging storage.

The `raysynapsemgdvnet` copies target the `dedpoolinmgdvnet` dedicated SQL pool and `FiscalSpark35`.
