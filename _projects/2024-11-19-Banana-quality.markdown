---

title: Optimizing Banana Quality- A Predictive Model
description: Python
date:  2024-11-19 14:03:36 +0900

---

# Description  
## Optimizing Banana Quality: A Predictive Models  

This project involves building a Random Forest Classifier to predict and evaluate high-quality bananas using the Banana Quality Dataset. Below is a detailed breakdown of the steps taken in the model development process, along with insights derived during the analysis.  
- The dataset : [Dataset_from_Kaggle](https://www.kaggle.com/datasets/mrmars1010/banana-quality-dataset/data)  

---  

```python
import seaborn as sns
sns.pairplot(bs_cleaned)
```  

<img width="782" alt="Screenshot 2024-12-10 at 3 26 38 PM" src="https://github.com/user-attachments/assets/9855bc5f-6735-485a-93de-18dcac802945">  

The dataset contains various features related to banana quality, such as ripeness, sugar content, firmness, and environmental factors. 
An initial exploration using scatter plots revealed non-linear relationships between variables, making Random Forests an appropriate choice due to their capability to handle such complexity.  

### 1. Loading the dataset  

```python
import pandas as pd
import sklearn
import matplotlib.pyplot as plt
import numpy as np

bs = pd.read_csv("/Users/leesangeun/Downloads/banana_quality_dataset.csv")
bs.head()
```

<img width="987" alt="Screenshot 2024-11-20 at 10 27 59 PM" src="https://github.com/user-attachments/assets/8b90d448-55c6-4a40-9d84-3d379572afcb">  

### 2. Handling Missing values  

```python
bs.isna().sum()
bs.info()
```

<img width="216" alt="Screenshot 2024-11-20 at 10 29 01 PM" src="https://github.com/user-attachments/assets/4438997e-6404-45e9-bb7c-599fc388d7f5">  
<img width="423" alt="Screenshot 2024-11-20 at 10 29 45 PM" src="https://github.com/user-attachments/assets/23bd75c8-10aa-45c8-8c86-6ebc04cc1920">  

### 3. Encoding Categorical variables

```python
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()

bs['variety'] = le.fit_transform(bs['variety'])
bs['region'] = le.fit_transform(bs['region'])
bs['quality_category'] = le.fit_transform(bs['quality_category'])
```  

### 4. Feature Selection
- Identify Unused Variables
- Check for Redundancy  

```python
bs_cleaned = bs.drop(columns = ['sample_id' ,'quality_score','ripeness_category', 'tree_age_years', 'altitude_m', 'rainfall_mm', 'harvest_date' ])
bs_cleaned.info()
```  

<img width="338" alt="Screenshot 2024-12-10 at 3 08 27 PM" src="https://github.com/user-attachments/assets/65a91a53-fe0a-4286-ac8e-44e0d8656fd7">  

### 5. Outlier Detection

```python
import seaborn as sns

col_bs = [
     "ripeness_index", "sugar_content_brix",
    "firmness_kgf", "length_cm", "weight_g", "soil_nitrogen_ppm"
]

plt.figure(figsize=(14, 10))
for i, col in enumerate( col_bs, 1):
    plt.subplot(3, 3, i)
    sns.boxplot(y=bs_cleaned[col], color="darkgreen")  
    plt.title(f"Boxplot of {col}")

plt.tight_layout()
plt.show()
```  

<img width="778" alt="Screenshot 2024-12-10 at 3 10 44 PM" src="https://github.com/user-attachments/assets/448f24b4-d477-4664-89df-f9293d081a26">   

## Model Building  
### 6. Random Forest classifier

RandomForestClassifier : [sckit learn explanation](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)  

A Random Forest Classifier was implemented to predict the quality_category of bananas. The dataset was split into training and testing sets with an 80:20 ratio.  


```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor

x = bs_cleaned.drop(columns=[ 'quality_category']) 
y = bs_cleaned["quality_category"]

x_train, x_test , y_train, y_test = train_test_split(x, y, test_size=0.2, random_state=42)
print(x_train.shape, x_test.shape, y_train.shape, y_test.shape)
```  

**(800, 8) (200, 8) (800,) (200,)**  

The model trained and evaluated as follows:  

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score 

clf = RandomForestClassifier(n_estimators=100, max_depth=20,random_state=0)
clf.fit(x_train,y_train)

predict1 = clf.predict(x_test)
print(accuracy_score(y_test,predict1))
```
**- Accuracy score is 0.895.**  

### 7. Corss validation Results
```python
from sklearn.model_selection import cross_validate

scores = cross_validate(
    clf, x, y, cv=5, scoring=['accuracy', 'f1_weighted']
)

print("Accuracy score:", scores['test_accuracy'])
print("F1-score :", scores['test_f1_weighted'])
print("Average Accuracy score:", scores['test_accuracy'].mean())
print("Average F1-score:", scores['test_f1_weighted'].mean())
```

### 8. Result 
Accuracy score: [0.925 0.89  0.875 0.935 0.895]  
F1-score : [0.91546266 0.88079787 0.86371544 0.92769708 0.88211976]  
Average Accuracy score: 0.9039999999999999  
Average F1-score: 0.8939585619159265  