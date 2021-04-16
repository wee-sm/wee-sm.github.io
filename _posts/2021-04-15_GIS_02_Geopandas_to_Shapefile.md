---
layout: single
title: "#GIS_02 Geopandas to Shapefile"
---

## Subject: Open Geopandas Dataframe at ArcGIS Pro

### Step 01. Make Geopandas Dataframe for Save


```python
import pandas as pd
from functools import reduce

import geopandas
from urllib.request import urlopen
from io import BytesIO
import os
from zipfile import ZipFile
import matplotlib.pyplot as plt

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```


```python
# GET GEOPANDAS DATAFRAME FROM SHAPEFILE
region_code_url = "https://raw.githubusercontent.com/wee-sm/database/master/GIS/shapefile_korea/Region_Code_Souyh_Korea.txt"
region_codes = pd.read_csv(region_code_url, delimiter = "\t", header = 0, encoding = "ANSI")

Name_City = "서울특별시"
Code_Seoul = str(region_codes[region_codes.iloc[:, 1] == Name_City].iloc[:, 0][0])[:2]

# function to unzip and save zipfile
def download_and_unzip(url, extract_to='.'):
    http_response = urlopen(url)
    zipfile = ZipFile(BytesIO(http_response.read()))
    zipfile.extractall(path=extract_to)
    
SIG_SHP_ZIP_url = "http://www.gisdeveloper.co.kr/download/admin_shp/SIG_202101.zip"
SIG_SHP_dir = "./temp/202101_SIG_SHP"

download_and_unzip(SIG_SHP_ZIP_url, SIG_SHP_dir)

def read_shapefile_in_dir(dir="."):
    files = os.listdir(dir)
    shapefile = list(filter(lambda shp: '.shp' in shp, files))[0]
    return geopandas.read_file(dir + '/' + shapefile)

shapefile_korea = read_shapefile_in_dir(SIG_SHP_dir)
shapefile_Seoul = shapefile_korea[shapefile_korea['SIG_CD'].str[:2] == Code_Seoul]
shapefile_Seoul["point"] = shapefile_Seoul["geometry"].centroid
shapefile_Seoul['coords'] = shapefile_Seoul['geometry'].apply(lambda x: x.representative_point().coords[:])
shapefile_Seoul['coords'] = [coords[0] for coords in shapefile_Seoul['coords']]
shapefile_Seoul = shapefile_Seoul.drop(columns='SIG_KOR_NM')

# shapefile_Seoul.plot()
```


```python
# SIG CODE DATAFRAME
Codes_SIG = region_codes[region_codes.iloc[:, 1].str.contains(Name_City)]
Codes_SIG = Codes_SIG[Codes_SIG.iloc[:, 1].str[-1] == '구']

Codes_SIG = Codes_SIG.iloc[:, [0, 1]]
Codes_SIG = Codes_SIG.rename(columns={"법정동코드": "SIG_CD", "법정동명": "sigName"})
Codes_SIG['SIG_CD'] = Codes_SIG['SIG_CD'].astype(str).str[:5]
Codes_SIG['sigName'] = Codes_SIG['sigName'].str.replace("서울특별시 ", "")

####################### READ DATA ############################
# BMI DATA
BMI_data_url = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/%EC%84%9C%EC%9A%B8_%EB%B3%B4%EA%B1%B4_%EC%84%9C%EC%9A%B8%EC%8B%9C%20%EB%B9%84%EB%A7%8C%EB%8F%84%20%ED%86%B5%EA%B3%84/2019.csv"

data_BMI_all = pd.read_csv(BMI_data_url)
data_BMI = data_BMI_all.iloc[:, [1, -1]]
data_BMI = data_BMI.set_axis(['sigName', 'BMI_avg'], axis='columns', inplace=False)

data_BMI_merged = pd.merge(Codes_SIG, data_BMI, on="sigName").drop(columns=['sigName'])
data_BMI_merged["BMI_avg"] = data_BMI_merged["BMI_avg"].astype(float)
# print(data_BMI_merged.head(3))

# Cultural Space DATA
CS_data_url = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/%EC%84%9C%EC%9A%B8_%EB%AC%B8%ED%99%94%EA%B4%80%EA%B4%91_%EC%84%9C%EC%9A%B8%EC%8B%9C%20%EB%AC%B8%ED%99%94%EA%B3%B5%EA%B0%84%20(%EA%B3%B5%EC%97%B0%EC%9E%A5)%20%ED%86%B5%EA%B3%84/2019.csv"
data_CS_all = pd.read_csv(CS_data_url)
data_CS = data_CS_all.iloc[:, 1:]
data_CS = data_CS.set_axis(['sigName', 'sc_sum', 'sc_public', 'sc_private', 'sc_large', 'sc_mid', 'sc_small'], axis='columns', inplace=False)
data_CS.head(3)

data_CS_merged = pd.merge(Codes_SIG, data_CS, on="sigName").drop(columns=['sigName'])
data_CS_merged = data_CS_merged.replace("-", 0)
data_CS_merged.iloc[:, 1:] = data_CS_merged.iloc[:, 1:].astype(int)
# print(data_CS_merged.head(3))

####################### SET DATA ############################
data_frames = [shapefile_Seoul]
data_frames.append(data_BMI_merged)
data_frames.append(data_CS_merged)

####################### MERGE DATA ############################
shapefile_Seoul_merged = reduce(lambda  left,right: pd.merge(left, right, on=['SIG_CD'], how='outer'), data_frames)

shapefile_Seoul_BMI = shapefile_Seoul.merge(data_BMI_merged)
shapefile_Seoul_CS = shapefile_Seoul.merge(data_CS_merged)
```


```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize  = (16, 8))

# FIG 1
ax1.set_title("Average of BMI by Gu in Seoul", fontsize=15)
shapefile_Seoul_BMI.plot(column='BMI_avg',
                         ax=ax1,
                         legend=True,
                         legend_kwds={'label': "Average BMI",
                                      'orientation': "horizontal"})

ax1.set_axis_off()
for idx, row in shapefile_Seoul_BMI.iterrows():
    ax1.annotate(row['SIG_ENG_NM'], xy=row['coords'], horizontalalignment='center')

# FIG 2
ax2.set_title("Cultural Space by Gu in Seoul", fontsize=15)
shapefile_Seoul_CS.plot(column='sc_sum',
                         ax=ax2,
                         legend=True,
                         legend_kwds={'label': "Sum of Cultural Space",
                                      'orientation': "horizontal"})

ax2.set_axis_off()
for idx, row in shapefile_Seoul_CS.iterrows():
    ax2.annotate(row['SIG_ENG_NM'], xy=row['coords'], horizontalalignment='center', color='white')
```


    
![png](\assets\images\2021-04-15\output_5_0.png)
    


### Step 02. Save Geopandas Dataframe as shapefile


```python
Save_filePath = "./temp/20210415_geopandas/"
Save_fileName_BMI = "seoul_BIM.shp"
Save_fileName_CS = "seoul_CS.shp"
    
### point and coords data occure error
shapefile_Seoul_BMI.drop(columns=['point', 'coords']).to_file(Save_filePath + Save_fileName_BMI)
shapefile_Seoul_CS.drop(columns=['point', 'coords']).to_file(Save_filePath + Save_fileName_CS)
```



### Step 03. Load shapefile on ArcGIS

![png](\assets\images\2021-04-15\arhgis.png)
