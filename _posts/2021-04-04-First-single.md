---
layout: single
title:  "First single"
---

```python
import os
import geopandas
import pandas as pd
import re

BASEPATH = "dbset/db_statistics_2/"
RESULTPATH = "dbset/db_reshaped/"
```


```python
# get encoding function
import chardet

def get_file_encoding(src_file_path):
    with open(src_file_path) as src_file:
        return src_file.encoding

def get_file_encoding_chardet(file_path):
    with open(file_path, 'rb') as f:
        result = chardet.detect(f.read())
        return result['encoding']
```


```python
def getsig_name(region = '서울특별시'):
    # code dataframe
    region_code = pd.read_excel('dbset/한국행정구역분류_2021.1.1.기준.xlsx', sheet_name = 2, header = None)
    region_code = region_code[region_code[0] == region]
    region_code = region_code[region_code[1] != region]
    
    gu_code = region_code[[1, 8]].drop_duplicates(1)
    gu_code[8] = gu_code[8].astype(str).str[:5]
    gu_code = gu_code.rename(columns={1:"GU_NAME", 8:"CODE_SIG"})
    sig_name = gu_code.set_index('CODE_SIG')
    
    return sig_name

def getsig_shp():
    FilePath = 'dbset/db_shp'
    FileName_emd = '/SIG_202101/TL_SCCO_SIG.shp'
    
    shp_sig = geopandas.read_file(FilePath + FileName_emd, encoding='EUC-KR')
    shp_sig = shp_sig[['SIG_CD', 'geometry']]
    shp_sig = shp_sig.rename(columns={'SIG_CD':'CODE_SIG'})
    shp_sig = shp_sig[shp_sig['CODE_SIG'].str.startswith('11')] # 11 = Seoul code
    shp_sig = shp_sig.set_index('CODE_SIG')
    
    return shp_sig

sig_shp, sig_name = getsig_shp(), getsig_name()
```


```python
def setReshapedData(FolderName, extension = "csv", encoding = "UTF-8", basePath = BASEPATH, resultPath = RESULTPATH):
    rootPath = basePath + FolderName
    resultFilePath = resultPath + FolderName
    
    year_df_list = []
    yearRanges = []
    for roots, dirs, files in os.walk(rootPath):
        if (roots == rootPath):
            for file in files:
                if re.search('-', file):
                    filePath = roots + '/' + file
                    fileName, fileExt = file.split('.')
                    yearRange = [int(i) for i in fileName.split('-')]
                    yearRanges.append(yearRange)

                    Range_df = pd.read_csv(filePath, sep="\t",
                                           low_memory=False,
                                           encoding=get_file_encoding_chardet(filePath),
                                           header=None)

                    for year in range(yearRange[1], yearRange[0] + 1, 1):
                        temp_df = Range_df[Range_df[0] == str(year)]
                        temp_columns = Range_df[:(Range_df.iloc[:, 0] == str(yearRange[1])).idxmax()].T
                        temp_columns.iloc[0, :] = 'Year'
                        temp_columns.iloc[1, :] = 'sigName'
                        temp_df.columns = temp_columns
                        year_df_list.append(temp_df)

    # make save folder
    if not os.path.exists(resultFilePath):
        os.makedirs(resultFilePath)

    # get total dataframe and save
    total_df = pd.concat(year_df_list).reset_index(drop=True)
    total_df.to_csv(resultFilePath + "/" + str(yearRanges[-1][0]) + '-' + str(yearRanges[0][-1]) + '.' + extension, encoding=encoding, index = False)

    # get separate dataframe and save
    for i in range(len(sig_name)):
        region_df = total_df[total_df.iloc[:, 1] == sig_name.iloc[i, 0]]
        region_df.to_csv(resultFilePath + "/" + sig_name.iloc[i, 0] + '.csv', encoding=encoding, index = False)

    for yearRange in yearRanges:
        for year in range(yearRange[0], yearRange[1] - 1, -1):
            year_df = total_df[total_df.iloc[:, 0] == str(year)]
            year_df.to_csv(resultFilePath + "/" + str(year) + '.csv', encoding=encoding, index = False)
```


```python
# for roots, dirs, files in os.walk(BASEPATH):
#     if (roots != BASEPATH):
#         print(roots.replace(BASEPATH, ""))
#         setReshapedData(roots.replace(BASEPATH, ""))
```


```python
def getReshapedData(dataPath, target, extension = "csv", encoding="UTF-8"):
    temp_df = pd.read_csv(RESULTPATH + dataPath + '/' + str(target) + "." + extension,
                       low_memory=False, encoding=encoding, header=None)
    
    if isinstance(target, int):
        df = temp_df[temp_df[0] == str(target)]
        df.columns = temp_df[:(temp_df.iloc[:, 0] == str(target)).idxmax()].T
    if isinstance(target, str):
        df = temp_df[temp_df[1] == str(target)]
        df.columns = temp_df[:(temp_df.iloc[:, 1] == str(target)).idxmax()].T
    
    return df
```


```python
def getPriceData(target):
    FolderPath = 'dbset/db_statistics_1'
    FolderName = '서울_도시관리_서울시 개별공시지가 정보'
    
    FileName = str(target) + '.csv'
    Path = FolderPath + '/' + FolderName + '/' + FileName
    df = pd.read_csv(Path, sep=",",
                     low_memory=False,
                     encoding='UTF-8',
                     header=0)
    return df
```
