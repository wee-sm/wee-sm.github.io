---
layout: single
title: "#GIS 11 Get all line's angles at each point"
---

# Get all angles at each points


```python
import pandas as pd
import numpy as np
import geopandas
import matplotlib.pyplot as plt
from shapely.geometry import Point, LineString

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```

### Functions


```python
import os
import geopandas

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
import math
 
def getAngle(a, b, c):
    ang = math.degrees(math.atan2(c[1]-b[1], c[0]-b[0]) - math.atan2(a[1]-b[1], a[0]-b[0]))
    return ang + 360 if ang < 0 else ang

def getMean(list):
    return sum(list) / len(list)

def getVsum(list):
    vsum = 0
    for x in list:
        vsum = vsum + (x - getMean(list)) ** 2
    return vsum / len(list)

def getStd(list):
    return math.sqrt(getVsum(list))
```


```python
def plot_point_line(FID, r=8):
    point = Npoint[Npoint["FID_1"] == FID].iloc[0, 1]
    line = Nline[Nline["FID_1"] == FID].iloc[:, 2]
    
    plt.scatter(round(point.coords[0][0], r), round(point.coords[0][1], r))
    for l in line:
        plt.plot((round(l.coords[0][0], r), round(l.coords[-1][0], r)),
                 (round(l.coords[0][1], r), round(l.coords[-1][1], r)))
    plt.show()

def plot_line(index):
    px=[]
    py=[]

    line = Nline.loc[index, "geometry"]

    for point in line.coords:
        plt.scatter(point[0], point[1])
        px.append(point[0])
        py.append(point[1])
    plt.plot(px, py)
    plt.show()
```

### Road Data


```python
Load_filePath = "./temp/custom_arcgis/Seoul_Networks_step03/"
Networks_line = read_shapefile_in_dir(Load_filePath + "Nline")
Networks_point = read_shapefile_in_dir(Load_filePath + "Npoint")
```


```python
Npoint = Networks_point.copy()
Nline = Networks_line.copy()

Npoint["angle_min"] = ""
Npoint["angle_max"] = ""
Npoint["angle_mean"] = ""
Npoint["angle_vsum"] = ""
Npoint["angle_std"] = ""

Npoint.head(3)
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
      <th>angle_min</th>
      <th>angle_max</th>
      <th>angle_mean</th>
      <th>angle_vsum</th>
      <th>angle_std</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>POINT (126.99742 37.52596)</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>POINT (126.99813 37.52575)</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.0</td>
      <td>POINT (126.99926 37.52566)</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>



### check data and plot


```python
# plot geometry by 'FID' of point and line
pid = 0

print("=====================[ Lines related to point ]=====================")
plot_point_line(pid)
print("[ Point ]-----------------------------------------------------------")
print(Npoint[Npoint["FID_1"] == pid])
print("[ Lines ]-----------------------------------------------------------")
print(Nline[Nline["FID_1"] == pid])
print("====================================================================")

print("\n\n")

# plot geometry by 'index' of line
pid = 0

print("=====================[ Points related to line ]=====================")
plot_line(pid)
print("[ Line ]------------------------------------------------------------")
print(Nline.loc(pid)[0])
print("====================================================================")
```

    =====================[ Lines related to point ]=====================
    


    
![png](\assets\images\2021-05-11\output_10_1.png)
    


    [ Point ]-----------------------------------------------------------
       FID_1                    geometry angle_min angle_max angle_mean  \
    0    0.0  POINT (126.99742 37.52596)                                  
    
      angle_vsum angle_std  
    0                       
    [ Lines ]-----------------------------------------------------------
           FID_1  Shape_Leng                                           geometry
    0        0.0    0.000011  LINESTRING (126.99742 37.52596, 126.99743 37.5...
    19156    0.0    0.000011  LINESTRING (126.99743 37.52597, 126.99742 37.5...
    19157    0.0    0.000011  LINESTRING (126.99742 37.52596, 126.99741 37.5...
    ====================================================================
    
    
    
    =====================[ Points related to line ]=====================
    


![png](\assets\images\2021-05-11\output_10_3.png)    
    


    [ Line ]------------------------------------------------------------
    FID_1                                                       0.0
    Shape_Leng                                             0.000011
    geometry      LINESTRING (126.9974224997486 37.5259639003138...
    Name: 0, dtype: object
    ====================================================================
    


```python
# 1st Try: Line을 기준으로 각도 구하기
# - 순서대로 line을 읽지 못해 오류가 발생한다
index = 0
stopper = 0

