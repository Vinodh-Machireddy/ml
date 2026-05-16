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
### SELECT and WHERE
```
SELECT * FROM battery_readings
WHERE fault_label = 'thermal_fault'
AND temperature > 40;
```   
### ORDER BY, DESC/ASC, LIMIT
```
-- 🔵 MLflow example
SELECT run_id, key, value    # Column filter
FROM metrics
WHERE key = 'f1_score'
ORDER BY value DESC, Temperature AS temp ASC, Voltage AS Vol DESC;  # select rows of value column in metrics table.
LIMIT 5;    # Row filter  
```
### GROUP BY
```
SELECT fault_label,
       COUNT(*)            AS total_batteries,    # count all rows in each group — no matter what values they have.
       MAX(temperature)    AS max_temp,
       MIN(temperature)    AS min_temp,
       AVG(temperature)    AS avg_temp
FROM battery_readings
GROUP BY fault_label;   # GROUP BY looks at the fault_label column and groups all rows that have the same value together.   
```
#### HAVING is the WHERE for GROUP BY.  
```
WHERE         Individual rows              Before GROUP BY
HAVING       Groups after aggregation       After GROUP BY
```
```
-- 🔵 Experiments that have more than 5 failed runs
SELECT experiment_id, COUNT(*) AS failed_runs
FROM runs
WHERE status = 'FAILED'
GROUP BY experiment_id
HAVING COUNT(*) > 5;
```
### LIKE
```
SELECT * FROM battery_readings
WHERE fault_label LIKE 'battery_fault%';       # '%' Any number of characters
WHERE battery_id LIKE 'BAT00_';                # '_' Exactly one character
``` 
### BETWEEN
-- 🔵 MLflow example — runs between two dates
SELECT run_id, status, start_time
FROM runs
WHERE start_time BETWEEN '2024-01-01' AND '2024-12-31';
WHERE cycle_count BETWEEN 300 AND 600;

### JOIN
JOIN combines two tables into one result based on a common column.  
```
SELECT br.battery_id,
       br.fault_label,
       br.temperature,
       bm.manufacturer,
       bm.location
FROM battery_readings br
JOIN battery_metadata bm ON br.battery_id = bm.battery_id;
```
#### JOIN Types
#### 1. INNER JOIN — only matching rows
```
SELECT br.battery_id, br.fault_label, bm.manufacturer
FROM battery_readings br
INNER JOIN battery_metadata bm ON br.battery_id = bm.battery_id;
```
> BAT099 and BAT100 will NOT appear — no match found.
> JOIN alone always means INNER JOIN by default.   

#### 2. LEFT JOIN — all rows from left table
LEFT JOIN — "Give me ALL rows from LEFT table (battery_readings), even if no match in RIGHT"
```
battery_readings (LEFT)    battery_metadata (RIGHT)
───────────────────────────────────────────────────
BAT001  ←────match───────→  BAT001   ✅ both appear
BAT099  ←────NO match        ❌       ✅ BAT099 still appears, manufacturer = NULL
                BAT100       ❌       ❌ BAT100 disappears — it's on RIGHT side
```
#### 3. RIGHT JOIN — all rows from right table
RIGHT JOIN — "Give me ALL rows from RIGHT table (battery_metadata), even if no match in LEFT"  
```
battery_readings (LEFT)    battery_metadata (RIGHT)
───────────────────────────────────────────────────
BAT001  ←────match───────→  BAT001   ✅ both appear
BAT099     ❌               ❌       ❌ BAT099 disappears — it's on LEFT side
           ❌  NO match──→  BAT100   ✅ BAT100 still appears, fault_label = NULL
```
#### 4. FULL JOIN — all rows from both tables  
```
SELECT br.battery_id, br.fault_label, bm.manufacturer
FROM battery_readings br
FULL JOIN battery_metadata bm ON br.battery_id = bm.battery_id;
```


### Window Functions
The problem with GROUP BY:  
```
-- GROUP BY collapses all rows into one row per group
SELECT fault_label, MAX(temperature) AS max_temp
FROM battery_readings
GROUP BY fault_label;
```
> You get max temperature — but you lose the battery_id. You can't see WHICH battery had that max temperature.  

**1. Only ORDER BY — rank ALL batteries together**
```
SELECT battery_id, fault_label, temperature,
       RANK() OVER (ORDER BY temperature DESC) AS rank    # OVER() It tells SQL: "Don't collapse rows like GROUP BY — instead calculate across a window of rows"
FROM battery_readings;
```
**2. Only PARTITION BY — no ranking, just grouping**
```
SELECT battery_id, fault_label, temperature,
       COUNT(*) OVER (PARTITION BY fault_label) AS total_in_group  # PARTITION BY fault_label means: "Start ranking from 1 again for each fault type"
FROM battery_readings;
```
> No ranking — just shows how many batteries exist in each fault group. All rows kept.  

**3. Both PARTITION BY + ORDER BY — rank WITHIN each group**
```
SELECT battery_id, fault_label, temperature,
       RANK() OVER (PARTITION BY fault_label ORDER BY temperature DESC) AS rank
FROM battery_readings;
```

When to use which:  
Just need the total count per group      ->        GROUP BY
Need total count AND individual row details together    ->   PARTITION BY










### Table-1
```
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
```
### Table-2
```  
CREATE TABLE battery_metadata (
    battery_id      VARCHAR(10),
    manufacturer    VARCHAR(50),
    model           VARCHAR(50),
    install_date    DATE,
    location        VARCHAR(50)
);

INSERT INTO battery_metadata VALUES
('BAT001', 'Panasonic', 'NCR18650', '2022-01-15', 'Plant-A'),
('BAT002', 'LG Chem',   'INR21700', '2021-06-20', 'Plant-B'),
('BAT003', 'Samsung',   'INR18650', '2020-03-10', 'Plant-A'),
('BAT004', 'Panasonic', 'NCR21700', '2022-08-05', 'Plant-C'),
('BAT005', 'LG Chem',   'INR18650', '2019-11-30', 'Plant-B'),
('BAT006', 'Samsung',   'INR21700', '2021-04-18', 'Plant-A'),
('BAT007', 'Panasonic', 'NCR18650', '2023-02-22', 'Plant-C');
```
```
-- Battery with no metadata
INSERT INTO battery_readings VALUES
('BAT099', 3.5, 36.0, 1.2, 95.0, 78, 95, 13.0, 0.3, 100, 'normal');

-- Metadata with no reading
INSERT INTO battery_metadata VALUES
('BAT100', 'Sony', 'VTC6', '2023-05-10', 'Plant-D');
``` 

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

