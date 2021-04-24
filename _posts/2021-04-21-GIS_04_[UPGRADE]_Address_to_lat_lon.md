---
layout: single
title: "#GIS_04 [UPGRADE]Address to Latitude and Longitude"
---

# Geocode: convert address to lat/lon

### Subject: '지하철주소' 데이터를 '위도/경도' 데이터로 변환


```python
# !pip install geopandas
# !pip install geopy

import pandas as pd
import numpy as np
from geopy.extra.rate_limiter import RateLimiter
from geopy.geocoders import Nominatim
import time

# 1 - conveneint function to delay between geocoding calls
locator = Nominatim(user_agent="myGeocoder")
geocode = RateLimiter(locator.geocode, min_delay_seconds=0.1)
```

### ▶ 데이터 로드


```python
subway_address_url = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210419_subway/subway/subway_address.csv"
subway_address_df = pd.read_csv(subway_address_url)
```


```python
df = subway_address_df[['역번호', '호선', '역명', '역주소']]
df = df.rename(columns={"역번호": "subwayNo",
                        "호선": "Line",
                        "역명": "subwayName",
                        "역주소": "address_kor1"})

# # 데이터 샘플링
# df = df.head(10)

# 역이름 보정
for index, row in df.iterrows():  
    if row['subwayName'][-1] == '역':
        df.loc[index, 'subwayName'] = row['subwayName'][:-1]
        

# 주소의 '(...)'를 모두 없애기
df['address_kor2'] = df['address_kor1'].str.split('(', 1).str[0]

# 역명을 이용하여 주소검색 텍스트 만들기
df['address_kor3'] = '서울특별시 ' + df['subwayName'] + '역'

df
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
      <th>subwayNo</th>
      <th>Line</th>
      <th>subwayName</th>
      <th>address_kor1</th>
      <th>address_kor2</th>
      <th>address_kor3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>150</td>
      <td>1</td>
      <td>서울</td>
      <td>서울특별시 중구 세종대로 지하 2 (남대문로 5가)</td>
      <td>서울특별시 중구 세종대로 지하 2</td>
      <td>서울특별시 서울역</td>
    </tr>
    <tr>
      <th>1</th>
      <td>151</td>
      <td>1</td>
      <td>시청</td>
      <td>서울특별시 중구 세종대로 지하 101 (정동)</td>
      <td>서울특별시 중구 세종대로 지하 101</td>
      <td>서울특별시 시청역</td>
    </tr>
    <tr>
      <th>2</th>
      <td>152</td>
      <td>1</td>
      <td>종각</td>
      <td>서울특별시 종로구 종로 지하 55 (종로1가)</td>
      <td>서울특별시 종로구 종로 지하 55</td>
      <td>서울특별시 종각역</td>
    </tr>
    <tr>
      <th>3</th>
      <td>153</td>
      <td>1</td>
      <td>종로3가</td>
      <td>서울특별시 종로구 종로 지하 129 (종로3가)</td>
      <td>서울특별시 종로구 종로 지하 129</td>
      <td>서울특별시 종로3가역</td>
    </tr>
    <tr>
      <th>4</th>
      <td>154</td>
      <td>1</td>
      <td>종로5가</td>
      <td>서울특별시 종로구 종로 지하 216  (종로5가)</td>
      <td>서울특별시 종로구 종로 지하 216</td>
      <td>서울특별시 종로5가역</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>287</th>
      <td>4134</td>
      <td>9</td>
      <td>송파나루</td>
      <td>서울특별시 송파구 오금로 165</td>
      <td>서울특별시 송파구 오금로 165</td>
      <td>서울특별시 송파나루역</td>
    </tr>
    <tr>
      <th>288</th>
      <td>4135</td>
      <td>9</td>
      <td>한성백제</td>
      <td>서울특별시 송파구 위례성대로 51</td>
      <td>서울특별시 송파구 위례성대로 51</td>
      <td>서울특별시 한성백제역</td>
    </tr>
    <tr>
      <th>289</th>
      <td>4136</td>
      <td>9</td>
      <td>올림픽공원</td>
      <td>서울특별시 송파구 양재대로 1233</td>
      <td>서울특별시 송파구 양재대로 1233</td>
      <td>서울특별시 올림픽공원역</td>
    </tr>
    <tr>
      <th>290</th>
      <td>4137</td>
      <td>9</td>
      <td>둔촌오륜</td>
      <td>서울특별시 송파구 강동대로 327</td>
      <td>서울특별시 송파구 강동대로 327</td>
      <td>서울특별시 둔촌오륜역</td>
    </tr>
    <tr>
      <th>291</th>
      <td>4138</td>
      <td>9</td>
      <td>중앙보훈병원</td>
      <td>서울특별시 강동구 명일로 117</td>
      <td>서울특별시 강동구 명일로 117</td>
      <td>서울특별시 중앙보훈병원역</td>
    </tr>
  </tbody>
</table>
<p>292 rows × 6 columns</p>
</div>



### ▶ goecode Test


```python
print("====== test 01 ======")
address_a = df.loc[0, 'address_kor1']
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
address_c = df.loc[0, 'subwayName']
location_c = locator.geocode(address_c)
print("address: " + address_c)
print("location: " + str(location_c))
print("Latitude = {}, Longitude = {}".format(location_c.latitude, location_c.longitude))
```

    ====== test 03 ======
    address: 서울
    location: 서울, 노원구, 대한민국
    Latitude = 37.5666791, Longitude = 126.9782914
    


```python
print("====== test 04 ======")
address_d = "대한민국 " + df.loc[0, 'subwayName']
location_d = locator.geocode(address_d)
print("address: " + address_d)
print("location: " + str(location_d))
print("Latitude = {}, Longitude = {}".format(location_d.latitude, location_d.longitude))
```

    ====== test 04 ======
    address: 대한민국 서울
    location: 서울, 노원구, 대한민국
    Latitude = 37.5666791, Longitude = 126.9782914
    

### Test 04로 진행

### 주소를 위도/경도로 변환 


```python
# # Try 01

