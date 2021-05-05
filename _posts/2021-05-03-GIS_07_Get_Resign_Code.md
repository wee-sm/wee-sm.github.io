---
layout: single
title: "#GIS_07 Resign Code (Seoul)"
---

# Get Resign Code from Korean name of resign (dong)


```python
import pandas as pd
import geopandas
```


```python
# shape file, data file load
Seoul_Job_dir= "temp/Seoul_Job.csv"
Seoul_Job = pd.read_csv(Seoul_Job_dir, sep='\t', header=2).iloc[:, :4]

Seoul_Job.head(3)
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
      <th>기간</th>
      <th>자치구</th>
      <th>동</th>
      <th>사업체수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019</td>
      <td>합계</td>
      <td>합계</td>
      <td>823,624.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019</td>
      <td>종로구</td>
      <td>소계</td>
      <td>39,679.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>종로구</td>
      <td>사직동</td>
      <td>3,574.0</td>
    </tr>
  </tbody>
</table>
</div>



### 1st Try


```python
# First EMD CODES
Korea_Code_csv= "temp/Region_Code_SouthKorea_EMD.csv"
Korea_Code = pd.read_csv(Korea_Code_csv, sep='\t', encoding="ANSI")

Seoul_Code = Korea_Code[Korea_Code["법정동코드"].between(1111000001, 1199999999)]
Seoul_Code = Seoul_Code[Seoul_Code["폐지여부"] == "존재"]

Seoul_Code["CITY"] = Seoul_Code["법정동명"].str.split(' ', expand=True)[0]
Seoul_Code["GU"] = Seoul_Code["법정동명"].str.split(' ', expand=True)[1]
Seoul_Code["DONG"] = Seoul_Code["법정동명"].str.split(' ', expand=True)[2]

Seoul_Code = Seoul_Code.drop(columns=["법정동명", "폐지여부"])
Seoul_Code = Seoul_Code.rename(columns={"법정동코드": "CODE"})

Seoul_Code.head(3) # Doesn't match with upper classification
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
      <th>CODE</th>
      <th>CITY</th>
      <th>GU</th>
      <th>DONG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>1111010100</td>
      <td>서울특별시</td>
      <td>종로구</td>
      <td>청운동</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1111010200</td>
      <td>서울특별시</td>
      <td>종로구</td>
      <td>신교동</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1111010300</td>
      <td>서울특별시</td>
      <td>종로구</td>
      <td>궁정동</td>
    </tr>
  </tbody>
</table>
</div>



### 2nd Try


```python
# Second EMD CODES
Korea_Code_xlsx= "temp/한국행정구역분류_2021.1.1.기준.xlsx"
Korea_Code = pd.read_excel(Korea_Code_xlsx, sheet_name=1, header=2)

Seoul_Code = Korea_Code[Korea_Code["소분류"].between(1100000, 1200000)]
Seoul_Code = Seoul_Code[Seoul_Code.읍면동.notna()].iloc[2:, 5:7]
Seoul_Code.head(3) # Matching with upper classification
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
      <th>소분류</th>
      <th>읍면동</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>1101055.0</td>
      <td>부암동</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1101056.0</td>
      <td>평창동</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1101057.0</td>
      <td>무악동</td>
    </tr>
  </tbody>
</table>
</div>



### Merge Results & Save


```python
# rename columns to join
Seoul_Job = Seoul_Job.rename(columns={"동": "DONG"})
Seoul_Code = Seoul_Code.rename(columns={"읍면동": "DONG"})

# replace to match the row
Seoul_Job['DONG'] = Seoul_Job['DONG'].str.replace('·','.')
Seoul_Code['DONG'] = Seoul_Code['DONG'].str.replace('·','.')

# Merge Data
df_Seoul_Job = Seoul_Code.merge(Seoul_Job, on = "DONG")

df_Seoul_Job['사업체수'] = df_Seoul_Job['사업체수'].str.replace('.0','')
df_Seoul_Job['사업체수'] = df_Seoul_Job['사업체수'].str.replace(',','')
df_Seoul_Job = df_Seoul_Job.astype({"소분류": int, "사업체수": int})
df_Seoul_Job = df_Seoul_Job.drop(columns=["기간"])

df_Seoul_Job.head(3)
```

    C:\ProgramData\Anaconda3\lib\site-packages\ipykernel_launcher.py:12: FutureWarning: The default value of regex will change from True to False in a future version.
      if sys.path[0] == '':
    




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
      <th>소분류</th>
      <th>DONG</th>
      <th>자치구</th>
      <th>사업체수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1101055</td>
      <td>부암동</td>
      <td>종로구</td>
      <td>599</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1101056</td>
      <td>평창동</td>
      <td>종로구</td>
      <td>761</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1101057</td>
      <td>무악동</td>
      <td>종로구</td>
      <td>647</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_Seoul_Job.to_csv("temp/Seoul_Job_Gu.csv")
```
