---
layout: post
title: Parking space management
subtitle: Parking space management through image recognition
categories: Graduation_Pj(Second)
tags: YOLOv5 coco dataset
---

### 알고리즘

 졸업 프로젝트를 수행하는 도중 현재 짜여진 알고리즘에 문제가 있는 것을 발견했다. 우선 저희 프로젝트 진행에는 영상 -> 객체 인식의 절차가 필수적이기 때문에 Djagno(서버) 프레임워크에서 수행되는 Yolov5를 통한 객체 인식을 위한 영상에 대한 처리가 중요하다.
 Django 내 라이브러리중 서버 컴퓨터에 연결되어 있는 웹캠에 접근할 수 있는 기능이 있다. 따라서 여러대의 카메라를 이용할 경우 다음 그림과 같은 새로운 시나리오를 구상하게 되었다.
 <p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/199650177-b577d672-1127-4b52-a2dd-9b7e8d17ebfc.png" width = 380>
</p>
 각 구역별 서버 컴퓨터와 연결된 웹캠을 통해 하나의 DB에 저장하고 각각의 영상을 API와 통신하는 본 서버에서 객체 인식을 처리하여 데이터를 전송한다.

### Yolv5

 객체 인식을 위해 처음 사용해 보는 Yolov5 알고리즘에 대해 분석해 보았다. 우선 모델을 살펴 볼 수 있다. 

 <p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/199650731-e2b6b7cb-9c80-4890-88c1-baad7ebcfd07.png" width = 380>
</p>
 출처: https://github.com/ultralytics/yolov5

 위 그림은 오픈소스의 소개되어 있는 Yolov5의 모델들이다. n, s, m, l, x의 5개의 모델이 존재하며 각각의 인식 속도, 정확성의 차이를 보인다. 우리 프로젝트에서는 주차장의 실시간 변화가 1초 단위의 급변하는 속도가 아니기 때문에 정확도에 중요성을 두어 x 모델을 사용하였다.

 ### 데이터셋
 학습 모델에 적용할 데이터는 처음에는 Yolov5에서 제공하는 cocodatset을 사용하려 했으나 여러가지 방법을 검색하던 도중 roboflow에서 제공하는 주차장에 대한 데이터셋이 있어 이를 학습해 보기로 결정하였다.
 라이브러리에서 요구하는 데이터셋의 형태를 만들기위해 데이터셋을 가공하였다.
 ```Python
from glob import glob
train_img_list = glob('/content/yolov5/pklot/train/images/*.jpg')
test_img_list = glob('/content/yolov5/pklot/test/images/*.jpg')
valid_img_list = glob('/content/yolov5/pklot/valid/images/*.jpg')
print(len(train_img_list), len(test_img_list), len(valid_img_list))
 ```
 위 코드에서 glob 명령어를 이용해 train, test, valid 데이터셋으로 나누었고 각각의 데이터 개수를 확인하였다.

 ```Python
import yaml

with open('/content/yolov5/pklot/train.txt', 'w') as f:
  f.write('\n'.join(train_img_list) + '\n')

with open('/content/yolov5/pklot/test.txt', 'w') as f:
  f.write('\n'.join(test_img_list) + '\n')

with open('/content/yolov5/pklot/valid.txt', 'w') as f:
  f.write('\n'.join(valid_img_list) + '\n')
 ```
 위 명령어를 통해 각각의 데이터셋의 대한 정보를 파일에 저장하였다.
 <p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/199651624-b681f0db-2549-47f0-8931-490e3ef339e0.png" width = 380>
</p>
 위 코드를 통해 가공한 최종 데이터 셋은 위 그림과 같다.

 ### 학습
  데이터를 Yolov5x 모델로 학습하기 위해서 컴퓨터의 성능 한계가 있어 구글에서 제공하는 코랩을 사용하기로 하였다. 무료 버전을 사용하던 도중 런타임 오류의 문제가 빈번하게 발생하여 결국 코랩 Pro를 사용하였다.
  `!python train.py --img 640 --batch 32 --epochs 50 --data ./pklot/data.yaml --cfg ./models/yolov5x.yaml --weight ' ' --name pklot_results`
  * img 640: 이미지 크기는 학습 데이터의 이미지 해상도와 최대한 비슷하게 하여 정확성을 올렸다.
  * batch 32: 배치 사이즈는 컴퓨터의 성능에 따라 달라진다. 너무 클 경우 런타임오류가 발생하여 조정해야 했다.
  * epochs 50: 학습 횟수의 조정으로 데이터가 많기 때문에 50회가 적당하다고 판단하였다.
  
  <p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/199651702-0fe50a7c-0be8-47a2-84d3-a02bf6941e7a.png" width = 380>
  </p>
  가공한 데이터셋과 model x를 이용하여 다음과 같이 학습을 완료하였다.