for index, point in Npoint.iterrows():
    stopper += 1
    if stopper > 2: break
    
    point_0 = point["geometry"].coords[0]
    FID = point["FID_1"]
    lines = Nline[Nline["FID_1"] == FID]
    angles = []
    
    for line1_index, line1 in lines.iterrows():
        for line2_index, line2 in lines.iterrows():
            # Skip same line
            if line1.geometry == line2.geometry: continue
                
            # Get three end points
            for end_point in line1.geometry.coords:
                if (end_point != point_0): point_a = end_point
            for end_point in line2.geometry.coords:
                if (end_point != point_0): point_b = end_point
            
            # Calculate Angle
            angle = getAngle(point_a, point_0, point_b)
            if angle > 180: angle = abs(360-angle)
            angles.append(angle)
            # print("Angle [between " + str(line1_index) + ", " + str(line2_index) + "]:" + str(round(angle, 2)))
        
        # Drop the first line to avoid overlap calculating
        lines = lines.drop(line1_index)
    
    ###################################################
    plot_point_line(FID)
    print("Angles of upper lines:", [round(alnge, 2) for alnge in angles])
    print("minimum angle:", min(angles))
    print("maximum angle:", max(angles))
    # print("mean:", getMean(angles)) # always 360/len(lines)
    # print("vsum:", getVsum(angles))
    print("std:", getStd(angles))
    ###################################################

# Angle을 계산하는 line의 순서가 정리안됨 -> 결과 오류
```


![png](\assets\images\2021-05-11\output_11_0.png)    
    


    Angles of upper lines: [27.23, 163.55, 169.21]
    minimum angle: 27.23336607948253
    maximum angle: 169.21497465475227
    std: 65.63664912230189
    


![png](\assets\images\2021-05-11\output_11_2.png)    
    


    Angles of upper lines: [179.73, 86.04, 94.23]
    minimum angle: 86.04183311344389
    maximum angle: 179.72853479237264
    std: 42.366523060627344
    


```python
# 2nd Try: Point를 기준으로 각도 구하기
# - 읽을 point의 순서를 만들어낸다

index = 0

for index, point in Npoint.iloc[:3].iterrows():
    FID = point["FID_1"]
    lines = Nline[Nline["FID_1"] == FID]["geometry"]
    
    point_A = point["geometry"].coords[0]
    point_B = (point_A[0] + 0.00001, point_A[1])
    # plt.scatter(point_A[0], point_A[1])
    # plt.scatter(point_B[0], point_B[1])

    end_points = []
    angles = []
    
    # get point and sort by order (from standard line angle)
    for line in lines:
        for end_point in line.coords:
            if (end_point != point_A):
                angle = getAngle(end_point, point_A, point_B)
                
                ######################################
                # point annotate
                anno = "(" + str(round(end_point[0], 2)) + ", "+ str(round(end_point[0], 2)) + ")"
                plt.scatter(end_point[0], end_point[1])
                plt.annotate(anno, end_point)
                ######################################
                
                # add list: point and angle relationship
                end_points.append({'angle': angle, 'point': end_point})
    
    end_points.sort(key=lambda x: x.get('angle'))
    
    for i in range(len(end_points)):
        j = i + 1
        if j == len(end_points): j = 0
        
        # 순서 반대로
        angles.append(getAngle(end_points[j]["point"], point_A, end_points[i]["point"]))
        
    ###################################################
    plot_point_line(FID)
    print("Point Index:", index)
    print("FID:", FID)
    print("Angles of upper lines:", [round(alnge, 2) for alnge in angles])
    print("minimum angle:", min(angles))
    print("maximum angle:", max(angles))
    # print("mean:", getMean(angles)) # always 360/len(lines)
    # print("vsum:", getVsum(angles))
    print("std:", getStd(angles))
    print()
    ###################################################
```


![png](\assets\images\2021-05-11\output_12_0.png)    
    


    Point Index: 0
    FID: 0.0
    Angles of upper lines: [163.55, 169.21, 27.23]
    minimum angle: 27.23336607948253
    maximum angle: 169.21497465475227
    std: 65.63664912230189
    
    


![png](\assets\images\2021-05-11\output_12_2.png)    
    


    Point Index: 1
    FID: 1.0
    Angles of upper lines: [94.23, 86.04, 179.73]
    minimum angle: 86.04183311344389
    maximum angle: 179.72853479237264
    std: 42.36652306062734
    
    



![png](\assets\images\2021-05-11\output_12_4.png)    
    


    Point Index: 2
    FID: 2.0
    Angles of upper lines: [86.21, 93.8, 179.99]
    minimum angle: 86.2100040115024
    maximum angle: 179.9923137994165
    std: 42.533919807508695
    
    


```python
# Final code

