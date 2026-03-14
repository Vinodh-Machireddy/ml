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

### 1. Component (@dsl.component)
A component is one single step in your pipeline. Each component is like one small independent worker that does one job.  
**Key points:**
Each component runs inside its own Docker container
Components are isolated — one component's failure doesn't corrupt another
You mention input/output types clearly so KFP knows how to pass data between steps
base_image tells which Docker image to use
packages_to_install installs dependencies at runtime  

### 2. Pipeline (@dsl.pipeline)
A pipeline connects multiple components together in a defined order.   

### 3. Compilation
After writing the pipeline in Python, you compile it into a YAML file. This YAML is what Kubernetes actually understands and executes.  

### 4. KFP Client (Submitting the Pipeline) 

**MLE → writes @dsl.component (logic, what runs inside pod)** 
What MLE owns:  
Component logic (training, feature processing, validation)  
Inputs & outputs of components  
Python code that runs inside the container  
Making components reusable & production-ready  
NOTE: ML engineers create reusable Kubeflow components containing the ML logic, which are then shared with the MLOps engineer. MLE decides what happens inside the pod.  

**MLOps → writes @dsl.pipeline (orchestration, how it runs in cluster)**  

### 1. First, I convert MLE's code into KFP components.

Once we receive raw code(Data Ingestion, Validation, Feature Eng, Training, Evaluation)  from MLE's. we convert it into KFP Components (@dsl.components). where we change: 
Change 1:  Add @dsl.component decorator        → KFP recognises it as pipeline step
Change 2:  Replace hardcoded input path         → Input[Dataset] — works on any machine
Change 3:  Replace local model save             → Output[Model] — saved to S3 automatically
Change 4:  Replace print() with Output[Metrics] → Metrics tracked in KFP UI
Change 5:  Add model metadata                   → Model labelled with framework, accuracy, etc.
Change 6:  Extract hyperparameters              → Configurable per run without code change
Change 7:  Move imports inside function         → Works inside KFP container
Change 8:  Create new file in components/       → MLE's original file stays untouched
Change 9:  Create Dockerfile                    → Production-grade containerisation
Change 10: Create requirements.txt              → Pinned versions for reproducibility
Change 11: Add stratify to train-test split     → Balanced class distribution
Change 12: Add n_jobs=-1                        → Use all CPU cores for faster training
Change 13: Add random_state=42                  → Same input = same output every time
> **NOTE:** INFRASTRUCTURE CHANGES (MLOps core work): 1 to 10 except 6th change. ||  CODE QUALITY IMPROVEMENTS (MLOps suggests to MLE or adds directly): 6,11,12,13.


 MLOps creates NEW FILE → components/training/train_fault_classifier.py
        Copies ML logic from MLE's file
        ADDS: @dsl.component, base_image, pinned packages
        REPLACES: hardcoded paths → Input/Output artifacts
        REPLACES: print → metric logging
        MOVES: imports inside function
        ADDS: metadata
        EXTRACTS: hyperparameters as function params


### 2. Second, I designed the pipeline definition  



