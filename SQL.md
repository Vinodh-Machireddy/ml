SQL
When MLflow connects to your PostgreSQL database, it automatically creates these tables:  MLflow creates and manages these tables. You just query them using SQL.
```
Your PostgreSQL Database (mlflow_db)
│
├── experiments       ← table
├── runs              ← table  ✅ this one
├── params            ← table
├── metrics           ← table
├── tags              ← table
├── registered_models ← table
└── model_versions    ← table
```
