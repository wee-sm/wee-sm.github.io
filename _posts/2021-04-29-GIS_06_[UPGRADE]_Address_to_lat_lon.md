---
layout: single
title: "#GIS_06 [UPGRADE]Address to Latitude and Longitude"
---

# Goepy: address to latitude/longitude


```python
import pandas as pd
import numpy as np
from geopy.extra.rate_limiter import RateLimiter
from geopy.geocoders import Nominatim
from functools import reduce
import time

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```


```python
subway_address_url = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210419_subway/subway/subway_address_origin.csv"
subway_address_df = pd.read_csv(subway_address_url)

subway_address_df = subway_address_df[['역번호', '호선', '역명', '역주소']]
subway_address_df = subway_address_df.rename(columns={"역번호": "subwayNo",
                                                      "호선": "Line",
                                                      "역명": "subwayName",
                                                      "역주소": "address_kor1"})

################################# Correction 1 #################################
# for index, row in subway_address_df.iterrows():
#     if row['subwayName'][-1] == '역':
#         subway_address_df.loc[index, 'subwayName'] = row['subwayName'][:-1]
################################# Correction 2 #################################
subway_address_df['address_kor2'] = subway_address_df['address_kor1'].str.split('(', 1).str[0]
################################# Correction 3 #################################
temp_address = subway_address_df['address_kor1'].str.split(' ', 2)
subway_address_df['address_kor3'] = temp_address.str[0] + " " + temp_address.str[1] + " " + subway_address_df['subwayName'] + '역'

subway_address_df.head(3)
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
      <td>서울역</td>
      <td>서울특별시 중구 세종대로 지하 2 (남대문로 5가)</td>
      <td>서울특별시 중구 세종대로 지하 2</td>
      <td>서울특별시 중구 서울역역</td>
    </tr>
    <tr>
      <th>1</th>
      <td>151</td>
      <td>1</td>
      <td>시청</td>
      <td>서울특별시 중구 세종대로 지하 101 (정동)</td>
      <td>서울특별시 중구 세종대로 지하 101</td>
      <td>서울특별시 중구 시청역</td>
    </tr>
    <tr>
      <th>2</th>
      <td>152</td>
      <td>1</td>
      <td>종각</td>
      <td>서울특별시 종로구 종로 지하 55 (종로1가)</td>
      <td>서울특별시 종로구 종로 지하 55</td>
      <td>서울특별시 종로구 종각역</td>
    </tr>
  </tbody>
</table>
</div>




```python
subway_usage_url = "https://raw.githubusercontent.com/wee-sm/database/master/Statistics/Korea_Seoul/20210419_subway/subway/usage/CARD_SUBWAY_MONTH_202103.csv"
subway_usage_df = pd.read_csv(subway_usage_url, index_col=False)

subway_usage_df = subway_usage_df[['사용일자', '노선명', '역명', '승차총승객수', '하차총승객수']]
subway_usage_df = subway_usage_df.rename(columns={"사용일자":"date",
                                                  "역번호": "subwayNo",
                                                  "노선명": "Line",
                                                  "역명": "subwayName",
                                                  "승차총승객수": "usage_on",
                                                  "하차총승객수": "usage_off"})

subway_usage_df.head(3)
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
      <th>date</th>
      <th>Line</th>
      <th>subwayName</th>
      <th>usage_on</th>
      <th>usage_off</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20210301</td>
      <td>중앙선</td>
      <td>양평</td>
      <td>1418</td>
      <td>1175</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20210301</td>
      <td>경원선</td>
      <td>덕계</td>
      <td>1186</td>
      <td>1077</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20210301</td>
      <td>7호선</td>
      <td>신풍</td>
      <td>4593</td>
      <td>4575</td>
    </tr>
  </tbody>
</table>
</div>




