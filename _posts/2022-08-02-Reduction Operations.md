---
layout: post
title: Reduction Operations
subtitle: Reduction
categories: BigData
tags: Reduction Operations
---

### Reduce
- RDD.reduce(<func>)
- 사용자가 지정하는 함수를 받아 여러 개의 값을 하나로 줄인다

```python
from pyspark import SparkConf, SparkContext
```


```python
conf = SparkConf().setMaster("local").setAppName("reduction-op")
sc = SparkContext(conf=conf)
```


```python
from operator import add
```


```python
sample_rdd = sc.parallelize([1, 2, 3, 4, 5])
sample_rdd
```




    ParallelCollectionRDD[0] at readRDDFromFile at PythonRDD.scala:274




```python
sample_rdd.reduce(add)
```




    15



* 파티션에 따라 결과물이 달라지기 때문에 분산된 파티션들의 연산과 합치는 부분을 나눠서 생각해야 한다

### lambda 조건식 계산 방법
1. [1, 2, 3, 4] 순서대로 파티션을 나눈다
2. 앞 파티션의 계산 결과를 새로운 x로 받아들여 계산한다
    ex) (1*2)+2=4 -> (4*2)+3=11 -> (11*2)+4=26


```python
sample_rdd = sc.parallelize([1, 2, 3, 4])
sample_rdd.reduce(lambda x, y : (x*2)+y)
```




    26




```python
# 파티션 1개로 지정
sc.parallelize([1, 2, 3, 4], 1).reduce(lambda x, y : (x*2)+y)
# [1, 2, 3, 4]가 들어가 있는 파티션을 1개 만든다
```




    26




```python
# 파티션 2개로 지정
sc.parallelize([1, 2, 3, 4], 2).reduce(lambda x, y : (x*2)+y)
# 파티션 3개로 지정
sc.parallelize([1, 2, 3, 4], 3).reduce(lambda x, y : (x*2)+y)
# 파티션에 데이터가 한개만 존재할 경우 reduce가 일어나지 않는다
```




    '\n2개의 경우\n각각의 파티션 [1, 2], [3, 4]가 생성되고 각각의 파티션에서 연산이 일어난 후\n결과를 바탕으로 마지막 연산이 이루어진다.\n3개의 경우\n데이터의 개수가 1개인 파티션들끼리 연산 후 데이터가 두개인 파티션을\n연산한 후 결과를 바탕으로 남은 연산을 진행한다\n'

### 파티션 개수에 따른 계산

#### 2개의 경우
    - 각각의 파티션 [1, 2], [3, 4]가 생성되고 각각의 파티션에서 연산이 일어난 후 결과를 바탕으로 마지막 연산이 이루어진다
#### 3개의 경우
    - 데이터의 개수가 1개인 파티션들끼리 연산 후 데이터가 두개인 파티션을 연산한 후 결과를 바탕으로 남은 연산을 진행한다

### Fold
- RDD.fold(zeroValue, <func>)
- reduce와 비슷하지만, zeroValue에 넣어놓고 싶은 시작값을 지정해서 reduce가 가능하다
- zeroValue는 파티션 마다 연산되는 값


```python
rdd = sc.parallelize([2, 3, 4], 4)
print(rdd.reduce(lambda x, y : (x*y)))
# 2*3 -> *4
print(rdd.fold(1, lambda x, y : (x*y)))
# 1*2*3*4
```

    24
    24
    


```python
print(rdd.reduce(lambda x, y : x+y))
print(rdd.fold(1, lambda x, y : x+y))
'''
fold의 시작값은 파티션 마다 부여된다
따라서 [1+1]+[2+1]+[3+1]+[4+1]의 연산 결과 값이 나온다
따라서 reduce의 연산 값은 9가 나오고
'''
```

    9
    14
    




    '\nfold의 시작값은 파티션 마다 부여된다\n따라서 [1+1]+[2+1]+[3+1]+[4+1]의 연산 결과 값이 나온다\n따라서 reduce의 연산 값은 9가 나오고\n'



### GroupBy
- RDD.groupBy(<func>)
- 그룹핑 함수를 받아 reduction


```python
rdd = sc.parallelize([1, 1, 2, 3, 5, 8])
result = rdd.groupBy(lambda x : x%2).collect()
sorted([x, sorted(y)] for (x,y) in result)
```




    [[0, [2, 8]], [1, [1, 1, 3, 5]]]



### Aggregate(action)
- RDD.aggregate(zeroValue, seqOp, combOp)
- zeroValue: 각 파티션에서 누적한 시작 값
- seqOp: 타입 변경 함수
    - 파티션 내에서 벌어지는 연산을 담당
- combOp: 모든 결과를 하나로 합쳐주는 연산을 담당
- 파티션 단위의 연산 결과를 합쳐주는 과정을 거치게 된다


```python
rdd = sc.parallelize([1, 2, 3, 4], 2)

seqOp = lambda x, y : (x[0] + y, x[1] + 1)
# 파티션 내 연산
combOp = (lambda x, y : (x[0] + y[0], x[1] + y[1]))
# 파티션의 모든 결과 최종 연산

print(rdd.aggregate((0, 0), seqOp, combOp))
```

    (10, 4)
    


```python
sc.stop()
```
