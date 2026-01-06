+++
title = "[Palantir Foundry] Titanic Survivor Prediction Project (2/2) — Model Training & Workshop Dashboard"
description = "Train a baseline model in Foundry Code Workspaces (JupyterLab), export predictions for Kaggle, and build a simple dashboard in Workshop using Ontology."
date = 2026-01-06T00:00:00+09:00
draft = false
slug = "titanic-foundry-training-workshop"
tags = ["Palantir", "Foundry", "Titanic", "ML", "Workshop", "Kaggle"]
categories = ["Palantir"]
+++

> This post continues from Part 1 (preprocessing). Using `titanic_cleaned_train`, we go through: **(1) model training in JupyterLab → (2) Kaggle submission → (3) visualization with a Workshop dashboard**.

## 1. Create a Code Workspace (JupyterLab)

1) Go back to the Titanic project home and select **Application**.

![Application](pasted-image-20260105230120.png)

2) Search for `code` → open **Code Workspaces** → click **Create new**.

![Create code workspace](pasted-image-20260105230247.png)

3) Select **JupyterLab** → click **Continue**.

![Select JupyterLab](pasted-image-20260105230351.png)

4) In **Select location**, choose the project `Titanic` → click **Continue**.

![Select location](pasted-image-20260105230431.png)

5) Review the summary and click **Create**.

![Create](pasted-image-20260105230551.png)

6) After a short wait, you should see a page like this.

![Workspace created](pasted-image-20260105230654.png)

7) Open the **Data** panel → click **Add Data** and load the datasets you created.

![Data tab](pasted-image-20260105231829.png)
![Add Data](pasted-image-20260105232116.png)
![Select datasets](pasted-image-20260105232200.png)

8) Create a new Python environment for the notebook and install `scikit-learn` from the library.

![Create python environment](pasted-image-20260105232242.png)
![Install scikit-learn](pasted-image-20260105232656.png)
![Install scikit-learn](pasted-image-20260105232709.png)
![Install done](pasted-image-20260105232814.png)

---

## 2. Load data & train a baseline model (RandomForest)

The following is a baseline setup to quickly validate end-to-end training and prediction.

### 2.1 Load data

```python
# 1. Load data
from foundry.transforms import Dataset
import pandas as pd
import numpy as np

titanic_cleaned_train = Dataset.get("titanic_cleaned_train").read_table(format="pandas")
test = Dataset.get("test").read_table(format="pandas")

print("✅ Data loading complete!")
print(f"Train data: {titanic_cleaned_train.shape}")
print(f"Test data: {test.shape}")
print("\nColumn list:")
print(titanic_cleaned_train.columns.tolist())
```

![Load data](pasted-image-20260105232356.png)

> If you see warnings, you can usually ignore them as long as the data loads successfully.

### 2.2 Preprocess (build training features)

```python
# 2. Data preprocessing and feature preparation

# Add Sex_Encoded to Test data
test["Sex_Encoded"] = test["Sex"].map({"male": 0, "female": 1})

# Missing value handling
test["Age"].fillna(titanic_cleaned_train["Age"].median(), inplace=True)
test["Fare"].fillna(test["Fare"].median(), inplace=True)

# Select features to use for learning
features = ["Pclass", "Sex_Encoded", "Age", "SibSp", "Parch", "Fare"]

# X, y separation
X_train = titanic_cleaned_train[features]
y_train = titanic_cleaned_train["Survived"]
X_test = test[features]

print("✅ Preprocessing completed!")
print(f"\nX_train shape: {X_train.shape}")
print(f"y_train shape: {y_train.shape}")
print(f"X_test shape: {X_test.shape}")
print(f"\nCheck for missing values:")
print(f"Train: {X_train.isnull().sum().sum()}")
print(f"Test: {X_test.isnull().sum().sum()}")
```

![Preprocessing](pasted-image-20260105232527.png)

### 2.3 Train a model (RandomForest) + cross validation

```python
# 3. Random forest model training
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

rf_model = RandomForestClassifier(
    n_estimators=100,
    max_depth=10,
    min_samples_split=5,
    min_samples_leaf=2,
    random_state=42,
)

print("🚀 Training the model...")
rf_model.fit(X_train, y_train)

cv_scores = cross_val_score(rf_model, X_train, y_train, cv=5, scoring="accuracy")

print("✅ Model training complete!")
print("\n📊 Cross-Validation Accuracy:")
print(f"   average: {cv_scores.mean():.4f}")
print(f"   standard deviation: {cv_scores.std():.4f}")
print(f"   Each Fold: {[f'{score:.4f}' for score in cv_scores]}")
```

![Train model](pasted-image-20260105232838.png)

### 2.4 Check feature importance

