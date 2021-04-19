---
layout: single
title: "#ML_01 Land Price Data Analysis"
---

# Land Price Data Analysis


### 1. Import Packages & Set options


```python
# # Basic
import pandas as pd
import numpy as np
from functools import reduce

import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns

# Plot setting
%matplotlib inline
plt.rc('font', family='Malgun Gothic') # set Korean font
mpl.rcParams['axes.unicode_minus'] = False # set minus font

# warnings.filterwarnings('ignore') # import warnings

# ipynb file loader
import import_ipynb # !pip install import_ipynb (if it isn't)
import DataController # load custom ipynb
```

### 2-1. Data Mining - Load Target Data


```python
guName = "종로구"
```


```python
# target Data : Price data
df_price = DataController.getPriceData(guName)
df_price = df_price.iloc[:, [0, 3]]
df_price = df_price.groupby('Year').mean()
df_price = df_price.reset_index()
df_price.columns=['Year', guName + '_price']
```


```python
# source Datas
df_population = DataController.getReshapedData("서울_인구_서울시 주민등록인구 (구별) 통계", guName)
df_population = df_population.iloc[:, [0, 3]]
df_population.iloc[:, 1] = df_population.iloc[:, 1].replace(",", "", regex=True)
df_population = df_population.astype(int)
df_population.columns=['Year', guName + '_population']

df_budget = DataController.getReshapedData("서울_일반행정_서울시 예산결산총괄 통계", guName)
df_budget = df_budget.iloc[:, [0, 2]]
df_budget.iloc[:, 1] = df_budget.iloc[:, 1].replace(",", "", regex=True)
df_budget = df_budget.astype(int)
df_budget.columns=['Year', guName + '_budget']
```

### 3.1 TF_Linear Regression sample


```python
# TF import
import pathlib
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

print(tf.__version__)
```

    2.3.0
    


```python
analysis_df_list = [df_price, df_budget, df_population]
analysis_df = reduce(lambda x, y: pd.merge(x, y, on = 'Year'), analysis_df_list)
analysis_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>종로구_price</th>
      <th>종로구_budget</th>
      <th>종로구_population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1996</td>
      <td>1.856732e+06</td>
      <td>140403</td>
      <td>199475</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1997</td>
      <td>1.943481e+06</td>
      <td>158784</td>
      <td>196611</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1998</td>
      <td>1.970869e+06</td>
      <td>153477</td>
      <td>190619</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1999</td>
      <td>1.765345e+06</td>
      <td>157747</td>
      <td>188865</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2000</td>
      <td>1.797014e+06</td>
      <td>180358</td>
      <td>188946</td>
    </tr>
  </tbody>
</table>
</div>




```python
def plot_lr(in_x, in_y):
    in_x = in_x.fillna(0)
    in_y = in_y.fillna(0)
    
    df_x = in_x.to_numpy().reshape(-1, 1)
    df_y = in_y.to_numpy().reshape(-1, 1)
    df_x_name = in_x.columns[0]
    df_y_name = in_y.columns[0]

    df_y_pred = get_LR(df_x, df_y)

    ######## Plot Result ################
    fig, axs = plt.subplots(1, 2, figsize=(10, 5))

    # [ ax_0 ]
    ax0_1 = axs[0]
    ax0_2 = axs[0].twinx()
    lns1 = ax0_1.plot(in_x, label = df_x_name, color='orange')
    lns2 = ax0_2.plot(in_y, label = df_y_name, color='green')

    # set frame
    lns = lns1+lns2
    labs = [l.get_label() for l in lns]
    axs[0].legend(lns, labs, loc='upper right')
    axs[0].grid()
    axs[0].set_title(df_x_name + ", " + df_y_name + " change graph")
    axs[0].set_xlabel("Year")
    axs[0].set_ylabel(r"" + df_x_name)

    # [ ax_1 ]
    scatter = axs[1].scatter(df_x, df_y, c = in_x.index);
    lr_line = axs[1].plot(df_x, df_y_pred)

    # set frame
    axs[1].legend(*scatter.legend_elements(), bbox_to_anchor=(1.05, 1), loc='upper left', title="Year")
    axs[1].grid()
    axs[1].set_title(df_x_name + ", " + df_y_name + " scatter graph")
    axs[1].set_xlabel(df_x_name)
    axs[1].set_ylabel(df_y_name)

    plt.show()
```


```python
from sklearn import datasets, linear_model
from sklearn.metrics import mean_squared_error, r2_score

def get_LR(df_x, df_y):
    ######## Linear Regression ################
    # Create linear regression object
    regr = linear_model.LinearRegression()

    # Train the model using the training sets
    regr.fit(df_x, df_y)

    # Make predictions using the testing set
    df_y_pred = regr.predict(df_x)
    
    return df_y_pred
```


```python
# test
plot_lr(analysis_df[["종로구_price"]], analysis_df[["종로구_population"]])
plot_lr(analysis_df[["종로구_price"]], analysis_df[["종로구_budget"]])
```

    
![png](\assets\images\2021-02-24\output_12_0.png)
    

    
![png](\assets\images\2021-02-24\output_12_1.png)
    
