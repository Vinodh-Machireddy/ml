# INTERVIEW TARGET
1. Ml Pipelines : Kubeflow Pipelines
2. Experiment Tracking + Model Registry: Mlflow 
3. Deployment and serving : ArgoCD + Kserve
4. Monitoring and alerting : Prometheus & Grafana
5. Ci/cd Pipelines : GitHub Actions
6. Cloud : AWS S3, ECR, EKS, IAM, SageMaker, Cost optimization, Security (AWS Secrets Manager)

# Production Ready Tech Stack: 
```Kubeflow Pipelines(KFP), Mlflow, ArgoCD, Kserve, Prometheus & Grafana, GitHub Actions, AWS, Git, GitHub, DVC, Docker, Kubernetes, Python, Linux.```

<div align="center"> 
 
# MONITORING (PROMETHEUS & GRAFANA) 

</div>

<img width="490" height="320" alt="image" src="https://github.com/user-attachments/assets/5e67dc93-67bb-42c7-b713-91598ca68f1e" />

## Prometheus Components

### Prometheus Server:  
It collects and stores metrics data.
- Retrieval: Pulls metrics from different targets (like servers or apps).
- TSDB (Time-Series Database): Stores the metrics in a format that makes it easy to query over time. It stores key and value pair format. Eg: 12:00:45 = HB 69
- HTTP Server: Allows you to run queries and interact with Prometheus through a web interface.  

### Prometheus Targets: (pull metrics)
These are the systems (like applications, pods, nodes, databases, or servers) that expose metrics. They often use exporters to provide the metrics in a format Prometheus understands.  

### Pushgateway: (push metrics)
 Normally Prometheus pulls metrics from targets.  But in some cases Applications push metrics to Pushgateway → Prometheus then scrapes Push-gateway.

Used for short-lived jobs like scripts, Run for very short time (batch jobs / cron jobs) which Do not expose /metrics endpoint continuously,  these jobs "push" their metrics to the Push-gateway which stores temporarily and Prometheus scraps it from Push-gateway.
Example:
Data pipeline job
ML training batch job
Nightly backup job

### Service Discovery  
Service Discovery means automatic target detection.  
It automatically finds the targets (applications, pods, nodes, services) that Prometheus needs to monitor.  
It eliminates the need to hard-code IP addresses and ports inside the prometheus.yml file.  

#### Why Service Discovery is Important in Kubernetes  
In Kubernetes:  
- Pods are created and deleted dynamically.
- IP addresses change frequently.
- Auto-scaling happens automatically.
- Deployments roll out new versions continuously.

Because of this dynamic nature, we cannot hardcode targets. Prometheus uses Kubernetes Service Discovery to handle this automatically.
Prometheus talks to the Kubernetes API to discover targets/services like Pods, Nodes, Endpoints, Ingress, drift_job, node-exporter, kube-state-metrics(k8s Objects). Prometheus service discovery = “Find all possible targets”

#### Types: 
- Kubernetes SD: ```kubernetes_sd_configs```  
- Static Service Discovery: ```statis_sd_configs```
- EC2 service discovery: ```ec2_sd_configs```
- DNS-based discovery: ```dns_sd_configs```
- File-Based Service Discovery: ```file_sd_configs```

#### How Service Discovery Works
Prometheus uses configured service discovery mechanisms to:
 - Detect running services (targets).
 - Fetch endpoint details (IP address and port).
 - Dynamically update the scrape target list.
 - Automatically start or stop scraping based on target availability.  
This ensures monitoring stays accurate even when infrastructure changes.

### Exporters:
- collect info from nodes and keep in API Endpoints (/metrics) and from there prometheus scrapes(it follows pull mechanism) the metrics and store in TSDB.  
For Kubernetes cluster monitoring: 1. Node Exporter/plugin/add-on 2. Kube State Metrics (KSM) 3. cAdvisor  4. API Server Metrics 5. kubelet metrics 6. etcd metrics 
For Web Servers: nginx_exporter, apache_exporter   
For Database: postgres_exporter, mongodb_exporter, redis_exporter ...etc  

#### 1. Node Exporter: (system-level metrics from the OS)
Node Exporter collects hardware and operating system metrics from a server (node) and exposes them for Prometheus to scrape.  
It Collects: CPU usage, Memory usage, Disk space & disk I/O, Network traffic, File system usage, Load average, System uptime

