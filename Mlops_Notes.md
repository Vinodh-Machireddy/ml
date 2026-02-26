MACHINE LEARNING:
Machine Learning (ML) is a process / technique of teaching a computers using data, instead of writing rules manually like old programs. A computer learns patterns from past data and uses that learning to make predictions or decisions on new data. 


Old (Traditional) way
You write rules
Computer follows rules
Example:
IF marks > 35 THEN pass
Machine Learning way
You give data + answers
Computer learns rules by itself
Example:
Past student marks + pass/fail result
Model learns what marks usually mean pass or fail

Types of Machine Learning

1. Supervised Learning:
The model learns from labeled examples (you tell it the “right answer”). our feed in input data (features) and the correct output (label). The algorithm discovers the mapping from input → output.

if output column data is categorical(yes/no) data then we go for Classification algorithm family.
supervised classification algorithms examples:
1. Decision Tree
2. Random Forest
3. Logistic Regression

if output column/target data is numerical labels than we go for Regression Algorithm under supervised technics.
supervised regression algorithms examples: 
1. Simple Linear Regression
2. Ridge Regression
3. Lasso Regression
4. Elastic Net

fixed evaluate/performance metrics for regression model
mean_squard_error
mean_absolute_error
r2_score


2. UnSupervised Learning:
The model learns from unlabelled data (you don’t provide “right answers”). Only inputs are given; no labels. The algorithm searches for patterns, groupings, or structure in the data.

When to use: Clustering (e.g., grouping customers by behaviour, features)

eg:-
K‑Means Clustering
Hierarchical Clustering 
Principal Component Analysis (PCA)
DBSCAN


3. Reinforcement Learning
Learns by trial and error using rewards.
Example: games, robots
NOTE: Structured / Unstructured is about DATA FORMAT
	Supervised / Unsupervised / RL is about LEARNING METHOD

Structured data :
Data organised in a fixed format – rows and columns
Usually stored in:  Tables, CSV, Excel, Databases

Who mainly works with structured data? 
Data Engineers—>Data Scientist—>ML Engineer—>MLOps Engineer
Unstructured data : 
Data without fixed rows & columns
Text (emails, chat, PDFs), Images, Audio, Video, documents, Logs (semi-structured)
Who mainly works with unstructured data?
Data Engineer, NLP Engineer, CV Engineer, LLM Engineer, Prompt Engineer, GenAI / LLMOps Engineer

Algorithm
An algorithm is just a step-by-step set of instructions to solve a problem or perform a task. It’s like a recipe a computer follows to get the desired result.
Input → Algorithm → Output
An algorithm is the mathematical method used to learn patterns from data.

Model
The result (output) of that learning process.

Model artefacts/File Formats
A file that stores a trained machine learning model so it can be loaded and used later.
.pkl, .joblib,  .onnx,  .pt,  .h5,  .sav

DS creates Models
Creates Dataset(small/big)
Split (20 - 80 or 30 - 70) ration
Algorithm 
Train
Test
Retraining
Save (.pkl, joblib)
API


What is MLOPS
DevOps influenced MLOps
DevOps is for Traditional Application
Mlops is for machine learning model
Traditional software + ML model = Intelligent system
In DevOps we  focuses on automating the lifecycle of traditional software/Application(SDLC), were as in MLOps it automates the lifecycle of a machine learning model(MDLC).


Youtube, Netflix: Traditional software: OTT streaming platform—>DevOps
ML model used: Recommendation engine—>MLops

PayPal: Traditional software: Online payment system
ML model used: Fraud detection model
  
Amazon: Traditional software: E-commerce platform
ML model used: Product recommendation + pricing

Banking Apps: Loan approval, Credit risk models

Swiggy/Zomato: Traditional software: Food delivery ML models used: Restaurant recommendation, Delivery time prediction
Machine Learning Life Cycle:
Problem definition
Data collection
Data clean
Feature Engineering
Model selection (Algo)
Model Training
Model Evaluation(test)
Hyperparameter
Deploying
Monitoring maintenance
ML Workflow:
ML workflow represents the automated and operational execution of those stages in production.

Comprehensive/Standalone/End2End MLOps Platforms
Kubeflow, MLFlow, SageMaker, Azure ML, Vertex AI
Data → Training → Pipeline → Model Registry → Deployment → Monitoring (all inside one ecosystem)

