# Kserve

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


### Endpoint			
Endpoint is the deployed model’s API address where applications send requests to get predictions.  
endpoint is a network-accessible URL (e.g., https://mycompany.com/predict)  

# KSERVE COMPONENTS
## 1. Deploy InferenceService CRD
	 kubectl apply -f ev-battery-inferenceservice.yaml  
This is Core Heart of KServe. Instead of managing pods manually, you just define a YAML file.  Main purpose is to deploy ml model in kubernetes cluster.  
Inside it, you mention:  
Model format (sklearn, pytorch, tf, xgboost etc.)  
Storage location  
Resources (CPU / RAM)  
Optional components  

why InferenceService is a Kubernetes Custom Resource (CRD)?  
Kubernetes by default understands only standard objects like: pods, services, deployments. But ML model deployment has special needs like model framework, storage, autoscale, Transformer…etc These are not supported in normal Kubernetes objects. So KServe extends Kubernetes by adding a new resource type called: InferenceService. This is done using CRD (Custom Resource Definition).   

CRD = Teaching Kubernetes a new “object type”.  
Just like Kubernetes knows pods, services, deployments. Now it also knows: InferenceService.  

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
This is where model actually runs.  In KServe, a Predictor is the component that actually runs your machine learning model and produces predictions.  
. loads the trained model  
. runs inference  
. returns prediction output  
> Every InferenceService must have a Predictor. Without it, there is no model serving.
> **Traffic flow for prediction:**  
Client → Istio Ingress Gateway → Knative → Transformer (preprocess) → Predictor → Transformer (postprocess) → Client

 **Inference code:**
 This is the actual code that runs when a prediction request comes in. In KServe, it typically involves three key methods in a custom model server:   
1. Model Loading (load)  
	. Load model file from storage into memory  
	. Load any dependencies like tokenizer, scaler, label encoder  
2. Preprocessing (preprocess)  
	. Transform raw input into model-ready format  
3. Prediction (predict)  
	. Run the actual model inference and get the output  
4. Postprocessing (postprocess)  
	. Transform model output into user-friendly response  
5. Server Startup (main)  
	. Initialize and start the KServe model server  

**Built-in Predictor**
. KServe knows how to load and run predict() automatically. But it does basic preprocessing only — like converting JSON to tensor format. It only does format    conversion. It assumes your input data is already clean and ready. It does NOT touch or transform the actual data values.  
. It does not handle custom preprocessing or postprocessing.  

**Custom Predictor**
. When KServe's built-in runtimes don't support your model or You have complex business logic , you write your own inference code and package it as a Docker container.  
. It handles dirty, raw, real-world data that needs cleaning and transformation before format conversion.  

**Step 1: Write Inference Code** 
**Step 2: Write Dockerfile**
**Step 3: Build and Push Docker Image**
**Step 4: Deploy Using InferenceService YAML**
**Step 5: Apply and Test**
> Deploy, Check status, Test prediction

## 4 Transformer
Transformer is an optional component inside an InferenceService. It sits between the client and the predictor to handle preprocessing and postprocessing.  
Preprocessing() and postprocess() run in transformer container only.  

**When to Use Transformer**  
Reason 1: Heavy Preprocessing  
- Imagine image preprocessing takes 10 seconds   
- Model prediction takes 2 seconds   
**Without Transformer:** One container handles both → If preprocessing is slow, everything is slow. You can't scale them separately. 
**With Transformer:**
Preprocessing container → Scale to 10 pods (because it's heavy)  
Prediction container    → Scale to 3 pods (because it's light)  
Each scales independently based on its own load.  

Reason 2: Reuse Same Preprocessing 
- One Transformer shared across Model A, B, C. Update once → all models get updated preprocessing.

Reason 3: Different Teams  
Team A manages Transformer container → preprocessing  
Team B manages Predictor container   → model  
No conflicts. Each team deploys independently.  

Reason 4: Update Without Redeploying Model  
- if you do Changes in  preprocessing → Redeploy only Transformer container not Model container.  Model container stays untouched (safe, no risk)  

> Pre-processing input: Cleaning raw data, Image resizing, Check missing values, Check data types, Reject bad requests early. This protects the model.  
> Post-processing Output(After prediction):  Map class IDs → human labels  
> Transformer = Separate container for preprocessing and postprocessing, keeping predictor focused only on prediction.

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

Explainer is separate endpoint (/explainer), because it is slow, heavy, and not needed for every request. Keeping it separate keeps predictions fast.  
**Request Flow (/explain):** Client → Istio → Knative → Explainer → Client  
```
Predictor:  Takes 100 milliseconds  (fast)
Explainer:  Takes 5-30 SECONDS      (very slow)

If combined:
Every prediction request waits 5-30 seconds
even when user just wants a quick answer ❌

If separate:
/predict → 100ms fast response ✅
/explain → 5-30 sec only when user ASKS for it ✅
```
> **Traffic flow for explanation (separate endpoint):**
Client → Istio Ingress Gateway → Knative → Explainer → Client
				
## 6 Knative Serving
Knative is a Kubernetes-based platform, KServe uses Knative for serverless model serving. It handles autoscaling, including the most important feature — **scale to zero.** that makes serverless model serving.  
```
Without Knative Serving:  
Normally in Kubernetes: Pods always run, Even if no user traffic it leads to Wastes CPU & memory  💸  
With Knative:  

With Knative Serving:  
You don’t need to keep pods running all the time 
No traffic → Pod shuts down (scale to zero) → Saves money 💰  
New request → Pod starts automatically → Serves prediction  
Heavy traffic → More pods created → Handles load  
```
**What Knative Serving Does for KServe (How it Works)**  
**1. Scale to Zero**
```
No requests for 5 minutes
     │
     ▼
Knative shuts down model pod
     │
     ▼
New request arrives
     │
     ▼
Knative starts pod again (cold start)
     │
     ▼
Prediction served
```
**2. Autoscaling (Scale Up/Down)**
```
10 requests/sec  → 1 pod running
100 requests/sec → Knative adds more pods
500 requests/sec → Knative scales to 10 pods
Back to 10 req/s → Knative scales down to 1 pod
0 requests       → Knative scales to 0 pods
```
**3. Traffic Routing**
```
Model v1 (90% traffic) ──┐
                          ├── Knative routes traffic
Model v2 (10% traffic) ──┘
```
**4. Revision Management**
```
Every time you update model → Knative creates a new revision
├── Revision 1: model-v1
├── Revision 2: model-v2
└── Revision 3: model-v3

You can route traffic to any revision
```
> **How KServe and Knative Work Together:** apply InferenceService YAML -> KServe Controller reads it -> KServe tells Knative: "Create a serverless service for this model" .  

**Cold Start Problem:**  The one downside of scale to zero:  
```
Pod is at zero
     │
     ▼
New request arrives
     │
     ▼
Knative starts pod (takes 5-30 seconds) ← COLD START
     │
     ▼
Prediction served (delayed)

After first request:
Next requests are instant (pod is warm now)
```
> **KServe Without Knative:** we can deploy with HPA but cannot achive scale to zero. atleast one pod runs continuosily. 

## 7 Istio ingress gateway
Istio Ingress Gateway is a secure traffic entry point(entry gate) that receives external requests and routes them to the correct service inside Kubernetes.  
**What It Does**
1. Single Entry Point
```
All requests enter through ONE door

Client A ──┐
Client B ──┼──→ Istio Ingress Gateway ──→ Routes to correct model
Client C ──┘
```
2. Traffic Routing — WHERE to send
Decides which model to send the request to based on URL/path. it Accepts External Traffic and Routes Traffic to Correct Service 
```
Client A: /v1/models/loan-model:predict    → Loan Model Pod
Client B: /v1/models/fraud-model:predict   → Fraud Model Pod
Client C: /v1/models/churn-model:predict   → Churn Model Pod
```
> **Problem it solves:** I have multiple different models, which one should handle this request?

3. Traffic Splitting — HOW MUCH to send
- Decides what percentage of traffic goes to different versions of the same model.
```
All requests hit: /v1/models/loan-model:predict

But Istio splits:
├── 90% requests → Loan Model v1 (old stable version)
└── 10% requests → Loan Model v2 (new version being tested)
```
> **Problem it solves:** I have two versions of the same model, how much traffic should each version get?

4. Load Balancing — WHO handles it
Decides which pod handles the request when multiple replicas of the same model version are running. Distributes requests fairly across pods
```
All requests hit: Loan Model v1 (3 replicas running)

Istio distributes:
├── Request 1 → Loan Model v1 - Pod 1
├── Request 2 → Loan Model v1 - Pod 2
└── Request 3 → Loan Model v1 - Pod 3
```
5. Security (TLS/HTTPS)
it provides mTLS (Mutual Transport Layer Security) encryption secure communication  

6.  Authentication & Authorization
```
Client sends request
     │
     ▼
Istio Ingress Gateway
├── Is this user allowed? (auth check)
├── Does this user have correct API key?
├── Yes → Forward to model
└── No → Reject request (401/403)
```

**HOW DO YOU DEPLOY THE MODEL?**
We deploy the model in a Kubernetes cluster using KServe. First, we create an InferenceService CRD YAML which defines the predictor, and optionally a transformer and explainer. Once we apply this YAML, the KServe Controller, which continuously watches for new InferenceService resources, detects it and starts validating — model framework, storage path, resource limits, whether transformer or explainer is present.
After validation, KServe asks Knative Serving to create serverless resources which handle autoscaling, scale to zero, and revision management. It also configures Istio Ingress Gateway for secure routing, TLS termination, and traffic management like canary rollouts.
Then pods start creating — the Predictor pod loads the model from storage (like S3), the Transformer pod (if configured) handles preprocessing and postprocessing, and the Explainer pod (if configured) handles model explainability on a separate endpoint.

**After InferenceService YAML Is Applied** 
```
Step 0: KServe
Step 1: KServe Controller
     │
     ├── Detects new InferenceService YAML
     ├── Validates configuration
     │   ├── Model framework correct?
     │   ├── Storage path valid?
     │   ├── Resource limits defined?
     │   ├── Transformer present?
     │   └── Explainer present?
     │
     ├── Creates pods
     │   ├── Predictor pod (loads model from storage)
     │   ├── Transformer pod (if configured)
     │   └── Explainer pod (if configured)
     │
     ├── Tells Knative → Handle autoscaling
     └── Tells Istio → Handle networking
         │
         ▼
Step 2: Knative Handles
     │
     ├── Autoscaling (scale up/down)
     ├── Scale to zero
     ├── Cold start (waking up from zero)
     └── Revision management (model versions)
         │
         ▼
Step 3: Istio Handles
     │
     ├── Creates endpoint URL
     ├── Secure routing (TLS/HTTPS)
     ├── Traffic routing (which model)
     ├── Traffic splitting (which version)
     ├── Load balancing (which pod)
     └── Authentication & Authorization
         │
         ▼
Model is READY to serve predictions
```


**Scripts**
1. Inference_code.py
2. InferenceService_Deployment.yaml




# ArgoCD
ArgoCD is a GitOps continuous delivery tool for Kubernetes. It automates the deployment of your YAML files to the Kubernetes cluster.  

**Without ArgoCD**
```
You manually deploy every time:

kubectl apply -f inferenceservice.yaml     ← You run this manually
kubectl apply -f inferencegraph.yaml       ← You run this manually
kubectl apply -f transformer.yaml          ← You run this manually

Every change → You run commands manually ❌
```
**With ArgoCD**
```
You push YAML to Git → ArgoCD automatically deploys to Kubernetes

git push (updated YAML) → ArgoCD detects → Deploys automatically ✅

No manual kubectl commands needed
```

**How It Works**
```
Step 1: You push YAML to GitHub
Step 2: ArgoCD watches GitHub repo
Step 3: ArgoCD detects change
Step 4: ArgoCD applies YAML to Kubernetes cluster
Step 5: KServe Controller detects new InferenceService
Step 6: KServe deploys the model (creates pods, knative, istio)
```
> ArgoCD automates YAML delivery from Git to Kubernetes. KServe takes over from there to deploy the model.

