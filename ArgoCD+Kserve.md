# Model Inference Platform(Kserve):

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
```
import joblib  
model = joblib.load(“traffic_sign_model.pkl") # load model  
def predict(input_data):  #This predict() is the inference function
    return model.predict(input_data)
print(predict([5.1, 3.5, 1.4, 0.2]))  # Output: Predicted class
```  
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
```
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
```
```
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
```  
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


