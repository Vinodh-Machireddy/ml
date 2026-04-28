# KUBEFLOW  
## What is Kubeflow?  
Kubeflow is a Kubernetes-native platform designed to run the entire machine learning lifecycle—data-loading, training, evaluation, pipelines, deployment, monitoring, retraining.  

## Why Kubeflow?  
Kubeflow is used to make Machine Learning production-ready models, the same disciplined way how we develope a traditional software app is built and deployed using DevOps.  

## Kubeflow Components:  
<img width="273" height="149" alt="image" src="https://github.com/user-attachments/assets/c7feb8cf-2211-4364-9cca-825e2751db32" />   

DSL:  
Kubeflow uses a Python-based DSL to declaratively define ML pipelines, which are later compiled into Kubernetes-executable YAML workflows.  

## Kubeflow Pipeline:
Kubeflow Pipelines (KFP) is a platform for building and running ML workflows as automated pipelines on Kubernetes.

We write pipelines using Python DSL  
Kubernetes executes it as a DAG of pods  
Each step runs in its own container, but all steps are connected using inputs and outputs.  

**Why?**  
Automation – no manual steps  
Reproducibility – same pipeline, same result  

## 1. Component (@dsl.component) 
A component is one single step in your pipeline. Each component is like one small independent worker that does one job.  
**Key points:**  
- Each component runs inside its own Docker container  
- Components are isolated — one component's failure doesn't corrupt another  

**Resources, retry, timeout:**   
They decide how the training pod runs in Kubernetes. They do NOT change ML logic.  
They control runtime behaviour of the pod.  
```
 		train_task.set_cpu_limit(“4")   This training pod can use maximum 4 CPU cores.  
        train_task.set_memory_limit(“8Gi")  # This pod can use maximum 8 GB RAM.  
        train_task.set_retry(3)  # If this step fails, Kubeflow will try again up to 3 times.  
        train_task.set_timeout(7200) # If this step runs more than 7200 seconds (2 hours), Kubeflow will stop it automatically.  
```  
**environment variables:**   
It adds an environment variable inside the training pod so your training code knows in which environment it is running.  
```
dsl.EnvVar(name="ENVIRONMENT", value=environment) this creates  
		ENVIRONMENT=prod   or dev qa …etc
Runtime: When the training pod starts, inside the container you will have:
		echo $ENVIRONMENT # output: prod
Your Python code can read it like this:
		import os
		env = os.getenv("ENVIRONMENT")
```  
**/secrets:**   
```
train_task.add_secret(secret_name=“ev-telemetry-secret", mount_path="/secrets" )  
 It securely gives sensitive information to the training pod.  
When the training pod starts, Kubernetes mounts the secret at:  /secrets  Inside that folder you will see files like:  
		/secrets/DB_PASSWORD
		/secrets/API_KEY
- Your training code can read them like:
   		with open("/secrets/DB_PASSWORD") as f:
   		 password = f.read()
```  
**Caching:**   
train_task.set_caching_options(True) # If you run the pipeline again It tells Kubeflow to reuse the result of this step if nothing has changed.  

**evaluation:**  
The training step created a model file, for example:   
/models/ev_fault_model.joblib  
That file location is saved as an output of the training step. Now we pass that output to the evaluation step. So you are telling Kubeflow: “Use the same model that was just trained.”  
This creates a dependency: Train step  →  Evaluate step. Evaluate will not start until training finishes.  

If evaluation says model is good → register it  
accuracy = 0.92  
metrics = "accuracy=0.92, recall=0.90”  

**Promotion Gate:**   
dsl.Condition(…) This creates a decision point in the pipeline. Run the next step only if this condition is true.”  
		eval_task.outputs["accuracy"] >= accuracy_threshold  

This is just another Kubeflow component you create, like your other steps.  
Inside that component, you write code that:  
Uses KServe SDK or kubectl  
Creates / updates an InferenceService  
Points it to your model in S3 / GCS / MinIO  

