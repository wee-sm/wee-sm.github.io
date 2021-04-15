---
layout: single
title: "#GIS_01 Geopandas"
---

## Subject: get the GIS data of South Korea and visualization

### Step 00. Import Related Modules


```python
import pandas as pd

# api for gis, shapefile
import geopandas
# import fiona
# import shapefile

from urllib.request import urlopen
from io import BytesIO
import os
from zipfile import ZipFile

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```

### Step 01. Get the Target Region's Shapefile

#### Find the target region code (in this case, Seoul)


```python
# get code-data url from git hub
region_code_url = "https://raw.githubusercontent.com/wee-sm/database/master/GIS/shapefile_korea/Region_Code_Souyh_Korea.txt"
region_codes = pd.read_csv(region_code_url, delimiter = "\t", header = 0, encoding = "ANSI")

# set target region: Seoul
Name_City = "서울특별시"

# get the specific code from url data
Code_Seoul = str(region_codes[region_codes.iloc[:, 1] == Name_City].iloc[:, 0][0])[:2]

print("Region Code for GIS data: " + Code_Seoul + "(" + Name_City + ")")
```

    Region Code for GIS data: 11(서울특별시)
    

#### Load shapefile data from url

##### 1. Download and unzip shapefile from url


```python
# function to unzip and save zipfile
def download_and_unzip(url, extract_to='.'):
    http_response = urlopen(url)
    zipfile = ZipFile(BytesIO(http_response.read()))
    zipfile.extractall(path=extract_to)
    
# target shapefile url (this case zipfile)
SIG_SHP_ZIP_url = "http://www.gisdeveloper.co.kr/download/admin_shp/SIG_202101.zip"
# target download directory
SIG_SHP_dir = "./temp/202101_SIG_SHP"

# save shapefile in directory and road shapefile to dataframe format
download_and_unzip(SIG_SHP_ZIP_url, SIG_SHP_dir)
```

##### 2. Read shapefile from directory


```python
# function to read shapefile data to dataframe from target directory
def read_shapefile_in_dir(dir="."):
    # get file list in directory
    files = os.listdir(dir)
    
    # get shapefile
    shapefile = list(filter(lambda shp: '.shp' in shp, files))[0]
    
    # return geopandas file by filepath
    return geopandas.read_file(dir + '/' + shapefile)

shapefile_korea = read_shapefile_in_dir(SIG_SHP_dir)
```

##### 3. Filtering with the target region code


```python
shapefile_Seoul = shapefile_korea[shapefile_korea['SIG_CD'].str[:2] == Code_Seoul]

# check the results
print("shapefile shape:", shapefile_Seoul.shape, type(shapefile_Seoul))
print()
print("[shapefile samples]")
print(shapefile_Seoul.iloc[:3, :2]) #sample row

shapefile_Seoul.plot()
```

    shapefile shape: (25, 4) <class 'geopandas.geodataframe.GeoDataFrame'>
    
    [shapefile samples]
        SIG_CD  SIG_ENG_NM
    140  11110   Jongno-gu
    141  11140     Jung-gu
    142  11170  Yongsan-gu
    




    <AxesSubplot:>




    
![png](/assets/images/2021-04-14/output_10_2.png)
    


### Step 02. Set the GeoDataFrame for visualization

##### 1. Create centroid point column


```python
shapefile_Seoul["point"] = shapefile_Seoul["geometry"].centroid
```

##### 2. Remove broken column (by encoding)

It could be fixed, but I'll remove it


```python
shapefile_Seoul = shapefile_Seoul.drop(columns='SIG_KOR_NM')
```


```python
shapefile_Seoul.head(3)
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
      <th>geometry</th>
      <th>point</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>140</th>
      <td>11110</td>
      <td>Jongno-gu</td>
      <td>POLYGON ((956615.453 1953567.199, 956621.579 1...</td>
      <td>POINT (953858.768 1955185.184)</td>
    </tr>
    <tr>
      <th>141</th>
      <td>11140</td>
      <td>Jung-gu</td>
      <td>POLYGON ((957890.386 1952616.746, 957909.908 1...</td>
      <td>POINT (955484.195 1951318.204)</td>
    </tr>
    <tr>
      <th>142</th>
      <td>11170</td>
      <td>Yongsan-gu</td>
      <td>POLYGON ((953115.761 1950834.084, 953114.206 1...</td>
      <td>POINT (954048.024 1948135.318)</td>
    </tr>
  </tbody>
</table>
</div>



### Step 03. Set the data to visualize

##### 1. read data

###### I'll use the opesity(BMI) data from Korea database


```python
# read csv file from url
BMI_data_url = 'https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/%EC%84%9C%EC%9A%B8_%EB%B3%B4%EA%B1%B4_%EC%84%9C%EC%9A%B8%EC%8B%9C%20%EB%B9%84%EB%A7%8C%EB%8F%84%20%ED%86%B5%EA%B3%84/2019.csv'
data_BMI_all = pd.read_csv(BMI_data_url)
data_BMI_all.head(3)
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
      <th>('Year', 'Year', 'Year')</th>
      <th>('sigName', 'sigName', 'sigName')</th>
      <th>('비만도', '비만도 분포', '비만(BMI≥25)')</th>
      <th>('비만도', '비만도 분포', '저체중(BMI&lt;18.5)')</th>
      <th>('비만도', '비만도 분포', '정상(18.5≤BMI＜25)')</th>
      <th>('비만도', '체질량지수(BMI)(평균)', '체질량지수(BMI)(평균)')</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019</td>
      <td>서울시</td>
      <td>31.8</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019</td>
      <td>종로구</td>
      <td>34.2</td>
      <td>5.3</td>
      <td>60.5</td>
      <td>23.8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>중구</td>
      <td>29.1</td>
      <td>5.3</td>
      <td>65.5</td>
      <td>23.4</td>
    </tr>
  </tbody>
