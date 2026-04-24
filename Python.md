
# PYTHON
**Introduction:**  
	- python is a dynamically typed programming language.  variables are key for any program lang bcz using variable  we can make our program dynamic.  Variables are 	  flagship in python.   
	- Dynamically typed programming languages: Python, Ruby, JavaScript, PHP, Perl, Lua, Tcl, Groovy.  
	- Statically Typed Languages:C, C++, Java, C#, Go.  

**Shell Scripting  VS Python Scripting:**  
	- in devops mostly we use linux systems bcz windows has less security. In Windows/Mac it uses rich UI. In Linux default CLI. The main purpose of Shell Scripting         is to interact with Linux systems and get information. We can write the commands one after the other or set of command in scriptFile.sh and execute.
	- Platform-specific behavior (e.g., Bash for Linux, PowerShell for Windows).  
	- Simpler for small tasks but syntax can become complex for larger scripts.  
	- Best for system-level tasks like command chaining and file operations.  

	. Cross-platform; runs on any system with Python installed.  
	. Easy to learn with a clean syntax; suitable for beginners.  
	. Suitable for writing complex programs, Interact with API, automation, and data processing.  

in real-world scenario we might get a chance to work on windows that’s why python is needed.  

> **NOTE:** As a devops engineer it not mandatory to use python to fetch the information from linux and windows machines we can use Ansible also to achieve this task.  
> **NOTE:** When we automating the things, normally we talk to API (OR) CLI.  
		- CLIs are great for immediate and straightforward automation tasks,  
		- while APIs provide more flexibility and are better suited for complex, programmatic integrations.  

## Virtual Env  
A virtual environment in Python is an isolated environment that allows you to install and manage packages separately from the global Python environment. If we want to work with multiple projects in on virtual machine/ec2-instance would prefer virtual Env.  It creates logical separation on virtual machine/ec2-instance for python packages.  
```
python -m venv vmtutes   #to create
source vmtutes/bin/activate
pip install requests numpy pandas
Pip list
deactivate
rm -rf venv
```  
## Command Line Arguments
	- Command-line arguments in Python are a way to give input to a Python program when you run it from the command line or terminal.  
	  Syntax: python example.py arg1 arg2 arg3  

## Environment variables
Environment variables in Python are like little notes your operating system keeps to tell your programs about important settings. They can store information such as:  
	- Paths to files or directories.
	- API keys or credentials.
	- Configuration details like debug settings.  

`Export password=“root123”`
`Export apitoken=“45rf6tgy78u9i9o0p9o”`

```
import os  
print(os.getenv(“password”)) # get password from environment variable
print(os.getenv(“apitoken”))
```
> **Note:** Builtin env variable : env

## IDLE  
	- It is the default Python IDE (Integrated Development Environment) that comes with the standard Python installation.
	- It's designed to be simple and beginner-friendly, making it great for learning Python, without writing py file.
	- To exit the IDLE terminal:  exit(). (Or) Control + d  

> **NOTE:** Python, operators are essential for the interpreter and compiler to understand and execute specific operations  

<details>
<summary><b>Python Core Topics (Must Know)</b></summary>  
	
1. Variables  
2. Data Types  
3. Type Conversion  
4. Operators  
5. Strings  
6. if / elif / else  
7. for loop  
8. while loop  
9. Lists  
10. Tuples  
11. Sets  
12. Dictionaries  
13. Functions  
14. Lambda Functions  
15. Built-in Functions  
16. OOP Basics (Classes & Objects)  
17. Modules & Import  
18. List Comprehensions  
19. Error Handling  
**Nice to Have**  
20. Input & Output  
21. File Handling  
22. Decorators (just recognize, not write)
23. dsl
24. generator
25. 
26. 
</details>

<details> 
<summary><b>Python Libraries/Packages for ML</b></summary>  
	
- numpy – Numerical Computing
- pandas – Data Manipulation
- scikit-learn – Core ML Library
- mlflow – Experiment Tracking & Model Registry
- dvc – Data & Model Versioning
- fastapi – API for Model Serving
- optuna & XGBoost – Hyperparameter Tuning
- pyyaml – YAML
- pytest – Tests
- joblib – Model Serialization
</details>	

## 1. Variables
variables are used to store data values. A variable is essentially a name that refers to a value. It store data, DATA can be any type like String, Float, Boolean….etc  > In Python, you don't declare types — just assign and go.  
```  
model_version = "v1.3"
threshold = 0.5
max_depth = 6
data_path = "s3://daimler-battery/data/train.csv"
```  
**Multiple Assignment:**
```
best_score = 0.91
current_score = 0.95
# current model is better, so swap
best_score, current_score = current_score, best_score
print(best_score)    # 0.95
```
**Unpack**  
x, y, z = 10, 20, 30  
#x=10  y=20  z=30  
`X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)`  
> left side variables = right side values, matched by position.  

**Types:**
Global Variable: Defined outside all functions — accessible everywhere.  
Local Variable : Defined inside a function — only lives inside that function.  

**Naming Conventions:**
- Variable names must start with a letter (a-z, A-Z) or an underscore (_).  
- The rest of the name can include letters, numbers, or underscores.  
- Best Practice only Lower case variable names  
- Snake casing: Vinodh_Machireddy_DevOps (or) Camel Casing: vinodh_Machireddy_DevOps  
- Keep variable names as descriptive as possible. vmd  
- Reserved keywords (e.g., if, while, class, import) cannot be used as variable names.  

