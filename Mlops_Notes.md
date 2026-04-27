# MACHINE LEARNING:
Machine Learning (ML) is a process / technique of teaching a computers using data, instead of writing rules manually like old programs. A computer learns patterns from past data and uses that learning to make predictions or decisions on new data. 

**Old (Traditional) way:**
You write rules, Computer follows rules   
	Example: IF marks > 35 THEN pass  

**Machine Learning way:**
You give data + answers. Computer learns rules by itself  
	Example: Past student marks + pass/fail result  
			 Model learns what marks usually mean pass or fail  

## Types of Machine Learning approachs  
### 1. Supervised Learning:
The model learns from labeled data. examples (you tell it the “right answer”). our feed in input data (features) and the correct output (label). The algorithm discovers the mapping from input → output.  

if output column data is categorical(yes/no), true/false, multi-class data then we go for Classification algorithm family.  
**supervised classification algorithms examples:** 
1. Decision Tree Classifier
2. Random Forest Classifier
3. Logistic Regression
4. XGBoost

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


### 2. UnSupervised Learning:
Unsupervised Learning is a machine learning approach where a model is trained on unlabeled data — meaning there are no correct output labels provided. The model must find hidden patterns, structure, or groupings in the data on its own.  

When to use: Clustering (e.g., grouping customers by behaviour, features)  

eg:-
K‑Means Clustering
Hierarchical Clustering 
Principal Component Analysis (PCA)
DBSCAN

### 3. Reinforcement Learning(RL)
Learn by trial and error — get rewarded for good actions, penalized for bad ones.
Algorithm: Q-Learning
Example: games, robots...etc
> **NOTE:** Structured / Unstructured is about DATA FORMAT  
	        Supervised / Unsupervised / RL is about LEARNING APPROACHS  

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


















## DATASETS
Kaggle Datasets:  https://www.kaggle.com/datasets  
GitHub Repositories  
Plotly : opensource datasets to train ml models  
uci machine learning repository:  https://archive.ics.uci.edu  

**built‑in datasets in python for pratice:**
1. scikit‑learn (sklearn.datasets)
2. seaborn (seaborn.load_dataset())
3. statsmodels (statsmodels.datasets)

**MODEL-HUB:**
models that were already trained on large datasets by big companies or research labs, and made available for others to reuse or fine-tune instead of training from scratch.  

Hugging Face, TensorFlow Hub, PyTorch Hub  
AWS SageMaker JumpStart → ready-to-use pre-trained models for NLP, vision, tabular  
Azure ML Model Registry → curated pre-trained models  
Google Cloud Vertex AI Model Garden → Google & open-source models  

**Terminology:**  
Row : --->	Sample / Instance / Observation / Record  
Column : ---->	Feature / Attribute / Variable  
output column : -----> is a special type of column:  target column/target variable/label/answer  

**data:**  
1. primary and
2. secondary data
train and test data  
input and output columns/data  

CPU: Central processing unit  
GPU: Graphical processing unit  
TUP: Tensor processing unit  
QPU: Quantum Processor unit  
DBU: Data Bricks Units  


**File formats for saving machine learning models:**  
1.  .pkl (Python Pickle)  
Primary Use: Saving models from libraries like Scikit-learn, XGBoost, and LightGBM. It's great for traditional machine learning models.  
2.  .h5 (Hierarchical Data Format)  
Primary Use: Saving models from deep learning frameworks, specifically for Keras and TensorFlow.  
3.  .pt / .pth (PyTorch)  
Primary Use: Saving and loading models in PyTorch.  
4.  .onnx (Open Neural Network Exchange)  
Primary Use: Deploying models in production, especially when the training framework is different from the inference environment.  