</table>
</div>




```python
# drop row for '서울시' and take columns needed
data_BMI = data_BMI_all.iloc[1:, [1, -1]]

# rename columns
data_BMI = data_BMI.rename(columns={"('sigName', 'sigName', 'sigName')": "sigName", "('비만도', '체질량지수(BMI)(평균)', '체질량지수(BMI)(평균)')": "average_BMI"})

# set type
data_BMI["average_BMI"] = data_BMI["average_BMI"].astype(float)

data_BMI.head(3)
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
      <th>sigName</th>
      <th>average_BMI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>종로구</td>
      <td>23.8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>중구</td>
      <td>23.4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>용산구</td>
      <td>23.5</td>
    </tr>
  </tbody>
</table>
</div>



##### join sigCode column by sigName column


```python
# get dataframe about relationship between sigCode and sigName
Codes_SIG = region_codes[region_codes.iloc[:, 1].str.contains(Name_City)]
Codes_SIG = Codes_SIG[Codes_SIG.iloc[:, 1].str[-1] == '구']

Codes_SIG = Codes_SIG.iloc[:, [0, 1]]
Codes_SIG = Codes_SIG.rename(columns={"법정동코드": "sigCode", "법정동명": "sigName"})
Codes_SIG['sigCode'] = Codes_SIG['sigCode'].astype(str).str[:5]
Codes_SIG['sigName'] = Codes_SIG['sigName'].str.replace("서울특별시 ", "")
Codes_SIG.head(3)
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
      <th>sigCode</th>
      <th>sigName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>11110</td>
      <td>종로구</td>
    </tr>
    <tr>
      <th>94</th>
      <td>11140</td>
      <td>중구</td>
    </tr>
    <tr>
      <th>179</th>
      <td>11170</td>
      <td>용산구</td>
    </tr>
  </tbody>
</table>
</div>



##### merge relationship dataframe with 'BMI' dataframe and 'Geopandas' dataframe


```python
# step 1
data_BMI_merge = pd.merge(Codes_SIG, data_BMI, on="sigName")
# step 2
shapefile_Seoul_BMI = pd.merge(shapefile_Seoul, data_BMI_merge, left_on="SIG_CD", right_on="sigCode")

shapefile_Seoul_BMI.head(3)
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
      <th>geometry</th>
      <th>point</th>
      <th>sigCode</th>
      <th>sigName</th>
      <th>average_BMI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11110</td>
      <td>Jongno-gu</td>
      <td>POLYGON ((956615.453 1953567.199, 956621.579 1...</td>
      <td>POINT (953858.768 1955185.184)</td>
      <td>11110</td>
      <td>종로구</td>
      <td>23.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11140</td>
      <td>Jung-gu</td>
      <td>POLYGON ((957890.386 1952616.746, 957909.908 1...</td>
      <td>POINT (955484.195 1951318.204)</td>
      <td>11140</td>
      <td>중구</td>
      <td>23.4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11170</td>
      <td>Yongsan-gu</td>
      <td>POLYGON ((953115.761 1950834.084, 953114.206 1...</td>
      <td>POINT (954048.024 1948135.318)</td>
      <td>11170</td>
      <td>용산구</td>
      <td>23.5</td>
    </tr>
  </tbody>
</table>
</div>



### Step 04. Plot

##### Plot Geopandas Data with matplotlib


```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize  = (8, 8))


shapefile_Seoul_BMI.plot(column='average_BMI',
                         ax=ax,
                         legend=True,
                         legend_kwds={'label': "Average BMI",
                                      'orientation': "horizontal"})

ax.set_title("Average of BMI by Gu in Seoul", fontsize=15)
ax.set_axis_off()

plt.axis();
```


    
![png](/assets/images/2021-04-14/output_24_0.png)
    

