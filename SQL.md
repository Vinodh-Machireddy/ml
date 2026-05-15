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
```  
```
SELECT fault_label,
       COUNT(*)            AS total_batteries,    # count all rows in each group — no matter what values they have.
       MAX(temperature)    AS max_temp,
       MIN(temperature)    AS min_temp,
       AVG(temperature)    AS avg_temp
FROM battery_readings
GROUP BY fault_label;   # GROUP BY looks at the fault_label column and groups all rows that have the same value together.   
```  

-- Step 1: Create the table
CREATE TABLE battery_readings (
    battery_id              VARCHAR(10),
    voltage                 DECIMAL(5,2),
    temperature             DECIMAL(5,2),
    current                 DECIMAL(5,2),
    capacity                DECIMAL(5,2),
    soc                     DECIMAL(5,2),
    soh                     DECIMAL(5,2),
    internal_resistance     DECIMAL(5,2),
    c_rate                  DECIMAL(5,2),
    cycle_count             INT,
    fault_label             VARCHAR(50)
);

-- Step 2: Insert your data
INSERT INTO battery_readings VALUES
('BAT001', 3.7, 35.2, 1.2, 98.5, 82, 98, 12.1, 0.3, 120,  'normal'),
('BAT002', 3.2, 42.1, 1.8, 87.3, 74, 87, 18.4, 0.6, 340,  'thermal_fault'),
('BAT003', 2.8, 55.6, 2.1, 76.2, 61, 76, 24.7, 0.8, 520,  'overvoltage'),
('BAT004', 4.3, 38.1, 2.5, 91.2, 98, 91, 15.2, 1.2, 210,  'overcharging'),
('BAT005', 3.1, 36.2, 1.1, 45.3, 55, 45, 38.9, 0.4, 890,  'cell_degradation'),
('BAT006', 3.6, 35.8, 1.3, 97.1, 79, 97, 42.3, 0.5, 150,  'internal_short_circuit'),
('BAT007', 3.5, 34.9, 1.2, 96.8, 31, 96, 13.1, 0.3, 95,   'undervoltage');

INSERT INTO battery_readings VALUES
('BAT008', 3.3, 43.5, 1.9, 85.0, 72, 86, 19.1, 0.6, 360, 'thermal_fault'),
('BAT009', 3.1, 44.2, 2.0, 83.0, 70, 84, 20.0, 0.7, 400, 'thermal_fault'),
('BAT010', 2.9, 56.0, 2.2, 75.0, 60, 75, 25.0, 0.8, 530, 'overvoltage'),
('BAT011', 3.6, 35.5, 1.2, 97.0, 80, 97, 12.5, 0.3, 110, 'normal'),
('BAT012', 3.7, 34.9, 1.1, 98.0, 83, 98, 12.0, 0.3, 100, 'normal');



## Metrics Table in PostGresSQL
| run_id | key        | value | step | timestamp   |
|--------|------------|-------|------|-------------|
| abc123 | f1_score   | 0.91  | 1    | 1704067200  |
| abc123 | precision  | 0.89  | 1    | 1704067200  |
| abc123 | recall     | 0.93  | 1    | 1704067200  |
| abc123 | accuracy   | 0.88  | 1    | 1704067200  |
| abc123 | auc        | 0.95  | 1    | 1704067200  |
| xyz456 | f1_score   | 0.85  | 1    | 1704067200  |
| xyz456 | precision  | 0.82  | 1    | 1704067200  |
