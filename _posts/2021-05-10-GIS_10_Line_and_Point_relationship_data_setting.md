---
layout: single
title: "# GIS 10 Line and Point relationship data setting"
---

# GIS 10 Line_and_Point_relationship_data_setting


```python
import pandas as pd
import numpy as np
import os
import geopandas
import matplotlib.pyplot as plt
from shapely.geometry import Point, LineString

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```


```python
def read_shapefile_in_dir(dir=".", i=0):
    files = os.listdir(dir)
    shapefile = list(filter(lambda shp: '.shp' in shp, files))[i]
    return geopandas.read_file(dir + '/' + shapefile)

def listToString(s): 
    str1 = "" 
    for ele in s: str1 += (str(ele) + ", ")
    return str1
```


```python
Networks_dir = "temp/custom_arcgis/Seoul_Newworks/Networks_point"
Networks_line = read_shapefile_in_dir(Networks_dir, 0)
Networks_point = read_shapefile_in_dir(Networks_dir, 2)
```


```python
Networks_line.head(3)
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
      <th>FID_1</th>
      <th>Shape_Leng</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>0.000011</td>
      <td>LINESTRING (126.99742 37.52596, 126.99743 37.5...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>0.000011</td>
      <td>LINESTRING (126.99812 37.52575, 126.99813 37.5...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>0.000011</td>
      <td>LINESTRING (126.99813 37.52575, 126.99814 37.5...</td>
    </tr>
  </tbody>
</table>
</div>




```python
Networks_point.head(3)
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
      <th>FID_1</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>POINT (126.99742 37.52596)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>POINT (126.99813 37.52575)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.0</td>
      <td>POINT (126.99926 37.52566)</td>
    </tr>
  </tbody>
</table>
</div>




```python
Npoint = Networks_point.copy()
Nline = Networks_line.copy()
```

### test plot function


```python
def plot_point_line(PID, r=8):
    point = Npoint[Npoint["FID_1"] == PID].iloc[0, -1]
    plt.scatter(round(point.coords[0][0], r), round(point.coords[0][1], r))

    line = Nline[Nline["FID_1"] == PID].iloc[:, -1]
    for l in line:
        plt.plot((round(l.coords[0][0], r), round(l.coords[-1][0], r)),
                 (round(l.coords[0][1], r), round(l.coords[-1][1], r)))
    plt.show()

def plot_line(index):
    px=[]
    py=[]

    line = Nline.loc[i, "geometry"]

    for point in line.coords:
        plt.scatter(point[0], point[1])
        px.append(point[0])
        py.append(point[1])
    plt.plot(px, py)
    plt.show()
```

### STEP 1


```python
# 꺽인 Linestring data 수정: 양끝 point를 연결하는 line으로 변경
########################################
error_index = 361
print("Sample Data (index:" + str(error_index) + ")")
print("Before: ", end="")
for li in Nline.loc[error_index, "geometry"].coords:
    print(li, end=" ")
plot_line(error_index)
########################################

## CODE
for index, line in Nline.iterrows():
    if (len(line.geometry.coords) > 2):
        Nline.loc[index, "geometry"] = LineString([line.geometry.coords[0], line.geometry.coords[-1]])

########################################
print("After: ", end="")
for li in Nline.loc[error_index, "geometry"].coords:
    print(li, end=" ")
plot_line(error_index)
########################################
```

    Sample Data (index:361)
    Before: (126.98701886083188, 37.57168751236884) (126.9870191000515, 37.57168709958006) (126.98702320006066, 37.571679199935204) 


    
![png](\assets\images\2021-05-10\output_10_1.png)
    


    After: (126.98701886083188, 37.57168751236884) (126.98702320006066, 37.571679199935204) 


    
![png](\assets\images\2021-05-10\output_10_3.png)
    


### STEP 2



```python
# 범위를 벗어나는 오류데이터 정리 (불필요)
# Networks_line = Networks_line[Networks_line["Shape_Leng"].between(0.000008, 0.000012)]
```

### STEP 3


```python
# round: arcgis intersect 연산에서 발생한 오차를 조정
# drop: 기준점에서 떨어진(불필요한) line 제거
r = 8 # round index

########################### Data sample [Before] ###########################
print("[ Before ]")
print("Point: ", str(len(Npoint)))
print("Line : ", str(len(Nline)))
plot_point_line(16645.0)
############################################################################

stopper = 0
for pindex, point in Npoint.iterrows():
    PID = point.FID_1
    Line_PID = Nline[Nline["FID_1"] == PID]
    
    # filter 1: point와 연결 안 된 line 지우기
    for lindex, line in Line_PID.iterrows():
        
        line_meet_point = False
        rounded_point = (round(point.geometry.x, r), round(point.geometry.y, r))
        
        for l in line.geometry.coords:
            if ((round(l[0], r), round(l[1], r)) == rounded_point):
                line_meet_point = True
        
        if not line_meet_point:
            Nline = Nline.drop(lindex)
            Line_PID = Line_PID.drop(lindex)
            
#             stopper = 1
#             break
#         if stopper:
#             break
#     if stopper:
#         break
            
    # filter 2: 2개 이하의 line set 지우기
    if (len(Line_PID) < 3):
        Npoint = Npoint.drop(pindex)
        Nline = Nline.drop(Line_PID.index)

########################### Data sample [After] ###########################
print("[ After ]")
print("Point: ", str(len(Npoint)))
print("Line : ", str(len(Nline)))
plot_point_line(16645.0)
############################################################################
```

    [ Before ]
    Point:  94304
    Line :  299732
    


    
![png](\assets\images\2021-05-10\output_14_1.png)
    


    [ After ]
    Point:  87774
    Line :  285160
    


    
![png](\assets\images\2021-05-10\output_14_3.png)
    


### Save shapefile


```python
Save_filePath = "./temp/20210510_geopandas/"
Save_Npoint = "Npoint.shp"
Save_Nline = "Nline.shp"

Npoint.to_file(Save_filePath + Save_Npoint)
Nline.to_file(Save_filePath + Save_Nline)
```
