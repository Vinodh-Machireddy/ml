
							PYTHON
 	python packages:
================
numpy - Numerical Computing
pandas - Data Manipulation
scikit-learn - Core ML Library
mlflow - Experiment Tracking & Model Registry
dvc - Data & Model Versioning
fastapi - API for Model Serving
optuna & XGBoost - Hyperparameter Tuning
yyaml - YAML
pytest - Tests
joblib - Model serialization
						=======

Introduction
Data Types
KeyWords

python is a dynamically typed programming language.  variables are key for any program lang bcz using variable  we can make our program dynamic.  Variables are flagship in python.

Dynamically typed programming languages:
Python, Ruby, JavaScript, PHP, Perl, Lua, Tcl, Groovy.

Statically Typed Languages:
C, C++, Java, C#, Go.


Shell Scripting  VS Python Scripting
============================
- in devops mostly we use linux systems bcz windows has less security. In Windows/Mac it uses rich UI. In Linux default CLI. The main purpose of Shell Scripting is to interact with Linux systems and get information. We can write the commands one after the other or set of command in scriptFile.sh and execute.

Platform-specific behavior (e.g., Bash for Linux, PowerShell for Windows).
Simpler for small tasks but syntax can become complex for larger scripts.
Best for system-level tasks like command chaining and file operations.

Cross-platform; runs on any system with Python installed.
 Easy to learn with a clean syntax; suitable for beginners.
Suitable for writing complex programs, Interact with API, automation, and data processing.

in real-world scenario we might get a chance to work on windows that’s why python is needed.
Python solves 


NOTE:- As a devops engineer it not mandatory to use python to fetch the information from linux and windows machines we can use Ansible also to achieve this task.
NOTE:- When we automating the things, normally we talk to API (OR) CLI.
	- CLIs are great for immediate and straightforward automation tasks, 
	- while APIs provide more flexibility and are better suited for complex, programmatic integrations.


DATATYPES:
==========
Sequence
	String   len(), split(), replace(), find(), strip(), lower()
	list.     numbers = [10, 20, 30, 40]
	tuple    data = (1, 2, 3)
	range.     for i in range(3):
Non-sequence data types
Set
Frozen Set
Dict
Numeric
   	 Int.      abs(-7)
    	Float      round(3.14159265359, 2)
Mapping
Boolean
Binary
None
Custome 

