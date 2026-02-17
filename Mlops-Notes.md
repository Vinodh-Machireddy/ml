# INTERVIEW TARGET
1. Ml Pipelines : Kubeflow Pipelines
2. Experiment Tracking + Model Registry: Mlflow
3. Deployment and serving : ArgoCD + Kserve
4. Monitoring and alerting : Prometheus & Grafana
5. Ci/cd Pipelines : GitHub Actions
6. Cloud : AWS S3, ECR, EKS, IAM, SageMaker, Cost optimization, Security (AWS Secrets Manager)

# Production Ready Tech Stack: 
```Kubeflow Pipelines(KFP), Mlflow, ArgoCD, Kserve, Prometheus & Grafana, GitHub Actions, AWS, Git, GitHub, DVC, Docker, Kubernetes, Python, Linux.```

# CI/CD Pipeline Using GitHub Actions
### To start ci/cd we need few source files like. 
- train.py which has main training script.
- api.py
- Dataset
- requirements.txt: file where dependencies (libraries/packages) that your project needs to run
- GitHub actions workflow file i.e main.yml:  This file defines the entire ci/cd pipeline. 
- .gitignore File: To avoid pushing unnecessary, temporary, or sensitive files into the repo. 
NOTE:- This files we get from DS and ML teams.

### What we do?(Manual work)
1. Initial Setup
2. DVC Setup (Data Version Control)
3. Push Model to S3
4. Kubernetes Cluster
5. KServe Setup
6. Test KServe Inference

To Automate Test, Build, And Deploy we use ci/cd pipelines using GitHub actions.

[GitHub Actions Repo Structure](<img width="281" height="442" alt="Screenshot 2026-02-11 at 3 11 23 PM" src="https://github.com/user-attachments/assets/97f4d798-7b12-43a0-9f4e-655594f48a66"/>)

### Ci/cd Pipeline Flow:
1. Checkout code
2. Setup Python
3. Install Dependencies
4. Unit tests (pytest)  # Code Validation
5. Lint check (flake8)
6. Security Scan (Trivy)
7. Generate Dataset 
8. Train Model 
9. Configure AWS Credentials
10. Install DVC
11. Push Model to S3 via DVC
12. Login to Amazon ECR
13. Build Inference Docker Image
14. Push Docker Image to ECR.  #CI Ends Here (Artifact + Image ready)
15. Update KServe Manifest inference.yaml with new image tag #CD starts here
16. Git Commit Deployment Config
17. ArgoCD Syncs Deployment
18. KServe Deploys Model pod
19. Monitoring + Alerts


`Note:- Trigger Kubeflow Training pipeline (optional). Big companies go instead of running training in GitHub runner it runes in cluster, it is heavy lift.`

### Continuous Integration (CI)
This job is triggered on every push, PR, workflow_dispatch to the main branch.

### Continuous Deployment (CD)
 -  ArgoCD
 -  KServe

