
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
- For Kubernetes cluster monitoring: 1. Node Exporter/plugin/add-on 2. Kube State Metrics (KSM) 3. cAdvisor  4. API Server Metrics 5. kubelet metrics 6. etcd metrics  
- For Web Servers: nginx_exporter, apache_exporter   
- For Database: postgres_exporter, mongodb_exporter, redis_exporter ...etc  

#### 1. Node Exporter: (system-level metrics from the OS)
What it is: A lightweight agent that runs on every node in the cluster and exposes hardware and OS-level metrics of that node.  
How it runs: As a DaemonSet — meaning Kubernetes automatically places one node-exporter pod on every node. If you have 5 nodes, you get 5 node-exporter pods.  
```
Node 1 ──→ node-exporter pod ──→ exposes :9100/metrics
Node 2 ──→ node-exporter pod ──→ exposes :9100/metrics
Node 3 ──→ node-exporter pod ──→ exposes :9100/metrics
                                        │
                                        ▼
                                   Prometheus scrapes all
```

**What metrics it exposes:**  
```
**K8s Object**        **Example Metrics**             **What it tells you** 
CPU      -  node_cpu_seconds_total          -  How much CPU time is used vs idle
Memory   -  node_memory_MemAvailable_bytes  - How much RAM is free on the node
Disk     -  node_filesystem_avail_bytes     - How much disk space is left
```  

#### 2. Kube State Metrics (Kubernetes object-level metrics to Prometheus.)  
What it is: A single service that talks to the Kubernetes API server and converts Kubernetes object states into Prometheus metrics.  
**What metrics it exposes:**  
```
**K8s Object**              **Example Metrics**                         **What it tells you**  
Pod             kube_pod_status_phase{phase="Running"}         Is the pod running, pending, or failed?
Pod             kube_pod_container_status_restarts_total       How many times has this container restarted? 
Deployment      kube_deployment_status_replicas_available      How many replicas are actually available? 
Deployment      kube_deployment_spec_replicas                  How many replicas were requested? 
Node            kube_node_status_condition{condition="Ready"}  Is the node in Ready state?
```


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
>  is most important layer for MLOps. This is where real business impact happens. If this layer fails → users are affected immediately.

  When your EV Battery model is deployed an InferenceService on KServe and serving predictions, you want to answer these questions:  
```
1. How many prediction requests are coming?
2. How fast is the model responding?
3. Are any requests failing?
4. Is the model overloaded?
```
To answer these, you need numbers (metrics). Someone has to count and measure every request. That "someone" is the queue-proxy sidecar. When you deploy an InferenceService on KServe, KNative automatically adds a **queue-proxy sidecar** container alongside your model container in every pod. 
```
┌─────────────── KServe Pod ───────────────┐
│                                           │
│  ┌─────────────┐    ┌──────────────────┐  │
│  │  Your Model  │    │   Queue-Proxy    │  │
│  │  Container   │◄──►│   (Sidecar)      │  │
│  │              │    │                  │  │
│  │  - Predictor │    │  - Receives all  │  │
│  │  - or        │    │    requests first│  │
│  │  - Transformer    │  - Measures      │  │
│  │              │    │    latency       │  │
│  │              │    │  - Counts        │  │
│  │              │    │    requests      │  │
│  │              │    │  - Exposes :9091 │  │
│  └─────────────┘    └──────────────────┘  │
│                                           │
└───────────────────────────────────────────┘
                    │
                    ▼
            Prometheus scrapes :9091  
```
**What Does Queue-Proxy Do?**
Every prediction request goes through queue-proxy FIRST, then reaches your model. Queue-proxy sits in the middle and records everything:
```
Client sends request
        │
        ▼
  Queue-Proxy receives it
        │
        ├── Records: "Request received at 10:00:01.000"
        ├── Records: "Request count + 1"
        │
        ▼
  Forwards to Your Model Container
        │
        ▼
  Model returns prediction
        │
        ▼
  Queue-Proxy receives response
        │
        ├── Records: "Response sent at 10:00:01.045"
        ├── Calculates: "Latency = 45ms"
        ├── Records: "Response code = 200 (success)"
        │
        ▼
  Client gets response
```
**Where Does Queue-Proxy Store These Numbers?**
Queue-proxy does NOT send these numbers anywhere on its own. It just holds them in memory and exposes them on a URL: ``` http://pod-ip:9091/metrics```  
**Who Reads These Numbers? PROMETHEUS**
Prometheus is a tool that visits the queue-proxy URL every 15 seconds and copies the numbers into its own database.
```
Every 15 seconds:
Prometheus ──HTTP GET──→ http://kserve-pod:9091/metrics
                              │
                              ▼
                    Queue-Proxy responds with:
                    "request_count = 4523"
                    "latency_bucket_50ms = 3800"
                    "error_count = 12"
                              │
                              ▼
                    Prometheus saves these numbers
                    with timestamp: 10:00:15
```
> This "visiting and copying" is called scraping. Prometheus scrapes metrics from queue-proxy every 15 seconds.

