import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# 1. Load data
data = pd.read_csv("s3://daimler-battery/data/train.csv")    # we load the dataset

X = pd.DataFrame(data.data, columns=data.feature_names)		  # Creating X and y from a dataset (convert features into a structured DataFrame and extract X, y)
y = data.target 											

# 2. Split data (train/test)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 3. Train model
model = RandomForestClassifier()
model.fit(X_train, y_train)

# 4. Predict
y_pred = model.predict(X_test) 

# 5. Evaluate
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy}") 