#### 2. Kube State Metrics (Kubernetes object-level metrics to Prometheus.)  
It tells the current state of Kubernetes objects, not resource usage.  
It collects:Pods (Running, Pending, Failed), Deployments (Desired vs Available replicas), ReplicaSets, StatefulSets, DaemonSets, Jobs & CronJobs, HPA (min/max replicas, current replicas), Nodes (Ready/NotReady status), PersistentVolumeClaims.  

#### 3. cAdvisor (container-level resource usage metrics)  
cAdvisor monitors how much CPU, memory, and disk each container is using.

### Alertmanager:
Sends alerts when something is wrong. For example, if your server’s CPU is too high, Prometheus triggers an alert. Alerts can be sent to email, Slack, PagerDuty, or other notification systems.

### Visualization Tools:
Prometheus has its own web interface to show metrics. It also integrates with tools like Grafana to create beautiful dashboards for better analysis.

### How Prometheus Works:
Prometheus finds targets using Service Discovery or a configuration file.  
It pulls metrics from these targets at regular intervals.  
The metrics are stored in the Time-Series Database (TSDB).  
You can query these metrics using PromQL, a query language.  
If something goes wrong, Alertmanager sends out notifications.  
You can view metrics and create dashboards using the Prometheus Web UI or Grafana.  


## Monitoring starts when ALL 3 are present:  
KServe internally uses Knative and Istio. Prometheus detects it → starts scraping metrics → monitoring starts. we no need to manually start monitoring.  
1. Prometheus (kube-prometheus-stack) is installed and running
2. ServiceMonitor created
3. Application exposes /metrics endpoint
4. Model pod running

## How to confirm monitoring is started (100% proof)
Go to Prometheus UI:  http://localhost:9090/targets
If you see:
```model-service      UP
mlflow-service     UP
kubeflow-service   UP
```
## Interview Questions
How do you design ML monitoring architecture?  
How do you monitor model performance in production?  
How do you implement alerting strategy?  
How do you monitor KServe serving metrics?  
Differentiating "System" vs. "Model" Metrics  

## ML monitoring architectures
1️⃣ Prometheus-Based Native Kubernetes Monitoring Architecture (Best for production-ready systems, This is the most traditional and stable approach.)  
```Tools Used: Prometheus, Grafana, KServe, kube-state-metrics, node-exporter, Alertmanager```

2️⃣ Observability Stack with Logging + Tracing + Metrics (Full Observability Architecture, This goes beyond metrics.)  
```Tools Used: Prometheus, Grafana, Loki, Jaeger, KServe```
```
Metrics → Prometheus
Logs → Loki
Traces → Jaeger
        ↓
Grafana (Unified View)
```
3️⃣ ML-Specific Monitoring Architecture (Drift + Quality Focused, This focuses more on model behaviour.)
```Tools Used: Prometheus, Evidently, Grafana, KServe```

# 1. Prometheus-Based Native Kubernetes Monitoring Architecture  
## 1. Infrastructure Layer  
Monitor cluster stability.  
Infrastructure = node level   

Metrics:  
-  Node CPU  
-  Node Memory
-  Disk usage
-  Network traffic
-  Node availability  

Collected by: node-exporter, kube-state-metrics  

### Step 1: Install Monitoring Stack 
we use kube-prometheus-stack it is a pre-packaged Helm chart that automatically installs a complete Kubernetes monitoring setup like Prometheus, Alertmanager, Grafana, Creates ServiceMonitor, NodeExporter, kube-state-metrics. It provides ready-made dashboards, alert rules, and Kubernetes service discovery.  

### Step 2 : Verify Node-Exporter is Running & CPU Metrics in Prometheus
kubectl get pods -n monitoring
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090  and http://localhost:9090
 
### Step 3 : Create CPU Usage Query (PromQL)
```100 - (avg by(instance)(
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100)
``` 
### Step 4 : Add CPU Panel in Grafana
  1. Open Grafana
  2. Create Dashboard
  3. Add Panel
  4. Select Prometheus as data source
  5. Paste above quer
  6. Set Unit → Percent (0–100)  
     Now you will see live node CPU usage.
     
