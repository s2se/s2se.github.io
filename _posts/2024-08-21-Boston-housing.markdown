---
layout: post
title:  "[Project] Analysis of Boston housing dataset"
date:   2024-08-21 21:03:36 +0900
categories: [Project, Python]

---

## Description  

+ The purpose
    - Develop a model to predict house prices using the Boston Housing dataset. 
    - Consider which factors have the most significant impact based on various elements.  

```python
import numpy as np 
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans
```

## Explore the data
Start by exploring the Boston Housing dataset.  
Brought the dataset from Kaggle  

[Kaggle](https://www.kaggle.com/datasets/zedekiaobuya/housingdatacsv)  
```python
ds = pd.read_csv("HousingData.csv")
ds.head()
# Examine the top 5 rows.
```
![dataset](https://github.com/user-attachments/assets/42f9529e-8a6f-4112-873b-3b77e0cc7a54)


These are the codes for exploring the loaded data.  
```python
ds.shape
# Print the number of row and columns
ds.info()
# Print the description of data
ds.describe()
# Print the summary statistics
ds.unique()
# Print the number of unique value
```  
(506, 14)  
row: 506 / column: 14  



## Processing the data  

#### 1. Handling missing values  
- Removing null values  
- Fill with its mean values  



```python
ds.isna().sum()
```
![nullcheck](https://github.com/user-attachments/assets/b43a2c9a-3037-4527-93ff-6a95f04673e5)
```python
ds['CRIM'].fillna(ds['CRIM'].mean(), inplace=True)
ds['ZN'].fillna(ds['ZN'].mean(), inplace=True)
ds['INDUS'].fillna(ds['INDUS'].mean(), inplace=True)
ds['CHAS'].fillna(ds['CHAS'].mean(), inplace=True)
ds['AGE'].fillna(ds['AGE'].mean(), inplace=True)
ds['LSTAT'].fillna(ds['LSTAT'].mean(), inplace=True)
```
Replace null value with its mean values.  
After filled it up with mean values : 506 row, 14 columns  



#### 2. Explore the outliers  
1) Visualization  
    1. From the above graphs, these can be observed that there are many outliers in the boxplots.
    2. To follow normalization, histograms should simultaneously exhibit a bell-shaped curve, but some variables do not follow this pattern.
    3. right skewed: CRIM, ZN, CHAS, DIS, RAD
    4. Left skewed: AGE, B  

```python
fig, ax = plt.subplots(nrows=2, ncols=7, figsize=(20, 8))

for i, column in enumerate(ds.columns):
    index = ax[i // 7, i % 7]   
    index.boxplot(ds[column], notch=True, whis=2.5)
    index.set_xlabel(f"{column}")

plt.tight_layout()
plt.show()
```  
![visu1](https://github.com/user-attachments/assets/b1fbb916-1e32-495f-b67e-72d866c334d8)

```python
fig, ax = plt.subplots(nrows=2, ncols=7, figsize=(20, 8))

for i, column in enumerate(ds.columns):
    index = ax[i // 7, i % 7]  
    index.hist(ds[column], bins=10)  
    index.set_xlabel(f"{column}")

plt.tight_layout()
plt.show()
```  
![visu2](https://github.com/user-attachments/assets/faf4be7a-056a-4c11-a6cc-a51c1c7b11c0)

  
1. Remove the dependenct variable: MEDA
2. Remove Categorical Feature: RAD  
```python  
medv = ds['MEDV']
ds_cluster = ds.drop(['MEDV', 'RAD'], axis=1)  
```  

  
## Check the correlation graph
```python  
correlation_matrix = ds.corr(numeric_only=True)
plt.figure(figsize=(30, 20))

mask = np.triu(np.ones_like(correlation_matrix, dtype=bool))
sns.heatmap(correlation_matrix, annot=True, cmap='RdYlBu_r', center=0, mask=mask)

plt.show()
```  

![correlation](https://github.com/user-attachments/assets/039d2ee1-c4e0-495d-bcdd-56535839d1ea)  

1. The highest value is RAD and TAX.( 0.91)  
2. With MEDV, RM and LSTAT is highly correlated.  
3. If MEDV decreases, LSTAT will increase about 72%.  
4. If MEDV increases, RM will increase about 70%.  
5. CHAS and MEDV has the smallest correlation.    




**Remove the variables RAD and TAX from model, as they have a correlation of 0.91.** 
```python 
sns.regplot(y=ds["RAD"], x=ds["TAX"])
```  
![RadnTax](https://github.com/user-attachments/assets/801651ae-5f6c-469c-a8fe-b73d1463c194)
<br/><br/>

## Linear Regression
```python 
from sklearn.model_selection import train_test_split
from sklearn import linear_model
from sklearn.metrics import r2_score,mean_squared_error
from sklearn.feature_selection import RFE

reg = linear_model.LinearRegression()

train, test = train_test_split(ds, test_size=0.2, random_state=142)
print(f"train shape:" , train.shape)
print(f"train shape:" , test.shape)
```  
train shape: (404, 14)  
train shape: (102, 14)  
```python 
X_train = train.drop(['MEDV','RAD','TAX'], axis=1)
y_train = train['MEDV']
X_test = test.drop(['MEDV','RAD','TAX'], axis=1)
y_test = test['MEDV']
print("X_train shape: ", X_train.shape)
print("y_train shape: ", y_train.shape)
print("X_test shape: ", X_test.shape)
print("y_test shape: ", y_test.shape)
```  
X_train shape:  (404, 11)  
y_train shape:  (404,)  
X_test shape:  (102, 11)  
y_test shape:  (102,)  



## Result  

```python 
reg.fit(X_train, y_train)
hat = reg.predict(X_test)
mse = mean_squared_error(y_test, hat)
rmse =np.sqrt(mse)

result = pd.DataFrame({'MSE': [mse], 'RMSE': [rmse]}, index=['mymodel'])
print(result)
```  

|         | MSE       | RMSE     |
|---------|-----------|----------|
| mymodel | 21.846483 | 4.674022 |





```python 
plt.scatter(y_test, hat, alpha=0.4)
plt.xlabel("Actual Price")
plt.ylabel("Predicted Price")
plt.title("LINEAR REGRESSION")
plt.show()
```  
![regression](https://github.com/user-attachments/assets/4db60e76-dee3-4134-9d01-f1bf3bb2d8cf)  



```python 
reg.coef_
```  
array([-3.48612839e-02,  2.99658699e-02, -9.19095137e-02,  3.32939857e+00,
       -1.26065897e+01,  4.30909169e+00, -7.35528335e-03, -1.44981983e+00,
       -8.77489529e-01,  7.68527041e-03, -5.42291786e-01])  