What MLOps owns:  
Wiring components together  
Execution order & dependencies  
Resources (CPU, memory, GPU)  
Retries & timeouts  
Secrets & env variables  
Conditions (promotion gates)  
Monitoring & alerts  
CI/CD integration  
NOTE:- MLOps engineer integrates these components into a pipeline, and decides how pods run in the cluster.  
```
COMPONENT 1: Ingest EV Battery Telemetry  
@dsl.component(
    base_image="python:3.9-slim",
    packages_to_install=["scikit-learn", “joblib"]
)
COMPONENT 2: Data Validation
def validate_battery_data
COMPONENT 3: Feature Engineering
def generate_battery_features
COMPONENT 4: Train Fault Classification Model
def train_fault_classifier
COMPONENT 5: Evaluate Model
def evaluate_fault_model
COMPONENT 6: Register Model
def register_fault_model
COMPONENT 7: Drift Monitoring Setup
def setup_drift_monitoring
COMPONENT 8: Notification / Monitoring Hook
def notify_pipeline_status
```
```
@dsl.pipeline(
    name="ev-battery-health-fault-pipeline",
    description="Production-grade Kubeflow pipeline for EV Battery Health Fault Classification"
)
def ev_battery_fault_pipeline(
    data_version: str = “v1",              #These three are pipeline parameters
    accuracy_threshold: float = 0.90,
    environment: str = "prod"
):

    # Exit handler → monitoring always runs
    with dsl.ExitHandler(
        notify_pipeline_status(status="completed")
    ):
        # 1️⃣ Ingest telemetry
        ingest_task = ingest_battery_data(data_version=data_version)

        # 2️⃣ Validate sensor data         # raw_data_path=“/data/battery_raw_v1.csv"
        validate_task = validate_battery_data(raw_data_path=ingest_task.outputs["raw_data_path"] )

        # 3️⃣ Generate features.         # clean_data_path= “/data/battery_clean_v1.csv"
        feature_task = generate_battery_features(clean_data_path=validate_task.outputs["clean_data_path"] )

        # 4️⃣ Train fault classifier
        train_task = train_fault_classifier(features_path=feature_task.outputs["features_path"])

        train_task.set_cpu_limit(“4")  
        train_task.set_memory_limit(“8Gi") 
        train_task.set_retry(3) 
        train_task.set_timeout(7200)

        train_task.add_env_variable(dsl.EnvVar(name="ENVIRONMENT", value=environment) )

        train_task.add_secret(secret_name=“ev-telemetry-secret", mount_path="/secrets" )
        train_task.set_caching_options(True)

        # 5️⃣ Evaluate model
        eval_task = evaluate_fault_model(
            model_path=train_task.outputs["model_path"],
            features_path=feature_task.outputs["features_path"]
        )

        # 6️⃣ Promotion gate (safety critical)
            with dsl.Condition(eval_task.outputs["accuracy"] >= accuracy_threshold ):

            register_fault_model(
                model_path=train_task.outputs["model_path"],
                metrics=eval_task.outputs["metrics"]
            )
# ✅ KServe deployment step
    deploy_task = deploy_model_kserve(
        model_uri=register_task.outputs["model_uri"],
        service_name="ev-battery-fault-classifier",
        namespace="mlops-prod"
               )

        # 7️⃣ Drift monitoring setup
        setup_drift_monitoring(
            reference_data=feature_task.outputs["features_path"],
            model_name="ev_battery_fault_classifier"
        )


	# 8. COMPILER: Python DSL → YAML

if __name__ == "__main__":
    kfp.compiler.Compiler().compile(
        pipeline_func=ev_battery_fault_pipeline,
        package_path="ev_battery_fault_pipeline.yaml"
    )
```  
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
evaluation:  
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

**How outputs are stored in Kubeflow**
When a component finishes, its outputs are stored in two different ways:   
```  
Parameters → small values (strings, numbers, booleans)  
accuracy = 0.92  
status = "success"  
model_version = "v3"  

Artifacts → files / large data (Anything stored as a file: datasets, models, logs, reports)  
/data/battery_raw_v1.csv  
/models/ev_fault_model.joblib  
/metrics/report.json
```  

## Interview Explanation:  
- I automated ML pipeline orchestration using Kubeflow Pipelines by separating ML logic and orchestration clearly.   
- ML engineers focused on built reusable components using @dsl.component, which runs inside the pod – like data ingestion, validation, feature engineering, model training, and evaluation.  

- I  integrated the orchestration layer using @dsl.pipeline that defines execution order, dependencies, promotion gates, and production controls like resources, retries, secrets, and environment variables.  

- I added drift monitoring and notification hooks so the pipeline continues into post-deployment phase.  

- One important part of the orchestration was adding promotion gates. Since we were working on a safety-critical system, I added a condition that checks model accuracy before allowing it to move to the registry.   
- So instead of trusting every trained model, the pipeline itself makes the decision:  
- Only models that meet quality standards are promoted.  
- This removed human error and brought strong governance into the ML lifecycle.  

**Next, I focused on making the pipeline production-ready.**  
I added full runtime controls at the orchestration level:
- CPU and memory limits for heavy training jobs
- Retry logic for temporary failures
- Timeouts to prevent long-running stuck jobs
I also injected:
- Environment variables, so the same pipeline works in dev, QA, and production
- Kubernetes secrets, so credentials are handled securely and never hard-coded
This way, ML engineers could focus on modeling, while the pipeline handled reliability, security, and consistency.
 

Automation did not stop at training and deployment.   
I extended the pipeline into the post-deployment phase by adding:  
- Drift monitoring setup  
- Notification hooks  
For drift monitoring, we stored the training data as reference and compared it continuously with new incoming data. If data patterns changed too much, the system raised alerts. This helped us decide when to retrain the model, instead of waiting for performance issues to appear in production.  
The notification hooks ensured that:  
- The team always knows when the pipeline succeeds or fails  
- Issues are caught early without manual checking  
So the pipeline became not just an execution tool, but a continuous monitoring system.  

Finally, I automated the build and deployment of the pipeline itself.  
Using the Kubeflow compiler, I converted the Python-based DSL into a YAML workflow. This YAML was version-controlled in Git and deployed using our CI/CD pipeline into Kubeflow.  
So the complete automation flow became:  
**Pipeline code → YAML → CI/CD → Kubeflow → Kubernetes execution**  

Finally, I compiled the Python DSL into YAML and deployed it through CI/CD into Kubeflow, so the entire ML lifecycle became fully automated, reproducible, and production-ready.  