### Step 5 : Create Alert rule and apply for High CPU
Create file  node-cpu-alert.yaml
```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: node-cpu-alert
  namespace: monitoring
spec:
  groups:
  - name: node.rules
    rules:
    - alert: HighNodeCPUUsage
      expr: 100 - (avg by(instance)(
              rate(node_cpu_seconds_total{mode="idle"}[5m])
            ) * 100) > 85
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage on node"
        description: "Node CPU usage is above 85% for 5 minutes"
```
```Apply: kubectl apply -f node-cpu-alert.yaml```

### Step 6 : Configure Alertmanager  
Alertmanager already installed via kube-prometheus-stack. configure Slack or Email receiver. When CPU > 85% for 5 mins → alert triggers.  

#### 1. Verify Alertmanager is Running  
   i/p: kubectl get pods -n monitoring
   o/p: alertmanager-monitoring-kube-prometheus-alertmanager-0   Running  (If running → ready to configure.)  
 
#### 2. Create Slack Webhook  
   Go to Slack → Settings  
   Create Incoming Webhook  
   Select channel  
   Copy webhook URL
   ```Example:  https://hooks.slack.com/services/T000/B000/XXXX```

#### 3. Create Alertmanager Config File
   Create file: alertmanager-config.yaml
```global:
  resolve_timeout: 5m

route:
  receiver: "slack-email"
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

receivers:
- name: "slack-email"

  slack_configs:
  - api_url: "https://hooks.slack.com/services/T000/B000/XXXX"
    channel: "#alerts"
    send_resolved: true
    title: "{{ .CommonAnnotations.summary }}"
    text: "{{ .CommonAnnotations.description }}"

  email_configs:
  - to: "your-email@gmail.com"
    from: "your-email@gmail.com"
    smarthost: "smtp.gmail.com:587"
    auth_username: "your-email@gmail.com"
    auth_password: "your-app-password"
    require_tls: true
    send_resolved: true
```
#### 4. Create Kubernetes Secret   
   - Alertmanager config must be stored as secret.
     ```
     kubectl create secret generic alertmanager-monitoring-kube-prometheus-alertmanager \
     --from-file=alertmanager.yaml=alertmanager-config.yaml \
     -n monitoring \
     --dry-run=client -o yaml | kubectl apply -f -
     ```
#### 5. Restart Alertmanager
   ```kubectl delete pod alertmanager-monitoring-kube-prometheus-alertmanager-0 -n monitoring```  

#### 6. Verify Configuration
   ```
   Port forward:  kubectl port-forward svc/monitoring-kube-prometheus-alertmanager -n monitoring 9093
   open:  http://localhost:9093
   ```
   Check:  
   Status → Config  
   Make sure Slack and email receivers are visible.  
   
#### 7. Test Alert  
   You can temporarily create a test alert:
```
- alert: TestAlert
  expr: vector(1)
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "Test Alert"
    description: "This is a test alert"
```  
  After 1 minute → Slack & Email should receive notification.  

### Step 7 : When Does Alert Trigger?
   ```
   If:
   Node CPU > 85%
   AND
   Condition lasts for 5 minutes
   Then alert becomes ACTIVE.
   You can check in: Prometheus → Alerts tab  or   Grafana → Alert panel
   ```
NOTE: Standard Process for Any Infra Metric,  For CPU, Memory, Disk, Node availability, Pod restarts — workflow is always same  

## 2. Platform Layer (Monitor Workloads)
This layer is above infrastructure.  
Infrastructure = node level  
Platform layer = workload level (pods, deployments, HPA)  
this layer ensures your ML workloads are stable.  

Metrics:  
-  Pod status
-  Restart count
-  Deployment health
-  HPA scaling
-  Autoscaling behaviour
-  Resource limits 

Alerts:
Pod CrashLoopBackOff
HPA max replicas reached
Pod not ready > 5 mins
This ensures deployment reliability.  

Tools Used: ```Prometheus, Grafana, kube-state-metrics(kube-state-metrics gives Kubernetes object metrics)```  

### 1. Pod Availability
Check Running pods: ```kube_pod_status_phase{phase="Running"}```  
If pod not running → issue.  