**This is where Prometheus Operator and ServiceMonitor come in.** 
**Problem:** Your cluster might have 50 pods. Prometheus can't visit all of them. It needs to know "which pods have metrics worth scraping?"  
**Solution:** You create a ServiceMonitor — a YAML file that tells Prometheus "scrape pods with THIS label":  ServiceMonitor tells Prometheus where to look.  
```
# servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: kserve-monitor
spec:
  selector:
    matchLabels:
      serving.knative.dev/service: "ev-battery-classifier"
  endpoints:
    - port: http-usermetric
      path: /metrics
      interval: 15s
```
**How Do We SEE These Numbers? GRAFANA**
Prometheus stores numbers, but it has no good visual dashboard. **Grafana** connects to Prometheus and shows beautiful graphs.  
```
Prometheus (database of numbers)
        │
        ▼
Grafana (asks Prometheus using PromQL)
        │
        ▼
Dashboard with graphs and panels
```
**Example Grafana query:** You want to see "requests per second" on a graph:  
```
rate(revision_request_count[5m])
```

This asks Prometheus: "How fast is request_count growing over the last 5 minutes?" — which gives you requests per second.  
```
Grafana Dashboard:
┌─────────────────────────────────────┐
│  Requests Per Second                │
│                                     │
│  15 ┤         ╭──╮                  │
│  10 ┤    ╭────╯  ╰──╮              │
│   5 ┤───╯           ╰───           │
│   0 ┤                               │
│     └──────────────────────────      │
│     10:00  10:15  10:30  10:45      │
└─────────────────────────────────────┘
```
**How Do We Get ALERTS? ALERTMANAGER**
You don't want to stare at dashboards 24/7. So you define rules — "if this number crosses this threshold, alert me":
```
- alert: ModelLatencyHigh
  expr: histogram_quantile(0.99, rate(revision_request_latencies_bucket[5m])) > 500
  for: 5m
```
**In simple English:** "If p99 latency goes above 500ms for 5 continuous minutes, fire an alert."
```
Prometheus checks rule every 15 seconds
        │
        ▼
Is p99 > 500ms? 
        │
   NO ──→ do nothing
   YES ──→ has it been YES for 5 minutes?
                │
           NO ──→ wait
           YES ──→ send alert to Alertmanager
                        │
                        ▼
                  Alertmanager routes to Slack
                        │
                        ▼
                  #ml-alerts: "p99 latency is 650ms!"
```
> **"p" means percentile.** The number after "p" tells you what percentage of people had a BETTER experience than this value.
> "p99 latency means 99% of requests are served within that time. We use percentiles instead of averages because averages hide tail latency — one slow request can make the average misleading, but p99 clearly shows the worst-case experience for real users."
> ``` p99 captures tail latency — the experience of 1 in 100 requests. ```
> ```p99 under 100–200ms is generally considered good. Under 50ms is excellent.```
  


