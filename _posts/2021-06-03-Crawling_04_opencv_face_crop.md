---
layout: single
title: "#Face cropping with Python: OpenCV2"
---


### 경로에 있는 이미지에 대해 얼굴을 추출하고, 일정한 크기로 다시 저장하는 함수

##### opencv-python API 사용 (cv2)


```python
# !pip install opencv-python
import cv2
import os
import numpy as np
```

#### CascadeClassifier 파일 로드
###### "https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml"


```python
# Load the cascade
face_cascade = cv2.CascadeClassifier('./datas/haarcascade_frontalface_default.xml')
```

#### 이미지에서 얼굴부분 추출, 일정한 크기로 저장하는 함수


```python
def face_crop_cv2(imageName, filePath=".", savePath="."):
    if not os.path.exists(savePath):
        os.makedirs(savePath)
    
    filePath = filePath + "/" + imageName
    saveName = str(imageName.encode("UTF-8")).replace("\\", "").replace("'", "")
    savePath = savePath + "/" + saveName
    stream = open(filePath.encode("utf-8"), "rb")
    bytes = bytearray(stream.read())
    numpyArray = np.asarray(bytes, dtype=np.uint8) ### file name ###
    
    ######################################################################
    print(filePath + " → " + savePath + "/...", end="")
    ######################################################################
    try:
        # img = cv2.imread(numpyArray)
        img = cv2.imdecode(numpyArray , cv2.IMREAD_UNCHANGED)

        # Convert into grayscale
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        # Detect faces
        faces = face_cascade.detectMultiScale(gray, 1.1, 4)
        # Draw rectangle around the faces and crop the faces

        face_boundary_ratio = 0.1

        if len(faces) > 0: # if face is detected
            for (x, y, w, h) in faces:
                y1 = int(y-face_boundary_ratio*h)
                y2 = int(y + (1+face_boundary_ratio)*h)
                x1 = int(x-face_boundary_ratio*w)
                x2 = int(x + (1+face_boundary_ratio)*w)

                # cv2.rectangle(img, (x1, y1), (x2, y2), (0, 0, 255), 0)
                faces = img[y1:y2,x1:x2]
                resized_faces = cv2.resize(faces, dsize=(400, 400), interpolation=cv2.INTER_AREA)
                cv2.imwrite(savePath, resized_faces)
                # cv2.imshow("face",resized_faces)
                print("[o]", end="")

            ### Display the output
            # cv2.imwrite('detcted.jpg', img)
            # cv2.imshow('img', img)
            # cv2.waitKey()
            
        else: # if face isn't detected
            print("[x]", end="")
            pass
        
    except:
        print("[x]", end="")
        pass
    ######################################################################
    print("")
```

#### 특정 경로에 있는 모든 이미지에 대해 함수 실행


```python
def print_file(data_dir):
    files = os.listdir(data_dir)
    for item in files :
        if os.path.isdir(data_dir+"/"+item) == True :
            print_file(data_dir+"/"+item)
        else:
            face_crop_cv2(item, data_dir, "Croped_"+data_dir)
```

#### 실행


```python
print_file("LearningData")
```