DS vs ML vs MLops Responsibilities:
DS: Responsible for 1 to 7 steps of Machine Learning Life Cycle
ML Engineer: Responsible for Optimise the model for api, scalability, performance, latency, throughput, memory…etc (production ready state)
and also develops API’s to the models end of the models will shift as a software applications or packages to integrate with backend systems, So that end users access it through Netflix mobile app/website/other apps which Netflix supports.
MLOps: Responsible for
 building reproducible training pipelines/Continuous Training(CT) 
CI/CD
Observability (Monitoring)
Model registry
Iac
Cost Optimization 
Enables DS & ML to ship the model faster and safer.


How mlops engineers helps DS
							DVC
DVC stands for Data Version Control. DVC is a tool used to version large data files and ML models, similar to how Git versions code.

Git is capable of doing auditing, versioning, RBAC for code repo’s
Same capabilities is required for Data also, But git not designed for  large files(GB, TB)

Why not Git for Data versioning
Wouldn’t handle large files    -  DVC handles  large files(GB, TB)
Slow while push & pull.        -   Fast
Cost in-effective                    -   Cost Effective(Cheap)
Local storage is limited         -   S3, Azure Blob, GCS (also supports versioning & Durability(99.99999))      

 
If you have datasets and python code in same folder.
Dvc add dataset -> .dvc file will generate which has dataset tracking info eg: wine.csv.dvc
Dvc push -> dvc/config file will generate which has datasets version info in remote storage s3.
Git push .dvc and dvc/config -> git (Git stores only a pointer file → wine.cvs.dvc and .dvc/config folder)

Dataset is stored in s3 bucket and metadata of dataset is stored in git for DS and other teams.

Push to Remote storage service
Dvc add winne.csv
aws sts get-caller-identity
Aws configure
Add S3 Permissions: AmazonS3FullAccess.
dvc remote list 
	If you need to change it: dvc remote modify myremote url s3://your-correct-bucket-name/path
python3 -m pip install dvc-s3
dvc remote add -d mlops-remote s3://mlops-ss3
Dvc push

Push wine.scv.dvc metadata of dataset file and .dvc folder to GitHub
Git add wine.csv
Git remote add origin https://github.com/Vinodh-Machireddy/MLOps-Project.git
Git push -u origin master


Deployment Strategies used by mlops engineers to deploy the model

Virtual machines
Kubernetes
Cloud: aws sagemaker
Kserve


Kubernetes Deployment Strategy

0. We collect the model file train.py form DS and app.py file from ML engineers.
Preparing Dockerfile, model container image, running the container locally. (This is also called docker architecture)
Push image to Model registry 	eg:- DockerHub: it is a centralised repo where it stores app container images, model container images for versioning, auditing, RBAC.
Local Kubernetes cluster for demo, POC, Cost-effective. eg:- kind, Minikube, k3s ..etc 
	- brew install kind
	- kind version
	- kind create cluster --name=ml-demo-cluster
	- kubectl cluster-info  # Verify cluster is running
	- kubectl get nodes
	- kind delete cluster
    NOTE: Org level: EKS, AKS, GKS, OPENSHIFT, SELF HOSTED K8S CLUSTER.

EKS:
Prerequisites
	- aws configure
       	- AWS CLI
	- kubectl
	- eksctl
2. create cluster
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --version 1.32 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  —managed

eksctl get cluster  #verify
aws eks describe-cluster --name my-cluster --region us-east-1 #if not “ACTIVE" Configure kubeconfig
kubectl cluster-info #If cluster is reachable, it will show API server details.


   This command automatically creates:
VPC, Subnets
EKS Control Plane
Managed Node Group
IAM roles & required resources

3. Configure kubeconfig
aws eks update-kubeconfig --region us-east-1 --name my-cluster
Kubectl config current-context
kubectl get nodes
kubectl get pods -n kube-system
4. Delete the Cluster (Cleanup)
eksctl delete cluster --name my-cluster --region us-east-1

User —>ingress—>LB—>SVC—>pod


Model Inference Platform(Kserve):

Inference
Inference means using a trained model make predictions on new/ unseen data.
Training = learning patterns from data (adjusting weights).
Inference = using those learned patterns to predict outcomes.
1. Batch Inference (Predict on a dataset at once, offline jobs)

