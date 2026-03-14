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

### 1. First, I convert MLE's code into KFP components.  
Once we receive raw code(Data Ingestion, Validation, Feature Eng, Training, Evaluation)  from MLE's. we convert it into KFP Components (@dsl.components). where we change:   
```  
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
```

> **NOTE:** INFRASTRUCTURE CHANGES (MLOps core work): 1 to 10 except 6th change. ||  CODE QUALITY IMPROVEMENTS (MLOps suggests to MLE or adds directly): 6,11,12,13.

**Final Converted Code of train.py**  
```  
# components/training/train_fault_classifier.py — MLOps creates this NEW file (Change 8)

from kfp import dsl                                                    # Change 1
from kfp.dsl import Input, Output, Dataset, Model, Metrics             # Change 2, 3, 4

@dsl.component(                                                        # Change 1
    base_image="python:3.9-slim",                                      # Change 1
    packages_to_install=["scikit-learn==1.3.0", "pandas==2.0.3",       # Change 10
                         "joblib==1.3.2"]
)
def train_fault_classifier(
    features_data: Input[Dataset],                                     # Change 2
    model_artifact: Output[Model],                                     # Change 3
    training_metrics: Output[Metrics],                                 # Change 4
    n_estimators: int = 200,                                           # Change 6
    max_depth: int = 15,                                               # Change 6
    test_size: float = 0.2                                             # Change 6
):
    import pandas as pd                                                # Change 7
    from sklearn.ensemble import RandomForestClassifier                # Change 7
    from sklearn.model_selection import train_test_split               # Change 7
    from sklearn.metrics import accuracy_score, f1_score               # Change 7
    import joblib                                                      # Change 7

    df = pd.read_csv(features_data.path)                               # Change 2

    X = df.drop(columns=["fault_type", "battery_id", "timestamp"])
    y = df["fault_type"]

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=test_size, random_state=42, stratify=y         # Change 6, 11, 13
    )

    model = RandomForestClassifier(
        n_estimators=n_estimators,                                     # Change 6
        max_depth=max_depth,                                           # Change 6
        random_state=42,                                               # Change 13
        n_jobs=-1                                                      # Change 12
    )
    model.fit(X_train, y_train)

    y_pred = model.predict(X_test)
    accuracy = accuracy_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred, average="weighted")

    training_metrics.log_metric("accuracy", float(accuracy))           # Change 4
    training_metrics.log_metric("f1_score", float(f1))                 # Change 4
    training_metrics.log_metric("n_estimators", n_estimators)          # Change 4
    training_metrics.log_metric("max_depth", max_depth)                # Change 4
    training_metrics.log_metric("training_samples", len(X_train))      # Change 4
    training_metrics.log_metric("test_samples", len(X_test))           # Change 4

    joblib.dump(model, model_artifact.path)                            # Change 3

    model_artifact.metadata["framework"] = "scikit-learn"              # Change 5
    model_artifact.metadata["model_type"] = "RandomForestClassifier"   # Change 5
    model_artifact.metadata["accuracy"] = float(accuracy)              # Change 5
    model_artifact.metadata["f1_score"] = float(f1)                    # Change 5
    model_artifact.metadata["training_samples"] = len(X_train)         # Change 5  
```
> NOTE: Same for remaining all components.

---


