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
1. Logistic Regression
2. Decision Tree Classifier
3. Random Forest Classifier
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






### Daimler = Tabular/structured Multi-class Classification problem
``` 
| battery_id | voltage (V) | temperature (°C) | current (A) | capacity (%) | soc (%) | soh (%) | internal_resistance (mΩ) | c_rate (C) | cycle_count | fault_label |
|---|---|---|---|---|---|---|---|---|---|---|
| BAT001 | 3.7 | 35.2 | 1.2 | 98.5 | 82 | 98 | 12.1 | 0.3 | 120 | normal |  # Multi-class — one battery reading gets one label from 7 possible classes. 
| BAT002 | 3.2 | 42.1 | 1.8 | 87.3 | 74 | 87 | 18.4 | 0.6 | 340 | thermal_fault |  # each row = one data point ✅
| BAT003 | 2.8 | 55.6 | 2.1 | 76.2 | 61 | 76 | 24.7 | 0.8 | 520 | overvoltage |
| BAT004 | 4.3 | 38.1 | 2.5 | 91.2 | 98 | 91 | 15.2 | 1.2 | 210 | overcharging |
| BAT005 | 3.1 | 36.2 | 1.1 | 45.3 | 55 | 45 | 38.9 | 0.4 | 890 | cell_degradation |
| BAT006 | 3.6 | 35.8 | 1.3 | 97.1 | 79 | 97 | 42.3 | 0.5 | 150 | internal_short_circuit |
| BAT007 | 3.5 | 34.9 | 1.2 | 96.8 | 31 | 96 | 13.1 | 0.3 | 95 | undervoltage |
```  
SOC — State of Charge level (%): How much charge is currently remaining right now.  
SOH — State of Health level (%): Overall long-term health of the battery — how degraded it is over its lifetime.    
C-Rate (C): How fast the battery is being charged or discharged relative   
Cycle Count (Integer): Total number of complete charge-discharge cycles the battery has completed in its lifetime   


> One full row  -> data point / sample / observation  
> `voltage`, `temperature`, `current`, `capacity`  ->  features / input variables  
> `battery_id` -> identifier — not a feature  
> `fault_label` ->  target / label / output  
>  All rows together -> dataset

```  
Problem Type  : Multi-class Classification
Data Type     : Structured / Tabular
Features      : voltage, temperature, current, capacity (sensor data)
Target        : fault_label normal, thermal_fault, overvoltage,  overcharging, undervoltage, cell_degradation, internal_short_circuit, 
Model         : XGBoost Classifier (Algorithm)
```
**In real MLOps projects you never directly jump to XGBoost.** we follow this process. "I evaluated multiple algorithms — Logistic Regression, Decision Tree, Random Forest, and XGBoost. XGBoost gave the best F1 score of 0.94 so I selected it as my final model."  This is called model selection — you try multiple algorithms and pick the best one.  
```
Step 1 — Baseline model:     Logistic Regression     ← simplest, quick benchmark

Step 2 — Better models
         Decision Tree           ← simple, explainable
         Random Forest           ← stronger than decision tree

Step 3 — Best model
         XGBoost                 ← best performance ✅ — your final model
```
- we used scikit-learn Library and XGBoost algorithm(actual model that learns patterns) in Daimler Project  
- XGBoost stands for eXtreme Gradient Boosting — it is a machine learning algorithm based on decision trees + boosting technique.   
> "I used scikit-learn Python library for preprocessing, metrics, and pipeline — and XGBoost Classifier as my algorithm for the model."     
> Random Forest, Logistic Regression, Decision Tree  , Yes — all work for multi-class tabular classification:   





Logistic Regression → can't capture non-linear patterns
        ↓ fix ↓
Decision Tree → captures non-linear but overfits badly
        ↓ fix ↓
Random Forest → fixes overfitting using many trees + voting
        ↓ fix ↓
XGBoost → fixes remaining errors using boosting → best performance ✅


