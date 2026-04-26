import pandas as pd
import mlflow
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder, StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import f1_score, classification_report
from xgboost import XGBClassifier
import joblib

## 1. Load data
data = pd.read_csv("s3://daimler-battery/data/train.csv")    # we load the dataset

X = pd.DataFrame(data.data, columns=data.feature_names)		  # Creating X and y from a dataset (convert features into a structured DataFrame and extract X, y)
y = data.target 											

## 2. Split data (train/test)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

## 3. Train model  
model = XGBClassifier(
    n_estimators=100,        # number of trees
    max_depth=6,             # how deep each tree grows
    learning_rate=0.05,      # how much each tree contributes
    random_state=42          # reproducibility
)
model.fit(X_train, y_train)

## 4. Predict
y_pred = model.predict(X_test) 
print(y_pred)  

## 5. Evaluate
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy}") 