In batch mode, you take a large dataset (like CSV of 1 lakh records, or folder of 10,000 images).
Run the trained model on the entire dataset at once.
Save predictions to a file, database, or data warehouse.
Used when predictions are not time-sensitive,  Periodic jobs (daily, weekly).

2. Real-Time Inference (API-based)
In real-time mode, the model gives predictions immediately whenever a request comes.
You expose the model as an API endpoint (using FastAPI, Flask, or KServe).
Used when predictions are time-critical, Instant predictions (seconds).

Inference Code This is the core logic (usually Python code) that loads the trained model and runs predictions on new/unseen data.
import joblib
model = joblib.load(“traffic_sign_model.pkl") # load model
def predict(input_data):  #This predict() is the inference function
    return model.predict(input_data)
print(predict([5.1, 3.5, 1.4, 0.2]))  # Output: Predicted class

Inference Service
A specific microservice or API that runs your inference code.
Example: A FastAPI app exposing /predict or a KServe InferenceService resource.
Focus is only on serving one trained model (or one pipeline of models).

Inference Pipeline
This is when we chain multiple inference steps together to handle more complex tasks. Instead of just calling the model we call model pipeline.
Step 1: Preprocess → Resize image to 224x224 pixels, normalize colors.
Step 2: Run Model A → Object detection model finds region with traffic sign.
Step 3: Run Model B → Classification model predicts “Speed Limit 60”.
Step 4: Postprocess → Convert model output (class ID 12) → Human-readable label (“Speed Limit 60”).
Useful for large systems (like self-driving cars, fraud detection, healthcare pipelines).

Inference checks are small tests we run after deployment (or in staging) to make sure the model loads and predicts correctly on sample input before sending traffic.
			
Endpoint is the deployed model’s API address where applications send requests to get predictions.
endpoint is a network-accessible URL (e.g., https://mycompany.com/predict)

Model Serving System
Def: Model Serving Systems are platforms that take trained machine learning models and make them available for real-time or batch predictions in production, we can manage deployment, scalability, Multi-Model Management, versioning, monitoring, and security.

KServe (Kubeflow Serving) – Kubernetes-native, supports multi-model, autoscaling, and inference pipelines.
Seldon Core – Advanced Kubernetes serving, supports canary, A/B testing, explainability.
TensorFlow Serving – Optimized for TensorFlow models, production-grade serving.
TorchServe – Co-developed by AWS & PyTorch team, for PyTorch models.
Cloud-managed: AWS SageMaker Endpoints, Azure ML Endpoints, Google Vertex AI Prediction, Databricks Model Serving.

“FastAPI is a Python web framework for building APIs.  It takes input data through HTTP requests, validates it, passes it to the model, and returns predictions as JSON.

Why FastAPI 
It is fast and  Handles many requests with low latency. 
It Validates and  Stops bad data before reaching the model.
Auto docs → /docs Swagger UI makes testing easy.
It integrates & Works well with  Docker + Kubernetes.



KServe is an open-source model serving platform that runs on Kubernetes. It is used to automate deploy, serving, and inference.
It Support All Frameworks: Scikit-Learn, TensorFlow, PyTorch, XGBoost
- Scale-to-Zero is a specialised server-less capability that automatically scales your model's computational resources down to zero replicas when there is no incoming request traffic.


Why Kserve:
Because manually deploying ML models in VM’s and Kubernetes is difficult.
KServe makes it easy with one YAML file.

KServe Architecture:
KServe runs on top of Kubernetes and mainly has three layers:

Installation:
1. kind create cluster --name=kserve-demo-intent
2. kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
	- kubectl get pods -n cert-manager
3. Install KServe CRDs
kubectl create namespace kserve

helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.16.0 \
  -n kserve \
  --wait

4. Install KServe controller:
helm install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment \
  --wait

5.Create an InferenceService to Deploy the Intent Classifier mode:
kubectl create namespace intent

cat <<EOF | kubectl apply -n intent -f -
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: intent-classifier
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: https://github.com/Vinodh-Machireddy/MLOps-Project/releases/download/v1.1/intent_model.pkl
      resources:
        requests:
          cpu: "100m"
          memory: "512Mi"
        limits:
          cpu: "1"
          memory: "1Gi"
EOF

Verify: kubectl get inferenceservice intent-classifier -n intent
 kubectl get inferenceservices sklearn-iris -n kserve-test #inference status
 kubectl logs kserve-controller-manager-7f7b6d54df-fhskf -n kserve
 Kubectl get horizontalpodautoscalers.autoscaling -n intent
 Kubectl get svc -n intent

6. Determine the ingress IP and ports
 kubectl get svc istio-ingressgateway -n istio-system
			or
  Port-forward to access the model
 kubectl  port-forward svc/intent-classifier-predictor 8080:80 -n intent --address 0.0.0.0
7. Inference the Model
curl -s -X POST http://localhost:8080/v1/models/intent-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances":["I want to cancel my subscription"]}' | jq
KSERVE COMPONENTS
1. Deploy InferenceService CRD
	# kubectl apply -f ev-battery-inferenceservice.yaml
This is Core Heart of KServe. Instead of managing pods manually, you just define a YAML file.  Main purpose is to deploy ml model in kubernetes cluster.
nside it, you mention:
Model format (sklearn, pytorch, tf, xgboost etc.)
Storage location
Resources (CPU / RAM)
Optional components

why InferenceService is a Kubernetes Custom Resource (CRD)? 
Kubernetes by default understands only standard objects like: pods, services, deployments. But ML model deployment has special needs like model framework, storage, autoscale, Transformer…etc These are not supported in normal Kubernetes objects. So KServe extends Kubernetes by adding a new resource type called: InferenceService. This is done using CRD (Custom Resource Definition). 

CRD = Teaching Kubernetes a new “object type”. 
Just like Kubernetes knows pods, services, deployments. Now it also knows: InferenceService. 

Command: kubectl apply -f model.yaml

apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: ev-battery-fault-classifier
  namespace: ev-battery
  annotations:
    autoscaling.knative.dev/minScale: "1"
    autoscaling.knative.dev/maxScale: "6"
    autoscaling.knative.dev/target: “10"

spec:
  predictor:
    model:
      runtime: kserve-sklearnserver
      modelFormat:
        name: sklearn
      storageUri: s3://ev-ml-models/battery-fault/v1/
      resources:
        requests:
          cpu: "500m"
          memory: "1Gi"
        limits:
          cpu: "2"
          memory: “2Gi"

  transformer:
    containers:
      - name: battery-preprocess
        image: your-org/ev-battery-transformer:latest
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
          limits:
            cpu: "2"
            memory: "1Gi"

explainer:
    alibi:
      type: AnchorTabular
      storageUri: s3://ev-ml-models/battery-fault/explainer/
      resources:
        requests:
          cpu: "500m"
          memory: "1Gi"
        limits:
          cpu: "2"
          memory: "2Gi"

  traffic:
    - percent: 80
      revisionName: ev-battery-fault-classifier-predictor-default
    - percent: 20
      revisionName: ev-battery-fault-classifier-v2

2. Kserve Controller:
Runs inside Kubernetes.
Role:
Reads every InferenceService YAML you apply
Validates configuration
Creates model runtime pods
Connects with Knative and Istio
Watches model health, scaling, lifecycle
Basically controller = manager of KServe

3 Predictor
This is where model actually runs.
In KServe, a Predictor is the component that actually runs your machine learning model and produces predictions.
loads the trained model
runs inference
returns prediction output
Without Predictor → no model serving.

4 Transformer
In KServe, a Transformer is the component that prepares the input data before it reaches the model (and can also post-process output).
It sits before the Predictor and makes sure the model receives data in the exact format it expects.

Pre-processing input: Cleaning raw data, Image resizing, Check missing values, Check data types, Reject bad requests early. This protects the model.
Post-processing Output(After prediction):  Map class IDs → human labels

5 Explainer(Model Explainability)
An Explainer is a component that provides human-understandable reasons for a machine learning model’s prediction by identifying which input features influenced the output and how.
 - Explainer runs as a separate pod
Alibi framework
KServe does not build explainers from scratch. It integrates Alibi as the Explainer component.
KServe Explainer = Alibi running inside a pod

- Flow:  User —> Transformer—>Predictor(model)—>Alibi Explainer/Explainer → Prediction + Explanation
Alibi supports multiple explanation methods, including:
i.SHAP = SHapley Additive exPlanations
how much each feature contributed to the prediction.

Prediction: Battery Degradation Fault
SHAP explanation:
SOH < 65% → +40% impact
Cell temperature high → +30% impact
Voltage imbalance → +20% impact
Normal charge cycles → -10% impact
So engineer clearly knows root cause.

ii.LIME = Local Interpretable Model-agnostic Explanations.
LIME explains a prediction by creating a small, simple model around that one data point.
Local = explanation for ONE specific prediction
Global = explanation of how the model works overall

Prediction: Overheating Fault
LIME explanation:
Temperature spike → main reason
Fast charging → secondary reason
Easy and quick explanation.

Temperature = 72°C              Temp = 70, 71, 72, 73, 74
Voltage     = 3.9V		Voltage = 3.8, 3.9
SOC         = 40%		SOC = 38, 40, 42
ChargeRate  = Fast		ChargeRate = Normal, Fast 
				
6 Knative Serving
Knative is a Kubernetes-based platform that makes applications serverless.
Normally in Kubernetes: Pods always run, Even if no user traffic it leads to Wastes CPU & memory
With Knative: You don’t need to keep pods running all the time
Scale to zero: No traffic → pods scale to 0
Auto scaling: Traffic comes → pods start automatically  #You pay resources only when used
Traffic split: Knative can send 80% traffic to v1 and 20% to v2. 

7 Istio ingress gateway
Istio Ingress Gateway is a secure traffic entry point that receives external requests and routes them to the correct service inside Kubernetes. 
	- it Accepts External Traffic and Routes Traffic to Correct Service
	- load balancing: Distributes requests fairly across pods:
	- it provides mTLS (Mutual Transport Layer Security) encryption secure communication

HOW YOU DEPLOY THE MODEL?
We deploy the model in Kubernetes cluster using kserve. First, we create an InferenceService CRD. KServe controller continuously watches new InferenceService Once it detects, it starts validating model framework, transformer present?, explainer present?, resource limits, storage path.  KServe now asks Knative to create Serving resources like autoscaling, traffic split etc…
The request first enters through Istio Ingress Gateway which handles secure routing & traffic management. Pods starts creating Predictor pod loads model from S3, transformer preprocesses request, and explainer handles explainability. 

Traffic flows: Client → Istio → Knative → Transformer → Predictor → Explainer → Client.
Knative handles autoscaling and Istio handles secure routing & traffic management.


KUBEFLOW
What is Kubeflow?
Kubeflow is a Kubernetes-native platform designed to run the entire machine learning lifecycle—data-loading, training, evaluation, pipelines, deployment, monitoring, retraining.

Why Kubeflow?
Kubeflow is used to make Machine Learning production-ready, the same disciplined way traditional software is built and deployed using DevOps. 


Kubeflow Components:





DSL:
Kubeflow uses a Python-based DSL to declaratively define ML pipelines, which are later compiled into Kubernetes-executable YAML workflows.

Kubeflow Pipeline: ML workflow automation
It helps us automate the full ML lifecycle – from data ingestion to training, evaluation, deployment, and monitoring.
Technically:
We write pipelines using Python DSL
Kubeflow compiles it into YAML
Kubernetes executes it as a DAG of pods
Each step runs in its own container, but all steps are connected using inputs and outputs.

Why?
Automation – no manual steps
Reproducibility – same pipeline, same result



MLE → writes @dsl.component (logic, what runs inside pod)
What MLE owns:
Component logic (training, feature processing, validation)
Inputs & outputs of components
Python code that runs inside the container
Making components reusable & production-ready
NOTE: ML engineers create reusable Kubeflow components containing the ML logic, which are then shared with the MLOps engineer. MLE decides what happens inside the pod.

MLOps → writes @dsl.pipeline (orchestration, how it runs in cluster)
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

Resources, retry, timeout:
They decide how the training pod runs in Kubernetes. They do NOT change ML logic.
They control runtime behaviour of the pod.

 train_task.set_cpu_limit(“4")   This training pod can use maximum 4 CPU cores.
        train_task.set_memory_limit(“8Gi")  # This pod can use maximum 8 GB RAM.
        train_task.set_retry(3)  # If this step fails, Kubeflow will try again up to 3 times.
        train_task.set_timeout(7200) # If this step runs more than 7200 seconds (2 hours), Kubeflow will stop it automatically.

environment variables:
It adds an environment variable inside the training pod so your training code knows in which environment it is running.

dsl.EnvVar(name="ENVIRONMENT", value=environment) this creates 
		ENVIRONMENT=prod   or dev qa …etc
Runtime: When the training pod starts, inside the container you will have:
		echo $ENVIRONMENT # output: prod
Your Python code can read it like this:
		import os
		env = os.getenv("ENVIRONMENT")

/secrets:
train_task.add_secret(secret_name=“ev-telemetry-secret", mount_path="/secrets" )
 It securely gives sensitive information to the training pod.
When the training pod starts, Kubernetes mounts the secret at:  /secrets  Inside that folder you will see files like:
		/secrets/DB_PASSWORD
		/secrets/API_KEY
- Your training code can read them like:
   		with open("/secrets/DB_PASSWORD") as f:
   		 password = f.read()

Caching:
train_task.set_caching_options(True) # If you run the pipeline again It tells Kubeflow to reuse the result of this step if nothing has changed.
evaluation:
The training step created a model file, for example: 
/models/ev_fault_model.joblib
That file location is saved as an output of the training step. Now we pass that output to the evaluation step. So you are telling Kubeflow: “Use the same model that was just trained.”
This creates a dependency: Train step  →  Evaluate step. Evaluate will not start until training finishes. 

If evaluation says model is good → register it
accuracy = 0.92
metrics = "accuracy=0.92, recall=0.90”

Promotion Gate:
dsl.Condition(…) This creates a decision point in the pipeline. Run the next step only if this condition is true.” 
		eval_task.outputs["accuracy"] >= accuracy_threshold


This is just another Kubeflow component you create, like your other steps.
Inside that component, you write code that:
Uses KServe SDK or kubectl
Creates / updates an InferenceService
Points it to your model in S3 / GCS / MinIO


Drift monitoring:
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

NOTE: Each Kubeflow pipeline component runs in its own isolated pod, but components are linked through the pipeline using input–output dependencies. The pipeline builds a DAG, and Kubeflow executes the pods in that order, passing outputs from one component to the next.
How outputs are stored in Kubeflow
When a component finishes, its outputs are stored in two different ways:
Parameters → small values (strings, numbers, booleans)
accuracy = 0.92
status = "success"
model_version = "v3"

Artifacts → files / large data (Anything stored as a file: datasets, models, logs, reports)
/data/battery_raw_v1.csv
/models/ev_fault_model.joblib
/metrics/report.json

Interview Explanation:
I automated ML pipeline orchestration using Kubeflow Pipelines by separating ML logic and orchestration clearly.
ML engineers focused on built reusable components using @dsl.component, which runs inside the pod – like data ingestion, validation, feature engineering, model training, and evaluation.

I  integrated the orchestration layer using @dsl.pipeline that defines execution order, dependencies, promotion gates, and production controls like resources, retries, secrets, and environment variables.

I added drift monitoring and notification hooks so the pipeline continues into post-deployment phase.

One important part of the orchestration was adding promotion gates. Since we were working on a safety-critical system, I added a condition that checks model accuracy before allowing it to move to the registry.
So instead of trusting every trained model, the pipeline itself makes the decision:
Only models that meet quality standards are promoted.
This removed human error and brought strong governance into the ML lifecycle.

Next, I focused on making the pipeline production-ready.
I added full runtime controls at the orchestration level:
CPU and memory limits for heavy training jobs
Retry logic for temporary failures
Timeouts to prevent long-running stuck jobs
I also injected:
Environment variables, so the same pipeline works in dev, QA, and production
Kubernetes secrets, so credentials are handled securely and never hard-coded
This way, ML engineers could focus on modeling, while the pipeline handled reliability, security, and consistency.

Automation did not stop at training and deployment. 
I extended the pipeline into the post-deployment phase by adding:
Drift monitoring setup
Notification hooks
For drift monitoring, we stored the training data as reference and compared it continuously with new incoming data. If data patterns changed too much, the system raised alerts. This helped us decide when to retrain the model, instead of waiting for performance issues to appear in production.
The notification hooks ensured that:
The team always knows when the pipeline succeeds or fails
Issues are caught early without manual checking
So the pipeline became not just an execution tool, but a continuous monitoring system.

Finally, I automated the build and deployment of the pipeline itself.
Using the Kubeflow compiler, I converted the Python-based DSL into a YAML workflow. This YAML was version-controlled in Git and deployed using our CI/CD pipeline into Kubeflow.
So the complete automation flow became:
Pipeline code → YAML → CI/CD → Kubeflow → Kubernetes execution

Finally, I compiled the Python DSL into YAML and deployed it through CI/CD into Kubeflow, so the entire ML lifecycle became fully automated, reproducible, and production-ready.

DATASETS
========
Kaggle Datasets:  https://www.kaggle.com/datasets
GitHub Repositories
Plotly : opensource datasets to train ml models
uci machine learning repository:  https://archive.ics.uci.edu


built‑in datasets in python for pratice:
1. scikit‑learn (sklearn.datasets)
2. seaborn (seaborn.load_dataset())
3. statsmodels (statsmodels.datasets)


						MODEL-HUB

models that were already trained on large datasets by big companies or research labs, and made available for others to reuse or fine-tune instead of training from scratch.

Hugging Face, TensorFlow Hub, PyTorch Hub
AWS SageMaker JumpStart → ready-to-use pre-trained models for NLP, vision, tabular
Azure ML Model Registry → curated pre-trained models
Google Cloud Vertex AI Model Garden → Google & open-source models



API for model serving/expose
=============================
models cannot directly interact with users, models shift through software applications which exposes the enduser. these are python frameworks.
FastAPI (Python)
Flask
KServe (Kubernetes-native)
Django
...etc


# data and Feature pipelines 


Row : --->	Sample / Instance / Observation / Record
Column : ---->	Feature / Attribute / Variable
output column : -----> is a special type of column:  target column/target variable/label/answer




ML Model:
--------- 
It’s the finished product you get after running that recipe on your data.
he will deploy the ML algorithms i.e ML Models into production.


data
1. primary and
2. secondary data
train and test data
input and output columns/data


CPU: Central processing unit
GPU: Graphical processing unit
TUP: Tensor processing unit
QPU: Quantum Processor unit
DBU: Data Bricks Units

connection to cloud applications
--------------------------------
1) google colab
2) vscode
3) pycharm
4) local anaconda jupyter notebook
5) databricks
6) GCP Benchmark
...etc