Alert: Pod Not Running
```
- alert: PodNotRunning
  expr: kube_pod_status_phase{phase!="Running"} > 0
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Pod not running"
```
### 2. Pod Restart Monitoring (Important for ML workloads.)
PromQL: ```increase(kube_pod_container_status_restarts_total[5m]) > 3```  
If restart count > 3 in 5 minutes → alert. Frequent restarts = memory leak or crash.  

Alert YAML:  
```
- alert: FrequentPodRestarts
  expr: increase(kube_pod_container_status_restarts_total[5m]) > 3
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Pod restarting frequently"
    description: "Container restarted more than 3 times in 5 minutes"
```
### 3. Deployment Availability
Check if available replicas < desired replicas:  
PromQL:  ```kube_deployment_status_replicas_available < kube_deployment_spec_replicas```  
Alert if mismatch > 5 minutes. This ensures rolling deployments succeed.  

Alert YAML:  
```
- alert: DeploymentReplicasMismatch
  expr: kube_deployment_status_replicas_available
        <
        kube_deployment_spec_replicas
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Deployment replicas mismatch"
    description: "Available replicas are less than desired replicas"
```
### 4. Horizontal Pod Autoscaler (HPA)  
Very important for ML serving.  
If current replicas = max replicas continuously → scaling issue.  
PromQL:  ```kube_hpa_status_current_replicas == kube_hpa_spec_max_replicas```  
If true for 10 minutes → cluster under-provisioned.  

Alert YAML:  
```
- alert: HPAMaxedOut
  expr: kube_hpa_status_current_replicas
        ==
        kube_hpa_spec_max_replicas
  for: 10m
  labels:
    severity: warning
  annotations:
    summary: "HPA reached maximum replicas"
    description: "Autoscaler is at max replicas for more than 10 minutes"
```  
Why 10m?  
Because short spikes are normal.  

### 5. Resource Requests vs Limits
Check if pods close to limit:  
CPU: ```container_cpu_usage_seconds_total / kube_pod_container_resource_limits_cpu_cores```  
Memory: ```container_memory_usage_bytes / kube_pod_container_resource_limits_memory_bytes```  
If usage > 90% of limit → alert.  
Prevents OOMKilled issues.  

### Platform Alert Strategy
Critical:  
- Deployment unavailable
- Pod crash loop
- HPA maxed out

Warning:  
- Restart count high
- Replica mismatch
- Resource usage near limit  
```Always use for: duration```

## 3. Model Serving Layer (KServe)
most important layer for MLOps. This is where real business impact happens. If this layer fails → users are affected immediately.

Core metrics:
revision_request_count
revision_request_latencies
Response codes
Autoscaling events  

Important:
Use p95 and p99 latency, not average.  

Example logic:
If p95 latency > 500ms for 5 mins → Warning
If error rate > 5% → Critical  

### revision_request_count
Shows incoming traffic rate.  
PromQL: ```sum(rate(revision_request_count[5m]))```  
If traffic suddenly drops to 0 for 5 mins → possible outage.

### revision_request_latencies
PromQL: ```histogram_quantile(0.95, sum(rate(revision_request_latencies_bucket[5m])) by (le))```  
This gives 95th percentile latency.

Alert:  
If p95 > 500ms for 5 minutes → Warning  
If p95 > 1s → Critical  
This ensures SLA protection.  

### Error Rate Monitoring
Error rate query:```sum(rate(revision_request_count{response_code!~"2.."}[5m]))
                  /
                  sum(rate(revision_request_count[5m]))
                 ```  
If error rate > 5% → Critical alert.

## 4. ML-Specific Layer
This layer monitors model behavior, not infrastructure or pods.  

Infra tells you system health.  
Platform tells you workload health.  
Serving tells you request health.  
ML layer tells you model quality health.  

Monitor:
- Feature distribution
- Prediction distribution
- Drift detection
- Model Confidence score
- Accuracy drop/degradation  

It is a shared responsibility. But implementation ownership is usually MLOps.  
 Model quality in production = MLOps + Data Science collaboration.  

Data Scientist:  
Defines evaluation metrics  
Provides baseline thresholds  

MLOps Engineer:  
Implements logging  
Builds dashboards  
Creates alert rules  
Monitors degradation  

predictor.py  
Model loading  
Model logic (predict)  
Monitoring logic (metrics)  
Response formatting  

