---
layout: single
title: "#GIS_03 Address to Latitude and Longitude"
---

# Geocode

### Subject: '지하철주소' 데이터를 '위도/경도' 데이터로 변환


```python
# !pip install geopandas
# !pip install geopy

import pandas as pd
from geopy.extra.rate_limiter import RateLimiter
from geopy.geocoders import Nominatim

locator = Nominatim(user_agent="myGeocoder")
```

### ▶ 데이터 로드


```python
subway_address_url = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210419_subway/subway/%EC%84%9C%EC%9A%B8%EA%B5%90%ED%86%B5%EA%B3%B5%EC%82%AC_%EC%97%AD%EC%A3%BC%EC%86%8C%20%EB%B0%8F%20%EC%A0%84%ED%99%94%EB%B2%88%ED%98%B8_20200715.csv"
subway_address_df = pd.read_csv(subway_address_url)
```


```python
df = subway_address_df[['역번호', '호선', '역명', '역주소']]
df.head(2)
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
      <th>역번호</th>
      <th>호선</th>
      <th>역명</th>
      <th>역주소</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>150</td>
      <td>1</td>
      <td>서울역</td>
      <td>서울특별시 중구 세종대로 지하 2 (남대문로 5가)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>151</td>
      <td>1</td>
      <td>시청</td>
      <td>서울특별시 중구 세종대로 지하 101 (정동)</td>
    </tr>
  </tbody>
</table>
</div>



### ▶ goecode Test


```python
print("====== test 01 ======")
address_a = df.loc[0, '역주소']
location_a = locator.geocode(address_a)
print("address: " + address_a)
print("location: " + str(location_a))
# print("Latitude = {}, Longitude = {}".format(location_a.latitude, location_a.longitude))
```

    ====== test 01 ======
    address: 서울특별시 중구 세종대로 지하 2 (남대문로 5가)
    location: None
    


```python
print("====== test 02 ======")
address_b = address_a.split('(', 1)[0]
location_b = locator.geocode(address_b)
print("address: " + address_b)
print("location: " + str(location_b))
print("Latitude = {}, Longitude = {}".format(location_b.latitude, location_b.longitude))
```

    ====== test 02 ======
    address: 서울특별시 중구 세종대로 지하 2 
    location: 세종대로, 소공동, 중구, 서울, 노원구, 04512, 대한민국
    Latitude = 37.5604063, Longitude = 126.9753256
    


```python
print("====== test 03 ======")
address_c = df.loc[0, '역명']
location_c = locator.geocode(address_c)
print("address: " + address_c)
print("location: " + str(location_c))
print("Latitude = {}, Longitude = {}".format(location_c.latitude, location_c.longitude))
```

    ====== test 03 ======
    address: 서울역
    location: 서울역, 한강대로, Namyeong-dong, 용산구, 서울, 04320, 대한민국
    Latitude = 37.5534443, Longitude = 126.9697112
    

### Test 02, 03으로 진행

### [1차 시도] 주소를 위도/경도로 변환 


```python
# Try 01

# df 주소의 '(...)'를 모두 없애기
df['address'] = df['역주소'].str.split('(', 1).str[0]

# 1 - conveneint function to delay between geocoding calls
geocode = RateLimiter(locator.geocode, min_delay_seconds=0.1)

# 2- - create location column
df['location'] = df['address'].apply(geocode)

# 3 - create longitude, laatitude and altitude from location column (returns tuple)
df['point'] = df['location'].apply(lambda loc: tuple(loc.point) if loc else None)

df.head(2)

# # 4 - split point column into latitude, longitude and altitude columns
# df[['latitude', 'longitude', 'altitude']] = pd.DataFrame(df['point'].tolist(), index=df.index)
```

    C:\ProgramData\Anaconda3\lib\site-packages\ipykernel_launcher.py:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      after removing the cwd from sys.path.
    




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
      <th>역번호</th>
      <th>호선</th>
      <th>역명</th>
      <th>역주소</th>
      <th>address</th>
      <th>location</th>
      <th>point</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>150</td>
      <td>1</td>
      <td>서울역</td>
      <td>서울특별시 중구 세종대로 지하 2 (남대문로 5가)</td>
      <td>서울특별시 중구 세종대로 지하 2</td>
      <td>(세종대로, 소공동, 중구, 서울, 노원구, 04512, 대한민국, (37.5604...</td>
      <td>(37.5604063, 126.9753256, 0.0)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>151</td>
      <td>1</td>
      <td>시청</td>
      <td>서울특별시 중구 세종대로 지하 101 (정동)</td>
      <td>서울특별시 중구 세종대로 지하 101</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



##### 일부 행에서 None이 발생한다 (이후단계 진행 불가)

### [2차 시도] 역 이름을 사용한 위도/경도 변환


```python
# Try 02
# 역 이름을 주소로 사용하기
df['address2'] = df['역명'] + '역'

# 1 - conveneint function to delay between geocoding calls
geocode = RateLimiter(locator.geocode, min_delay_seconds=0.1)

# 2- - create location column
df['location'] = df['address2'].apply(geocode)

# 3 - create longitude, laatitude and altitude from location column (returns tuple)
df['point'] = df['location'].apply(lambda loc: tuple(loc.point) if loc else None)

# 4 - split point column into latitude, longitude and altitude columns
df[['latitude', 'longitude', 'altitude']] = pd.DataFrame(df['point'].tolist(), index=df.index)

df.head(2)
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
      <th>역번호</th>
      <th>호선</th>
      <th>역명</th>
      <th>역주소</th>
      <th>address</th>
      <th>location</th>
      <th>point</th>
      <th>address2</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>altitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>150</td>
      <td>1</td>
      <td>서울역</td>
      <td>서울특별시 중구 세종대로 지하 2 (남대문로 5가)</td>
      <td>서울특별시 중구 세종대로 지하 2</td>
      <td>(서울역, 한강대로, Namyeong-dong, 용산구, 서울, 04320, 대한민...</td>
      <td>(37.5534443, 126.9697112, 0.0)</td>
      <td>서울역역</td>
      <td>37.553444</td>
      <td>126.969711</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>151</td>
      <td>1</td>
      <td>시청</td>
      <td>서울특별시 중구 세종대로 지하 101 (정동)</td>
      <td>서울특별시 중구 세종대로 지하 101</td>
      <td>(시청, 중앙대로, 연산2동, 연산동, 연제구, 부산, 47606, 대한민국, (3...</td>
      <td>(35.1797067, 129.0766143, 0.0)</td>
      <td>시청역</td>
      <td>35.179707</td>
      <td>129.076614</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



##### 변환 성공

### ▶ 필요 column 추출


```python
result = df[['역번호', '호선', '역명', 'latitude', 'longitude']]
result = result.rename(columns={"역번호": "subwayNo",
                                "호선": "Line",
                                "역명": "subwayName"})
result.to_csv('subway_address.csv')
```
