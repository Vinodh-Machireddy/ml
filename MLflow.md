
## What is Experiment Tracking?
When Data Scientist trains a model, they try many things(Different algorithms, hyperparameters, feature sets, datasets, evaluation metrics), we record every ML experiment details properly.  If we don’t track properly:  
- We don’t know which model gave best accuracy  
- We can’t reproduce results  
- We can’t compare experiments

### So experiment tracking stores:
- Parameters (learning rate, epochs, batch size)
- Metrics (accuracy, precision, recall, F1)
- Model artifacts
- Dataset version
- Code version
- Timestamp
- Who ran the experiment
## What is MLflow?
MLflow is an open-source platform for managing the end-to-end machine learning lifecycle.   
It provides four core components:   
1. Tracking (logging parameters, metrics, and artifacts from experiments)
2. Projects (packaging ML code in a reproducible format)
3. Models (a standard format for packaging and deploying models across different serving environments)
4. Model Registry (a centralized store for managing model versions, stages, and annotations).
   > Widely used for Tracking and Model Registry.


## Who is Responsible? DS or MLOps?
✅ Data Scientist Responsibility:
- Logging parameters & metrics
- Running experiments
- Comparing models
- Selecting best model
- Registering model (sometimes)  

✅ MLOps Engineer Responsibility:
- Setting up MLflow server
- Configuring backend store (DB)
- Configuring artifact store (S3)
- Access control & security
- Automating model promotion
- CI/CD integration
- Governance & compliance
- Production deployment

### Who handles experiment tracking?  
“In our project, Data Scientists log experiments using MLflow inside Kubeflow Pipelines. As an MLOps Engineer, I set up and managed the MLflow tracking server,  I configured aws S3 for artifact store, integrated it with CI/CD, and implemented model promotion workflow from staging to production.”  

Components:
MLflow → Runs inside EKS (Deployment)  
RDS PostgreSQL → Backend store  
S3 → Artifact store  
IAM Role → Secure S3 access  
ALB / Ingress → Access MLflow UI  

## MLflow deployed as Deployment in EKS 
🔹 Step 1: Create RDS PostgreSQL  
🔹 Step 2: Create S3 Bucket  
🔹 Step 3: Configure IAM Role for EKS  
🔹 Step 4: Create Docker Image for MLflow  
🔹 Step 5: Create Kubernetes Deployment YAML  
🔹 Step 6: Create Service  
🔹 Step 7: Expose via Ingress (ALB)  
🔹 Step 8: Integrate with Kubeflow  
🔹 Step 9: Manage via ArgoCD (GitOps)  

## Configure Backend Store with PostgreSQL (RDS)  
 Backend Store: The database used to store experiment tracking information. This backend store contains metadata.  

> Backend Store = Database  
  Metadata = Data stored inside that database

Metadata means: "Data about data"  
In MLflow, metadata includes: ```Experiment name, Run ID, Parameters, Metrics, Tags, info, Timestamp, Model name, version, stage ``` All this is metadata.  

Step 1️⃣ Create RDS PostgreSQL  
Example DB details:  
```
DB name: mlflowdb
Username: mlflowuser
Password: ********
Port: 5432
```
Step 2️⃣ Install PostgreSQL Driver   
When MLflow connects to PostgreSQL database, it needs a Python driver to talk to the database.  
In MLflow Docker image:  
```
FROM python:3.9-slim
RUN pip install mlflow psycopg2-binary boto3
CMD ["mlflow", "server"]
```
Step 3️⃣ Start MLflow with Backend Store  
Connection string format:  ```postgresql://username:password@host:port/dbname```  
Example:   
```
mlflow server \
--backend-store-uri postgresql://mlflowuser:password@rds-endpoint:5432/mlflowdb \
--default-artifact-root s3://my-mlflow-artifacts \
--host 0.0.0.0 \
--port 5000
```
Now:  
✅ Metadata → RDS  

## Configuring artifact store (S3)
Artifact store → actual files: ```model.pkl, Pickle files, plots.png, confusion_matrix.png, feature_importance.csv, entire model folder```  
Production Architecture, Since you are using EKS + RDS + S3:  
```
Data Scientist
     ↓
MLflow Server (EKS Pod)
     ↓
Metadata → RDS
Artifacts → S3
```


Step 1️⃣ Create S3 Bucket  
Step 2️⃣ Give Permission to MLflow  
   - MLflow running inside EKS must access S3. 👉 Use IAM Role for Service Account (IRSA)

Step 3️⃣ Install S3 Dependency  
   - Inside MLflow Docker image, install:  ``` RUN pip install boto3 ```  
   - boto3 is AWS SDK for Python. Without this, MLflow cannot upload files to S3.

Step 4️⃣ Start MLflow with S3 Artifact Root  
```
mlflow server \
--backend-store-uri postgresql://user:pass@rds-endpoint:5432/mlflowdb \
--default-artifact-root s3://company-mlflow-artifacts-prod \
--host 0.0.0.0 \
--port 5000
```
Now  
Artifacts → S3  
## Automating model promotion
Standard Production Flow  
```
1. Training Completed
2. Model Logged to MLflow
3. Model Registered
4. Evaluate Metrics
5. If metrics pass threshold
       → Promote to Staging
6. Run validation tests
7. If approved
       → Promote to Production
8. Trigger deployment (KServe)
```
Step 1️⃣ Register Model (Inside Training Code)  
```
import mlflow

result = mlflow.register_model(
    "runs:/<run_id>/model",
    "customer-churn-model"
)
```
Now model appears in Model Registry.  
Step 2️⃣ Automatically Promote Based on Metrics  
```
from mlflow.tracking import MlflowClient

client = MlflowClient()

model_name = "customer-churn-model"
version = result.version

if accuracy > 0.90:
    client.transition_model_version_stage(
        name=model_name,
        version=version,
        stage="Staging"
    )
```
This is Automated Model Promotion Logic.  

###  CI/CD integration
### Model in Production Stage  
- Only Production stage models are deployed.
- CI Updates Deployment YAML

STEP 3 — ArgoCD Detects Change

**artifact**
An artifact is any output file generated during training, testing, or pipeline execution.  
Examples: ```plots/metrics, CSV files, images, config files, serialized models, log files.```  

**Model Artifact (Specific Artifact)**
A model artifact is the saved trained model file. It contains everything required to load the model later and perform inference.  
```
Examples: model.pkl, model.joblib, model.pt, model.h5, model.onnx.

import joblib
joblib.dump(model, "model.pkl")

model.pkl → model artifact
```
**Artifact Store**
Artifact Store is the storage location where all artifacts (including model artifacts) are physically saved. It's the backend storage — could be your local filesystem, Amazon S3, Azure Blob Storage, Google Cloud Storage, etc.

  


