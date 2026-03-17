# Kserve (Model Inference Platform)

Core Components:  

InferenceService  
InferenceGraph  
Predictor    
Transformer  
Explainer  

Infrastructure Components:  

KServe Controller  
Ingress Gateway (Istio / Kourier)  
Knative Serving  
ModelMesh  

Storage Components:  
Storage Initializer  

Serving Runtimes:  
ClusterServingRuntime  
ServingRuntime  

Monitoring & Logging:  
Logger  
Batcher  

Networking:  
Istio / Kourier  
Cert Manager  

### Model Serving 
- It takes input data through HTTP requests, validates it, passes it to the model, and returns predictions as JSON.

**KServe**
KServe is an open-source model serving platform that runs on Kubernetes. It is used to automate deploy, serving, and inference.  
It Support All Frameworks: Scikit-Learn, TensorFlow, PyTorch, XGBoost  
- Scale-to-Zero is a specialised server-less capability that automatically scales your model's computational resources down to zero replicas when there is no incoming request traffic.

**Why Kserve:**  
- Because manually deploying ML models in VM’s and Kubernetes is difficult. KServe makes it easy with one YAML file.  

**KServe Architecture:**   
- KServe runs on top of Kubernetes and mainly has three layers  

**Kserve  VS  FastAPI** 
```  
- Built-in (scale to zero) Autoscaling         ->      You manage it yourself
- Automatic Health Checks(readiness/liveness)  ->      You write them manually
- Built-in Canary/A-B Testing  				   ->      You build it yourself
- Built-in Model Versioning                    ->      You handle it yourself
- Protocol Standard v1/v2 inference protocol   ->      You define your own endpoints
- Deployment Kubernetes-native                 ->      Run anywhere (Docker, VM, local)
```  

## Inference
Inference means using a trained model make predictions on new/ unseen data.  
- Training = learning patterns from data (adjusting weights).  
- Inference = using those learned patterns to predict outcomes.
```
KServe Inference
├── InferenceService (single model)
│   ├── Real-time inference
│   └── Batch inference
│
└── InferenceGraph (pipeline / multi-model)
    ├── Real-time inference
    └── Batch inference
```  
### 1. Batch Inference (Predict on a dataset at once, offline)  
In batch mode, you take a large dataset (like CSV of 1 lakh records, or folder of 10,000 images).  
Run the trained model on the entire dataset at once.  
Save predictions to a file, database, or data warehouse.  
Used when predictions are not time-sensitive,  Periodic jobs (daily, weekly).  

### 2. Real-Time Inference (API-based)  
In real-time mode, the model gives predictions immediately whenever a request comes.  
You expose the model as an API endpoint (using FastAPI, Flask, or KServe).  
Used when predictions are time-critical, Instant predictions (seconds).  

### InferenceService
**As Inference Type:**  
InferenceService serves a single model behind one endpoint. It follows a simple request-response pattern where a client sends input, the model processes it through preprocess → predict → postprocess, and returns the prediction. It supports both real-time and batch inference modes.  
**As Deployment:**
InferenceService is a Kubernetes Custom Resource (CRD). You write a YAML file specifying your model location, serving runtime, and resources. When you apply this YAML, KServe automatically deploys your model on the Kubernetes cluster, exposes an endpoint, handles autoscaling, health checks, version management, and traffic routing.  

> **NOTE** InferenceService YAML → Does 2 Jobs   
	. Deploys the model on Kubernetes   
	. Serves predictions through an endpoint   

**InferenceService vs InferenceGraph**
``` 
Single model    			->     Multiple models
Simple request 				->	   responseComplex workflow/pipeline  
Straightforward prediction  ->     Chaining, routing, ensemble  
No concept of nodes    		->     Has Sequence, Parallel, Switch, Splitter  
SimpleMore                  ->     complex
```

### InferenceGraph (Inference Pipeline)
**As Inference Type:** 
InferenceGraph serves multiple models connected together as a pipeline. It supports four routing patterns — Sequence, Ensemble, Switch, and Splitter. Each step in the graph can be an InferenceService. It is used when a single model cannot handle the complete task and you need chaining, parallel execution, or conditional routing.  
**As Deployment:**    
InferenceGraph is also a Kubernetes CRD. You write a YAML file defining the pipeline structure, nodes, and routing logic. However, each individual model must be deployed as an InferenceService first. Then the InferenceGraph connects them together.  

**4 Types of Nodes in InferenceGraph** 
1. Sequence  
	- Models run one after another  
	- Output of model A goes into model B  
``` Input → Model A → Model B → Model C → Output ```  
2.  Parallel (Ensemble)
	- Models run at the same time  
	- All results are combined  
``` ┌→ Model A ─┐
Input ───┼→ Model B ──┼→ Combined Output
         └→ Model C ─┘
```
3. Switch (Router)
Based on some condition, traffic goes to different models
```
┌→ Condition met     → Model A
Input ───┤
         └→ Condition not met → Model B
```
4. Splitter
Splits traffic by percentage
```
┌→ 80% traffic → Model A
Input ───┤
         └→ 20% traffic → Model B
```
**InferenceGraph-inference**
```
apiVersion: serving.kserve.io/v1alpha1
kind: InferenceGraph
metadata:
  name: my-pipeline
spec:
  nodes:
    root:
      routerType: Sequence
      steps:
        - serviceName: preprocessing-service
        - serviceName: prediction-service
        - serviceName: postprocessing-service
```
This creates a simple pipeline:```Input → Preprocessing → Prediction → Postprocessing → Output```  