```python
# 4. Check Feature Importance
feature_importance = pd.DataFrame(
    {"feature": features, "importance": rf_model.feature_importances_}
).sort_values("importance", ascending=False)

print("📊 Feature Importance:")
print(feature_importance.to_string(index=False))
```

![Feature importance](pasted-image-20260105232858.png)

---

## 3. Export predictions & submit to Kaggle

### 3.1 Run predictions

```python
# 5. perform predictions
predictions = rf_model.predict(X_test)
prediction_proba = rf_model.predict_proba(X_test)

print("✅ prediction complete!")
print("\nSummary of prediction results:")
print(f"  - Total number of predictions: {len(predictions)}")
print(f"  - survival prediction: {sum(predictions == 1)}")
print(f"  - death prediction: {sum(predictions == 0)}")
print(f"  - survival rate: {sum(predictions == 1) / len(predictions) * 100:.2f}%")
```

![Predictions](pasted-image-20260105232939.png)

### 3.2 Create the submission dataframe

```python
# 6. Create a result dataframe
result_df = pd.DataFrame(
    {
        "PassengerId": test["PassengerId"],
        "Survived": predictions,
    }
)

print("📋 Sample prediction results (first 10):")
print(result_df.head(10))

print("\n📋 Sample prediction results (last 10):")
print(result_df.tail(10))
```

![Result dataframe](pasted-image-20260105232952.png)

### 3.3 Write the result back to a Foundry dataset (Export)

```python
# 7. Extract results
gender_submission = Dataset.get("gender_submission")
gender_submission.write_table(result_df)
```

![Write dataset](pasted-image-20260105233007.png)

### 3.4 Submit to Kaggle

1) Go back to the project home and click the exported `gender_submission` dataset.

![Select gender_submission](pasted-image-20260105233126.png)

2) **All actions** → **Download as CSV**

![Download as CSV](pasted-image-20260105233243.png)

3) On Kaggle, open **Submit Predictions**, upload the CSV, and submit.

![Submit Predictions](pasted-image-20260105233450.png)
![Upload CSV](pasted-image-20260105233424.png)

4) You should see a score (e.g., 0.76315).

![Score](pasted-image-20260105233505.png)

> From here, you can improve the score by iterating on features, model choice, and hyperparameters.

---

## 4. Visualize results in Workshop (Ontology)

### 4.1 Create an Ontology (connect the result dataset)

1) In **Application**, search for `ontology` → open **Ontology Manager**.

![Ontology Manager](pasted-image-20260106225045.png)

2) Click **New** → **Object type**.

![New object type](pasted-image-20260106225125.png)

3) Choose **Use existing datasource** → **Select datasource** → pick `gender_submission`.

![Select datasource](pasted-image-20260106225221.png)
![Select gender_submission](pasted-image-20260106225302.png)

4) Click **Next** → and **Next** again on Step 2.

![Next](pasted-image-20260106225332.png)
![Step2 Next](pasted-image-20260106225459.png)

5) Set **Primary key** and **Title** to `Passenger Id` → click **Create**.

![Primary key](pasted-image-20260106225533.png)
![Create ontology](pasted-image-20260106225608.png)

6) Click **Save**.

![Save](pasted-image-20260106225637.png)

### 4.2 Build a Workshop dashboard

1) In **Application**, search for `Workshop` → **Create new**.

![Create workshop](pasted-image-20260106225812.png)

2) Click **Save**.

![Save workshop](pasted-image-20260106225850.png)

3) On the left, click **Add widget** → choose **Object list**.

![Add widget - object list](pasted-image-20260106230215.png)

4) To set the input data, click **New object set variable**.

![New object set variable](pasted-image-20260106230233.png)

5) Click **Select starting object set**, search for `gender`, and select it.

![Select object set](pasted-image-20260106230351.png)

6) Click **Add property** → select `Survived`.

![Add property](pasted-image-20260106230510.png)
![Object list result](pasted-image-20260106230531.png)

Now you can see whether each passenger survived or not in the list.

7) On the right, click **Add widget** → choose **Chart: Pie**.

![Add widget - pie](pasted-image-20260106230720.png)

8) Set the input to `gender_submission` and set **GROUP BY** to `Survived` to visualize the survival/death ratio.

![Pie chart](pasted-image-20260106230907.png)

9) You now have a simple dashboard. If needed, you can also deploy it so others can access it.

![Dashboard](pasted-image-20260106231026.png)

---

## Wrap-up

This completes the Kaggle Titanic project workflow in Foundry—from **preprocessing → training/prediction → Kaggle submission → dashboard visualization**.  
In the next post, we can extend this by improving model performance (feature engineering, validation strategy, and model comparisons).


