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
 
# CI/CD Pipeline Using GitHub Actions

</div>

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

<details>
  <summary><b>Click to expand: MLOps ci/cd Pipeline Stages</b></summary>

  ### MLOps ci/cd Pipeline
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
   </details>

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

## 1️⃣ After Model is Trained → We Build Docker Image
### 🔵 Case A – Model Embedded Inside Image
If after training you build Docker image and include model file, then image contains:
- Base OS (python:3.9-slim)  
- Python runtime  
- All dependencies (scikit-learn, pandas, etc.)  
- Inference code (FastAPI / Flask app)  
- Model file (model.pkl)  
Example structure inside image:  
```
/app
  ├── app.py
  ├── model.pkl
  ├── requirements.txt
```
So this image is:  👉 Code + Dependencies + Model file  
This means every retraining → new Docker image build.  Used in small systems or simple deployments.  

### 🟢 Case B – Model NOT Inside Image (Production Setup – Your Stack)
In modern production (like KServe + S3):  
After training:  
- Model saved in S3  
- Model registered in MLflow  
- Docker image is NOT rebuilt   

The image contains:  
- Base OS  
- Python  
- Inference server code  
- Logic to download/load model from S3  
It does NOT contain model file. KServe loads model dynamically from S3 path.  
```So retraining → update model URI → redeploy``` No image rebuild required.  
This is scalable and recommended.  


## Training Image:   
Kubeflow does:  
👉 Pull that image  
👉 Create Pod  
👉 Run training inside that container  
So this image is used as: 🟢 Training Environment Image  
It contains:  
Python  
ML libraries  
Training script  
Kubeflow uses it to execute pipeline steps.  

## inference image: 
If image is inference server image:
Then:
ArgoCD deploys it
Kubernetes creates Pod
Container starts
Model loaded from S3
API starts
So image is used as:
🟢 Inference Server Environment
It contains:
FastAPI code
Model loading logic
Dependencies