## 2. Data Types
**int — Integer:** Whole numbers, no decimals.  
```
n_estimators = 100
max_depth = 6
num_classes = 4         # fault types in battery classifier
print(type(n_estimators))   # <class 'int'>
```
**float — Decimal Numbers:**
```
learning_rate = 0.05
threshold = 0.5
f1_score = 0.923
print(type(learning_rate))   # <class 'float'>
```
**str — String:**
```
model_name = "XGBoost"
experiment_name = "battery_fault_classifier"
data_path = "s3://daimler-battery/data/train.csv"
print(type(model_name))   # <class 'str'>
```
**bool — Boolean:** Only two values — True or False.
```
is_trained = False
log_to_mlflow = True

print(type(is_trained))   # <class 'bool'>
```
> Important — In Python, booleans are actually integers underneath
**Check Data Type:**
```
print(type(100))          # <class 'int'>
print(type(0.95))         # <class 'float'>
print(type("XGBoost"))    # <class 'str'>
  
# use isinstance() — more useful in real code
print(isinstance(0.95, float))    # True
print(isinstance(100, int))       # True
print(isinstance(1, (int, float)))  # True ✅ — accepts both
```
## 3.Type Conversion
Converting a value from one data type to another.  

| Function | Converts To | Input Example | Output | Watch Out For |
|---|---|---|---|---|
| `int()` | Integer | `int("100")` | `100` | `int(3.9)` → `3` (truncates, no rounding) |
| `float()` | Float | `float("0.05")` | `0.05` | `float("abc")` crashes — wrap in try/except |
| `str()` | String | `str(100)` | `"100"` | |
| `bool()` | Boolean | `bool(0)` | `False` | `bool("False")` → `True` ⚠️ any non-empty string is True |
| `list()` | List | `list((1,2,3))` | `[1, 2, 3]` | |

**bool():**
```
# These are all False
print(bool(0))          # False
print(bool(0.0))        # False
print(bool(""))         # False — empty string
print(bool(None))       # False
print(bool([]))         # False — empty list

# Everything else is True
print(bool(1))          # True
print(bool(0.95))       # True
print(bool("XGBoost"))  # True
print(bool([1,2,3]))    # True
```
**Mapping(dict):** 
In Python, a mapping data type is a type that stores data in key-value pairs. he main mapping data type in Python is the dict (dictionary). 

student = {
    "name": "Vinodh",
    "age": 30,
    "role": "MLOps Engineer"
}  

## 4. Operators
**Arithmetic Operators:**
```
a = 10
b = 3

print(a + b)    # 13  — addition
print(a - b)    # 7   — subtraction
print(a * b)    # 30  — multiplication
print(a / b)    # 3.3333 — division (always float)
print(a // b)   # 3   — floor division (drops decimal)
print(a % b)    # 1   — modulus (remainder)
print(a ** b)   # 1000 — power (10³)
```
**Comparison Operators:**
Always returns True or False.  
```
f1_score = 0.923
threshold = 0.90

print(f1_score > threshold)     # True  — greater than
print(f1_score < threshold)     # False — less than
print(f1_score >= 0.923)        # True  — greater than or equal
print(f1_score <= 0.90)         # False — less than or equal
print(f1_score == 0.923)        # True  — equal
print(f1_score != 0.923)        # False — not equal
```
**Logical Operators:**
Combine multiple conditions.  
```
# and — both must be True
# or  — at least one must be True
# not — flips True to False

f1_score = 0.94
drift_detected = False
model_age_days = 10

# Deploy only if f1 is good AND no drift
if f1_score > 0.90 and not drift_detected:
    print("Safe to deploy")         # ✅ this runs

# Retrain if drift detected OR model is old
if drift_detected or model_age_days > 30:
    print("Trigger retraining")     # ❌ doesn't run here
```
**Assignment Operators:**
Shortcut for updating a variable.  
```
score = 0.90

score += 0.03       # same as score = score + 0.03  → 0.93
score -= 0.01       # same as score = score - 0.01  → 0.92
score *= 2          # same as score = score * 2     → 1.84
score /= 2          # same as score = score / 2     → 0.92
```
**Membership Operators:**
Check if a value exists inside a collection.  (in & not in)
```
supported_models = ["XGBoost", "RandomForest", "LightGBM"]
requested_model = "XGBoost"

if requested_model in supported_models:
    print(f"Loading {requested_model}...")
else:
    print("Model not supported")

# Check if feature exists in dataframe columns
required_features = ["voltage", "temperature", "current"]
if "voltage" not in df.columns:
    raise ValueError("Missing required feature: voltage")
```
**Identity Operators:**
Check if two variables point to same object in memory — not just equal value.  
```
# is / is not

# use 'is' for None checks — standard Python practice
model = None

if model is None:
    print("Model not loaded yet")      # ✅ correct way

if model is not None:
    print("Model ready")
```
> ### Python follows math order: ** → * / // % → + -
result = 2 + 3 * 4      # 14, not 20 — multiplication first
result = (2 + 3) * 4    # 20 — brackets first  
> ### MLOps example
accuracy = correct / total * 100    # division first, then multiply

## 5. Strings
Any text data wrapped in quotes — single 'battery_fault_classifier' or double "XGBoost", both work the same.  

**String Indexing & Slicing:**
Every character has a position starting from 0.  
```
name = "XGBoost"
#       0123456

print(name[0])      # X — first character
print(name[-1])     # t — last character
print(name[0:3])    # XGB — from index 0 to 2
print(name[2:])     # Boost — from index 2 to end
print(name[:3])     # XGB — from start to index 2
```
**String Manipulation:**
- String manipulation means cleaning, searching, or modifying that sentence.
```  
log = "  ERROR: Battery voltage drop at 14:32  "

# 1. strip() → removes spaces from both sides
log.strip()           # "ERROR: Battery voltage drop at 14:32"

# 2. lower() → converts everything to lowercase
log.lower()           # "  error: battery voltage drop at 14:32  "

# 3. upper() → converts everything to uppercase
log.upper()           # "  ERROR: BATTERY VOLTAGE DROP AT 14:32  "

# 4. replace() → replaces a word
log.replace("ERROR", "ALERT")   # "  ALERT: Battery voltage drop..."

# 5. split() → breaks string into a list
log.split(":")        # ["  ERROR", " Battery voltage drop at 14", "32  "]

# 6. startswith() → checks beginning
log.strip().startswith("ERROR")   # True

# 7. endswith() → checks ending
log.strip().endswith("32")        # True

# 8. in → checks if word exists
"voltage" in log      # True ✅
"thermal" in log      # False ❌
```
**Chaining Methods — Do Multiple Things at Once:**  
```
log = "  ERROR: Battery voltage drop  "

# Instead of writing 3 lines
log = log.strip()
log = log.lower()
log = log.replace("error", "alert")

# Write in one line ✅
log = log.strip().lower().replace("error", "alert")
print(log)    # "alert: battery voltage drop"

# Capitalize inside f-string
print(f"Model: {model.upper()}, Accuracy: {accuracy}")
# Model: XGBOOST, Accuracy: 0.94
```