workflow/process for training and evaluating any ML model
=========================================================
Process:
• We need to divide data into two parts train_data and test_data
• Both train_data and test_data includes input columns(X) and output columns(y)
• Next we divide train_data to X_train and y_train
• Next we divide test_data to X_test and y_test
• Model will be developed based on train data (X_train and y_train)
• Model predictions performed on X_test data the result we called as y_predictions
• Finally we compare y_test with y_predictions


	ML pipeline


Deployment Pipeline (CD Pipeline)

1.Trigger → model approved in registry.
2.Package model (Docker/container).
3.Provision infra (Kubernetes, SageMaker, Vertex AI, etc.).
4.Rollout strategy (Canary, Blue-Green, A/B, etc.).
5.Run Smoke Tests → API ping, sample inference, latency check (200 OK response). 
If smoke tests pass → continue rollout (canary, blue-green, etc.).
If smoke tests fail → rollback immediately.

Deployment Strategies:

Shadow Deployment/Dark Launch:
- The new model runs in parallel with the current production model.
- It gets the same live input data, but its predictions are not shown to users (only logged).
Users still see results from the old stable model(Champion Model).
Both outputs are stored in logs/metrics DB.
Later, you compare shadow vs champion
Why : To test new model performance, predictions with old model  on real production data without risk.
Blue-Green deployment
Blue-Green deployment is a safe release strategy where you keep two separate but identical environments:
Blue = current production (old stable model)
Green = new version (new model)
At any time, only one (Blue or Green) serves user traffic.
To switch, you just change the traffic routing from Blue → Green.
Why: To deploy new models safely without disturbing running users. To allow instant rollback if something goes wrong.