```python
def getLine(n):
    subway_line = subway_address_df[subway_address_df.Line == n]

    locator = Nominatim(user_agent="myGeocoder")

    print("Errors List: ", end="")
    errorsList = []

    for index, row in subway_line.iterrows():
        location = locator.geocode(row['address_kor3'])
        if not location:
            location = locator.geocode(row['address_kor1'])
        if not location:
            location = locator.geocode(row['address_kor2'])

        if location:
            subway_line.loc[index, 'latitude'] = location[1][0]
            subway_line.loc[index, 'longitude'] = location[1][1]
        else:
            errorsList.append(row.subwayName)
            print(row.subwayName, end=' ')

    if not errorsList:
        print("None")
    else:
        print()

    ########################
    subway_usage_line = subway_usage_df[subway_usage_df.Line == str(n) + "호선"]
    subway_usage_line = subway_usage_line.groupby(by="subwayName").sum()[["usage_on", "usage_off"]].reset_index()

    subway_usage_line['subwayName'] = subway_usage_line['subwayName'].str.split('(', 1).str[0]

    # ### inner merge
    # print("dropped subway: " + str(set(subway_line.subwayName) - set(subway_usage_line.subwayName)))

    subway_line_usage = subway_line.merge(subway_usage_line, on="subwayName", how="outer")
    
    return subway_line_usage[["subwayNo", "Line", "subwayName", "latitude", "longitude", "usage_on", "usage_off"]]
```


```python
Line = []
for i in range(1, 9):
    Line.append(getLine(i))
```

    Errors List: None
    Errors List: None
    Errors List: None
    Errors List: 총신대입구 
    Errors List: 강동 상일동 
    Errors List: None
    Errors List: None
    Errors List: 암사 산성 신흥 수진 
    


```python
Lind_final = reduce(lambda left,right: left.append(right), Line)
Lind_final
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
      <th>latitude</th>
      <th>longitude</th>
      <th>usage_on</th>
      <th>usage_off</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>150.0</td>
      <td>1.0</td>
      <td>서울역</td>
      <td>37.554330</td>
      <td>126.972384</td>
      <td>1103795.0</td>
      <td>1037732.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>151.0</td>
      <td>1.0</td>
      <td>시청</td>
      <td>37.565277</td>
      <td>126.977104</td>
      <td>531441.0</td>
      <td>531883.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>152.0</td>
      <td>1.0</td>
      <td>종각</td>
      <td>37.570183</td>
      <td>126.982902</td>
      <td>877966.0</td>
      <td>837757.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>153.0</td>
      <td>1.0</td>
      <td>종로3가</td>
      <td>37.570431</td>
      <td>126.992225</td>
      <td>693427.0</td>
      <td>635888.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>154.0</td>
      <td>1.0</td>
      <td>종로5가</td>
      <td>37.570976</td>
      <td>127.001918</td>
      <td>604736.0</td>
      <td>601693.0</td>
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
    </tr>
    <tr>
      <th>12</th>
      <td>2823.0</td>
      <td>8.0</td>
      <td>남한산성입구</td>
      <td>37.451553</td>
      <td>127.159813</td>
      <td>365328.0</td>
      <td>335488.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2824.0</td>
      <td>8.0</td>
      <td>단대오거리</td>
      <td>37.445085</td>
      <td>127.156778</td>
      <td>282125.0</td>
      <td>273558.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2825.0</td>
      <td>8.0</td>
      <td>신흥</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>124204.0</td>
      <td>131424.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2826.0</td>
      <td>8.0</td>
      <td>수진</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>140092.0</td>
      <td>128819.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2827.0</td>
      <td>8.0</td>
      <td>모란</td>
      <td>37.431366</td>
      <td>127.129030</td>
      <td>126999.0</td>
      <td>99937.0</td>
    </tr>
  </tbody>
</table>
<p>284 rows × 7 columns</p>
</div>



### ▶ Save to csv file


```python
Lind_final.to_csv('Lind_final.csv')
```


```python

```
