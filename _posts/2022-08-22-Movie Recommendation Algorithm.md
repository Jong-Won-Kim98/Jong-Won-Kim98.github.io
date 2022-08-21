---
layout: post
title: Movie Recommendation Algorithm
subtitle: Machine Learning Practice
categories: Machine_Learning
tags: machine_learning
---

### 영화 추천 알고리즘

#### 원리

- A와 B가 동일한 영화에 대해 비슷한 별점을 주었을 경우 영화 취향이 비슷하다 판단 후 상대방이 보지 않은 영화에 대해 높은 별점의 영화가 있다면 그것을 추천해 준다.

- 한 사람이 모든 영화를 볼 수 없기 때문에 모든 영화에 대한 평점을 알 수 없다, 따라서 비슷한 영화 취향을 가지고 있는 사람들을 비교하여 서로 평점을 주지 않은 영화에 대한 평점을 예측하여 이를 바탕으로 추천한다.

#### Matrix

![matrix](https://user-images.githubusercontent.com/77920565/185779618-96d8c2c6-6811-491d-9a4d-6a05f7e486b0.png)

- User Matrix: 영화에 대한 Matrix로 특정 영화를 본 user
- Item Matrix: User에 대한 Matrix로 특정 사람이 본 영화
- Rating Matix: User Matrix X Item Matrix로 표현할 수 있다
- User Matrix의 값과 Item Matrix의 값은 랜덤하게 채워지고, Item 행렬을 고정 시키고 User 행렬을 최적화 한다, 반대로 User 행렬을 고정 시키고 Item 행렬을 최적화 시킨다.

=> 결과적으로 User Matrix의 값과 Item Matrix의 결과물을 합쳐 Rating Matrix와 최대한 가까운 Matrix가 생성되게 되고, 이 Matrix는 빈 칸에 있는 값들이 모두 채워진 형식이 된다.