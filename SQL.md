# SQL  
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
SELECT * FROM battery_readings
WHERE fault_label = 'thermal_fault'
AND temperature > 40;  

-- 🔵 MLflow example
SELECT run_id, key, value    # Column filter
FROM metrics
WHERE key = 'f1_score'
ORDER BY value DESC, Temperature AS temp ASC, Voltage AS Vol DESC;  # select rows of value column in metrics table.
LIMIT 5;    # Row filter  


SELECT fault_label,
       COUNT(*)            AS total_batteries,    # count all rows in each group — no matter what values they have.
       MAX(temperature)    AS max_temp,
       MIN(temperature)    AS min_temp,
       AVG(temperature)    AS avg_temp
FROM battery_readings
GROUP BY fault_label;   # GROUP BY looks at the fault_label column and groups all rows that have the same value together.   