**f-string Formatting :**
```
print(f"Model: {model}, F1: {f1}, Version: v{version}")
o/p: Model: XGBoost, F1: 0.923, Version: v3  

f1 = 0.92345
print(f"F1 Score: {f1:.4f}")        # F1 Score: 0.9235

print(f"Epoch: {epoch} | Loss: {loss:.4f}")     # Epoch: 10 | Loss: 0.0432

model_path = f"s3://{bucket}/models/{version}/model.pkl"  

experiment_name = f"{model_name}_{env}_{date}"  
```
**Multi-line Strings:**
```
# triple quotes — used for long messages, SQL queries, prompts
query = """
    SELECT feature, value
    FROM battery_data
    WHERE fault_type = 'thermal'
    AND timestamp > '2024-01-01'
"""

# also used for docstrings in functions
def train_model(X, y):
    """
    Trains XGBoost classifier on battery fault data.
    Returns trained model and f1 score.
    """
    pass
```
## 6. if / elif / else
Controls which block of code runs based on a condition. Every condition returns True or False.  
```
f1_score = 0.94

if f1_score >= 0.90:
    print("Model is good — ready for deployment")
elif f1_score >= 0.75:
    print("Model is average — needs improvement")
else:
    print("Model is poor — retrain immediately")

# Output: Model is good — ready for deployment
```
**Multiple Conditions — and / or:**
```
f1_score = 0.94
drift_detected = False

# both must be True
if f1_score >= 0.90 and not drift_detected:
    print("Safe to deploy")

# at least one must be True
if f1_score < 0.75 or drift_detected:
    print("Trigger retraining")
```
> we can use `in / not in` ,  `is / is not` also.

## 7. for loop
Repeats a block of code for each item in a collection — list, tuple, dictionary, range, or any iterable.  
```
# range(start, stop, step)
for i in range(0, 100, 10):
    print(i)        # 0 10 20 30 40 50 60 70 80 90
```

| Syntax | Use | MLOps Example |
|---|---|---|
| `for item in list` | Loop over a list | Iterate over feature names |
| `for i in range(n)` | Loop n times | Training epochs |
| `for i, item in enumerate(list)` | Loop with index | Track model number in search |
| `for a, b in zip(list1, list2)` | Loop two lists together | Compare actual vs predicted labels |
| `for k, v in dict.items()` | Loop over dictionary | Print hyperparameter config |
| `break` | Exit loop early | Stop when target F1 is hit |
| `continue` | Skip current iteration | Skip None or missing features |

## 8. while loop
Repeats a block of code as long as a condition is True — unlike for loop, you don't know upfront how many times it will run.  
```
epoch = 0

while epoch < 5:
    print(f"Training epoch {epoch}")
    epoch += 1          # ⚠️ must update — otherwise runs forever!

# Output:
# Training epoch 0
# Training epoch 1
# Training epoch 2
# Training epoch 3
# Training epoch 4
```
> break — Exit Early
> continue — Skip Current Iteration

## 9. Lists
An ordered, mutable collection that stores multiple values in a single variable.  
```
features = ["voltage", "temperature", "current", "capacity"]
f1_scores = [0.91, 0.93, 0.87, 0.95]
mixed = ["XGBoost", 100, 0.05, True]       # can hold different types
empty = []                                  # empty list
```
**Indexing & Slicing:**
```
features = ["voltage", "temperature", "current", "capacity"]
#                 0            1           2           3

print(features[0])      # voltage   — first item
print(features[-1])     # capacity  — last item
print(features[1:3])    # ['temperature', 'current'] — index 1 to 2
print(features[:2])     # ['voltage', 'temperature'] — first two
print(features[2:])     # ['current', 'capacity']    — from index 2
```
**Most Useful List Methods:**
```
features = ["voltage", "temperature", "current"]

# Add
features.append("capacity")            # adds to end
features.insert(1, "resistance")       # adds at index 1

# Remove
features.remove("resistance")          # removes by value
features.pop()                         # removes last item
features.pop(0)                        # removes at index 0

# Info
print(len(features))                   # number of items
print("voltage" in features)           # True — membership check
print(features.index("current"))       # returns index of value
print(features.count("voltage"))       # counts occurrences

# Order
features.sort()                        # sorts in place A-Z
features.reverse()                     # reverses in place
sorted_list = sorted(features)         # returns new sorted list

# Copy
features_copy = features.copy()        # safe copy — not reference
```
**List of Lists — 2D:**
```
# each inner list is one training sample
# [voltage, temperature, current, capacity]
battery_data = [
    [3.7, 35.2, 1.2, 98.5],
    [3.2, 42.1, 1.8, 87.3],
    [3.9, 28.5, 0.9, 99.1]
]

print(battery_data[0])          # [3.7, 35.2, 1.2, 98.5] — first sample
print(battery_data[0][1])       # 35.2 — first sample, temperature
```
**Common Mistakes**
```
# ❌ reference vs copy
a = [1, 2, 3]
b = a               # b points to same list!
b.append(4)
print(a)            # [1, 2, 3, 4] ← a also changed!

# ✅ always use .copy()
b = a.copy()
b.append(4)
print(a)            # [1, 2, 3] ← safe
``` 
**Useful Built-in Functions on Lists**
```
scores = [0.91, 0.87, 0.95, 0.93]

print(len(scores))          # 4     — number of items
print(max(scores))          # 0.95  — highest value
print(min(scores))          # 0.87  — lowest value
print(sum(scores))          # 3.66  — total
print(sorted(scores))       # [0.87, 0.91, 0.93, 0.95]
```
## 10. Tuples
An ordered, immutable collection — like a list but cannot be changed after creation.  
```
# list   → mutable   → can change
# tuple  → immutable → cannot change

features = ("voltage", "temperature", "current", "capacity")
model_info = ("XGBoost", "v1.3", 0.94)
empty = ()
single = ("voltage",)      # ⚠️ single item needs trailing comma!
```
**List vs Tuple**
# Python — List vs Tuple Comparison