# String: 
	- A sequence of characters. Enclosed in single ('), double (") or triple quotes.
                 Eg:- name = "Vinodh"

String concatenation:
Syntax: result = string1 + string2
str1 = "Hello"
str2 = "World"
result = str1 + " " + str2
print(result)

str(object)
object: The object you want to convert to a string. This could be of any type, such as numbers, lists, dictionaries, etc.
Note:- we cannot concatenate string to integer. If you want Use str()

Syntax: len(object)
object: This can be a sequence (e.g., string, list, tuple, range)
text = "Welcome To VmTutes"
length = len(text)
print("wow length", length)

text = “Welcome To VmTutes"
uppercase = text.upper()
lowercase = text.lower()
print("Uppercase:", uppercase)
print("Lowercase:", lowercase)

Syntax: string.replace(old, new, count)
text = "Python is awesome"
new_text = text.replace("awesome", "great")
print("Modified text:", new_text)

Syntax: string.strip([chars])  
text = “   Some spaces around      "
stripped_text = text.strip()
print("Stripped text:", stripped_text)
Note:- It removes spaces from both start and end of the string. but not from the middle

text = "Welcome To VmTutes"
vm = "is"
if vm in text:
    print(vm, "found in the text")

Syntax: string.split(separator, maxsplit)   
text = "Welcome To VmTutes"
words = text.split(“-")[1]
print("Words:", words)    

	List:
	A collection of ordered, changeable (mutable) elements. Enclosed in square brackets [ ]

            Vm_list = [“Vinodh”,  1, 2, 3  “Meghan”,  10.8] # any datatype we can add. It just consider as one element.
            Vm_list.append(“nethra”)
             Vm_list.remove(“nethra”)
	print(Vm_list[0])
	print(len(Vm_list))
	new_list = Vm_list[0:3], print(new_list) only 0, 1, 2 elements print from the list.
         
	numbers = [1, 10, 5, 2]
	print(sorted(numbers))  # Output: [1, 2, 5, 10]


	Tuple:
	A collection of ordered, unchangeable (immutable) elements. Enclosed in parentheses ( ).
	     eg:- info = (1, "apple", 3.5)

           Set:
          A set in Python is an unordered collection of unique elements. It is useful for mathematical operations like union, intersection, and difference. 
           Eg:-  my_set = {1, 2, 3, 4, 5}
            set1 = {1, 2, 3, 4}
		set2 = {3, 4, 5, 6}

union_set = set1.union(set2)  # Union of sets
intersection_set = set1.intersection(set2)  # Intersection of sets
difference_set = set1.difference(set2)  # Difference of sets

print("Union of set1 and set2:", union_set)
print("Intersection of set1 and set2:", intersection_set)
print("Difference of set1 and set2:", difference_set)


# Float
result1 = num1 / num2
print("Addition:", result1)
result5 = round(3.14159265359, 2)  # Rounds to 2 decimal places
print("Rounded:", result5)


# Integer 
num1 = 10
num2 = 5

result1 = num1 // num2
print("Integer Division:", result1)
# Absolute Value
result3 = abs(-7)
print("Absolute Value:", result3)


Mapping(dict): 
In Python, a mapping data type is a type that stores data in key-value pairs. he main mapping data type in Python is the dict (dictionary). 

student = {
    "name": "Vinodh",
    "age": 30,
    "role": "MLOps Engineer"
}

print(student["name"])          # To access a value
student["city"] = "Chennai"     # Add or update a new key-value pair
del student["city"]              # Delete a key-value pair
print(student.keys())           # Get all keys
print(student.values())         # Get all values
print(student.items())         # Get all items (key-value pairs)
student.clear()                 # Clear all items

ec2_instances_info = [           # number of dictionaries in a list
    {
        "id": "instance-001",
        "type": "t2.micro",
    },
    {
        "id": "instance-002",
        "type": "t2.medium",
    },
    {
        "id": "instance-003",
        "type": "t2.xlarge",
    }
]

print(ec2_instances_info[1][“id"])

Github API Call:

import requests
response = requests.get('https://api.github.com/repos/kubernetes/kubernetes/pulls')
results = response.json()

for i in results:
    print(i['user']['login'])


Boolean:
In Python, the Boolean data type (bool) is used to represent truth values — that means something is either True or False.  Data type name : bool

Boolean values are mainly used in if-else, loops, and comparisons.
print(x > y)   # True


if age >= 18:
    print("Eligible to vote")
else:
    print("Not eligible")

In Python, some values are automatically considered False in Boolean context:
False
None
0
"" (empty string)
[] (empty list)
{} (empty dict)
() (empty tuple)
Everything else is treated as True.
Binary Data Types:
In Python, the binary data type is used to store raw binary information — that is, data in the form of bytes (0s and 1s) instead of text.
This type is commonly used when dealing with files, images, network data, or serial communication, where data is not human-readable.
Python provides three built-in binary types:


None data type:
In Python, the None data type represents the absence of a value or nothingness.
When do we use None?  -> To show empty or no value



							KeyWords:
							=========


If
Else
Elif


							Variables:
							========

variables are used to store data values. A variable is essentially a name that refers to a value.
- It store data, DATA can be any type like String, Float, Boolean….etc
Types:
———
Global Variable
Local Variable

Naming Conventions:
Variable names must start with a letter (a-z, A-Z) or an underscore (_).
The rest of the name can include letters, numbers, or underscores.
Best Practice only Lower case variable names
Snake casing: Vinodh_Machireddy_DevOps (or) Camel Casing: vinodh_Machireddy_DevOps
Keep variable names as descriptive as possible. vmd
Reserved keywords (e.g., if, while, class, import) cannot be used as variable names.


							FUNCTIONS
							==========
In Python, functions are blocks of reusable code that perform a specific task. They make programs modular, easier to understand, and more maintainable.

Python provides several built-in functions like print(), len(), type(), def() etc.

Reusability
Readability
Debugging

Functions always follows 3 principles I.e 
Taking Input
Execute the desired logic
Return the Output 

Best practice functions:
——————————
def addition(n1, n2):
    Add = n1 + n2
    return add

def sub(n1, n2):
   s = n1 - n2
    return sub
     
print(addition(2, 5)) ——> invoking & printing function output
print(sub(2, 5))


							MODULES
							=========
In Python, modules are .py files containing Python code (e.g., functions, variables, or classes) that can be imported and reused in other Python programs. Modules are collection of functions. (Modularity Approach).

Built-in Modules:
These come pre-installed with Python.
Examples:
math: Provides mathematical functions.
os: Interacts with the operating system.
sys: Provides access to system-specific parameters and functions.
datetime: Deals with date and time.

Third-party Modules:
Here are some widely-used packages:
Aws - boto3
GitHub - GitHub
Jira - jira
NumPy: Numerical computing
Pandas: Data manipulation
Matplotlib/Seaborn: Data visualization
Flask/Django: Web development
TensorFlow/PyTorch: Machine learning
Requests: HTTP requests

NOTE: when we call packages indirectly it is a module. Inside module collection of functions.

User-defined Modules:
import mymodule 
print(mymodule.greet("Alice"))  


PACKAGES/LIBRARIES
===================
packages are collection of modules.
Eg:- suppose we’ve amazon.com ecommerce application where we will not write entire app code in a single .py file.   We use diff .py files for diff microservices like add to cart, my acc, orders ….etc

Why Use Packages?
Organization: Makes large projects easier to manage by grouping related code into separate files.
Reusability: Code in a package can be reused in other projects.

Package Structure
A package is a directory that contains:
Sub-packages (optional)
Modules (Python files with .py extension)
An optional __init__.py file, which defines the package’s behavior when it’s imported.

PYPI - python package index where we can download modules and packages just like docker registry to download container images.

PIP - Python’s package installer.
python -m pip install --upgrade pip

Virtual Env
=========
A virtual environment in Python is an isolated environment that allows you to install and manage packages separately from the global Python environment. If we want to work with multiple projects in on virtual machine/ec2-instance would prefer virtual Env.

- It creates logical separation on virtual machine/ec2-instance for python packages.

python -m venv vmtutes   #to create
source vmtutes/bin/activate
pip install requests numpy pandas
Pip list
deactivate
rm -rf venv

Command Line Arguments:
=====================
Command-line arguments in Python are a way to give input to a Python program when you run it from the command line or terminal.

Syntax: python example.py arg1 arg2 arg3

import sys

def addition(n1, n2):
    return n1 + n2
def subtract (n1, n2):
    return n1 - n2
def multiply (n1, n2):
    return n1 * n2

n1 = int(sys.argv[1])
operation = sys.argv[2]
n2 = int(sys.argv[3])

if operation == "addition":
    print(addition(n1, n2))
elif operation == "subtract":
    print(subtract(n1, n2))
elif operation == "multiply":
    print(multiply(n1, n2))

print("Arguments:", sys.argv)
print("Number of arguments:", len(sys.argv))
print("First argument:", sys.argv[1])
print("Second argument:", sys.argv[2])
print("Third argument:", sys.argv[3])



Environment variables:
—————————-
Environment variables in Python are like little notes your operating system keeps to tell your programs about important settings. They can store information such as:
Paths to files or directories.
API keys or credentials.
Configuration details like debug settings.

Export password=“root123” 
Export apitoken=“45rf6tgy78u9i9o0p9o”

import os
print(os.getenv(“password”)) # get password from environment variable
print(os.getenv(“apitoken”))

Note: Builtin env variable : env



								OPERATOR
								==========

Regular Division (/)	16/3 which gives division value i.e 5.3333							
Floor Division (//):- It rounds down the result to the nearest whole number. 16//3 i.e 5
(%) modulus:  16%3  Remainder is 5

Assignment operators: 
X = 10
x=x+10
x+=10

Identity Operators:
is: Returns True if two variables point to the same object.
is not: Returns True if two variables do not point to the same object.
A = 10
B = 20
A is b ——> false
B is not a —-> true

logical operator:
Logical operators in Python are used to combine multiple conditions and determine whether they are True or False.
and
Returns True if both conditions are True. If one condition is false then False.
 or
Returns True if at least one condition is True.
not
Reverses the result of a condition.
If the condition is True, not makes it False, and vice versa.

Relational Operator:
Relational operators in Python are used to compare two values. They check the relationship between the values and return True or False based on the result.
print(5 == 5)  # True
print(5 == 3)  # False

What is IDLE?
It is the default Python IDE (Integrated Development Environment) that comes with the standard Python installation.
It's designed to be simple and beginner-friendly, making it great for learning Python, without writing py file.
To exit the IDLE terminal:  exit(). (Or) Control + d

NOTE:-  Python, operators are essential for the interpreter and compiler to understand and execute specific operations

						

						Conditional Handling 
						==================



import sys

vinodh = sys.argv[1]

if vinodh == "hello":
    print("Hello, Vinodh!")
elif vinodh == "hi":
    print("Hi, Vinodh!")
elif vinodh == "bye":
    print("Bye, Vinodh!")
elif vinodh == "goodbye":
    print("Goodbye, Vinodh!")
else:
    print("Unrecognized greeting.")
        
						Loops
						=====	
Loops are a fundamental concept in programming, and they allow you to perform repetitive tasks efficiently. In Python, there are two primary types of loops: "for" and "while."

Syntax:- for variable in sequence:

numbers = [1, 2, 3, 4, 5]
tuple = (10, 20, 30, 40, 50)
colors = ["red", "green", "blue"]

for i in colors:
    print(i)

Break statement:
numbers = [1, 2, 3, 4, 5]
for number in numbers:
    if number == 3:
        break
    print(number)

Continue statement:
numbers = [1, 2, 3, 4, 5]
for number in numbers:
    if number == 3:
        continue
    print(number)
#### While Loop

The "while" loop continues to execute a block of code as long as a specified condition is true. It's often used when you don't know in advance how many times the loop should run.

**Syntax:**

```python
while condition:
    # Code to be executed as long as the condition is true
```

**Example:**

```python
count = 0
while count < 5:
    print(count)
    count += 1


							Python Libraries for ML
							=======================

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

# Load dataset
data = load_breast_cancer()
X = data.data
y = data.target

# Split dataset into training and testing sets  
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=None
                                                    )
# Initialize and train the Logistic Regression model
model = LogisticRegression(max_iter=10000)
model.fit(X_train, y_train)

# Make predictions on the test set
pred = model.predict(X_test)
accuracy = accuracy_score(y_test, pred)

print(f"accuracy: {accuracy * 100:.5f}%")

# Save the trained model to a file
import joblib
joblib.dump(model, 'logistic_regression_model.pkl')


NOTE: random_state=None —> it changes the split samples for each code run.

random_state=0,1,7,42,100,999.  So numbers are needed to freeze randomness. same split for every code run. all work exactly the same.

test_size only decides how many samples go to test
random_state only decides which samples are selected


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


import joblib
import mlflow
import mlflow.sklearn
from mlflow.models import infer_signature

from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# 1. Load dataset
data = load_breast_cancer()
X = data.data
y = data.target

# 2. Split dataset (use fixed seed for reproducibility)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
# 3. Start an MLflow run and do training, logging, saving, and verification
with mlflow.start_run():

    # Train model
    model = LogisticRegression(max_iter=10000)
    model.fit(X_train, y_train)

    # Predict and evaluate
    preds = model.predict(X_test)
    acc = accuracy_score(y_test, preds)

    # Log parameter and metric
    mlflow.log_param("max_iter", 10000)
    mlflow.log_metric("accuracy", acc)

    # Create signature and input_example to avoid MLflow warnings
    signature = infer_signature(X_test[:50], model.predict(X_test[:50]))
    input_example = X_test[:5]

    # Log model to MLflow (use 'name' not deprecated 'artifact_path')
    mlflow.sklearn.log_model(
        sk_model=model,
        name="model",
        signature=signature,
        input_example=input_example
    )

    # Also save model locally with joblib (optional, for local use)
    joblib.dump(model, "logistic_regression_model.pkl")

    # Load the saved local model and verify predictions
    loaded_model = joblib.load("logistic_regression_model.pkl")
    loaded_preds = loaded_model.predict(X_test)
    loaded_acc = accuracy_score(y_test, loaded_preds)

    # Log loaded model accuracy too (optional)
    mlflow.log_metric("loaded_accuracy", loaded_acc)

    # Print results
    print(f"Training run accuracy: {acc * 100:.5f}%")
    print(f"Loaded model accuracy: {loaded_acc * 100:.3f}%")


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

pip install mlflow
Mlflow ui # http://127.0.0.1:5000


DVC: DVC = Git for Data + Models 
Git cannot store: large datasets, model files, intermediate artifacts So DVC handles those.

pip install dvc
from dvc.repo import Repo
repo = Repo(".")
stages = repo.stages
print(stages)

pip install dvc[s3].  # cloud storage integration
pip install dvclive.  # internal storage engine
dvclive 			     # ML metrics & plots logging
 

pip install dvc
git init. #Inside your project folder:
dvc init. #Inside your project folder:
dvc add data.csv
git add .
git commit -m "tracked data with dvc"



FASTAPI:
This code creates a FastAPI web service that loads a trained ML model (iris_model.pkl) and exposes an API /predict to make predictions.

 pip install fastapi uvicorn

from fastapi import FastAPI
from pydantic import BaseModel
import joblib

app = FastAPI()

# Load model
model = joblib.load("iris_model.pkl")

# Input schema
class Input(BaseModel):
    sepal_length: float
    sepal_width: float
    petal_length: float
    petal_width: float

@app.post("/predict")
def predict(data: Input):
    features = [[
        data.sepal_length,
        data.sepal_width,
        data.petal_length,
        data.petal_width
    ]]

    pred = model.predict(features)[0]
    return {"prediction": int(pred)}

uvicorn app:app --reload
http://127.0.0.1:8000/docs


								XGBOOST:
								========
XGBoost is a machine learning algorithm that builds many small decision trees, each tree correcting the mistakes of the previous tree.

Used for Fast training, Very high accuracy, Works well for CSV, excel(structured  data)

								OPTUNA:
								========
Optuna is a tool that automatically searches for the best hyperparameters for your ML model. Because choosing good parameters manually is difficult.


							PYTEST:
							=======
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
							======






							DOCKER:
							=======


							MLOPS CI/CD:	
							============

Traditional DevOps CI/CD = build → test → deploy code
MLOps CI/CD = build → test → train model → evaluate → version → deploy model







Access key : AKIAV6HFIYWSAKH657NZ
Secret access key: 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


eksctl create cluster \
  --name mlops-eks-cluster \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3

aws eks update-kubeconfig --region us-east-1 --name mlops-eks-cluster

eksctl delete cluster --name mlops-eks-cluster --region us-east-1



Ec2-instance
sudo apt update && sudo apt upgrade -y

client_loop: send disconnect: Broken pipe
Client-Side
1. vi .ssh/config
2. Host *
    ServerAliveInterval 60
    ServerAliveCountMax 5

                 (OR)
Host *
    ServerAliveInterval 10
    ServerAliveCountMax 30
    TCPKeepAlive yes

3. chmod 600 ~/.ssh/config

Server-Side
1.Edit /etc/ssh/sshd_config on the server.
2. Add or uncomment:
	ClientAliveInterval 60
	ClientAliveCountMax 5
3. Restart the SSH service (sudo service sshd restart).




PROJECT
=======

							train.py file:
							—————
import joblib
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier

# Load the iris dataset
iris = load_iris()
X, y = iris.data, iris.target

# Train a random forest classifier
model = RandomForestClassifier()
model.fit(X, y)

# Save the trained model
joblib.dump(model, 'vinodh.pkl')

print("Model trained and saved as vinodh.pkl")

NOTE:
import joblib
model = joblib.load('vinodh.pkl')

# Predict one sample
sample = [[5.1, 3.5, 1.4, 0.2]]
print("Prediction:", model.predict(sample))

test_load.py is a small testing script used to check two things:
✔️ A) Your saved model file (vinodh.pkl) can be loaded
✔️ B) The model can make a prediction correctly
It is NOT part of the training. It is only used to verify the model works before you move to FastAPI / Docker / EKS deployment.

						FastAPI(Server.py File):
							———-
from fastapi import FastAPI
import joblib
import numpy as np

model = joblib.load('app/vinodh.pkl')

class_names = np.array(['setosa', 'versicolor', 'virginica'])

app = FastAPI()

@app.get('/')
def read_root():
    return {'message': 'Iris model API'}

@app.post('/predict')
def predict(data: dict):
    """
    Predicts the class of a given set of features.

    Args:
        data (dict): A dictionary containing the features to predict.
        e.g. {"features": [1, 2, 3, 4]}

    Returns:
        dict: A dictionary containing the predicted class.
    """
    features = np.array(data['features']).reshape(1, -1)
    prediction = model.predict(features)
    class_name = class_names[prediction][0]
    return {'predicted_class': class_name}
NOTE:- To Test locally: uvicorn app.server:app --reload


client-side inference test script.

import json
import requests

data = [[4.3, 3. , 1.1, 0.1],
       [5.8, 4. , 1.2, 0.2],
       [5.7, 4.4, 1.5, 0.4],
       [5.4, 3.9, 1.3, 0.4],
       [5.1, 3.5, 1.4, 0.3],
       [5.7, 3.8, 1.7, 0.3],
       [5.1, 3.8, 1.5, 0.3],
       [5.4, 3.4, 1.7, 0.2],
       [5.1, 3.7, 1.5, 0.4],
       [4.6, 3.6, 1. , 0.2],
       [5.1, 3.3, 1.7, 0.5],
       [4.8, 3.4, 1.9, 0.2]]

url = 'http://0.0.0.0:8000/predict/'

predictions = []
for record in data:
    payload = {'features': record}
    payload = json.dumps(payload)
    response = requests.post(url, data=payload)
    predictions.append(response.json()['predicted_class'])

print(predictions)

							Dockerfile:

FROM python:3.11

WORKDIR /code

COPY ./requirements.txt /code/requirements.txt

RUN pip install --no-cache-dir -r /code/requirements.txt

COPY ./app /code/app

EXPOSE 8000

CMD ["uvicorn", "app.server:app", "--host", "0.0.0.0", "--port", "8000"]

NOTE:- docker build -t mlops-fastapi .
	  docker run -p 8000:8000 mlops-fastapi

						PUSH IMAGE TO AWS ECR
aws ecr create-repository --repository-name mlops-fastapi

Login to ECR
aws ecr get-login-password --region us-east-1 \
| docker login --username AWS --password-stdin \
408502715812.dkr.ecr.us-east-1.amazonaws.com
Tag & push image
docker tag mlops-fastapi:latest \
408502715812.dkr.ecr.us-east-1.amazonaws.com/mlops-fastapi:latest

docker push \
408502715812.dkr.ecr.us-east-1.amazonaws.com/mlops-fastapi:latest

						KUBERNETES DEPLOYMENT (EKS)


k8s-deployment.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: mlops

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mlops-api
  namespace: mlops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mlops-api
  template:
    metadata:
      labels:
        app: mlops-api
    spec:
      containers:
      - name: mlops-api
        image: 408502715812.dkr.ecr.us-east-1.amazonaws.com/mlops-fastapi:latest
        ports:
        - containerPort: 8000

---
apiVersion: v1
kind: Service
metadata:
  name: mlops-api-service
  namespace: mlops
spec:
  type: LoadBalancer
  selector:
    app: mlops-api
  ports:
    - port: 80
      targetPort: 8000
                                                                                                     

kubectl apply -f k8s-deployment.yaml
Test inference
curl -X POST http://<LB-DNS>/predict \
-H "Content-Type: application/json" \
-d '{"data": [[5.1,3.5,1.4,0.2]]}'

					CI/CD WITH GITHUB ACTIONS
Push code to GitHub
Dockerfile
app/
model.joblib
k8s-deployment.yaml
.github/workflows/deploy.yml

Add GitHub Secrets
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
ECR_REGISTRY
ECR_REPOSITORY
EKS_CLUSTER_NAME

GitHub Actions pipeline
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - uses: aws-actions/amazon-ecr-login@v2

      - run: docker build -t ${{ secrets.ECR_REPOSITORY }} .

      - run: |
          docker tag ${{ secrets.ECR_REPOSITORY }}:latest \
          ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest

      - run: docker push \
          ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:latest

      - run: |
          aws eks update-kubeconfig \
          --region ${{ secrets.AWS_REGION }} \
          --name ${{ secrets.EKS_CLUSTER_NAME }}

      - run: kubectl rollout restart deployment mlops-api -n mlops

Code push → GitHub Actions → Docker build → ECR push → EKS rollout → Live ML prediction



HOW TO WRITE THIS ON YOUR RESUME
Project: End-to-End MLOps Pipeline on AWS
Built ML inference service using FastAPI + scikit-learn
Containerised application using Docker
Pushed images to AWS ECR
Provisioned AWS EKS cluster using eksctl
Deployed application using Kubernetes Deployments & Services
Debugged IAM, CloudFormation, and nodegroup issues
Exposed service via AWS LoadBalancer
Implemented live inference and validated predictions

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