Canary Deployment
Canary Deployment means releasing a new model slowly to a small part of traffic/users first.
Start with maybe 5–10% of requests going to the new model.
Rest 90–95% still go to the old stable model.
If new model works fine gradually increase traffic until 100%.
If new model performs badly → rollback immediately to the old model.
Why: To reduce risk when deploying new ML models. To check performance on real production users without impacting everyone.

A/B Deployment
A/B Deployment (also called A/B Testing or Champion–Challenger) means you run two models in production at the same time.
Group A (users) see predictions from the old model (Champion).
Group B (users) see predictions from the new model (Challenger).
Why: To safely test new models on a subset of users before rolling out fully.


File formats for saving machine learning models
1.  .pkl (Python Pickle)
Primary Use: Saving models from libraries like Scikit-learn, XGBoost, and LightGBM. It's great for traditional machine learning models.
2.  .h5 (Hierarchical Data Format)
Primary Use: Saving models from deep learning frameworks, specifically for Keras and TensorFlow.
3.  .pt / .pth (PyTorch)
Primary Use: Saving and loading models in PyTorch.
4.  .onnx (Open Neural Network Exchange)
Primary Use: Deploying models in production, especially when the training framework is different from the inference environment.




Fine-Tuning:
=========
Transfer Learning Technics
Fine-Tuning?
Fine-tuning means taking a pre-trained model (like ResNet, MobileNet, BERT, etc.) that already learned general patterns (edges, shapes, language, etc.)
You remove/freeze the last layer and replace it with a new layer for traffic sign classes.
Then you fine-tune on your dataset (Indian traffic sign images).