| | List | Tuple |
|---|---|---|
| Syntax | `[ ]` | `( )` |
| Mutable | ✅ Yes | ❌ No |
| Use when | Data changes | Data is fixed |
| Speed | Slower | Faster |
| Example | Collecting F1 scores across runs | Fixed feature set for training |
| Methods | `.append()` `.remove()` `.sort()` | `.count()` `.index()` only |
| Convert | `tuple(list)` to lock | `list(tuple)` to edit |

> Indexing & Slicing — Same as List

**Immutable — Cannot Change:**
```
features = ("voltage", "temperature", "current")

features[0] = "resistance"      # ❌ TypeError — cannot modify!
features.append("capacity")     # ❌ AttributeError — no append!

# if you need to change — convert to list, edit, convert back
features = list(features)
features.append("capacity")
features = tuple(features)
print(features)     # ('voltage', 'temperature', 'current', 'capacity')
```
**Unpacking :**
```
model_info = ("XGBoost", "v1.3", 0.94)

# unpack into variables
model_name, version, f1_score = model_info

print(model_name)   # XGBoost
print(version)      # v1.3
print(f1_score)     # 0.94
```

## 11. Sets
An unordered, unindexed collection of unique values — duplicates are automatically removed.  
```
fault_types = {"thermal", "voltage", "current", "capacity"}
empty_set = set()           # ⚠️ not {} — that creates a dictionary!
```

| | List | Set |
|---|---|---|
| Ordered | ✅ Yes | ❌ No |
| Duplicates | ✅ Allowed | ❌ Auto removed |
| Indexing | ✅ `list[0]` | ❌ Not supported |
| Use when | Order matters | Unique values matter |

**Most Useful Set Methods:**
```
fault_types = {"thermal", "voltage", "current"}

# Add / Remove
fault_types.add("capacity")            # adds one item
fault_types.remove("voltage")          # removes — raises error if not found
fault_types.discard("voltage")         # removes — no error if not found

# Info
print(len(fault_types))                # number of items
print("thermal" in fault_types)        # True — fast membership check
```
**Set Operations — Most Powerful Feature:**
```
expected = {"voltage", "temperature", "current", "capacity"}
actual   = {"voltage", "temperature", "resistance"}

# union — all items from both
print(expected | actual)
# {'voltage', 'temperature', 'current', 'capacity', 'resistance'}

# intersection — only common items
print(expected & actual)
# {'voltage', 'temperature'}

# difference — in expected but NOT in actual
print(expected - actual)
# {'current', 'capacity'}  ← these are missing!

# symmetric difference — in either but NOT in both
print(expected ^ actual)
# {'current', 'capacity', 'resistance'}
```
> convert to list if you need indexing  

## 12. Dictionaries
A collection of key-value pairs — like a lookup table. Instead of index numbers, you access values by meaningful keys.  
```
model_config = {
    "model_name": "XGBoost",
    "n_estimators": 100,
    "max_depth": 6,
    "learning_rate": 0.05
}
```
**Accessing Values:**
```
# by key
print(model_config["model_name"])       # XGBoost

# .get() — safer, returns None if key missing
print(model_config.get("max_depth", 6))         # 6 — default value ✅
```  
> .get(key, default) means — "give me the value for this key, but if the key doesn't exist, give me this default value instead."
  
**Adding, Updating & Removing:**
```
# add new key
model_config["n_estimators"] = 100
model_config["learning_rate"] = 0.05

# update existing key
model_config["learning_rate"] = 0.01   # overwrites old value

# remove by key
del model_config["debug"]

# remove and return value
n = model_config.pop("n_estimators")
print(n)            # 100
```  
**Dictionary Methods:**
print(config.keys())        
print(config.values())       
print(config.items())    

#### Merge Dict
```
config = {"model": "XGBoost", "n_estimators": 100}
extra  = {"max_depth": 6, "learning_rate": 0.05}

**.update() — modifies original dict:**
config.update(extra)        # extra gets added INTO config
print(config)

** **unpacking — creates new dict ✅ preferred**
merged = {**config, **extra}    # creates brand new dict, originals untouched
print(merged)
```  
**What if Same Key Exists in Both?**
```
config  = {"learning_rate": 0.05}   # left  — goes in first
updates = {"learning_rate": 0.01}   # right — goes in second, overwrites!

merged = {**config, **updates}
print(merged)   # {"learning_rate": 0.01}  ← 0.01 won
```
**Looping Over Dictionary:**
```
# loop keys
for key in config:
    print(key)

# loop values
for value in config.values():
    print(value)

# loop both — most useful ✅
for key, value in config.items():
    print(f"{key} = {value}")

```  
## 13. Functions
A reusable block of code that runs only when called. Write once, use many times. Python provides several built-in functions like print(), len(), type(), range(), input(), etc.  

