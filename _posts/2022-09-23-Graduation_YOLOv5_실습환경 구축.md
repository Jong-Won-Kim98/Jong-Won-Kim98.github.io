---
layout: post
title: Parking space management
subtitle: Parking space management through image recognition
categories: Graduation_Pj(Second)
tags: YOLOv5 Django 
---

객체 인식을 위해 우리는 PyTorch에서 지원하는 YOLOv5를 사용하기로 하였다. YOLOv5를 사용하기 위한 실습환경 구축을 실시하였다.

---
필요 프로그램
1. Python - 3.8버전 이상
2. PyTorch
3. CUDA - GPU용 병렬 컴퓨팅 플랫폼
4. YOLOv5 Repository
5. Pycham
---

### 1. Python

Python은 원래 설치되어 있어 version 확인만 하였다.

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/191902474-e5192111-aa27-4be9-8080-6a125852fe3b.png" width = 380>
</p>

- window 명령 프롬프트를 이용해 python과 anaconda 버전 확인

### 2. PyTorch

Pytroch 홈페이지에 접속할 경우 다음 그림과 같이 자신의 작업 환경에 맞는 Pytorch를 다운 받을 수 있습니다.

<p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/192978787-ae373f4e-86f5-4ae5-be5a-825e0059b453.png" width = 380>
</p>

위 그림과 같이 자신의 작업 조건과 맞는 사항을 선택한 후 밑에 나와 있는 코드를 입력합니다
저의 경우 `conda install pytorch torchvision torchaudio cpuonly -c pytorch` 를 입력했습니다.

### 3. YOLOv5

위 작업이 끝난 후 YOLOv5 파일을 다운로드 받습니다
다운 링크: https://github.com/ultralytics/yolov5
zip 파일을 다운 받은 후 압축 해제하고 python 명령프롬프트에서 yolov5-master 파일로 이동 후 파일 내에 있는 requirements.txt 파일을 pip 해줍니다.
* `pip install -r .\requirements.txt`
* requrements.txt 파일에는 YOLOv5을 실행하는데 필요한 모듈들이 저장되어 있습니다.

<p align="center">
    <img src = "https://user-images.githubusercontent.com/77920565/192981339-63a02708-aea5-4210-8830-c507be6498af.png">
</p>

저와 같은 경우 `pip install` 도중 위와 같은 오류가 발생해 구글링해본 결과 인코딩 오류였고 python 내부 디코딩 파일 내용을 변경하여 해결하였습니다.

### 4. CUDA
CUDA의 경우 그래픽 카드가 설치되어 있어야 하는 조건 사항이 있어 노트북으로 작업 환경을 만드려고 했던 터라 집에 있는 데스크 탑으로 옮겨서 다시 시도해볼 예정입니다.

### 5. Pycham

Pycham: YOLOv5 내부 cocodataset과 코드를 확인하기 위한 Tool로 실시간으로 영상에서 확인된 객체를 확인할 수 있다.
