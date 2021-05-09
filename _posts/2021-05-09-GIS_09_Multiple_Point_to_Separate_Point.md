---
layout: single
title: "#GIS_09 MultiPoints to Separate Points"
---

# Change muiltipoints from Arcgis to points shp file


```python
import pandas as pd
import os
import geopandas
import matplotlib.pyplot as plt

# set warning disable
pd.options.mode.chained_assignment = None  # default='warn'
```

### Read shp file from Arcgis


```python
def read_shapefile_in_dir(dir="."):
    files = os.listdir(dir)
    shapefile = list(filter(lambda shp: '.shp' in shp, files))[0]
    return geopandas.read_file(dir + '/' + shapefile)
```


```python
Networks_Intersect_DeleteIdentical_dir = "temp/custom_arcgis/Seoul_Newworks/Networks_Intersect_DeleteIdentical"
Networks_Intersect_DeleteIdentical = read_shapefile_in_dir(Networks_Intersect_DeleteIdentical_dir)
Networks_Intersect_DeleteIdentical.plot()
```




    <AxesSubplot:>




![png](\assets\images\2021-05-09\output_4_1.png)
    


### Change muiltipoint object to point


```python
points = []

multipoints = Networks_Intersect_DeleteIdentical["geometry"]
for multipoint in multipoints:
    for point in multipoint:
        points.append(point)
    
df_point = pd.DataFrame(points, columns = {"geometry"})
gdf_point = geopandas.GeoDataFrame(df_point)
gdf_point.crs = ({'init' :'epsg:4326'})

print(gdf_point.crs)
gdf_point.head(3)
```

    {'init': 'epsg:4326'}
    




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
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (126.99742 37.52596)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (126.99813 37.52575)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (126.99926 37.52566)</td>
    </tr>
  </tbody>
</table>
</div>



### Save shp file


```python
Save_filePath = "./temp/20210509_geopandas/"
Save_fileName = "points.shp"

gdf_point.to_file(Save_filePath + Save_fileName)
```