**Drift monitoring:** 
In real life, after a model is deployed, the data it sees can slowly change. When data changes too much, the model’s accuracy also drops. This problem is called data drift or model drift. This step makes sure your system can detect that change early.  
Compiler:  
This part is used to convert your Kubeflow pipeline Python code into a YAML file.  
Kubeflow cannot run Python directly. It runs pipelines from a YAML definition.  
Output: ev_battery_fault_pipeline.yaml     which we upload in Kubeflow UI  
Final:  
That YAML is:  
Versioned in Git  
Deployed via CI/CD  
Uploaded to Kubeflow UI  

> **NOTE:** Each Kubeflow pipeline component runs in its own isolated pod, but components are linked through the pipeline using input–output dependencies. The pipeline builds a DAG, and Kubeflow executes the pods in that order, passing outputs from one component to the next.  

## 3. Compilation
After writing the pipeline in Python, you compile it into a YAML file. This YAML is what Kubernetes actually understands and executes.  

## 4. KFP Client (Submitting the Pipeline) 






## DSL — Domain Specific Language  
DSL stands for Domain Specific Language — a language or syntax designed for one specific purpose rather than general programming.  
```
from kfp import dsl

@dsl.pipeline(
    name        = "Battery Fault Pipeline",
    description = "EV Battery Fault Classification"
)
def battery_fault_pipeline():
    pass
```
Here dsl means — KFP provides its own special syntax specifically for defining ML pipelines.  
kfp.dsl is not general Python — it is a mini language on top of Python designed only for:  
- defining pipeline steps  
- connecting components  
- passing artifacts between steps  
> kfp.dsl = Python + special pipeline syntax = DSL for ML pipelines.



# ML Pipeline End To End
Once Data Scientist Gives You in Jupyter Notebook:  
Problems:  
```
❌ hardcoded file path
❌ no S3 support
❌ no error handling
❌ no logging
❌ no @component — cannot run on Kubernetes
❌ no input/output — cannot connect to next step
❌ no Reusable functions 
❌ imports outside function
❌ no docstring
```
we need to convert into a KFP components i.e @dsl.component()  

1. we create Separate Python files for each component and Create Project Structure in github. (load_data.py, validate_data.py, preprocess.py, train.py, evaluate.py, register.py)   

**After MLOps Engineer — load_data.py:**
```
# ✅ components/load_data.py
# MLOps engineer converted to KFP component 

import kfp
from kfp import dsl
from kfp.dsl import component, pipeline, Input, Output, Dataset, Model, Metrics
from components.load_data     import load_data		# from your own components package
from components.validate_data import validate_data
from components.preprocess    import preprocess_data

@component(						# @componentwraps function as Docker container		
    packages_to_install = [     # what MLOps adds ✅
        "pandas==1.5.3",
        "boto3==1.26.0"
    ],
    base_image = "python:3.9"   # what MLOps adds ✅
)
def load_data(
    bucket   : str,             # what MLOps adds ✅ — no hardcoding
    data_path: str,             # what MLOps adds ✅ — no hardcoding
    dataset  : Output[Dataset]  # what MLOps adds ✅ — pass to next step
):

    # imports INSIDE function        # what MLOps adds ✅
    import pandas as pd
    import logging

    logging.basicConfig(level=logging.INFO)
    logger = logging.getLogger(__name__)

    # build S3 path
    s3_path = f"s3://{bucket}/{data_path}"
    logger.info(f"Loading data from: {s3_path}")

    try:
        # load data
        df = pd.read_csv(s3_path)
        logger.info(f"Loaded {len(df)} samples")

        # save to output — passes to next component
        df.to_csv(dataset.path, index=False)
        logger.info("Data saved to output ✅")

    except Exception as e:
        logger.error(f"Failed to load data: {e}")
        raise
```
**After MLOps Engineer — validate_data.py:**
``` 
# ✅ components/validate_data.py
# MLOps engineer converted to KFP component

from kfp.dsl import component, Input, Output, Dataset

@component(
    packages_to_install = [     # what MLOps adds ✅
        "pandas==1.5.3"
    ],
    base_image = "python:3.9"   # what MLOps adds ✅
)
def validate_data(
    dataset          : Input[Dataset],   # what MLOps adds ✅ — receives from load_data
    validated_dataset: Output[Dataset]   # what MLOps adds ✅ — passes to preprocess
):
    """
    Validates battery sensor data quality.    # what MLOps adds ✅

    Args:
        dataset          : input data from load_data component
        validated_dataset: cleaned data passed to preprocess component
    """
    # imports INSIDE function        # what MLOps adds ✅
    import pandas as pd
    import logging
    import sys

    # logging setup                  # what MLOps adds ✅
    logging.basicConfig(
        level  = logging.INFO,
        format = "%(asctime)s | %(levelname)s | %(message)s",
        stream = sys.stdout
    )
    logger = logging.getLogger(__name__)

    logger.info("Starting data validation...")
	# error handling                 # what MLOps adds ✅
```  

