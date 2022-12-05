---
layout: post
title: Parking space management
subtitle: Parking space management through image recognition
categories: Graduation_Pj(Second)
tags: YOLOv5
---

Yolov5라는 객체 인식 프레임 워크를 여러 방면으로 공부해보고 처음 사용해 보면서 처음 계획했던 과정과 많이 변했다. 우선 첫번 째로 저번 포스트에서 사용하려 했던 학습 데이터의 경우 정확도 측면에서 너무 떨어진다 판단하여 인식한 객체에 대한 Label을 기준으로 판단하기로 결심했다. 과정은 다음과 같다.

1. 테스트 동영상을 같은 프레임으로 이미지화
2. 이미지에 대한 주차구역 좌표면에 대한 좌표 검색
3. 이미지마다 객체 인식 시 생성되는 exp 파일의 label 값 사용
4. 객체가 해당 좌표에 인식될 경우 주차 불가로 판단

### 테스트 동영상을 같은 프레임으로 이미화

```Python
import cv2

cap = cv2.VideoCapture('C:/Graduation_Pj/yolov5-master/data/images/pest.mp4')  # 동영상 불러오기
num = 0

while (cap.isOpened()):
    ret, frame = cap.read()
    if ret:
        cv2.imshow('frame', frame)
        # 이미지의 각 이름을 자동으로 지정
        path = 'C:/Graduation_Pj/yolo/final_pklot/yolov5-master/data/images/snapshot_661.jpg'
        cv2.imwrite(path, frame)  # 영상 -> 이미지로 저장
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    num += 1

cap.release()
cv2.destroyAllWindows()
```

- 이미지화는 cv를 이용하였다. 미리 찍어둔 동영상(640*640)을 이용해 같은 프레임의 이미지로 저장하였다.

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/205485185-f322d04a-2984-4c35-a5f0-5d3ebfdbac93.png" width = 380>
</p>

- 결과는 위 그림과 같이 기본 저장 이름 snapshot_num를 지정하여 jpg 파일로 저장 되었다 여기서 num은 검색해본 결과 1프레임에 해당하는 결과 값으로 대략 44초의 영상이 660 프레임이기 때문에 1초에 15프레임을 추측할 수 있다.

### 이미지에 대한 주차구역 좌표면에 대한 좌표 검색

```Python
import cv2

img = cv2.imread('C:/Graduation_Pj/yolo/final_pklot/yolov5-master/data/images/snapshot_650.jpg')

x_pos,y_pos,width,height = cv2.selectROI("location", img, False)
print("x position, y position : ",x_pos, y_pos)
print("width, height : ",width, height)

cv2.destroyAllWindows()
```
- 위에서 자른 한개의 이미지는 동영상 프레임과 같은 프레임이기 때문에 동영상에서 해당하는 좌표면과 같다고 볼 수 있다 따라서 아래 이미지를 기준으로 한개의 주차장에 대한 모든 좌표면을 구하였다. 

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/205485325-33a94efd-e94f-4e0d-8cf3-6e8581d9366c.jpg" width = 380>
</p>

### 이미지마다 객체 인식 시 생성되는 exp 파일의 label 값 사용

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/205485389-0a29229f-6e27-4375-8576-0adb82c638a4.png" width = 380>
</p>

- Yolov5의 detect 파일을 실행할 때 --save-txt 옵션을 추가해줄 경우 위 그림과 같이 runs/dtext/exp/labels라는 폴더가 생기가 이미지의 이름과 함께 txt 파일이 생성된다.

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/205485447-9e0602c3-28e6-4012-8f36-2b2ed7c2bbe6.png" width = 380>
</p>

- txt 파일의 내용은 다음과 같고 한줄의 순서대로 class 이름, 객체의 중점 x좌표, y좌표, 넓이, 길이 순으로 저장된다. 우리는 이 생성되는 label의 내용을 이용하였다.

### 최종 코드

Yolov5의 detect 파일과 이를 이용해 결과 값이 label을 이용하여 객체를 인식하고 API에서 가시화 하기 위해 Firebase에서 제공하는 Realtime database를 이용하여 저장하고 이를 실시간으로 주고받았다.