# # 2- - create location column
# for index, row in df.iterrows():
#     address = locator.geocode(row['address_kor2'])
    
#     if not address:
#         address = locator.geocode(row['address_kor3'])
    
#     df.loc[index, 'location'] = address[0]
#     df.loc[index, 'latitude'] = address[1][0]
#     df.loc[index, 'longitude'] = address[1][1]

# df
```

#### 검색결과 오류가 있음


```python
# Try 02
# 역 이름을 주소로 사용하기 (대한민국을 더하기, 안할 시 중국, 일본의 지하철역 검색)
start = time.process_time()

# 2- - create location column
df['location'] = df['address_kor3'].apply(geocode)

# 3 - create longitude, laatitude and altitude from location column (returns tuple)
df['point'] = df['location'].apply(lambda loc: tuple(loc.point) if loc else None)

# 4 - split point column into latitude, longitude and altitude columns
df[['latitude', 'longitude', 'altitude']] = pd.DataFrame(df['point'].tolist(), index=df.index)

print("Runtime: " + str(time.process_time() - start))

df
```

    Runtime: 0.859375
    




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
      <th>subwayNo</th>
      <th>Line</th>
      <th>subwayName</th>
      <th>address_kor1</th>
      <th>address_kor2</th>
      <th>address_kor3</th>
      <th>location</th>
      <th>point</th>
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
      <td>서울</td>
      <td>서울특별시 중구 세종대로 지하 2 (남대문로 5가)</td>
      <td>서울특별시 중구 세종대로 지하 2</td>
      <td>서울특별시 서울역</td>
      <td>(서울역, 한강대로, Namyeong-dong, 용산구, 서울, 04320, 대한민...</td>
      <td>(37.5534443, 126.9697112, 0.0)</td>
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
      <td>서울특별시 시청역</td>
      <td>(시청, 127, 서소문로, 소공동, 중구, 서울, 노원구, 04515, 대한민국,...</td>
      <td>(37.5652772, 126.977104, 0.0)</td>
      <td>37.565277</td>
      <td>126.977104</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>152</td>
      <td>1</td>
      <td>종각</td>
      <td>서울특별시 종로구 종로 지하 55 (종로1가)</td>
      <td>서울특별시 종로구 종로 지하 55</td>
      <td>서울특별시 종각역</td>
      <td>(종각, 우정국로, 종로1·2·3·4가동, 명동, 종로구, 서울, 노원구, 0452...</td>
      <td>(37.5701834, 126.9829019, 0.0)</td>
      <td>37.570183</td>
      <td>126.982902</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>153</td>
      <td>1</td>
      <td>종로3가</td>
      <td>서울특별시 종로구 종로 지하 129 (종로3가)</td>
      <td>서울특별시 종로구 종로 지하 129</td>
      <td>서울특별시 종로3가역</td>
      <td>(종로3가, 돈화문로, 종로3가, 종로1·2·3·4가동, 종로구, 서울, 노원구, ...</td>
      <td>(37.5704309, 126.9922249, 0.0)</td>
      <td>37.570431</td>
      <td>126.992225</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>154</td>
      <td>1</td>
      <td>종로5가</td>
      <td>서울특별시 종로구 종로 지하 216  (종로5가)</td>
      <td>서울특별시 종로구 종로 지하 216</td>
      <td>서울특별시 종로5가역</td>
      <td>(종로5가, 대학로, 종로5·6가동, 종로구, 서울, 노원구, 03129, 대한민국...</td>
      <td>(37.5709758, 127.0019179, 0.0)</td>
      <td>37.570976</td>
      <td>127.001918</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>287</th>
      <td>4134</td>
      <td>9</td>
      <td>송파나루</td>
      <td>서울특별시 송파구 오금로 165</td>
      <td>서울특별시 송파구 오금로 165</td>
      <td>서울특별시 송파나루역</td>
      <td>(송파나루역, 오금로, 송파1동, 송파구, 서울, 05642, 대한민국, (37.5...</td>
      <td>(37.5096527, 127.1130858, 0.0)</td>
      <td>37.509653</td>
      <td>127.113086</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>288</th>
      <td>4135</td>
      <td>9</td>
      <td>한성백제</td>
      <td>서울특별시 송파구 위례성대로 51</td>
      <td>서울특별시 송파구 위례성대로 51</td>
      <td>서울특별시 한성백제역</td>
      <td>(한성백제, 51, 위례성대로, 오륜동, 송파구, 서울, 05507, 대한민국, (...</td>
      <td>(37.5166702, 127.1162125, 0.0)</td>
      <td>37.516670</td>
      <td>127.116213</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>289</th>
      <td>4136</td>
      <td>9</td>
      <td>올림픽공원</td>
      <td>서울특별시 송파구 양재대로 1233</td>
      <td>서울특별시 송파구 양재대로 1233</td>
      <td>서울특별시 올림픽공원역</td>
      <td>(올림픽공원역, 양재대로, 오륜동, 송파구, 서울, 05649, 대한민국, (37....</td>
      <td>(37.5165067, 127.1308993, 0.0)</td>
      <td>37.516507</td>
      <td>127.130899</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>290</th>
      <td>4137</td>
      <td>9</td>
      <td>둔촌오륜</td>
      <td>서울특별시 송파구 강동대로 327</td>
      <td>서울특별시 송파구 강동대로 327</td>
      <td>서울특별시 둔촌오륜역</td>
      <td>(둔촌오륜, 강동대로, 둔촌동, 강동구, 서울, 05408, 대한민국, (37.51...</td>
      <td>(37.5198411, 127.1384179, 0.0)</td>
      <td>37.519841</td>
      <td>127.138418</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>291</th>
      <td>4138</td>
      <td>9</td>
      <td>중앙보훈병원</td>
      <td>서울특별시 강동구 명일로 117</td>
      <td>서울특별시 강동구 명일로 117</td>
      <td>서울특별시 중앙보훈병원역</td>
      <td>(중앙보훈병원, 동남로, 둔촌동, 강동구, 서울, 05344, 대한민국, (37.5...</td>
      <td>(37.5282799, 127.1482582, 0.0)</td>
      <td>37.528280</td>
      <td>127.148258</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>292 rows × 11 columns</p>
</div>



##### 변환 성공

### ▶ 필요 column 추출


```python
result = df[['subwayNo', 'Line', 'subwayName', 'address_kor1', 'latitude', 'longitude']]
result.to_csv('subway_address.csv')
```