Functions always follows 3 principles I.e 
- Taking Input
- Execute the desired logic
- Return the Output 
```
def function_name(parameters):
    # code
    return result
```
```  
def addition(n1, n2):
    Add = n1 + n2
    return add

def check_fault(score, threshold=0.8):
    if score < threshold:
        return "FAULT"
    return "OK"
     
print(addition(2, 5)) ——> invoking & printing function output
print(check_fault(0.75, 0.7))   # OK
```  

### Parameters in Functions, There are 5 types:
**1. Positional Parameters — order matters:**
```
def train_model(model_name, learning_rate, n_estimators):
    print(f"{model_name} | lr={learning_rate} | n={n_estimators}")

# must pass in exact order
train_model("XGBoost", 0.05, 100)       # ✅
train_model(0.05, "XGBoost", 100)       # ❌ wrong — model_name gets 0.05

#Output
XGBoost | lr=0.05 | n=100
0.05 | lr=XGBoost | n=100
```
**2. Default Parameters — optional, has fallback:**
```
def train_model(model_name, learning_rate=0.05, n_estimators=100):
    print(f"{model_name} | lr={learning_rate} | n={n_estimators}")

train_model("XGBoost")                  # uses defaults for rest
train_model("XGBoost", 0.01)            # overrides learning_rate only
train_model("XGBoost", 0.01, 200)       # overrides both
```  
```
def train_model(model_name, learning_rate=0.05):
#               ↑                ↑
#         no = sign           has = sign
#         positional          default parameter
```
> Rule — default parameters must always come after positional:
> Simple rule — if it has =, it's a default parameter. If it doesn't, it's positional.
> Python reads left to right — required arguments first, optional ones at the end.  

