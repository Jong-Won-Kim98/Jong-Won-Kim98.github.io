---
layout: post
title: Machine Learning Principle
subtitle: MLlib
categories: Machine_Learning
tags: machine_learning
---

### MLlib
- Machine Learning Library
- 머신러닝을 확장성 있게 적용하기 위한 라이브러리
- 데이터 프레임과 연동이 쉽고 머신러닝 개발을 빠르게 할 수 있다.
  
![MLlib 원리](https://user-images.githubusercontent.com/77920565/185777407-051183d0-2407-4ce1-9242-d74110915d61.png)

- DataFrame API를 기반으로 작동된다.
- RDD를 기반으로 적용 가능한 MLlib도 존재 했었지만 데이터프레임을 이용한 구조로 통합되어 현재는 RDD 기반으로는 MLlib를 사용할 수 없다.

### Machine Learning
    - 많은 양의 데이터를 기반으로 데이터의 패턴을 분석하는 학문
    - 프로그래밍 구문에 의한 프로그램이 만들어 지는 것이 아닌 데이터 패턴을 분석하는 학문
    - ex) Data -> Model -> y(결과값)

### MLlib Component
    - y = f(x)
      - y: Target, Label
      - f(): Model
      - x: Feature Engineering

#### 알고리즘
  - Classification(분류): 카테고리를 바탕으로 수치로 표현 
  - Regression(회기): 강수량, 연봉 등과 같이 실수를 바탕으로 수치 파악
  - Clustering(군집): 비지도 학습의 일종으로 특정 기준을 바탕으로 군집 현성
  - Recommendation(추천)

#### 파이프라인
  - Training(훈련)
  - Evaluating(평가)
  - Persistence(전처리)
  - 데이터를 가공할 어떤 모델에 대한 오류, 결측치 등을 판단하는 과정
  - 데이터 로딩 -> 전처리 -> 학습 -> 모델 평가

#### Feature Engineering
  - Extraction
  - Transfromation
  - 입력된 데이터를 처리하기 위한 engineering

#### Utils
  - Linear Algebra(환영 대수)
  - Statistics(확률)

### 파라미터 튜닝
  - 모델 파라미터: 머신 러닝 알고리즘(학습 대상)
  - 하이퍼 파라미터: 사람이 모델에게 적용하는 알고리즘

### Transformer
  - Feature Transformation을 담당
  - 모든 Transformer는 transform() 함수를 가지고 있다
  - 머신러닝이 가능한 형태로 데이터의 포맷을 바꿔준다(문자열 형식의 데이터를 범주를 이용해 정수형으로 읽어 들일 수 있도록 변환)
  - 원본 DF를 변형해 새로운 Df를 생성한다(보통 하나 이상의 컬럼을 더하는 작업)

### Estimator
  - 모델의 학습을 담당한다
  - 모든 Estimator는 fit() 함수를 가지고 있다
  - fit()은 DataFrame을 입력 받아 학습한 다음 Model을 반환한다
  - 모델은 하나의 Transformer라고 볼 수 있다

### Pipeline

![Pipeline](https://user-images.githubusercontent.com/77920565/185779149-141c9404-f8c2-449d-a5a3-bef99b145a81.png)

  - 머신러닝의 전체적인 워크 플로우를 연결시켜 주는 역할을 한다.
  - 여러 stage를 담고 있다
    - 각 stage 마다 담당하는 과정이 있으며 stage는 로딩, 전처리, 학습, 모델 평가 등 각각의 과정들을 담당한다
