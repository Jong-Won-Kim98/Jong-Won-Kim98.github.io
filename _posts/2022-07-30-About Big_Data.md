---
layout: post
title: About Big_Data Engineering
subtitle: Big_Data Engineering Theory
categories: BigData
tags: Big_Data engineering 
---

## Big_Data engineering

### 빅데이터 엔지니어링

빅데이터 기반 의사결정을 만들기 위한 인프라 구성, 인사이트 추출

### 과거 데이터 아키텍쳐(문제점)
1. 시스템 구축에 비용이 많이 든다
2. 데이터의 용도가 정해져있다
3. 데이터 수집처가 일정하다

### GIGO(Garbege In Garbage Out)

좋은 데이터를 수집하고 잘 관리하여 처리하는 것이 훨씬 효율적이다

### ETL

데이터를 추출한 후 선 저장하고 쓰임에 따라 변환한다

#### Extract(추출)

기존의 DB에서 데이터를 가져온다

#### Transform(변환)

미리 정해 놓은 스키마에 맞게 데이터를 변환한다

#### Load(적재)

변환이 완료된 데이터를 원하는 스키마에 INSERT하는 과정

### ELT

최근에는 데이터를 추출하여 저장한 후 쓰임에 따라 데이터를 가공한다

### 데이터 아키텍처 분야

#### 소스

비지니스와 운영 데이터 생성

#### 수집 및 변환

ELT

#### 저장

데이터를 시스템이 사용할 수 있도록 처리 후 저장, 비용과 확장성 면으로 최적화

#### 과거, 미래 예측

1. 저장된 과거 데이터를 통해 인사이트 생성(Query)
2. 쿼리를 실행하고 필요시 분산 처리(Processing)
3. 과거에 일어난 일, 미래에 일어날 일 추측(Machine Learning)

![Dataflow](https://user-images.githubusercontent.com/77920565/182418566-b387fb1b-2867-4027-bd96-c629dbe84df0.png)

### Batch Processing(다수 동시)

Batch: 일괄, Processing: 처리

1. 많은 양의 데이터를 정해진 시간에 한번에 처리
2. 전통적으로 사용한 데이터 처리 방법
3. 실시간성을 보장하지 않아도 될 때 사용
4. 무거운 처리시 사용
5. 마이크로 배치: 데이터를 조금씩 모아서 프로세싱하는 방식

### Stream Processing(실시간)

1. 실시간으로 쏟아지는 데이터를 계속 처리하는 방식
2. 이벤트가 생길 때, 데이터를 들어올 때 마다 처리
3. 불규칙적으로 데이터가 들어오는 환경
