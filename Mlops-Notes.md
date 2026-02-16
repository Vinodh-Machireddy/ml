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
# MONITORING

<img width="490" height="320" alt="image" src="https://github.com/user-attachments/assets/5e67dc93-67bb-42c7-b713-91598ca68f1e" />
## Prometheus Components:
### Prometheus Server:  
It collects and stores metrics data.
- Retrieval: Pulls metrics from different targets (like servers or apps).
- TSDB (Time-Series Database): Stores the metrics in a format that makes it easy to query over time. It stores key and value pair format. Eg: 12:00:45 = HB 69
- HTTP Server: Allows you to run queries and interact with Prometheus through a web interface.  

#### Prometheus Targets: (pull metrics)
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

#### Exporters
collect info from nodes and keep in API Endpoints (/metrics) and from there prometheus scrapes(it follows pull mechanism) the metrics and store in TSDB.  
1. Node Exporter/plugin/add-on 2. Kube State Metrics (KSM) 3. cAdvisor  4. API Server Metrics 5. kubelet metrics 6. etcd metrics  


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


### Monitoring starts when ALL 3 are present:
1. Prometheus (kube-prometheus-stack) is installed and running
2. ServiceMonitor created
3. Application exposes /metrics endpoint

### How to confirm monitoring is started (100% proof)
Go to Prometheus UI:  http://localhost:9090/targets
If you see:
```model-service      UP
mlflow-service     UP
kubeflow-service   UP
```

Monitoring starts automatically as soon as:  

KServe pod is running and /metrics endpoint is available , KServe internally uses Knative and Istio. Prometheus detects it → starts scraping metrics → monitoring starts. we no need to manually start monitoring.

How do you design ML monitoring architecture?
How do you monitor model performance in production?
How do you implement alerting strategy?
How do you monitor KServe serving metrics?
Differentiating "System" vs. "Model" Metrics

## 3 strong ML monitoring architectures
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

Reason:
If infrastructure fails → ML serving fails.

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
