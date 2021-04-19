---
layout: single
title: "#ML_02 GIS data analysis 01"
---


### Correlation between the rate of admission to universities and the number of private institutes and teachers per student by region

## Step 01. Load and Set Data


```python
import pandas as pd
from functools import reduce
```


```python
# Set parameter
target_year = 2019 # after 2012
analysis_year = target_year - 1
sigName = "성동구"
```


```python
# get SIG Code
url_SIG_Code = "https://raw.githubusercontent.com/wee-sm/database/master/GIS/shapefile_korea/Seoul_SIG_Code"
SIG_Code = pd.read_csv(url_SIG_Code, delimiter = ",")

############################ LABEL ############################
url_enrollment_rate = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210418_schools_after2011/%EC%84%9C%EC%9A%B8%EC%8B%9C%20%EC%83%81%EA%B8%89%ED%95%99%EA%B5%90%20%EC%A7%84%ED%95%99%EB%A5%A0%20%ED%86%B5%EA%B3%84.txt"
data_univ = pd.read_csv(url_enrollment_rate, delimiter = "\t", header = [0, 1, 2])

data_univ = data_univ.iloc[:, :7]
data_univ.columns = ["year", "sigName", "student_all", "student_goto_university", "student_goto_university_rate", "student_goto_job", "student_goto_etc"]
data_univ["sigName"] = data_univ["sigName"].str.replace("합계", "서울특별시")

data_univ = data_univ.merge(SIG_Code, on="sigName")

print("University enrollment rate for ["
      + str(target_year) + " | " + sigName + "] :"
      + str(data_univ[(data_univ.year == target_year) & (data_univ.sigName == sigName)]["student_goto_university_rate"].values[0])
      + "%")

############################ DATA 1 ############################
url_teacher_per_student_rate = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210418_schools_after2011/%EC%84%9C%EC%9A%B8%EC%8B%9C%20%EA%B5%90%EC%9B%90%201%EC%9D%B8%EB%8B%B9%20%ED%95%99%EC%83%9D%EC%88%98(%EA%B5%AC%EB%B3%84)%20%ED%86%B5%EA%B3%84.txt"
data_tps = pd.read_csv(url_teacher_per_student_rate, delimiter = "\t", header = [0, 1, 2])

data_tps = data_tps.iloc[:, [0, 1, -3, -2, -1]]
data_tps.columns = ["year", "sigName", "student_all", "teacher_all", "tps"]

data_tps = data_tps.loc[data_tps['year'] == analysis_year]
data_tps["sigName"] = data_tps["sigName"].str.replace("합계", "서울특별시")

data_tps = data_tps.merge(SIG_Code, on="sigName")

print("Teacher Per student rate for ["
      + str(target_year) + " | " + sigName + "] :"
      + str(data_tps[(data_tps.year == analysis_year) & (data_tps.sigName == sigName)]["tps"].values[0])
      + " students per one Teacher")

############################ DATA 2 ############################
url_academy_per_student_rate = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210418_schools_after2011/%EC%84%9C%EC%9A%B8%EC%8B%9C%20%ED%95%99%EC%83%9D%201%EB%A7%8C%EB%AA%85%EB%8B%B9%20%EC%82%AC%EC%84%A4%ED%95%99%EC%9B%90%EC%88%98%20%ED%86%B5%EA%B3%84.txt"
data_aps = pd.read_csv(url_academy_per_student_rate, delimiter = "\t", header = 0)

data_aps.columns = ["year", "sigName", "student_all", "academy_all", "aps"]
data_aps["sigName"] = data_aps["sigName"].str.replace("합계", "서울특별시")

data_aps = data_aps.merge(SIG_Code, on="sigName")

print("Academy Per student rate for ["
      + str(target_year) + " | " + sigName + "] :"
      + str(data_aps[(data_aps.year == analysis_year) & (data_aps.sigName == sigName)]["aps"].values[0])
      + " academy per 100,000 students")
```

    University enrollment rate for [2019 | 성동구] :50.9%
    Teacher Per student rate for [2019 | 성동구] :10.1 students per one Teacher
    Academy Per student rate for [2019 | 성동구] :136.0 academy per 100,000 students
    


```python
label = data_univ[data_univ.year == target_year][["SIG_CD", "student_goto_university_rate"]]
label.columns = ["SIG_CD", "label"]

datas = []
datas.append(label)
datas.append(data_aps[data_aps.year == analysis_year][["SIG_CD", "aps"]])
datas.append(data_tps[data_tps.year == analysis_year][["SIG_CD", "tps"]])

merged_df = reduce(lambda  left, right: pd.merge(left, right, on=['SIG_CD'], how='outer'), datas)
merged_df.head(3)
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
      <th>SIG_CD</th>
      <th>label</th>
      <th>aps</th>
      <th>tps</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11000</td>
      <td>55.3</td>
      <td>156.3</td>
      <td>11.9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11110</td>
      <td>54.7</td>
      <td>148.4</td>
      <td>11.3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11140</td>
      <td>43.7</td>
      <td>80.7</td>
      <td>10.9</td>
    </tr>
  </tbody>
</table>
</div>



## Step 02. Load and Merge shapefile


```python
import geopandas
from urllib.request import urlopen
from io import BytesIO
import os
from zipfile import ZipFile
import matplotlib.pyplot as plt
import matplotlib as mpl

