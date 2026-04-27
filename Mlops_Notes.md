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
**supervised classification algorithms:** 
1. Decision Tree Classifier
2. Random Forest Classifier
3. Logistic Regression
4. XGBoost
5. LightGBM
6. CatBoost

if output column/target data is numerical labels than we go for Regression Algorithm under supervised technics.  
**supervised regression algorithms:**
1. Simple Linear Regression
2. Ridge Regression
3. Lasso Regression
4. Elastic Net  

> **NOTE:** fixed evaluate/performance metrics for regression models  
	- mean_squard_error  
	- mean_absolute_error  
	- r2_score  

### 2. UnSupervised Learning:
Unsupervised Learning is a machine learning approach where a model is trained on unlabeled data — meaning there are no correct output labels provided. The model must find hidden patterns, structure, or groupings in the data on its own.  
**When to use:** Clustering (e.g., grouping customers by behaviour, features)  

**UnSupervised algorithms:**
1. K‑Means Clustering
2. Hierarchical Clustering 
3. Principal Component Analysis (PCA)

### 3. Reinforcement Learning(RL):
Learn by trial and error — get rewarded for good actions, penalized for bad ones.  
Algorithm: Q-Learning  
Example: games(famous), robots...etc

## Data
	1. Structured / Unstructured is about DATA FORMAT  
	2. Supervised / Unsupervised / RL is about LEARNING APPROACHS  

### Structured data:  
Structured data is data that is organized in a fixed, predefined format — typically rows and columns — making it easy to store, search, and analyze. In one line: `"Data that fits neatly into a table."`  Usually stored in:  Tables, CSV, Excel, Databases

| Battery_ID   |   Voltage (V) |   Temperature (°C) |   SOC (%) | Fault_Label     |
|:-------------|--------------:|-------------------:|----------:|:----------------|
| BAT_001      |          3.85 |                 42 |        78 | Healthy         |
| BAT_002      |          3.21 |                 61 |         9 | Thermal_Runaway |
| BAT_003      |          3.9  |                 38 |        55 | Healthy         |

> Every row = one record, every column = one feature. Clean, consistent, queryable.

### Unstructured data:
Data without fixed rows & columns. Data that does NOT fit into a table. Requires specialized models — CNNs for images, RNNs/Transformers for text/audio   
**Examples:** Text (emails, chat, PDFs), Images, Audio, Video, documents, Logs (semi-structured)   

**DATASETS:**
Kaggle Datasets:  https://www.kaggle.com/datasets  
GitHub Repositories  
Plotly : opensource datasets to train ml models  
uci machine learning repository:  https://archive.ics.uci.edu  

### Algorithm:
An algorithm is just a step-by-step set of instructions to solve a problem or perform a task. It’s like a recipe a computer follows to get the desired result.  
Input → Algorithm → Output   
An algorithm is the mathematical method used to learn patterns from data.  

### Model:
The result (output) of that learning process.  

**File formats:**  
A file that stores a trained machine learning model so it can be loaded and used later.   
1.  .pkl (Python Pickle)  
Primary Use: Saving models from libraries like Scikit-learn, XGBoost, and LightGBM. It's great for traditional machine learning models.  
2.  .h5 (Hierarchical Data Format)  
Primary Use: Saving models from deep learning frameworks, specifically for Keras and TensorFlow.  
3.  .pt / .pth (PyTorch)  
Primary Use: Saving and loading models in PyTorch.  
4.  .onnx (Open Neural Network Exchange)  
Primary Use: Deploying models in production, especially when the training framework is different from the inference environment.   

### MLOPS
Machine Learning Operations which is influenced from DevOps. DevOps is for Traditional Application, Mlops is for machine learning model(Traditional software + ML model = Intelligent system).  
In DevOps we  focuses on automating the lifecycle of traditional software/Application(SDLC), were as in MLOps it automates the lifecycle of a machine learning model(MDLC).  

**Examples:**
1. Youtube, Netflix: Traditional software: OTT streaming platform—>DevOps   
   ML model used: Recommendation engine—>MLops    
2. PayPal: Traditional software: Online payment system 
   ML model used: Fraud detection model   
3. Amazon: Traditional software: E-commerce platform  
   ML model used: Product recommendation + pricing  
4. Banking Apps: Loan approval, Credit risk models  
5. Swiggy/Zomato: Traditional software: Food delivery ML models used: Restaurant recommendation, Delivery time prediction

### Machine Learning Life Cycle:
Business problem  
Data collection  
Data validation  
Feature engineering  
Model training  
Experiment tracking  
Model evaluation  
Model registry  
Deployment  
Monitoring  
Retraining  

## MODEL-HUB
models that were already trained on large datasets by big companies or research labs, and made available for others to reuse or fine-tune instead of training from scratch.   

- Hugging Face, TensorFlow Hub, PyTorch Hub  
- AWS SageMaker JumpStart → ready-to-use pre-trained models for NLP, vision, tabular  
- Azure ML Model Registry → curated pre-trained models  
- Google Cloud Vertex AI Model Garden → Google & open-source models  

## Terminology
Row : --->	Sample / Instance / Observation / Record / Data Point
Column : ---->	Feature / Attribute / Variable / Predictor / Input / Input Feature
Label/Class : -----> Target Variable / Target Column / Output / Response / Answer / Ground Truth / Target Label

CPU: Central processing unit  
GPU: Graphical processing unit  
TUP: Tensor processing unit  
QPU: Quantum Processor unit  
DBU: Data Bricks Units  