## 2. @dsl.pipeline (orchestration, how it runs in cluster) 
```  
@dsl.pipeline(name="ev-battery-fault-pipeline")
def ev_battery_fault_pipeline(
    data_version: str = "v1",
    accuracy_threshold: float = 0.90,
    environment: str = "prod"
):
    with dsl.ExitHandler(notify_pipeline_status(status="completed")):

        # ──────────────────────────────────────────
        # Step 1: Ingest — LIGHT component
        # ──────────────────────────────────────────
        ingest = ingest_battery_data(data_version=data_version)
        ingest.set_cpu_limit("1")              # just downloading file, 1 CPU enough
        ingest.set_memory_limit("2Gi")         # small memory
        ingest.set_retry(3)                    # S3 network can fail, retry 3 times
        ingest.set_timeout(1800)               # 30 min max
        ingest.set_caching_options(True)       # same version = skip download
        ingest.add_secret(secret_name="aws-credentials", mount_path="/secrets")  # needs S3 access
        ingest.set_display_name("Ingest Battery Telemetry")

        # ──────────────────────────────────────────
        # Step 2: Validate — LIGHT component
        # ──────────────────────────────────────────
        validate = validate_battery_data(
            raw_data=ingest.outputs["raw_data"]
        )
        validate.set_cpu_limit("2")            # some pandas processing
        validate.set_memory_limit("4Gi")       # data fits in 4GB
        validate.set_retry(1)                  # data validation rarely fails randomly
        validate.set_timeout(1800)             # 30 min max
        validate.set_caching_options(True)
        validate.set_display_name("Validate Sensor Data")
        # ❌ No secrets needed — just reading data from previous step
        # ❌ No GPU needed — just pandas operations

        # ──────────────────────────────────────────
        # Step 3: Feature Engineering — MEDIUM component
        # ──────────────────────────────────────────
        features = generate_battery_features(
            clean_data=validate.outputs["clean_data"]
        )
        features.set_cpu_limit("2")            # rolling window calculations
        features.set_memory_limit("4Gi")
        features.set_retry(1)
        features.set_timeout(3600)             # 1 hour max
        features.set_caching_options(True)
        features.set_display_name("Generate Battery Features")
        # ❌ No secrets needed
        # ❌ No GPU needed

        # ──────────────────────────────────────────
        # Step 4: Training — HEAVIEST component
        # ──────────────────────────────────────────
        train = train_fault_classifier(
            features_data=features.outputs["features_data"],
            n_estimators=200,
            max_depth=15
        )
        train.set_cpu_limit("4")               # heavy computation
        train.set_memory_limit("8Gi")          # large dataset in memory
        train.set_gpu_limit("1")               # if deep learning model
        train.set_retry(3)                     # OOM/network failures possible
        train.set_timeout(7200)                # 2 hours max — training takes time
        train.set_caching_options(True)
        train.add_secret(secret_name="aws-credentials", mount_path="/secrets")
        train.add_env_variable(dsl.EnvVar(name="ENVIRONMENT", value=environment))
        train.add_node_selector("node_type", "gpu-node")   # run on GPU machine
        train.add_toleration(key="nvidia.com/gpu", operator="Exists", effect="NoSchedule")
        train.set_display_name("Train EV Battery Fault Classifier")

        # ──────────────────────────────────────────
        # Step 5: Evaluate — MEDIUM component
        # ──────────────────────────────────────────
        evaluate = evaluate_fault_model(
            model_artifact=train.outputs["model_artifact"],
            features_data=features.outputs["features_data"]
        )
        evaluate.set_cpu_limit("2")            # prediction + metric calculation
        evaluate.set_memory_limit("4Gi")
        evaluate.set_retry(1)
        evaluate.set_timeout(1800)             # 30 min max
        evaluate.set_caching_options(False)    # ❌ always evaluate fresh, never cache
        evaluate.set_display_name("Evaluate Fault Model")

        # ──────────────────────────────────────────
        # Step 6: Register — LIGHT component
        # ──────────────────────────────────────────
        with dsl.Condition(evaluate.output >= accuracy_threshold):
            register = register_fault_model(
                model_artifact=train.outputs["model_artifact"],
                eval_metrics=evaluate.outputs["eval_metrics"]
            )
            register.set_cpu_limit("1")            # just API calls to MLflow
            register.set_memory_limit("2Gi")
            register.set_retry(3)                  # MLflow server might be busy
            register.set_timeout(600)              # 10 min max — just registration
            register.set_caching_options(False)    # ❌ never cache registration
            register.add_secret(secret_name="mlflow-credentials", mount_path="/secrets")
            register.add_env_variable(dsl.EnvVar(name="MLFLOW_TRACKING_URI", value="http://mlflow-server:5000"))
            register.set_display_name("Register Model in MLflow")  
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

## 3. Compilation
After writing the pipeline in Python, you compile it into a YAML file. This YAML is what Kubernetes actually understands and executes.  

## 4. KFP Client (Submitting the Pipeline) 


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






