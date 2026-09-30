# Synapse PySpark

Local development workspace for Azure Synapse notebooks in:

- Workspace: `synw-demoworkspace-001`
- Resource group: `rg-synapse-demo-001`
- Subscription: `rayfelipe-fdpo`
- Synapse folder: `FisCAL`

## Project structure

```text
notebooks/
  FisCAL/
    QueryMoviesDB.ipynb
Synapse-PySpark.code-workspace
```

## Open the workspace

Open `Synapse-PySpark.code-workspace` in Visual Studio Code.

## Publish the notebook to Synapse

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

The notebook reads up to 100 rows from `sqldedpool1.dbo.moviesDB` through the Azure Synapse Spark connector.