**3. Keyword Arguments — pass by name, order doesn't matter:**
```
def train_model(model_name, learning_rate, n_estimators):
    pass

# keyword — very readable ✅
train_model(
    model_name="XGBoost",
    n_estimators=100,
    learning_rate=0.05      # order doesn't matter here
)
```
**4.*args(When you don't know how many values will be passed:)**  
```
def log_metrics(*args):
    for value in args:      # args is a tuple
        print(value)

log_metrics(0.94)                       # one metric
log_metrics(0.94, 0.92, 0.96)          # three metrics — all work ✅
```
**5. **kwargs — variable number of keyword arguments:**
```
def log_metrics(**kwargs):
    for name, value in kwargs.items():  # kwargs is a dict
        print(f"{name} = {value}")

log_metrics(f1=0.94, precision=0.92, recall=0.96)
# f1 = 0.94
# precision = 0.92
# recall = 0.96
```
## 14. Lambda Functions  
- A small, one-line anonymous function — no def, no name, no return statement needed.  
```
lambda parameters: expression
#                  ↑ automatically returned
```

```
# lambda
add = lambda x, y: x + y
print(add(3, 5))    # 8

# MLOps example
weighted_score = lambda precision, recall: (2 * precision * recall) / (precision + recall)
print(weighted_score(0.92, 0.96))   # f1 score calculation
```
<details> 
<summary><b>15. Buil-in Functions</b></summary> 
**1.print():**  
Outputs text or values to the console. Most used function in Python — for logging, debugging, monitoring.  
```
model = "XGBoost"
f1 = 0.94567

print("Training started")           						    # Print string
print(["voltage", "temperature"])  							    # Print list
print("Model:", model, "F1:", f1)   						    # Print Multiple Values. comma separated — adds space
print("XGBoost", 0.94, "production", sep=" | ")  			    # XGBoost | 0.94 | production
print(model, f1, sep = " | ", end=" -> ")  					    # default end is newline  # custom end — stays on same line
print(f"my_model: {model} | model_f1_score: {f1}", end=" -> ")  # f-string
print(f"Epoch: {epoch} | F1: {f1:.4f}") 					    # Epoch: 10 | F1: 0.9457
print("-" * 50)													# print("Training Complete") # -----------------------
```

**2.len():**  
- Returns the total number of items in a collection or sequence — list, tuple, set, dictionary, string.   
> not support: int, float, bool, None  ❌ TypeError  

**3.range(start, stop, step):**  
range(0, 100, 10)   # 0, 10, 20, 30, 40, 50, 60, 70, 80, 90  
> print(list(range(5)))   # [0, 1, 2, 3, 4] — convert if you need a list   

**4.enumerate():**    
- Loops over a collection and gives you both the index and the value at the same time — so you don't have to manually track position.   
```
features = ["voltage", "temperature", "current", "capacity"]

for i, feature in enumerate(features, start=1):
    print(f"{i}: {feature}")

# Output:
# 0: voltage			# default starts from 0
# 1: temperature		# start=1 to start from 1
# 2: current
# 3: capacity
```  
**5.zip():**  
- Combines two or more lists together — pairs up items by position so you can loop over them at the same time.    
```  
features   = ["voltage", "temperature", "current"]
importance = [0.45, 0.32, 0.23]
dtypes     = ["float", "float", "float"]

for feature, score, dtype in zip(features, importance, dtypes):
    print(f"{feature} ({dtype}): {score}")

# voltage (float): 0.45
# temperature (float): 0.32
# current (float): 0.23
```
**Unequal Length Lists — zip() Stops at Shortest:**
```
features   = ["voltage", "temperature", "current", "capacity"]
importance = [0.45, 0.32, 0.23]        # only 3 items!

for feature, score in zip(features, importance):
    print(f"{feature}: {score}")

# Output — capacity is ignored!
# voltage: 0.45
# temperature: 0.32
# current: 0.23
```
**Convert zip to List or Dict:**
```
features   = ["voltage", "temperature", "current"]
importance = [0.45, 0.32, 0.23]

# convert to list of tuples
pairs = list(zip(features, importance))
print(pairs)
# [('voltage', 0.45), ('temperature', 0.32), ('current', 0.23)]

# convert to dictionary — very useful ✅
feature_importance = dict(zip(features, importance))
print(feature_importance)
# {'voltage': 0.45, 'temperature': 0.32, 'current': 0.23}
```

**6.map():**  
Applies a function to every item in a collection — transforms each item one by one and returns the result.   
**Syntax:**   map(function, collection)   

map() Returns an Iterator — Always Convert to list  
```
result = map(lambda x: x * 2, [1, 2, 3])
print(result)           # <map object at 0x...> ← not useful!
print(list(result))     # [2, 4, 6] ✅
```
map() With Built-in Functions:
```
# convert list of strings to integers
values = ["100", "200", "300"]
integers = list(map(int, values))
print(integers)         # [100, 200, 300]

# convert list of strings to floats
scores = ["0.94", "0.91", "0.88"]
floats = list(map(float, scores))
print(floats)           # [0.94, 0.91, 0.88]

# convert list of numbers to strings
versions = [1, 2, 3, 4]
labels = list(map(str, versions))
print(labels)           # ['1', '2', '3', '4']
```
map() With Custom Function  
```
def classify(proba):
    return "fault" if proba >= 0.5 else "normal"

probabilities = [0.87, 0.43, 0.91, 0.35, 0.62]

labels = list(map(classify, probabilities))
print(labels)
# ['fault', 'normal', 'fault', 'normal', 'fault']
```
**7.filter():**  
Goes through a collection and keeps only items that match a condition — filters out everything else.  
Syntax:  filter(function, collection)
```
f1_scores = [0.94, 0.78, 0.91, 0.65, 0.88]

good_scores = list(filter(lambda x: x >= 0.85, f1_scores))
print(good_scores)      # [0.94, 0.91, 0.88]
```
> Same behaviour as map() — lazy, needs list() to execute.  

filter() With Custom Function  
```
def is_good_model(score):
    return score >= 0.85        # returns True or False

f1_scores = [0.94, 0.78, 0.91, 0.65, 0.88]

good_scores = list(filter(is_good_model, f1_scores))
print(good_scores)      # [0.94, 0.91, 0.88]
```

**8.sorted():**  
Returns a new sorted list from any collection — without modifying the original.  
sorted() — creates NEW list, original untouched   
```
scores = [0.91, 0.87, 0.95, 0.93]

# ascending — default
print(sorted(scores))                   # [0.87, 0.91, 0.93, 0.95]

# descending
print(sorted(scores, reverse=True))     # [0.95, 0.93, 0.91, 0.87]

print(sorted(models))                   # alphabetical A-Z
# ['LightGBM', 'RandomForest', 'SVM', 'XGBoost']
``` 
**9.max() and min():**  
max() — returns the highest value  
min() — returns the lowest value  

```
scores = [0.91, 0.87, 0.95, 0.93]

print(max(scores))      # 0.95 — highest
print(min(scores))      # 0.87 — lowest

models = ["XGBoost", "RandomForest", "LightGBM"]

print(max(models))      # XGBoost      — last alphabetically
print(min(models))      # LightGBM     — first alphabetically
```
**10.sum():**  
Returns the total of all values in a collection — works on numbers only.  
```
scores = [0.91, 0.87, 0.95, 0.93]

# start=0 is default
print(sum(scores, 0))       # 3.66

# start=1 — adds 1 to total
print(sum(scores, 1))       # 4.66
```
Calculate Average  
```
f1_scores = [0.91, 0.87, 0.95, 0.93]

average = sum(f1_scores) / len(f1_scores)
print(f"Average F1: {average:.4f}")     # 0.9150
```
**11.round():**  
Rounds a number to a specified number of decimal places.  
```
f1 = 0.94567

print(round(f1))        # 1    — rounds to nearest integer
print(round(f1, 1))     # 0.9  — 1 decimal place

# Rounding Rules
# .5 rounds to nearest EVEN number — Python standard
print(round(0.5))       # 0 — rounds to even
print(round(1.5))       # 2 — rounds to even
print(round(2.5))       # 2 — rounds to even
print(round(3.5))       # 4 — rounds to even

# normal rounding
print(round(0.94))      # 1
print(round(0.44))      # 0

# Round to Tens, Hundreds
# negative decimal places
print(round(1567, -1))      # 1570  — round to tens
print(round(1567, -2))      # 1600  — round to hundreds
print(round(1567, -3))      # 2000  — round to thousands
```
**12.abs():**  
Returns the absolute value of a number — removes the negative sign, always gives positive result.  
```
print(abs(10))          # 10  — already positive
print(abs(-10))         # 10  — negative becomes positive
print(abs(0.15))        # 0.15
print(abs(-0.15))       # 0.15
print(abs(0))           # 0
```
**any() and all():**  
- any() — returns True if at least one item is True  
- all() — returns True if all items are True   

```
checks = [True, True, False, True]

print(any(checks))      # True  — at least one is True
print(all(checks))      # False — not all are True

# Simple Rule
# any() — like OR — one True is enough
# all() — like AND — every single one must be True

any([False, False, False])  # False — none are True
any([False, True,  False])  # True  — one is enough ✅

all([True, True,  True])    # True  — all True ✅
all([True, False, True])    # False — one False breaks it
```

</details>

## 16. OOP Basics (Classes & Objects)
A way to organize code into reusable blueprints called classes. Instead of writing separate functions and variables, you group related data and behaviour together.  
	- function OUTSIDE class — standalone function
	- function INSIDE class — called method 
	- Most functions in sklearn are inside classes — that's why you create objects first.  
```  
# Class  — blueprint (like a template)
# Object — actual instance created from blueprint

# Class is like a blueprint of a battery
# Object is an actual battery built from that blueprint

class Battery:
    pass

battery1 = Battery()    # object 1 — instance of Battery
battery2 = Battery()    # object 2 — another instance
```
**__init__ — Constructor:** 
Runs automatically when object is created — It stores the values you passed — into the object.

**self:** When you create multiple objects from same class — each object has its own data. self tells Python which object's data to use.
> self = battery1

```
class Battery:
    def __init__(self, battery_id, voltage, temperature):
        self.battery_id  = battery_id
        self.voltage     = voltage
        self.temperature = temperature
        self.status      = "normal"

    def check_status(self):
        if self.voltage < 3.0:
            self.status = "fault"
        else:
            self.status = "normal"
        return self.status

    def get_info(self):
        return f"{self.battery_id} | voltage={self.voltage} | status={self.status}"


# create object and use methods
battery1 = Battery("BAT001", 2.8, 35.2)
print(battery1.check_status())      # fault
print(battery1.get_info())          # BAT001 | voltage=2.8 | status=fault
```
**Class Variables vs Instance/Object Variables:**
```
class Battery:
    manufacturer = "Daimler"          # class variable — shared by ALL objects
    total_batteries = 0				 # class variable — defined OUTSIDE __init__

    def __init__(self, battery_id, voltage):
        # instance variables — unique to each object
        self.battery_id = battery_id   # anything with self. is an object variable.
        self.voltage    = voltage
        Battery.total_batteries += 1    # update class variable   # every time object created, add 1

battery1 = Battery("BAT001", 3.7)
battery2 = Battery("BAT002", 3.2)

print(battery1.manufacturer)        # Daimler — class variable
print(battery2.manufacturer)        # Daimler — same for all
print(Battery.total_batteries)      # 2 — shared count
```
**Inheritance:**
One class inherits from another — gets all its methods and can add more. 
inheritance:  child class gets parent's methods/functions
super()calls parent class __init__ 

**Why OOP Matters for MLOps — sklearn is Built on It:**
you are creating an object from it and calling its methods. You are already using OOP without realizing it!   
Every sklearn Component is a Class  
```
from sklearn.preprocessing import StandardScaler      # class
from sklearn.ensemble import RandomForestClassifier   # class
from sklearn.pipeline import Pipeline                 # class
from xgboost import XGBClassifier                     # class
```

## 17. Modules 
In Python, modules are .py files containing Python code (e.g., functions, variables, or classes) that can be imported and reused in other Python programs. Modules are collection of functions. (Modularity Approach).
 
**1. Built-in — comes with Python, no install needed:**  
Examples:
	- math: Provides mathematical functions.
	- os: Interacts with the operating system.
	- sys: Provides access to system-specific parameters and functions.
	- datetime: Deals with date and time.

**2. Third-party — install via pip, then import:**
```
Daily use:
pandas, numpy, scikit-learn,
xgboost, mlflow, boto3,
pyyaml, python-dotenv

Frequently:
requests, matplotlib,
great-expectations, prometheus-client

Sometimes:
flask, pytorch/tensorflow,
docker sdk, kubernetes client
```  
NOTE: when we call packages indirectly it is a module. Inside module collection of functions.

**3. Your own — files you created yourself:**
import my_utils
import train

### 4 Ways to Import
```
# 1. import entire module
import os
os.getenv("MODEL_VERSION")      # must use module name prefix

# 2. import specific item — no prefix needed ✅
from os import getenv
getenv("MODEL_VERSION")

# 3. import with alias — shortens long names ✅
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# 4. import everything — avoid this ❌
from os import *                # pollutes namespace — hard to debug
```
### Most Common MLOps Imports
```
# data handling
import pandas as pd
import numpy as np

# machine learning
from sklearn.model_selection import train_test_split
from sklearn.metrics import f1_score, precision_score, recall_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from xgboost import XGBClassifier

# experiment tracking
import mlflow
import mlflow.sklearn

# environment & file handling
import os
import sys
import json

# date & time
from datetime import datetime

# warnings
import warnings
warnings.filterwarnings("ignore")
```
**full-path:**
```
# short way — package hides module
from sklearn.metrics import f1_score

# full path — with module
from sklearn.metrics._classification import f1_score
#    ↑        ↑            ↑                ↑
#  library  package      module           function
```
> Because package's __init__.py already imports everything from its modules internally — so module is invisible to you. 


## 18. List Comprehensions  
- List comprehension is a shortcut to create a new list in one line.  
```  
scores = [0.91, 0.74, 0.88, 0.65, 0.95]

# Old way — 4 lines
faults = []
for s in scores:
    if s < 0.8:
        faults.append(s)

print(faults)   # [0.74, 0.65]

# New way — 1 line ✅
faults = [s for s in scores if s < 0.8]
print(faults)   # [0.74, 0.65]
```  
```
# 1. Basic
[round(score, 2)        for score in f1_scores]

# 2. With filter
[score                  for score in f1_scores   if score >= 0.85]

# 3. With if/else
["fault" if p >= 0.5
 else "normal"          for p     in probabilities]
```

**"Go through every score — if it is less than 0.8 — keep it"**  
```
faults = [ s   for s in scores   if s < 0.8 ]
           ↑        ↑                ↑
        keep it   go through      only if
                  every score     score < 0.8
``` 
> Whatever name you give in for — same name must be used in the expression. [round(s, 2) for s in f1_scores]

## 19. Error Handling
	- A way to catch and handle errors gracefully instead of letting your program crash.  
```
try:
    learning_rate = float("abc")        # code that might cause error  bcoz cant convert abc to float

except ValueError:
    print("Invalid learning rate")      # ✅ runs — error occurred
    learning_rate = 0.05                # ✅ safe fallback assigned

else:
    print(f"Learning rate set to {learning_rate}")  # runs only if NO error happened  # only runs if no error

finally:
    print("Learning rate configuration done")       # runs ALWAYS — error or no error # runs no matter what ✅
```
>  **Fallback** = backup value used when original value fails.  
   **Safe** = value that is valid and won't break your program.  


**Python — Common Error Types:**

| Error | When it Happens | Example |
|---|---|---|
| `ValueError` | Wrong value type | `int("abc")` |
| `TypeError` | Wrong data type | `"a" + 1` |
| `KeyError` | Dictionary key not found | `dict["missing_key"]` |
| `IndexError` | List index out of range | `list[99]` |
| `FileNotFoundError` | File doesn't exist | `open("missing.csv")` |
| `ZeroDivisionError` | Dividing by zero | `100 / 0` |
| `AttributeError` | Attribute doesn't exist | `None.predict()` |
| `ImportError` | Module not found | `import missing_module` |
| `NameError` | Variable not defined | `print(undefined_var)` |
| `Exception` | Catches ALL errors | use as last resort only |


> **Rule** — always catch specific errors first (`ValueError`, `KeyError`) and use `Exception` only as last resort.


## 20. Input and Output
 
				
## 21. File Handling
Think of file handling like opening a notebook, writing or reading, then closing it.  
**Writing to a File:**
```
# "w" means write mode
with open("results.txt", "w") as f:
    f.write("Model: XGBoost\n")    # \n means new line
    f.write("Accuracy: 0.94\n")
```
**Reading from a File:**
```
# "r" means read mode
with open("results.txt", "r") as f:
    content = f.read()      # reads entire file at once
    print(content)
```
**Reading Line by Line:**
```
with open("results.txt", "r") as f:
    for line in f:
        print(line.strip())   # strip removes \n at end
```
**Appending to a File:**  
```
# "a" means append mode → adds to existing file
with open("results.txt", "a") as f:
    f.write("Version: 2\n")   # doesn't delete old content
```
> Always use with — it closes file automatically. No tension! ✅ otherwise we need to close manually by f.close().  
> f is just a variable name that holds the opened file.  You can name it anything!  
> **Delete a File:**  # Always use try/except before deleting  
> **os:** os is a built in Python module that lets you talk to your operating system — delete files, check paths, list folders etc.
You must always import it first before using.


## 22. Decorators










				

## Python Libraries for ML
						
numpy – Numerical Computing
pandas – Data Manipulation
scikit-learn – Core ML Library
mlflow – Experiment Tracking & Model Registry
dvc – Data & Model Versioning
fastapi – API for Model Serving
optuna & XGBoost – Hyperparameter Tuning
pyyaml – YAML
pytest – Tests
joblib – Model Serialization


Numpy: 
It is a Python library used for fast mathematical calculations
	- pip install numpy
	- import numpy as np

Pandas:
A Python library used to work with data in table format.   Rows = records,  Columns = features
Very useful for data cleaning, loading CSV files, feature engineering.
pip install pandas
import pandas as pd

scikit-learn(Sklearn):
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
from sklearn.datasets import load_breast_cancer

Joblib:
Joblib is a small Python library used to save and load machine learning models.
after training a model you must save it so that:
- You can use it later
- You can deploy it
- You can share it
You don’t need to train again every time
- pip install joblib
- import joblib
joblib.dump(model, ‘vinodh.pkl’) #creates a file called vinodh.pkl
loaded_model = joblib.load(‘model.pkl') #to load
loaded_model.predict(X_test)

	NOTE: Why not use pickle?
pickle can also save models, but: joblib is faster for large numpy arrays, 
sklearn officially recommends joblib, better compression support. So joblib is preferred in ML workflows.

mlflow:
MLflow is an open-source MLOps tool used to manage the complete Machine Learning lifecycle, including:
Experiment tracking
Model versioning
Model packaging
Model deployment
It works with any ML library (Scikit-Learn, TensorFlow, PyTorch, XGBoost, etc.)

with mlflow.start_run():
This starts a new MLflow run. A “run” means one experiment attempt. Everything inside this with block will automatically be tracked and saved in MLflow. It creates a folder like: mlruns. 
So MLflow knows:
   which parameters you used ✔ which metrics you got ✔ which model you trained
mlflow.log_param("max_iter", 10000) # Saves a parameter to MLflow.
mlflow.log_metric("accuracy", acc) # Saves a metric (model perf score).
mlflow.sklearn.log_model(model, “model") # Saves your trained model into MLflow
This creates files inside MLflow:  model/model.pkl

MLflow stores these 3 files in mlruns/
	signature: The input–output schema of the model, describing what kind of data the model expects (shape, column count, data types) and what it produces. Stored in ‘MLmodel’ file
signature = infer_signature(X_test[:50], model.predict(X_test[:50])) 

	input_example: A small sample of the input data (few rows) that shows the exact format the model expects.
input_example = X_test[:5]  # input_example.json 
Model (sk_model): The actual trained machine learning model (LogisticRegression, RandomForest, etc.) that learns patterns from data and makes predictions. MLflow saves it as model.pkl.

XGBOOST:	 					
XGBoost is a machine learning algorithm that builds many small decision trees, each tree correcting the mistakes of the previous tree.  
Used for Fast training, Very high accuracy, Works well for CSV, excel(structured  data)  

OPTUNA:	  				
Optuna is a tool that automatically searches for the best hyperparameters for your ML model. Because choosing good parameters manually is difficult.  

PYTEST:  				
pytest is a Python testing library used to check if your code is working correctly. It is like a quality checker, If your code is wrong → pytest catches it before deployment.  

Model prediction code
✔ Data preprocessing code
✔ FastAPI endpoints
✔ Training scripts
✔ Output shapes & types
✔ Data validation
CI/CD checks

It checks Whether ML model gives correct output or not, FastAPI endpoint returns correct response or not,  Your function calculates correct values or not.
pytest is run every time you: push code,  create PR,  run CI/CD,  deploy a model  
pip install pytest  

PyYAML:
							


## Python Interview Questions & Answers
**Whenever you want to COUNT something → use Dictionary:**
```
numbers = [3,4,4,5,6,7,8,9,2,2,9,9]

count = {}   # empty dictionary

for n in numbers:
    if n in count:
        count[n] += 1    # already exists → add 1
    else:
        count[n] = 1     # first time seen → set to 1

print(count)
```







     