# Plot setting
%matplotlib inline
plt.rc('font', family='Malgun Gothic') # set Korean font
mpl.rcParams['axes.unicode_minus'] = False # set minus font
```


```python
# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'

# function to unzip and save zipfile
def download_and_unzip(url, extract_to='.'):
    http_response = urlopen(url)
    zipfile = ZipFile(BytesIO(http_response.read()))
    zipfile.extractall(path=extract_to)
    
SIG_SHP_ZIP_url = "http://www.gisdeveloper.co.kr/download/admin_shp/SIG_202101.zip"
SIG_SHP_dir = "./temp/202101_SIG_SHP"

download_and_unzip(SIG_SHP_ZIP_url, SIG_SHP_dir)

# function to read shp file package
def read_shapefile_in_dir(dir="."):
    files = os.listdir(dir)
    shapefile = list(filter(lambda shp: '.shp' in shp, files))[0]
    return geopandas.read_file(dir + '/' + shapefile, encoding = 'ANSI')

shapefile_korea = read_shapefile_in_dir(SIG_SHP_dir)

Code_Seoul = str(SIG_Code[SIG_Code.sigName == "서울특별시"]['SIG_CD'].values[0])[:2] # 11
shapefile_Seoul = shapefile_korea[shapefile_korea['SIG_CD'].str[:2] == Code_Seoul]
shapefile_Seoul['coords'] = shapefile_Seoul['geometry'].apply(lambda x: x.representative_point().coords[:])
shapefile_Seoul['coords'] = [coords[0] for coords in shapefile_Seoul['coords']]
shapefile_Seoul = shapefile_Seoul.astype({'SIG_CD': 'int64'})

merged_shapefile = pd.merge(shapefile_Seoul, merged_df, on=['SIG_CD'], how='inner')
merged_shapefile.head(3)
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
      <th>SIG_CD</th>
      <th>SIG_ENG_NM</th>
      <th>SIG_KOR_NM</th>
      <th>geometry</th>
      <th>coords</th>
      <th>label</th>
      <th>aps</th>
      <th>tps</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11110</td>
      <td>Jongno-gu</td>
      <td>종로구</td>
      <td>POLYGON ((956615.453 1953567.199, 956621.579 1...</td>
      <td>(952930.8679692361, 1955652.3804498552)</td>
      <td>54.7</td>
      <td>148.4</td>
      <td>11.3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11140</td>
      <td>Jung-gu</td>
      <td>중구</td>
      <td>POLYGON ((957890.386 1952616.746, 957909.908 1...</td>
      <td>(955219.2141431344, 1951067.0058507507)</td>
      <td>43.7</td>
      <td>80.7</td>
      <td>10.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11170</td>
      <td>Yongsan-gu</td>
      <td>용산구</td>
      <td>POLYGON ((953115.761 1950834.084, 953114.206 1...</td>
      <td>(954097.8285944711, 1948107.993268157)</td>
      <td>55.9</td>
      <td>88.5</td>
      <td>10.7</td>
    </tr>
  </tbody>
</table>
</div>



## Step 03. Visualization Data on map


```python
fig, axs = plt.subplots(1, 3, figsize  = (16, 8))

# FIG 1
axs[0].set_title("University enrollment rate per student", fontsize=15)
merged_shapefile.plot(column='label',
                         ax=axs[0],
                         legend=True,
                         legend_kwds={'label': "Label",
                                      'orientation': "horizontal"})
# FIG 2
axs[1].set_title("Nomber of Academy per student", fontsize=15)
merged_shapefile.plot(column='aps',
                         ax=axs[1],
                         legend=True,
                         legend_kwds={'label': "aps",
                                      'orientation': "horizontal"})
# FIG 3
axs[2].set_title("Nomber of Teacher per student", fontsize=15)
merged_shapefile.plot(column='tps',
                         ax=axs[2],
                         legend=True,
                         legend_kwds={'label': "tps",
                                      'orientation': "horizontal"})
for ax in axs:
    ax.set_axis_off()
    for idx, row in merged_shapefile.iterrows():
        ax.annotate(row['SIG_KOR_NM'], xy=row['coords'], horizontalalignment='center', color='white')
```


    
![png](\assets\images\2021-04-18\output_10_0.png)
    


## Step 04. Data Anlysis..?
### 4.1 Normalization


```python
from sklearn import preprocessing
min_max_scaler = preprocessing.MinMaxScaler()

df = merged_df.drop(columns = 'SIG_CD')

x = df.values # returns a numpy array
x_scaled = min_max_scaler.fit_transform(x)

df = pd.DataFrame(x_scaled)
df.plot()
```




    <AxesSubplot:>




    
![png](\assets\images\2021-04-18\output_12_1.png)
    


### 4.2 Linear Regression


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
    
    # [ ax_1 ]
    scatter = axs[1].scatter(df_x, df_y, c = in_x.index);
    lr_line = axs[1].plot(df_x, df_y_pred)

    plt.show()
    
# test
print("The Relationship between Enrollment rate with Number of Academy")
plot_lr(df[[0]], df[[1]])
```

    The Relationship between Enrollment rate with Number of Academy
    


    
![png](\assets\images\2021-04-18\output_14_1.png)
    