> **You just define the pipeline structure — which models, what order, what routing type.**  

> **NOTE** InferenceGraph YAML → Does Only 1 Job
	. Connects already deployed InferenceServices together as a pipeline
	. Deploys models → NO, it does NOT deploy any model

### Inference checks:
Inference checks are small tests we run after deployment (or in staging) to make sure the model loads and predicts correctly on sample input before sending traffic.  
**Health Checks (Probes)**
Readiness Probe — Is the model loaded and ready to accept requests?  
Liveness Probe — Is the serving container still alive and healthy?  

**Input/Output Validation**
Checking if the incoming request has the correct data format, shape, and type  
Validating that model output is as expected  

### Inference code:
This is the actual code that runs when a prediction request comes in. In KServe, it typically involves three key methods in a custom model server:   
- preprocess() — Transform raw input into the format the model expects  
- predict() — Run the actual model inference and get the output  
- postprocess() — Transform model output into a user-friendly response   

### Endpoint			
Endpoint is the deployed model’s API address where applications send requests to get predictions.  
endpoint is a network-accessible URL (e.g., https://mycompany.com/predict)  

### Install and Verify
Verify: kubectl get inferenceservice intent-classifier -n intent
 kubectl get inferenceservices sklearn-iris -n kserve-test #inference status
 kubectl logs kserve-controller-manager-7f7b6d54df-fhskf -n kserve
 Kubectl get horizontalpodautoscalers.autoscaling -n intent
 Kubectl get svc -n intent

# KSERVE COMPONENTS
## 1. Deploy InferenceService CRD
	 kubectl apply -f ev-battery-inferenceservice.yaml  
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
```
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
```  
## 2. Kserve Controller
Runs inside Kubernetes.  
Role:  
Reads every InferenceService YAML you apply  
Validates configuration  
Creates model runtime pods  
Connects with Knative and Istio  
Watches model health, scaling, lifecycle  
Basically controller = manager of KServe  

## 3 Predictor
This is where model actually runs.  
In KServe, a Predictor is the component that actually runs your machine learning model and produces predictions.  
loads the trained model  
runs inference  
returns prediction output  
Without Predictor → no model serving.  

## 4 Transformer
In KServe, a Transformer is the component that prepares the input data before it reaches the model (and can also post-process output).  
It sits before the Predictor and makes sure the model receives data in the exact format it expects.  

Pre-processing input: Cleaning raw data, Image resizing, Check missing values, Check data types, Reject bad requests early. This protects the model.  
Post-processing Output(After prediction):  Map class IDs → human labels  

## 5 Explainer(Model Explainability)
An Explainer is a component that provides human-understandable reasons for a machine learning model’s prediction by identifying which input features influenced the output and how.  
 - Explainer runs as a separate pod  
Alibi framework  
KServe does not build explainers from scratch. It integrates Alibi as the Explainer component.  
KServe Explainer = Alibi running inside a pod  

- Flow:  User —> Transformer—>Predictor(model)—>Alibi Explainer/Explainer → Prediction + Explanation  
Alibi supports multiple explanation methods, including:  
### i.SHAP = SHapley Additive exPlanations  
how much each feature contributed to the prediction.  

Prediction: Battery Degradation Fault  
SHAP explanation:  
SOH < 65% → +40% impact  
Cell temperature high → +30% impact  
Voltage imbalance → +20% impact  
Normal charge cycles → -10% impact  
So engineer clearly knows root cause.  

### ii.LIME = Local Interpretable Model-agnostic Explanations.
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
				
## 6 Knative Serving
Knative is a Kubernetes-based platform that makes applications serverless.  
Normally in Kubernetes: Pods always run, Even if no user traffic it leads to Wastes CPU & memory  
With Knative: You don’t need to keep pods running all the time  
Scale to zero: No traffic → pods scale to 0  
Auto scaling: Traffic comes → pods start automatically  #You pay resources only when used  
Traffic split: Knative can send 80% traffic to v1 and 20% to v2.   

## 7 Istio ingress gateway
Istio Ingress Gateway is a secure traffic entry point that receives external requests and routes them to the correct service inside Kubernetes.  
	- it Accepts External Traffic and Routes Traffic to Correct Service  
	- load balancing: Distributes requests fairly across pods:  
	- it provides mTLS (Mutual Transport Layer Security) encryption secure communication  

## HOW YOU DEPLOY THE MODEL?
We deploy the model in Kubernetes cluster using kserve. First, we create an InferenceService CRD. KServe controller continuously watches new InferenceService Once it detects, it starts validating model framework, transformer present?, explainer present?, resource limits, storage path.  KServe now asks Knative to create Serving resources like autoscaling, traffic split etc…  
The request first enters through Istio Ingress Gateway which handles secure routing & traffic management. Pods starts creating Predictor pod loads model from S3, transformer preprocesses request, and explainer handles explainability.   

Traffic flows: Client → Istio → Knative → Transformer → Predictor → Explainer → Client.  
Knative handles autoscaling and Istio handles secure routing & traffic management.  