# 4. ML-Specific Monitoring Layer
This layer monitors model behavior, not infrastructure or pods.  

- Infra tells you system health.  
- Platform tells you workload health.  
- Serving tells you request health.  
- ML layer tells you model quality health.  

**Why Do We Need ML-Specific Monitoring?**  
Your EV Battery model is deployed on KServe.  
Prometheus says:  
  - Pod is running         ✅
  - CPU usage normal       ✅
  - Memory usage normal    ✅
  - Latency p99 = 80ms    ✅
  - Error rate = 0%        ✅
  - All 3 replicas ready   ✅

Everything looks PERFECT from infrastructure side.  

BUT...  

The model is predicting "healthy battery" for 98% of requests.  
In reality, 30% of those batteries have thermal risk.  
The model is SILENTLY giving WRONG predictions.  
No alert fired. No error. No crash.  
> This is the scariest problem in ML — the system is healthy but the model is wrong. Infrastructure monitoring cannot catch this. You need ML-specific monitoring.

**Monitor:**
1. Data Drift (Feature Distribution Monitoring)
2. Prediction Drift (Output Distribution Monitoring)
3. Data Quality Monitoring
4. Model Confidence Score Monitoring
5. Model Performance Monitoring (Accuracy Degradation)

## 1. Data Drift
What it means: The data coming to your model in production looks different from the data it was trained on.
EV Battery example:  
```
TRAINING DATA (what model learned from):
  - Voltage range: 350V to 420V
  - Temperature range: 20°C to 45°C
  - Battery age: 0 to 5 years

PRODUCTION DATA (what model receives today):
  - Voltage range: 280V to 380V       ← shifted DOWN!
  - Temperature range: -10°C to 15°C  ← shifted DOWN! (winter season)
  - Battery age: 3 to 8 years         ← shifted UP! (older fleet)
```
**Why it happens:**
- Seasonal changes (summer → winter)
- New battery hardware model introduced
- Different driving conditions (highway vs city)

### Step 1 — Log Prediction Data to S3
Before you can detect drift, you need the raw data. Every time a prediction request comes to KServe, the Transformer logs the input features to S3.  
Why Transformer and not Predictor?  
```
Client ──request──→ Transformer ──→ Predictor ──→ Transformer ──→ Client
                    (preprocess)    (model)       (postprocess)

Transformer sees BOTH:
- Raw input features (before prediction)
- Model output (after prediction)

So Transformer is the best place to log.  
```

### Step 2 — KFP Monitoring Pipeline Pulls Data 
Now a scheduled KFP pipeline runs every 6 hours. Its first job is to pull the logged data from S3 and convert it into a pandas DataFrame.  

#### Component 1: Pull Reference Data (Training Data via DVC)

To detect drift, Evidently needs **two datasets**: 
```
Reference data = what the model was trained on (the "normal" data)
Current data   = what the model is receiving now (production data)

Evidently compares these two and says "are they similar or different?"
```
#### Component 2: Pull reference data via DVC 

#### Component 3: Run Evidently drift Detection report
Now the core part — Evidently compares these two datasets.

**What Evidently does internally:**
```
For each feature (voltage, temperature, current, battery_age, soc):

1. Take reference data distribution:
   voltage reference: mean=388, std=15, range=[350, 420]

2. Take production data distribution:
   voltage production: mean=340, std=25, range=[280, 400]

3. Apply statistical test (PSI by default):
   PSI = Σ (P_prod - P_ref) × ln(P_prod / P_ref)
   
   PSI for voltage = 0.32
   
4. Compare PSI against threshold:
   PSI < 0.1  → No drift
   PSI 0.1-0.25 → Moderate drift
   PSI > 0.25 → Significant drift ← voltage is HERE!
   
5. Repeat for every feature
```  

#### Component 4: Push drift Scores metrics to Prometheus via Pushgateway

