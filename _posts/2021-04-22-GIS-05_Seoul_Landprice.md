---
layout: single
title: "#GIS_05 Check Data and Shape File"
---

# [Data Mining] Land Price Data 


```python
import pandas as pd
import numpy as np
from geopy.extra.rate_limiter import RateLimiter
from geopy.geocoders import Nominatim
import time

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```


```python
path_landprice = "temp/20210422_landprice"
fileName_landprice = "공시지가_2020년.csv"

df_landprice = pd.read_csv(path_landprice + "/" + fileName_landprice, encoding="ANSI", low_memory=False)

print("[ Seoul Land Price Data ]")
print("Data Length: " + str(len(df_landprice)))
df_landprice.head(3)
```

    [ Seoul Land Price Data ]
    Data Length: 909979
    




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
      <th>시도명</th>
      <th>시군구명</th>
      <th>법정동명</th>
      <th>토지코드</th>
      <th>공시지가(원/㎡)</th>
      <th>시군구코드</th>
      <th>법정동코드</th>
      <th>필지구분코드</th>
      <th>필지구분명</th>
      <th>본번</th>
      <th>부번</th>
      <th>기준년도</th>
      <th>기준년월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울특별시</td>
      <td>성북구</td>
      <td>하월곡동</td>
      <td>1129013600100820125</td>
      <td>851400</td>
      <td>11290</td>
      <td>13600</td>
      <td>1</td>
      <td>토지</td>
      <td>0082</td>
      <td>0125</td>
      <td>2020</td>
      <td>2020-01-01</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>성북구</td>
      <td>하월곡동</td>
      <td>1129013600200050003</td>
      <td>1452000</td>
      <td>11290</td>
      <td>13600</td>
      <td>2</td>
      <td>임야</td>
      <td>0005</td>
      <td>0003</td>
      <td>2020</td>
      <td>2020-01-01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울특별시</td>
      <td>성북구</td>
      <td>하월곡동</td>
      <td>1129013600101040169</td>
      <td>1801000</td>
      <td>11290</td>
      <td>13600</td>
      <td>1</td>
      <td>토지</td>
      <td>0104</td>
      <td>0169</td>
      <td>2020</td>
      <td>2020-01-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_landprice_emd = df_landprice.groupby(by=["시도명", "시군구명", "법정동명", "시군구코드", "법정동코드", "필지구분코드"]).mean()["공시지가(원/㎡)"].reset_index()
df_landprice_emd = df_landprice_emd[df_landprice_emd.필지구분코드 == 1]

print("[ Seoul Land Price Data By EMD ]")
print("Data Length: " + str(len(df_landprice_emd)))
print("=== Data Type ===")
print(str(df_landprice_emd.dtypes))
print("=================")
df_landprice_emd.head(3)
```

    [ Seoul Land Price Data By EMD ]
    Data Length: 467
    === Data Type ===
    시도명           object
    시군구명          object
    법정동명          object
    시군구코드          int64
    법정동코드          int64
    필지구분코드         int64
    공시지가(원/㎡)    float64
    dtype: object
    =================
    




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
      <th>시도명</th>
      <th>시군구명</th>
      <th>법정동명</th>
      <th>시군구코드</th>
      <th>법정동코드</th>
      <th>필지구분코드</th>
      <th>공시지가(원/㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>개포동</td>
      <td>11680</td>
      <td>10300</td>
      <td>1</td>
      <td>4.609040e+06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현동</td>
      <td>11680</td>
      <td>10800</td>
      <td>1</td>
      <td>9.614150e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>대치동</td>
      <td>11680</td>
      <td>10600</td>
      <td>1</td>
      <td>1.080679e+07</td>
    </tr>
  </tbody>
</table>
</div>




```python
data = {'EMD_CD': (df_landprice_emd["시군구코드"]*100000 + df_landprice_emd["법정동코드"])/100, 'landprice': df_landprice_emd["공시지가(원/㎡)"]}
df_landprice_concised = pd.DataFrame(data=data)
df_landprice_concised = df_landprice_concised.sort_values(by=['EMD_CD']).astype('int32').reset_index(drop = True)


print("[ Concised Data ]")
print("Data Length: " + str(len(df_landprice_concised)))
print("=== Data Type ===")
print(str(df_landprice_concised.dtypes))
print("=================")
df_landprice_concised.head(3)
```

    [ Concised Data ]
    Data Length: 467
    === Data Type ===
    EMD_CD       int32
    landprice    int32
    dtype: object
    =================
    




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
      <th>EMD_CD</th>
      <th>landprice</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11110101</td>
      <td>2942890</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11110102</td>
      <td>3013425</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11110103</td>
      <td>3439445</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df_landprice_concised.to_csv('seoul_landprice_emd.csv')
```


```python
import os
import geopandas
```


```python
# function to read shapefile data to dataframe from target directory
def read_shapefile_in_dir(dir="."):
    # get file list in directory
    files = os.listdir(dir)
    
    # get shapefile
    shapefile = list(filter(lambda shp: '.shp' in shp, files))[0]
    
    # return geopandas file by filepath
    return geopandas.read_file(dir + '/' + shapefile)

SIG_SHP_dir= "./temp/LSMD_CONT_LDREG_서울"
shapefile_korea = read_shapefile_in_dir(SIG_SHP_dir)
```


```python
shapefile_korea.plot()
```




    <AxesSubplot:>




        
![png](\assets\images\2021-04-22\output_8_1.png)    



```python
shapefile_korea.head()
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
      <th>PNU</th>
      <th>JIBUN</th>
      <th>BCHK</th>
      <th>SGG_OID</th>
      <th>COL_ADM_SE</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1111011000100770013</td>
      <td>77-13´ë</td>
      <td>1</td>
      <td>161542</td>
      <td>11110</td>
      <td>POLYGON ((197116.811 453133.392, 197125.491 45...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1111011000100770014</td>
      <td>77-14´ë</td>
      <td>1</td>
      <td>161543</td>
      <td>11110</td>
      <td>POLYGON ((197109.234 453133.127, 197116.811 45...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1111017400105950099</td>
      <td>595-99 ´ë</td>
      <td>1</td>
      <td>161544</td>
      <td>11110</td>
      <td>POLYGON ((200936.723 452669.842, 200936.392 45...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1111017400105950247</td>
      <td>595-247 ´ë</td>
      <td>1</td>
      <td>161545</td>
      <td>11110</td>
      <td>POLYGON ((200916.229 452635.784, 200911.963 45...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1111011000100020001</td>
      <td>2-1´ë</td>
      <td>1</td>
      <td>161547</td>
      <td>11110</td>
      <td>POLYGON ((197195.436 453130.981, 197196.816 45...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_landprice = df_landprice[["토지코드", "공시지가(원/㎡)"]].rename(columns={"토지코드":"PNU", "공시지가(원/㎡)":"price"}).head()
df_landprice
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
      <th>PNU</th>
      <th>price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1129013600100820125</td>
      <td>851400</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1129013600200050003</td>
      <td>1452000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1129013600101040169</td>
      <td>1801000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1129013600100960051</td>
      <td>1006000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1129013600100960119</td>
      <td>871200</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_landprice["PNU"] = df_landprice["PNU"].astype(str)
shapefile_korea["PNU"] = shapefile_korea["PNU"].astype(str)
df_landprice.dtypes
```




    PNU      object
    price     int64
    dtype: object




```python
shapefile_korea.dtypes
```




    PNU             object
    JIBUN           object
    BCHK            object
    SGG_OID          int64
    COL_ADM_SE      object
    geometry      geometry
    dtype: object



## => Join at ArcGIS
