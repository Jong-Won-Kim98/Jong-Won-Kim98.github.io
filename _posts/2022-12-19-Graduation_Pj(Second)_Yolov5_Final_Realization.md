---
layout: post
title: Parking space management impression
subtitle: Parking space management through image recognition
categories: Graduation_Pj(Second)
tags: YOLOv5
---

### 결과

앞 포스터에서 게시글에서 테스트를 마치고 마지막 결과물을 위해 더 넓은 범위에서 실행을 했다.

* 실행 화면
 <iframe width="729" height="498" src="https://www.youtube.com/embed/Ay0GgrNaSG8" title="졸업 프로젝트 시연 영상" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### 오류 사항

#### 스레드, 속도 관리 

먼저 객체 인식을 시행하면서 제일 어려웠던 부분은 detect.py 파일의 내용을 파악하는 일이였다. 처음에는 파일 내부에서 코드를 수정하여 인식한 객체에 대한 label에 접근 하였다.

 ```Python
for *xyxy, conf, cls, in reversed(det):
    if save_txt: #Write to file
    xyxw = (xyxy2xyxh(torch.tensor(xyxy).view(1, 4))/gn).view(-1).tolist()
    line = (cls, *xyxh, conf) if save_conf else (cls, *xyxh)
    with open(txt_path + '.txt', 'a') as f:
        f.write(('%g' * lne(line)).rstrip() % line + '\n')
        if save_img or save_crop or view_img:
            label = None if hide_labels else (names[c] if hide_conf else f'{names[c]} {conf:.2f}')
            annotator.box_label(xyxy, label, color=colors(c, True))
        if save_crop:
            save_one_box(xyxy, imc, file=save_dir /'crops'/names[c]/f'{p.stem}.jpg', BGR=Ture)
 ```
위 코드는 Yolov5 detect.py 파일 내의 result를 도출할 때 어떤 것을 저장할지에 대한 부분으로 여기서 바로 값을 가져와 주차장 좌표 범위 안에 있는지를 비교하는데 많은 시간이 소요되고, 쓰레드 문제를 해결하지 못하고 시간이 오래 걸려 따로 파일을 만들어 실행하였다. 이번 프로젝트에서 제일 관건은 실제 영상과 데이터를 생성하는데 걸리는 시간이 어느정도 맞아야 하는 점이 중요했다. 따라서 파일 경로를 절대 파일 경로로 바꾸는 등 최대한 시간 절약을 위해 많은 공부가 되었다.

#### 최적의 conf 값 도출
   
위 detect.py의 명렁어를 보면 --conf 에 대한 설정을 주는 부분이 있는데 적절한 conf 값을 찾는 것에도 많은 시도가 필요했다.
 <p align="center">
 <img src = "https://user-images.githubusercontent.com/77920565/208360868-36f85014-b863-4be7-96e5-3c6691d1e139.png" width = 380>
</p>
conf 기준을 너무 낮게 잡을 경우 그림의 왼쪽 상단 부분과 같이 바닥에 쌓여있는 흰 눈을 차로 인식하는 경우가 발생하였습니다. 이는 넓은 면을 촬영하기 위해 카메라의 위치가 높이(멀리) 위치하고 있고, 카메라의 화질, 유리를 관통하여 촬영한 점 등의 한계가 있어 이런 상황이 발생 했다. 이에 저희는 어쩔 수 없는 상황을 받아 들이고 최종 데이터에 방해가 되지 않고 객체의 정확도를 최대한으로 발현할 수 있는 conf를 찾기위해 노력했다. 흰색 차의 경우 햇빛, 눈, 밝기 등에 영향을 받고 검은색 차량의 경우 어두운 정도 그림자에 영향을 받아 생각보다 최적의 conf를 찾는 것이 쉽지 않았고 모든 시간별 이미지의 객체의 정확도를 확인하며 최적의 값을 검색했다.

#### 주차 여부를 판단하는 좌표의 넓이
 객체의 유무를 판단하기 위해서 주차장의 좌표를 검색 하였었는데 이를 너무 넓게 잡는 경우 자동차가 이동하는 경우에도 해당 좌표 범위 안에 들어가 주차가 되어 있음으로 인식하는 경우가 발생하여 주차 면적의 좌표를 줄임과 동시에 값을 변경해 가며 인식이 되는 경우와 되지 않는 경우를 살펴 가며 최적의 좌표 값을 검색했다.


### 느낀점
 이번 프로젝트는 모르는 사람과 처음으로 합을 맞춰 하는 팀 프로젝트였기 때문에 각자 개인의 실력도 아무것도 알지 못하는 분들과 만나 처음 도전해 보는 객체 인식에 대한 분야를 도전했다. 그만큼 어려움도 팀장으로서 모르는 팀원을 도와주는 일도 모두 하기에 어려움이 있었기 때문에 프로젝트의 원래 계획을 변경해야 하는 일도 발생했다. 하지만 처음 도전에 대한 두려움을 한번 더 극복할 수 있었던 좋은 경험이였고, 많은 개인적인 환경의 변화와 일들이 있었지만 그래도 마무리 해야 한다는 책임감에 어떻게든 마무리 하겠다는 의지가 제일 빛났던 시기였던 것 같다. 팀원과의 프로젝트가 많은 면에서 힘들었지만 정말 좋은 경험을 하고 실력도 한 층 성장한 계기라고 생각하기에 만족한다.
 