Now you have drift scores. You need to get them into Prometheus so Grafana can display them and Alertmanager can alert on them.

**Why Pushgateway?**
```
PROBLEM:
  Prometheus works by PULLING (scraping) metrics from running pods.
  But KFP pipeline pod runs for 5 minutes and DIES.
  Prometheus cannot scrape a dead pod.

SOLUTION:
  Pipeline pod PUSHES metrics to Pushgateway.
  Pushgateway is ALWAYS running.
  Prometheus scrapes Pushgateway.
```

**First — Install Pushgateway on your cluster:** This creates a Pushgateway pod in the `monitoring` namespace with a ServiceMonitor so Prometheus automatically scrapes it.

After this push, Pushgateway holds these metrics: Prometheus Scrapes Pushgateway. This happens **automatically** because when you installed Pushgateway with `serviceMonitor.enabled=true`, a ServiceMonitor was created: You don't need to do anything for this step. It's automatic.  

### Step 3 — Wire Everything as a KFP Pipeline  (@dsl.pipeline)   
KFP Monitoring Pipeline:
  - @dsl.component → pull_data, pull_reference, detect_drift, push_metrics
  - @dsl.pipeline  → wires components together
    
### Step 4 - submit pipeline(submit_monitoring_pipeline.py) to kubeflow to run every 6 hours.  

### Step 5 — Grafana Dashboard
 Now Grafana can query Prometheus and show drift metrics.  

### Step 4 — Alertmanager checks rules 
Slack: "Data drift detected! voltage PSI=0.32, temperature PSI=0.45"  
PSI Threshold:  
```  
**PSI Value**    **Interpretation**                  **Action** 
< 0.1                No drift                      No action needed
0.1 - 0.25           Moderate drift                Investigate, monitor closely
> 0.25               Significant drift             Trigger retraining pipeline
```
### Step 5 GitHub Actions runs retraining Pipeline.  
Step 1: Alertmanager sends webhook to GitHub  
Step 2: GitHub Actions workflow listens for repository_dispatch  
Step 3: After PR is merged → ArgoCD deploys -> ArgoCD watches main branch -> ArgoCD detects change and sync 

> It is a shared responsibility. But implementation ownership is usually MLOps.  
> Model quality in production = MLOps + Data Science collaboration.  
> Data Scientist:  Defines evaluation metrics  and Provides baseline thresholds . 
> MLOps Engineer:  Implements logging,  Builds dashboards,  Creates alert rules,  Monitors degradation  

## 2. prediction Drift:
The model’s output (predictions) is changing over time compared to what it used to produce earlier. 
Compare Historical Predictions (Last 30 Days — Reference) vs Current Predictions (Last 6 Hours — Today)

**classes:** Your EV Battery model is a classification model. it predicts ONE of these answers: These 4 possible answers are called classes. 
```
Class 1: "healthy"
Class 2: "thermal_risk"
Class 3: "overvoltage"
Class 4: "degradation"
```
**What is "Class Distribution"?**
Class distribution means how many predictions fall into each class over a period of time.  
```
healthy:       6,000 predictions → 60%
thermal_risk:  2,000 predictions → 20%
overvoltage:   1,500 predictions → 15%
degradation:     500 predictions →  5%
                                  ─────
                          Total = 100%

This is the CLASS DISTRIBUTION:
  healthy = 60%, thermal_risk = 20%, overvoltage = 15%, degradation = 5%
```
**What is "Class Distribution SHIFT"?**
It means the percentages changed significantly compared to before.  
```
LAST 30 DAYS (normal):
  healthy:       ████████████████████████████████  60%
  thermal_risk:  ██████████                        20%
  overvoltage:   ███████                           15%
  degradation:   ██                                 5%

LAST 6 HOURS (shifted):
  healthy:       ███████████████████████████████████████████████  95%
  thermal_risk:  █                                                2%
  overvoltage:   █                                                2%
  degradation:   ░                                                1%

The distribution has SHIFTED!
"healthy" jumped from 60% to 95%
All other classes almost disappeared
```
> This shift is the problem. The model is suddenly predicting "healthy" for almost every battery. This is abnormal.