Why Fine-Tuning?
Saves time → You don’t train from scratch.
Saves compute cost → Less GPU needed.
Works with smaller datasets.
Gives better accuracy because the model already knows basic patterns.

Types of Fine-Tuning
Fixed Feature Extraction → Freeze the pre-trained backbone, train only the  head i.e last layer(Classification Layer).
                 Small dataset + similar task → Fixed Feature Extractor.
You stop updating their weights during training.
Full Fine-Tuning → Unfreeze more layers, retrain them on your dataset.
                 Bigger dataset + different task → Full Fine-Tuning.
Many teams start with fixed extractor, and if results are not good, they move to progressive/full fine-tuning.
Steps:
- Load a pre-trained backbone
Example: ResNet50/MobileNet pre-trained on ImageNet.
- Replace the last layer (classifier head)
Old head = 1000 classes → New head = your classes (e.g., 43).
- Freeze backbone layers: Set requires_grad = False (PyTorch) or mark layers non— trainable (Keras).
- Train only the head
Evaluate
Register & package
Deploy & monitor

When would it be “Training from Scratch”?
If you design your own CNN architecture and train only on your traffic sign dataset (with random weights, no pre-training).
This requires lakhs of images and huge compute (GPUs).
Usually, only big research labs or companies attempt this.