```Python
import os
import sys
import time
import firebase_admin
from firebase_admin import credentials
from firebase_admin import db

tmp_count_filename = 0
tmp_count_exp = 2
img_source_original = "snapshot_"
tmp_reading_txt = " "
count_occupied = 0
count_empty = 0
cred = credentials.Certificate(
        "C:/Graduation_Pj/yolo/final_pklot/yolov5-master/yolov5-f7e84-firebase-adminsdk-7dbnc-6e25b09e81.json")
firebase_admin.initialize_app(cred, {
        'databaseURL': 'Firebase 저장 경로'
})
ref = db.reference()

sys.stdout = open('output1.txt', 'at')

while tmp_count_filename < 663:

    print("filename : ", tmp_count_filename)

    img_name = img_source_original + str(tmp_count_filename)

    tmp_command = 'python detect.py --img 640 --weight yolov5x.pt --source "data/images/' + img_name + '.jpg" --view-img --save-txt --line-thickness 2 --hide-labels --hide-conf'

    os.system(tmp_command)
    #time.sleep(5)

    tmp_reading_txt = "runs/detect/exp" + str(tmp_count_exp) + "/labels/" + img_name + ".txt"
    original_txt = open(tmp_reading_txt, 'r')

    data_tmp = original_txt.read().splitlines()
    data_list = []
    tmp_list = []
    tmp_list2 = []
    SPOT_list = [0 for i in range(12)]
    final_list = []
    final_list_width = []
    tmp = 0

    for i in range(len(data_tmp)):
        tmp_list = data_tmp[i]
        data_list.append(data_tmp[i])

    for i in range(0, len(data_tmp), 1):
        tmp_list2 = data_list[i].split(" ")
        for j in range(1, 3, 1):
            tmp = round(float(tmp_list2[j]) * 640)
            final_list.append(tmp)

    for i in range(0, len(final_list), 2):
        # SPOT1
        if final_list[i] in range(282, 360) and final_list[i + 1] in range(323, 375):
            print("SPOT1 occupied")
            SPOT_list[0] = 1

        # spot2
        elif final_list[i] in range(357, 428) and final_list[i + 1] in range(321, 372):
            print("SPOT2 occupied")
            SPOT_list[1] = 1

        # spot3
        elif final_list[i] in range(357, 505) and final_list[i + 1] in range(319, 372):
            print("SPOT3 occupied")
            SPOT_list[2] = 1

        # spot4
        elif final_list[i] in range(485, 853) and final_list[i + 1] in range(319, 370):
            print("SPOT4 occupied")
            SPOT_list[3] = 1

        # spot5
        elif final_list[i] in range(552, 639) and final_list[i + 1] in range(317, 368):
            print("SPOT5 occupied")
            SPOT_list[4] = 1

        # spot6
        elif final_list[i] in range(70, 150) and final_list[i + 1] in range(427, 477):
            print("SPOT6 occupied")
            SPOT_list[5] = 1

        elif final_list[i] in range(131, 264) and final_list[i + 1] in range(408, 498):
            print("SPOT7 occupied")
            SPOT_list[6] = 1

        # spot8
        elif final_list[i] in range(231, 344) and final_list[i + 1] in range(407, 498):
            print("SPOT8 occupied")
            SPOT_list[7] = 1

        elif final_list[i] in range(332, 434) and final_list[i + 1] in range(407, 496):
            print("SPOT9 occupied")
            SPOT_list[8] = 1

        # spot10
        elif final_list[i] in range(451, 518) and final_list[i + 1] in range(430, 475):
            print("SPOT10 occupied")
            SPOT_list[9] = 1

        # spot11
        elif final_list[i] in range(511, 637) and final_list[i + 1] in range(404, 500):
            print("SPOT11 occupied")
            SPOT_list[10] = 1

    for i in range(0, 12, 1):
        if int(SPOT_list[i]) == 1:
            count_occupied += 1
        else:
            count_empty += 1
        print("SPOT", i + 1, ":", int(SPOT_list[i]))

    print("Occupied : ", count_occupied)
    print("Empty : ", count_empty, "\n")

    #초기화
    count_occupied = 0
    count_empty = 0

    tmp_count_filename += 65
    # 1초: 15
    tmp_count_exp += 1
    dictionary = {(i+1): str(SPOT_list[i]) for i in range(0, len(SPOT_list))}
    print(dictionary)
    #with open('C:/Graduation_Pj/yolo/final_pklot/yolov5-master/data/persons.json', 'w') as f:
    #    json.dump(dictionary, f, indent=4)
    ref = db.reference('pklot/test1')
    ref.update(dictionary)
sys.stdout.close()
```

- 기본적인 프로그램 생명 주기는 먼저 이미지에 대한 기본 이름, filename, label읠 txt 파일을 읽기 위한 변수, 주차 공간에 대한 occupied, empty 저장 변수, firebase에 저장하기 위한 url과 json 파일을 정의해 주었다.
- 총 나누어진 이미지의 수가 660장 이기 때문에 while 반복문을 이용하여 시스템이 돌아 갈 수 있도록 하였다.
- os.system()함수를 이용해 Yolov5의 detect.py 파일을 실행 시켰고 한번 실행할 때마다 위에서 설명한 exp 파일이 생성되고 그 밑에 있는 label 파일의 txt 파일로 접근하였다
- final_list는 우리가 접근한 label의 txt 파일에서 인식한 객체에 대한 x, y 중심 좌표를 저장한 공간으로 한 객체에 대해 2개의 좌표가 있기 때문에 2를 주기로 for 반복문을 실행 시켰다.
- final_list에 저장되어 있는 인식한 객체에 대한 좌표값 만큼 해당 주차면 좌표에 객체가 인식될 경우 0으로 초기화 되어있는 SPOT_list[]의 값을 1로 변경하여 자동차가 이미 주차되어 있음을 판단하였고 occupied 변수에 1을 증가시켜 주었다.
- 원래는 이 결과 값(SPOT_list[])를 Json 파일로 저장하기 위해 dictionary를 생성하여 이를 저장하였지만 firebase에 실시간으로 저장하기 위해 바로 접근하여 저장하였다. 

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/205486030-eb818887-46d7-40b0-8754-f2dd0f477a2f.png" width = 700>
</p>

- 그림과 같이 지정한 주차 구역 별로 실시간으로 firbase의 값이 변하는 것을 볼 수 있었다. 이제 테스트 영상이 아닌 실제 구현 영상을 바탕으로 많은 차들의 변화를 인식하여 저장해서 그것을 APP으로 가시화해볼 생각이다.