# PIPELINE DEFINITION
```
# pipeline/pipeline.py

from kfp import dsl
from components.load_data     import load_data
from components.validate_data import validate_data

@dsl.pipeline(name="Battery Fault Pipeline")
def battery_fault_pipeline(
    bucket   : str = "daimler-battery",
    data_path: str = "data/processed/train.csv"
):
    # step 1 — load data
    load_task = load_data(
        bucket    = bucket,
        data_path = data_path
    )

    # step 2 — validate
    # receives dataset OUTPUT from load_data ✅
    validate_task = validate_data(
        dataset = load_task.outputs["dataset"]
    )
		validate.set_cpu_limit("2")            # some pandas processing
        validate.set_memory_limit("4Gi")       # data fits in 4GB
        validate.set_retry(1)                  # data validation rarely fails randomly
        validate.set_timeout(1800)             # 30 min max

    # step 3 — preprocess (next component)
    # receives validated_dataset OUTPUT from validate_data ✅
    # preprocess_task = preprocess(
    #     dataset = validate_task.outputs["validated_dataset"]
    # )
```
> @componentwraps function as Docker container — runs on Kubernetes pod  
> @dsl.pipelinewraps function as KFP pipeline — connects all components

**How Kubernetes Runs This:**   
```
GitHub Actions triggers pipeline
        ↓
KFP creates Pod 1 — load_data
    pulls python:3.9 image
    pip install pandas boto3
    runs load_data()
    saves load_data.CSV to shared storage
    pod shuts down
        ↓
KFP creates Pod 2 — validate_data
    pulls python:3.9 image
    pip install pandas
    reads load_data.CSV from shared storage
    runs validate_data()
    saves validated_data.CSV to shared storage
    pod shuts down
        ↓
KFP creates Pod 3 — preprocess
    ... and so on
```
> KFP uses Kubernetes Persistent Volume (PV) / S3

**Feature Store:**  
A Feature Store is a centralized repository that stores, manages, and serves features for ML models — both for training and real-time serving.
```
Raw Data (BMS Sensors)
        ↓
Feature Engineering
        ↓
┌───────────────────────────────────┐
│           FEATURE STORE           │
│                                   │
│  Offline Store    Online Store    │
│  (S3 / Hive)      (Redis / DDB)  │
│  historical data  real-time data  │
│  training         serving         │
└───────────────────────────────────┘
        ↓               ↓
  Training           Serving
  Pipeline           Pipeline
```

> Feast    ->    opensource    ->  most common in MLOps
> AWS SageMaker Feature Store -> aws managed

**Example:**   
Data Scientist computes in training,  MLOps Engineer computes in serving:, different code — different result!   
With Feature Store — Solution  
```
Step 1 — Define feature ONCE in Feature Store:

    feature name : voltage_rolling_mean
    definition   : average voltage of last 5 readings
    computed by  : pandas rolling(5).mean()
    updated every: 5 seconds

Step 2 — Training fetches from Feature Store:
    voltage_rolling_mean = 3.48  ← from Feature Store ✅

Step 3 — Serving fetches from Feature Store:
    voltage_rolling_mean = 3.48  ← SAME value from Feature Store ✅

# SAME value everywhere → no confusion ✅
```