### 1. Prediction distribution
Prediction distribution represents how model outputs are distributed across classes or value ranges in production over a given time window.   
Example (Classification):    
If a fraud model predicts :  Fraud,  Not Fraud  
And in last 10 minutes:  
Fraud → 15%  
Not Fraud → 85%  
That percentage split is the prediction distribution.  

Example (Regression):  
If a price model predicts house prices:  
Average predicted price = ₹12L  
70% predictions between ₹10L–₹15L  
That spread of predicted values is the prediction distribution.  

Prediction Distribution Count = How many times model predicted a specific output.   

#### Step-1: Add Counter in Serving File
```
from prometheus_client import Counter

prediction_counter = Counter(
    "model_predictions_total",
    "Total predictions by class",
    ["class"]
)
```
This creates a Prometheus metric with label class.  
Metric will look like:
```
model_predictions_total{class="approved"}  1523
model_predictions_total{class="rejected"}  412
```  
##### Step-2: Update Metric Inside predict()
```
def predict(self, request: dict):

    data = request["instances"]

    predictions = self.model.predict(data)

    for pred in predictions:
        prediction_counter.labels(class=str(pred)).inc()

    return {"predictions": predictions.tolist()}
```  
Now every prediction increases counter.  
#### Step-3: Convert to Percentage Using PromQL
We do NOT alert on raw counter like 1523  or 412  
We use rate().  
PromQL: 
```
sum(rate(model_predictions_total{class="approved"}[5m]))
/
sum(rate(model_predictions_total[5m]))
```  
This gives percentage of "approved" predictions in last 5 minutes.  
#### Step-4: Create Grafana Panel  
Add two panels:  
Approved %  
Rejected %  

##### Step-5: Create Alert rule for Sudden Shift
Example:  
Training baseline:  
Approved = 80%  
Alert if:  
Approved < 50% for 10 minutes  
Alert YAML:  
```prediction-shift-alert.yaml```  
```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: prediction-distribution-alert
  namespace: monitoring
spec:
  groups:
  - name: ml.rules
    rules:
    - alert: PredictionDistributionShift
      expr: (
              sum(rate(model_predictions_total{class="approved"}[5m]))
              /
              sum(rate(model_predictions_total[5m]))
            ) < 0.5
      for: 10m
      labels:
        severity: warning
        service: ml-model
      annotations:
        summary: "Prediction distribution shifted"
        description: "Approved class ratio dropped below 50% for 10 minutes"
```
Apply it: ```kubectl apply -f prediction-shift-alert.yaml```  Now Prometheus can fire alert.  

#### Step-6: Configure Alertmanager routing
Create:  ```alertmanager-config.yaml```  
```
global:
  resolve_timeout: 5m

route:
  receiver: "ml-alerts"
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
  - match:
      severity: warning
    receiver: "ml-alerts"

receivers:
- name: "ml-alerts"

  slack_configs:
  - api_url: "https://hooks.slack.com/services/T000/B000/XXXX"
    channel: "#ml-alerts"
    send_resolved: true
    title: "{{ .CommonAnnotations.summary }}"
    text: "{{ .CommonAnnotations.description }}"

  email_configs:
  - to: "ml-team@company.com"
    from: "alerts@company.com"
    smarthost: "smtp.gmail.com:587"
    auth_username: "alerts@company.com"
    auth_password: "APP_PASSWORD"
    require_tls: true
    send_resolved: true
```  
#### Step-7: Create Kubernetes Secret  
Alertmanager config must be stored as secret:  
```
kubectl create secret generic alertmanager-monitoring-kube-prometheus-alertmanager \
  --from-file=alertmanager.yaml=alertmanager-config.yaml \
  -n monitoring \
  --dry-run=client -o yaml | kubectl apply -f -
```
#### Step-8: Restart Alertmanager  
It will restart automatically. ```kubectl delete pod alertmanager-monitoring-kube-prometheus-alertmanager-0 -n monitoring```   

#### Step-9: Verify  
```
   kubectl port-forward svc/monitoring-kube-prometheus-alertmanager -n monitoring 9093
   http://localhost:9093  
```  