What is Classification Layer.
At the end of every model, there is a final layer that gives predictions. Its job is to take the features extracted by the backbone and map them into predicted classes.
Example:
In ResNet50 trained on ImageNet → last layer has 1000 outputs (for 1000 categories).
In your Traffic Sign project → you replace that last layer with 43 outputs (because you have 43 traffic sign classes).
This last part is called the classification layer (Head or fully connected layer / dense layer).

Command to load pre-trained model from hugging face:
pip install transformers datasets
from transformers import AutoModelForSequenceClassification, AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)

Notes:
In a neural network, each connection between neurons has a number attached → that number is called a weight.
A Traffic Sign Recognition project most commonly falls under the category of deep learning, specifically using a Convolutional Neural Network (CNN).







# Strange
What are Guardrails?
STRANGE
—————
endpoint deployment
endpoint prediction
online and batch endpoints
AutoML
hyperparameters
pyspark:
- PySpark is primarily used for handling and processing large‑scale data in a distributed fashion.
parallel processing and fast.

- Machine Learning Libraries: Strong experience with machine learning libraries and frameworks such as scikit-learn, TensorFlow, PyTorch, Keras, etc.
      - For deep learning → TensorFlow, PyTorch, Keras.
      -	For traditional ML → Scikit-learn.
      - For boosting models → XGBoost, LightGBM, CatBoost.
      - For NLP → Hugging Face Transformers.
- Experience with RestAPI Frameworks like FastAPIs, Flask