**What is Confidence?**
Every time your model makes a prediction, it doesn't just say the class — it also says how sure it is. This "sureness" is called confidence score (or probability).  ```  
Model receives battery data and returns:

{
    "prediction": "thermal_risk",    ← the answer (class)
    "confidence": 0.92               ← how sure the model is (0 to 1)
}  
```
Confidence is always between 0 and 1:
```
confidence = 0.0 → model is COMPLETELY UNSURE (random guess)
confidence = 0.5 → model is CONFUSED (50-50, coin flip)
confidence = 0.8 → model is FAIRLY SURE
confidence = 0.95 → model is VERY SURE
confidence = 1.0 → model is 100% CERTAIN  
```

You don't calculate PSI manually. Evidently does it automatically.  













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
Accuracy=```(TP+TN)/Total```    

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
Out of all predicted positives, how many are correct?  
Precision=```TP/(TP+FP)```   

Suppose model predicts 10 batteries as faulty.    
But in real:  
7 are really faulty  
3 are actually healthy  
```
So:  
TP = 7  
FP = 3  
Precision = 7 / (7 + 3) = 7 / 10 = 70%  
Meaning:  
Out of all predicted faults, 70% were correct.  
```  
* It tells, How reliable your positive prediction is.   
* If precision is low, Model gives too many false alarms.  

- When to Choose: Used when false alarms are costly and dangerous. You want to maintain high trust in positive prediction.  
- model has predicted mail to be spam but actual value is not spam. If this is important for you then choose Precision and reduce FP to ZERO. It Depends on small or big problem. In some cases like Fraud Detection, Cybersecurity ....etc.  

Medical Diagnosis: Real Examples  
If model says patient has disease but actually not:  
Unnecessary stress  
Expensive tests  
Precision important.  

###### Recall (Sensitivity)  
- Out of all real positive cases, how many did the model correctly detect?
- Recall=```TP/(TP+FN)```  
```
So:  
TP = 7  
FN = 3  
Recall = 7 / (7 + 3)  
Recall = 7 / 10 = 70%  
Meaning:  Model detected 70% of real faults.  
- It tells, How good the model is at finding real positives.  
```  
Suppose:  
10 batteries are actually faulty.  
Model detected only 7 faulty.  

Medical Example: If 100 patients actually have cancer, Model detects only 60. Recall = 60%  
This means: ```40 patients missed. Very dangerous.```  

If you increase recall: 
You reduce FN 
But FP may increase. 

If you reduce FP too much: 
Recall may decrease. 

###### F1-Score
- Balance between Precision and Recall, It combines both into one number.  
- Formula: ```F1=2×(Precision×Recall)/(Precision+Recall)``` It is harmonic mean (not normal average).    

Why We Need F1?:  
Sometimes:  
Precision high  
Recall low  
OR  
Recall high  
Precision low  
Accuracy will not show this properly. F1 gives balance.  
```
Suppose:  
Precision = 80%  
Recall = 60%  

F1 =
2 × (0.8 × 0.6) / (0.8 + 0.6)
= 2 × 0.48 / 1.4
= 0.96 / 1.4
= 0.68

So F1 = 68%
It shows balanced performance.
```  
When To Use F1-Score?  
Use F1 when:  
- Data is imbalanced  
- You care about both FP and FN  
- You want single metric for model selection  


**scripts**
node-cpu-alert.yaml  
Alertmanager_config.yaml  

transformer.py — log prediction data to s3 
pipeline_monitoring.py    # Wire all 4 components as a KFP Pipeline
submit_monitoring_pipeline.py # Schedule it to run every 6 hours:  
retrain_trigger_server.py (updated for GitHub) 