### 2. Drift Detection. 
a change/shift in data patterns of production data and training data.   
1️⃣ Data Drift (Input changes).  
2️⃣ Concept Drift (Relationship changes).  
3️⃣ Prediction Drift (Output distribution changes).  
Do NOT compute drift inside predict(). Drift must run as batch/Cron job.   
#### Store Production inference data
the input data coming from users for prediction. we must compare: Training data vs Production data, So we store production inputs in:  
Database (Postgres, MySQL).  
Object storage (S3).  
Data warehouse.  
Feature store ...etc    
#### Create drift_job.py
Drift detection is:  
- Heavy statistical computation.  
- Needs historical data.  
- Not real-time per request. 
So we create a separate Python script whose only job is:  
Read production data (from DB/S3).  
Load baseline training stats.  
Compute drift score.  
Export drift score as Prometheus metric.   
MLOps implements that logic in drift_job.py  
Data Scientist tells:```“Use PSI threshold 0.3”```
It contains:  
1. Data loading logic.  
2. Drift calculation. 
3. Prometheus metric export. 
Example simplified structure:  
```
# drift_job.py

from prometheus_client import Gauge
import pandas as pd

drift_score = Gauge("model_drift_score", "Drift score")

def compute_drift():

    # 1. Load baseline stats
    baseline_mean = 35

    # 2. Load production data
    production_data = pd.read_csv("prod_data.csv")

    # 3. Compute drift
    production_mean = production_data["age"].mean()

    score = abs(production_mean - baseline_mean) / baseline_mean

    drift_score.set(score)

if __name__ == "__main__":
    compute_drift()
```  
It is just a separate batch script.  
#### Deploy as Kubernetes CronJob
Why CronJob?  
Because we want drift to run:  
Every 5 minutes  
Every 1 hour  
Every day.  So we schedule it.  
CronJob simply means:```“Kubernetes, please run this container periodically.”```  
MLOps creates this YAML.  
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: drift-job
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: drift-container
            image: your-drift-image
            ports:
            - containerPort: 8001
          restartPolicy: OnFailure
```
#### Create Service for Drift Job  
Prometheus scrapes via Service.  
drift-service.yaml  
```
apiVersion: v1
kind: Service
metadata:
  name: drift-service
  namespace: mlops
  labels:
    app: drift-job
spec:
  selector:
    app: drift-job
  ports:
  - name: metrics
    port: 8001
    targetPort: 8001
```
Apply it.  

#### Create ServiceMonitor (MLOps Engineer creates this)
drift-servicemonitor.yaml  
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: drift-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: drift-job
  namespaceSelector:
    matchNames:
      - mlops
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```
Apply it.  
Now check Prometheus → Targets → drift-service should be UP.  

#### Create Prometheus Alert Rule
We alert if:  
Drift score > 0.3 for 15 minutes.  

Prometheus service discovery = “Find all possible targets to monitor”  
ServiceMonitor tells Prometheus: Out of everything you discovered, scrape only the ones matching these labels  

If you manually configure Prometheus like this:  
```
scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
```  
Then Prometheus:  
Talks to Kubernetes API  
Discovers ALL pods  
Scrapes ALL pods (if not filtered)  
This is broad and risky.  

When you use kube-prometheus-stack, it automatically installs Prometheus Operator inside your Kubernetes cluster. Prometheus is managed by the Prometheus  Operator. Prometheus Operator changes the way Prometheus config works. Instead of manually writing: ```kubernetes_sd_configs``` file   

Prometheus Operator is a Kubernetes controller that:  
Watches custom resources (CRDs)  
Automatically configures Prometheus  
Manages lifecycle (create/update/delete)  
It removes the need to manually edit prometheus.yml.  

How It Actually Works (Step-by-Step)  
Step 1 You create a ServiceMonitor for an application.  
Step 2 Prometheus Operator detects this ServiceMonitor (it watches CRDs).  
Step 3 Operator automatically updates Prometheus configuration.  
Step 4 Prometheus starts scraping that target.  
No manual restart needed.  

Prometheus works in pull model. But Prometheus does NOT automatically know which service to scrape. So we tell Prometheus: Please scrape this Kubernetes Service every 30 seconds. That instruction is called: ServiceMonitor   
ServiceMonitor is a Kubernetes object (CRD) created by kube-prometheus-stack.  
It tells Prometheus:  
Which namespace  
Which service (via labels)  
Which port  
Which path (/metrics)  
How often to scrape  

ServiceMonitor does NOT talk to Pod directly. It talks to Service.  