### Logistic Regression:
Logistic Regression is a supervised machine learning algorithm used for classification — it predicts the probability that a given input belongs to a particular class.  

"Logistic Regression was my baseline model in the battery fault classifier project. It uses a sigmoid function to convert a weighted sum of input features into a probability between 0 and 1, then applies a threshold to assign the final class. For my 7-class problem, it uses Softmax instead of Sigmoid. It gave an F1 score of 0.81 on my dataset — good enough as a benchmark but not sufficient for production fault detection, which is why I moved to XGBoost and achieved 0.94 F1."  

**if Random Forest Already Gives F1 = 0.94:**
You stop at the simplest model that meets your performance requirement. Complexity must always be justified.
```  
Logistic Regression → F1 = 0.81
Decision Tree       → F1 = 0.85
Random Forest       → F1 = 0.94 ✅ ← stop here, no need for XGBoost
XGBoost             → F1 = 0.94 (same — not worth extra complexity)
```
> **The Golden Rule in MLOps:**  
"Always use the simplest model that solves the problem well enough."  
XGBoost is powerful but complex — harder to explain, harder to debug, heavier to deploy. You only go there when simpler models genuinely fall short.  

> If XGBoost fails and your data is good — the problem is the algorithm's limit. That's exactly when you escalate to LightGBM → CatBoost.


### Decision Tree
A Decision Tree is a supervised machine learning algorithm that splits data into branches based on feature conditions — like a flowchart — until it reaches a final prediction at the leaf node.  
```
Is Temp > 55°C?
                   /               \
                YES                 NO
                 |                   |
        Is Voltage < 3.3V?      Is SOC > 95%?
           /        \              /       \
         YES         NO          YES        NO
          |           |           |          |
   Thermal_Runaway  Healthy   Overcharge   Healthy
```

| Part Name | Meaning |
| :--- | :--- |
| **Root Node** | Top node: The first split and most important feature in the dataset. |
| **Internal Nodes** | Middle nodes: Represent feature conditions where the data is further split. |
| **Edges** | Branches: Represent the "Yes" / "No" paths or outcomes of a condition. |
| **Leaf Nodes** | End nodes: The final prediction or class label; no further splitting occurs here. |

**Overfitting and Underfitting Comparison:**
| Meaning | Simple Definition | Analogy |
| :--- | :--- | :--- |
| **Underfitting** | Model is too simple — fails even on training data | Student didn't study at all; cannot answer any questions. |
| **Overfitting** | Model memorizes training data — fails on new data | Student memorizes specific answers, but cannot solve new/unseen problems. |
| **Perfect Fit** | Learns patterns — performs well on new data | Student understood concepts and can solve any related problem. ✅ |  

**Visualized:**
```
Underfitting          Perfect Fit           Overfitting
(too simple)          (just right ✅)        (too complex)

    *                     *                      *
  *   *                 *   *                  *   *
 *     *               *     *                *     *
───────────           ~~~~~~~~~~~          ~~~^~v~^~v~~~

Straight line        Smooth curve         Follows every
misses everything    fits well            noise point
```
**How to Fix:**
Underfitting:	Increase max_depth, add more features, use stronger algorithm
Overfitting:	Decrease max_depth, pruning, add more training data, use Random Forest


### Random Forest:
Random Forest is a supervised machine learning algorithm that builds multiple Decision Trees and combines their predictions through voting to produce a more accurate and stable result.  In one line: `"Instead of trusting one tree — build hundreds of trees and let them vote."`    

Why Random Forest?:
```
Decision Tree
→ One tree
→ Overfits easily
→ Unstable (small data change = completely different tree)
→ High variance ❌

Random Forest
→ 100s of trees
→ Each tree sees different data + different features
→ All trees vote → majority wins
→ Variance cancels out ✅
```

Key Parameters:
n_estimators	Number of trees
max_depth		Depth of each tree
max_features 	Features considered per split
min_samples_split Minimum samples to split node




overfitting and underfitting
Key Parameters 