index = 0

for index, point in Npoint.iterrows():
    FID = point["FID_1"]
    lines = Nline[Nline["FID_1"] == FID]["geometry"]
    
    point_A = point["geometry"].coords[0]
    point_B = (point_A[0] + 0.00001, point_A[1])

    end_points = []
    angles = []
    
    # get point and sort by order (from standard line angle)
    for line in lines:
        for end_point in line.coords:
            if (end_point != point_A):
                angle = getAngle(end_point, point_A, point_B)
                
                # add list: point and angle relationship
                end_points.append({'angle': angle, 'point': end_point})
    
    end_points.sort(key=lambda x: x.get('angle'))
    
    for i in range(len(end_points)):
        j = i + 1
        if j == len(end_points): j = 0
        angles.append(getAngle(end_points[j]["point"], point_A, end_points[i]["point"]))
        
    Npoint.loc[index, "angle_min"] = min(angles)
    Npoint.loc[index, "angle_max"] = max(angles)
    Npoint.loc[index, "angle_mean"] = getMean(angles)
    Npoint.loc[index, "angle_vsum"] = getVsum(angles)
    Npoint.loc[index, "angle_std"] = getStd(angles)
```


```python
Npoint
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
      <th>angle_min</th>
      <th>angle_max</th>
      <th>angle_mean</th>
      <th>angle_vsum</th>
      <th>angle_std</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>POINT (126.99742 37.52596)</td>
      <td>27.233366</td>
      <td>169.214975</td>
      <td>120.0</td>
      <td>4308.169708</td>
      <td>65.636649</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>POINT (126.99813 37.52575)</td>
      <td>86.041833</td>
      <td>179.728535</td>
      <td>120.0</td>
      <td>1794.922276</td>
      <td>42.366523</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.0</td>
      <td>POINT (126.99926 37.52566)</td>
      <td>86.210004</td>
      <td>179.992314</td>
      <td>120.0</td>
      <td>1809.134334</td>
      <td>42.53392</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3.0</td>
      <td>POINT (126.99854 37.52566)</td>
      <td>72.718127</td>
      <td>180.01819</td>
      <td>120.0</td>
      <td>1999.990815</td>
      <td>44.721257</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4.0</td>
      <td>POINT (126.99883 37.52563)</td>
      <td>76.959394</td>
      <td>190.416309</td>
      <td>120.0</td>
      <td>2520.126478</td>
      <td>50.200861</td>
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
      <th>87769</th>
      <td>94299.0</td>
      <td>POINT (127.07564 37.61763)</td>
      <td>22.025435</td>
      <td>85.713707</td>
      <td>51.428571</td>
      <td>474.019149</td>
      <td>21.771981</td>
    </tr>
    <tr>
      <th>87770</th>
      <td>94300.0</td>
      <td>POINT (127.03887 37.55822)</td>
      <td>9.948566</td>
      <td>133.109134</td>
      <td>51.428571</td>
      <td>1393.91248</td>
      <td>37.335137</td>
    </tr>
    <tr>
      <th>87771</th>
      <td>94301.0</td>
      <td>POINT (127.03296 37.59790)</td>
      <td>16.909632</td>
      <td>92.687333</td>
      <td>51.428571</td>
      <td>758.073831</td>
      <td>27.533141</td>
    </tr>
    <tr>
      <th>87772</th>
      <td>94302.0</td>
      <td>POINT (127.07599 37.58017)</td>
      <td>26.116079</td>
      <td>75.172801</td>
      <td>45.0</td>
      <td>272.375982</td>
      <td>16.503817</td>
    </tr>
    <tr>
      <th>87773</th>
      <td>94303.0</td>
      <td>POINT (127.03544 37.56114)</td>
      <td>20.585732</td>
      <td>56.233336</td>
      <td>36.0</td>
      <td>129.446998</td>
      <td>11.377478</td>
    </tr>
  </tbody>
</table>
<p>87774 rows × 7 columns</p>
</div>




```python
Save_filePath = "./temp/20210510_geopandas2/"
Save_Npoint = "Npoint_angles.shp"

Npoint.to_file(Save_filePath + Save_Npoint)
```
