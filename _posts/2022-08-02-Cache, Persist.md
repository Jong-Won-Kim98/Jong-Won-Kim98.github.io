---
layout: post
title: Cache, Persist
subtitle: Using Persist
categories: BigData
tags: Spark Persist
---

### Transformations & Actions

#### Transformations: 새로운 RDD 반환(지연 실행(Lazy Executions))

- 메모리를 최대한 활용할 수 있다
- 데이터를 다루는 task는 반복되는 경우가 많다 
- Cache(), Persist(): 데이터를 메모리에 저장해두고 사용이 가능하다

#### Cache
- default Storage Lvevel을 사용한다
- RDD: MEMORY_ONLY
- DataFrame(DF): MEMORY_AND_DISK
#### Persist
- Storage Leel을 사용자가 원하는 대로 지정이 가능하다

#### Actions

연산 결과 출력 및 저장(즉시실행(Eager Execution))

```python
from pyspark import SparkConf, SparkContext
```


```python
conf = SparkConf().setMaster("local").setAppName("restaurant-review-average")
sc = SparkContext(conf=conf)
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "restaurant_reviews.csv"
```


```python
lines = sc.textFile(f"file:///{directory}\\{filename}")
```


```python
lines.collect()
```




    ['id,item,cateogry,reviews,',
     '0,짜장면,중식,125,',
     '1,짬뽕,중식,235,',
     '2,김밥,분식,32,',
     '3,떡볶이,분식,534,',
     '4,라멘,일식,223,',
     '5,돈가스,일식,52,',
     '6,우동,일식,12,',
     '7,쌀국수,아시안,312,',
     '8,햄버거,패스트푸드,12,',
     '9,치킨,패스트푸드,23']




```python
# 첫번째 row 설명 제거
header = lines.first()
filtered_lines = lines.filter(lambda row : row != header)
filtered_lines.collect()
```




    ['0,짜장면,중식,125,',
     '1,짬뽕,중식,235,',
     '2,김밥,분식,32,',
     '3,떡볶이,분식,534,',
     '4,라멘,일식,223,',
     '5,돈가스,일식,52,',
     '6,우동,일식,12,',
     '7,쌀국수,아시안,312,',
     '8,햄버거,패스트푸드,12,',
     '9,치킨,패스트푸드,23']




```python
def parse(row):
    fields = row.split(",")
    category = fields[2]
    reviews = int(fields[3])
    
    return category, reviews
```


```python
# persist를 사용하지 않는 방식
categoryReviews = filtered_lines.map(parse)
# transformations을 수행할 RDD 생성
categoryReviews.collect()
```




    [('중식', 125),
     ('중식', 235),
     ('분식', 32),
     ('분식', 534),
     ('일식', 223),
     ('일식', 52),
     ('일식', 12),
     ('아시안', 312),
     ('패스트푸드', 12),
     ('패스트푸드', 23)]




```python
result1 = categoryReviews.take(10)
# action을 곧바로 실행
result2 = categoryReviews.mapValues(lambda x : (x, 1)).collect()
```

- categoryReivews는 resul1, result2 두번 만들어 진다
- categoryReviews에서 데이터를 꺼내올 뿐 변경은 일어나지 않기 때문에
- persist를 이용해 categoryReivews를 메모리에 넣어 놓는다


```python
result1, result2
```




    '\ncategoryReivews는 resul1, result2 두번 만들어 진다\ncategoryReviews에서 데이터를 꺼내올 뿐 변경은 일어나지 않기 때문에\npersist를 이용해 categoryReivews를 메모리에 넣어 놓는다\n'




```python
# persist 사용
categoryReviews = filtered_lines.map(parse).persist()
# categoryReviews RDD는 하나만 존재하는 RDD
categoryReviews
```




    PythonRDD[7] at RDD at PythonRDD.scala:53




```python
result1 = categoryReviews.take(10)
result2 = categoryReviews.mapValues(lambda x : (x,1)).collect()
```


```python
sc.stop()
```