### name: MLOps ci/cd Pipeline
```
on:
  push:
    branches: [cicd]
  workflow_dispatch:
  repository_dispatch:

env:
  AWS_REGION: us-east-1
  S3_BUCKET: churn-model-bucket-cicd-abhi
  ECR_REPOSITORY: churn-model-repo
  IMAGE_TAG: ${{ github.sha }}

jobs:
  train-and-deploy:
    runs-on: ubuntu-latest

    permissions:
      contents: write

    steps:

    # ✅ 1. Checkout Code
    - name: Checkout code
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.GITHUB_TOKEN }}

    # ✅ 2. Setup Python
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    # ✅ 3. Install Dependencies
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest flake8

    # ==================================================
    # ✅ NEW STAGE — CODE VALIDATION
    # ==================================================

    # ✅ Unit Tests
    - name: Run Unit Tests
      run: pytest

    # ✅ Lint Check
    - name: Run Lint Check
      run: flake8 .

    # ✅ Security Scan (Trivy)
    - name: Security Scan using Trivy
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'

    # ==================================================
    # ✅ ML TRAINING
    # ==================================================

    # ✅ 4. Generate Dataset
    - name: Generate dataset
      run: python generate_data.py

    # ✅ 5. Train Model
    - name: Train model
      run: python train.py

    # ==================================================
    # ✅ MODEL VERSIONING
    # ==================================================

    # ✅ 6. Configure AWS Credentials
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    # ✅ 7. Install DVC
    - name: Install DVC
      run: pip install dvc[s3]

    # ✅ 8. Push Model to S3 via DVC
    - name: Push model using DVC
      run: |
        dvc push
        echo "Model pushed to S3"

    # ==================================================
    # ✅ CONTAINER BUILD + REGISTRY
    # ==================================================

    # ✅ 9. Login to Amazon ECR
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    # ✅ 10. Build Docker Image
    - name: Build Docker image
      run: |
        docker build -t ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }} .

    # ✅ 11. Push Docker Image to ECR
    - name: Push Docker image
      run: |
        docker push ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}

    # ==================================================
    # ✅ GITOPS DEPLOYMENT PREP
    # ==================================================

    # ✅ 12. Update KServe inference.yaml
    - name: Update inference.yaml with new image
      run: |
        NEW_IMAGE="${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}"
        sed -i "s|image: .*|image: $NEW_IMAGE|g" k8s/inference.yaml

    # ✅ 13. Commit Changes → Triggers ArgoCD Sync
    - name: Commit updated inference.yaml
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add k8s/inference.yaml
        git commit -m "Update inference image to $IMAGE_TAG [skip ci]" || echo "No changes"
        git push || echo "No changes to push"

    # ==================================================
    # ✅ OPTIONAL — DEPLOYMENT NOTIFICATION / MONITORING HOOK
    # ==================================================

    - name: Deployment Notification
      run: echo "Deployment committed. ArgoCD + KServe will take over.”
Note:- ArgoCD / KServe / Monitoring → cannot be inside GitHub YAML (they run after commit)

    # ------------------------------------------------
    # ✅ Wait for ArgoCD Sync
    # ------------------------------------------------
    - name: Wait for ArgoCD Sync
      run: |
        echo "Waiting for ArgoCD deployment..."
        sleep 60

    # ------------------------------------------------
    # ✅ Health Check After Deploy
    # ------------------------------------------------
    - name: KServe Health Check
      run: |
        curl -f https://your-kserve-endpoint/health || exit 1

    # ------------------------------------------------
    # ✅ Slack Notification
    # ------------------------------------------------
    - name: Slack Notification
      if: success()
      uses: slackapi/slack-github-action@v1
      with:
        payload: |
          {
            "text": "✅ Churn Model Deployment Success - Image Tag: ${{ env.IMAGE_TAG }}"
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

14.  ArgoCD
15.  KServe
16. Monitoring + alerting
```
### Continuous Training(CT)
CT means automatically retraining the model when new data or drift is detected.

#### CT Triggered When
 1. Scheduled retraining time 
 2. and New Data Uploaded 
#### CT Steps Flow
1. Night 2 AM → New production data pulled
2. Data validation
3. Train Model
4. Evaluate Model (If accuracy > threshold → register model)
5. Push Model to DVC/S3
6. Trigger CD → Deploy new model

#### Ex:-
```on:
  schedule:
    - cron: "0 2 * * *"    # Runs daily at 2 AM
  workflow_dispatch:       # Manual trigger also allowed
```
#### Ex:-
```on:
  repository_dispatch:
    types: [new-data-arrived]
```
# MONITORING (PROMETHEUS & GRAFANA)

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
Prometheus uses Kubernetes API to discover Pods, Services, Nodes, Endpoints, Ingress. When a new pod comes → Prometheus automatically starts scraping it. When pod deletes → Prometheus stops scraping.