NOTE:  
Why Drift Is Separate From predictor.py?  
Because:  
If 1000 users request per second,  
you cannot calculate statistical tests inside predict().  
It will:  
Increase latency. 
Increase CPU. 
Break SLA. 
So we separate serving path and monitoring path. 

#### Data Drift (Input Changes/Shifts)
The statistical distribution of input features changes compared to training data.  
Example:  
Model trained on:  
Age mean = 35  
Income mean = 50,000  

Production data:   
Age mean = 60  
Income mean = 1,20,000  
Input changed → Model may behave differently.  

What Changes?: Input features only.  
Model logic same.  
Relationship same.  

How We Monitor:  
We compare:  
Training feature distribution  
VS  
Production feature distribution  

Methods:  
Mean / Std comparison  
Histogram comparison  
PSI (Population Stability Index)  
KS test  
KL divergence  

Usually implemented in: drift_job.py (batch job)  


#### Concept Drift (Relationship Changes)  
The relationship between input and output changes.  
Example:  
Earlier:  
Age 30 → High loan approval chance  

Now:  
Age 30 → Low loan approval chance  

What Changes?: Input may look normal. But prediction correctness drops.   

How We Monitor:  
Concept drift cannot be detected from input alone.  

We need:  
Ground truth feedback.  
So we monitor:  
- Accuracy
- F1-score
- RMSE
- AUC
- Precision/Recall

 #### 3. Prediction Drift (Output Distribution Changes)   
Model output distribution changes.  
Example:  
Earlier:  
Approved = 80%  

Now:  
Approved = 20%  
Output distribution changed.  

##### What Changes?: Model output behavior. Even if inputs similar.  
next steps same as infra/platform layers.  PromQL: ```rate(model_predictions_total[5m])``` Compare class ratio to baseline.  

##### Key Differences:  
Data Drift → Feature shift  
Prediction Drift → Output shift  
Concept Drift → Model performance drop  / relationship changes  

##### Confusion matrix:
A confusion matrix is a table used to check how well a classification model is performing.
[Confusion Matrix]<img width="550" height="450" alt="image" src="https://github.com/user-attachments/assets/152b42ef-0df5-44a4-9c1f-1c04c70d3614" />

1️⃣ True Positive (TP)  
Model predicted Positive, and it is actually Positive.  
✔ Correct detection.  
2️⃣ True Negative (TN)  
Model predicted Negative, and it is actually Negative.  
✔ Correct rejection.  
3️⃣ False Positive (FP)  
Model predicted Positive, but actually Negative.  
❌ False alarm.  
4️⃣ False Negative (FN)  
Model predicted Negative, but actually Positive.  
❌ Missed detection.  
Example: Model says battery healthy, but actually faulty.  
⚠ Very dangerous in healthcare, automotive, cybersecurity.  

###### Accuracy:  
Out of all predictions, how many are correct?  
Accuracy=(TP+TN)/Total  

Number of positives ≈ Number of negatives  
Example:  
50 faulty  
50 healthy  
Then accuracy gives good idea.  

If:  
False positive = small problem  
False negative = small problem  
Then accuracy is fine.  

When NOT To Use Accuracy:   
Case: Imbalanced Data  
Suppose:  
1000 batteries  
990 healthy  
10 faulty  
Model predicts: Always "Healthy"  

Then:  
Correct = 990  
Wrong = 10  
Accuracy = 990/1000 = 99%    
Looks great 😲  
But model never detected any fault.  
Recall = 0%  
This is dangerous.  

In production: We rarely monitor only accuracy.  
We monitor: Precision, Recall, F1, False Negative Rate, Drift. Because business impact matters more than overall percentage.  

###### Precision:
Precision=TP/(TP+FP)  
Out of predicted positives, how many are correct?  
Used when false alarms are costly.  

When to Choose: model has predicted mail to be spam but actual value is not spam. If this is important for you then choose Precision and reduce FP to ZERO. It Depends on suituation. In some cases like Fraud Detection, Cybersecurity ....etc.

Medical Diagnosis: Real Examples  
If model says patient has disease but actually not:  
Unnecessary stress  
Expensive tests  
Precision important.  

###### Recall (Sensitivity)  
Recall=TP/(TP+FN)  
Out of actual positives, how many detected?  
Used when missing a fault is dangerous.  

###### 