#### Types: 
- Kubernetes Service Discovery
- Static Service Discovery
- EC2 service discovery
- DNS-based discovery
- File-Based Service Discovery

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


## 1️⃣ Prometheus-Based Native Kubernetes Monitoring Architecture
  
1️⃣ Infrastructure Layer
Monitor cluster stability.  

Metrics:  
Node CPU  
Node Memory
Disk usage
Network traffic
Node availability  

Collected by:
node-exporter
kube-state-metrics    

Standard Process for Any Infra Metric  
For CPU, Memory, Disk, Node availability, Pod restarts — workflow is always same:   

Step 1 : Install Monitoring Stack 
we use kube-prometheus-stack it is a pre-packaged Helm chart that automatically installs a complete Kubernetes monitoring setup like Prometheus, Alertmanager, Grafana, NodeExporter, kube-state-metrics. It provides ready-made dashboards, alert rules, and Kubernetes service discovery.  
Step 2 : Verify Node-Exporter is Running & CPU Metrics in Prometheus
kubectl get pods -n monitoring
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090  and http://localhost:9090
 
Step 3 : Create CPU Usage Query (PromQL)
```100 - (avg by(instance)(
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100)
``` 
Step 4 : Add CPU Panel in Grafana
  1. Open Grafana
  2. Create Dashboard
  3. Add Panel
  4. Select Prometheus as data source
  5. Paste above quer
  6. Set Unit → Percent (0–100)  
     Now you will see live node CPU usage. 
Step 5 : Create Alert rule and apply for High CPU
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


Step 6 : Configure Alertmanager  
Alertmanager already installed via kube-prometheus-stack.
You configure Slack or Email receiver.
When CPU > 85% for 5 mins → alert triggers.  

1. Verify Alertmanager is Running  
   i/p: kubectl get pods -n monitoring
   o/p: alertmanager-monitoring-kube-prometheus-alertmanager-0   Running  (If running → ready to configure.)  
 
2. Create Slack Webhook  
   Go to Slack → Settings  
   Create Incoming Webhook  
   Select channel  
   Copy webhook URL
   ```Example:  https://hooks.slack.com/services/T000/B000/XXXX```

3. Create Alertmanager Config File
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
4. Create Kubernetes Secret
   Alertmanager config must be stored as secret.
   ```kubectl create secret generic alertmanager-monitoring-kube-prometheus-alertmanager \
  --from-file=alertmanager.yaml=alertmanager-config.yaml \
  -n monitoring \
  --dry-run=client -o yaml | kubectl apply -f -
```
5. Restart Alertmanager
```kubectl delete pod alertmanager-monitoring-kube-prometheus-alertmanager-0 -n monitoring```
6. Verify Configuration
```Port forward:  kubectl port-forward svc/monitoring-kube-prometheus-alertmanager -n monitoring 9093
   open:  http://localhost:9093
   Check:
   Status → Config
   Make sure Slack and email receivers are visible.
```
7. Test Alert  
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


   
Step 7 : When Does Alert Trigger?  
```
If:
Node CPU > 85%
AND
Condition lasts for 5 minutes
Then alert becomes ACTIVE.
You can check in: Prometheus → Alerts tab  or   Grafana → Alert panel
```





2️⃣ Kubernetes Platform Layer
Monitor workloads.  

Metrics:
Pod status
Restart count
Deployment health
HPA scaling
Resource limits vs usage  

Alerts:
Pod CrashLoopBackOff
HPA max replicas reached
Pod not ready > 5 mins
This ensures deployment reliability.  

3️⃣ Model Serving Layer (KServe)
When model is deployed using KServe, monitor:

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

4️⃣ ML-Specific Layer
This differentiates MLOps from DevOps.  

Monitor:
- Feature distribution
- Prediction distribution
- Drift detection
- Confidence score
Accuracy drop (if feedback loop available)  

Expose these as custom Prometheus metrics from model code.  


