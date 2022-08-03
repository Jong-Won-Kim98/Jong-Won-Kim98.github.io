---
layout: post
title: Key-Value RDD
subtitle: Operations, join
categories: BigData
tags: Spark RDD Key-Value
---

### Key-Value RDD

- (Key, Value) 쌍을갖기 때문에 Pairs RDD라고도 한다
- Key를 기준으로 고차원적인 연산이 가능하다
  - Single Value RDD는 단순한 연산을 수행한다
- Key Value RDD는 다양한 집계 연산이 가능하다

### Key-Value RDD Reductin 연산

reduceByKey(<task>): key를 기준으로 task 처리
groupbyKey(): key를 기준으로 value 묶기
sortByKey(): key를 기준으로 정렬
keys(): key 추출
values(): value 추출
mapValues(): Kye에 대한 변경이 없을 경우
flatMapValues()

![reduction](https://user-images.githubusercontent.com/77920565/182549925-ce5fc0ca-da06-4ea2-b3f5-286b5577395f.png)


```python
from pyspark import SparkConf, SparkContext
conf = SparkConf().setMaster("local").setAppName("key-value_rdd_op_join")
sc = SparkContext(conf=conf)
```

### Operations
#### groupByKey
- KeyValueRDD.groupByKey()
- 그룹핑 후 특정 Transformations 같은 연산
- key 값이 있는 상태에서 시작
#### groupBy()
- RDD.groupBy(numPartitions=None, partitionFunc=<function portable_hash>)
- 함수에 의해서 그룹이 생기는 연산
- Transformations
    - 실제로 연산이 되지 않는다


```python
rdd = sc.parallelize([
    ("짜장면", 15),
    ("짬뽕", 10),
    ("짜장면", 5)
])

g_rdd = rdd.groupByKey()
# 정해진 key값을 기준으로 데이터를 모아준다
g_rdd.collect()
```




    [('짜장면', <pyspark.resultiterable.ResultIterable at 0x21d292bf850>),
     ('짬뽕', <pyspark.resultiterable.ResultIterable at 0x21d292bfa60>)]




```python
g_rdd.mapValues(len).collect()
```




    [('짜장면', 2), ('짬뽕', 1)]




```python
g_rdd.mapValues(list).collect()
```




    [('짜장면', [15, 5]), ('짬뽕', [10])]




```python
grouped = sc.parallelize([
    "c",
    "python",
    "c++",
    "java",
    "SCR"
]).groupBy(lambda x : x[0]).collect()
# groupBy는 자신의 조건 func를 넣어 key를 생성해 데이터를 묶어준다
grouped
```




    [('c', <pyspark.resultiterable.ResultIterable at 0x21d2916fe80>),
     ('p', <pyspark.resultiterable.ResultIterable at 0x21d29336b20>),
     ('j', <pyspark.resultiterable.ResultIterable at 0x21d29336b80>),
     ('S', <pyspark.resultiterable.ResultIterable at 0x21d29336c10>)]




```python
for k, v in grouped:
    print(k, list(v))
```

    c ['c', 'c++']
    p ['python']
    j ['java']
    S ['SCR']
    


```python
x = sc.parallelize([
    ("MATH", 7), ("MATH", 2), ("ENGLISH", 7),
    ("SCIENCE", 7), ("ENGLISH", 4), ("ENGLISH", 9),
    ("MATH", 8), ("MATH", 3), ("ENGLISH", 4),
    ("SCIENCE", 6), ("SCIENCE", 9), ("SCIENCE", 5)
], 3)

y = x.groupByKey()
```


```python
print(y.getNumPartitions())
```

    3
    


```python
y = x.groupByKey(2)
# 파티션의 개수 변경
y.getNumPartitions()
```




    2




```python
y.collect()
```




    [('MATH', <pyspark.resultiterable.ResultIterable at 0x21d2931af10>),
     ('ENGLISH', <pyspark.resultiterable.ResultIterable at 0x21d293361f0>),
     ('SCIENCE', <pyspark.resultiterable.ResultIterable at 0x21d29336c40>)]




```python
for t in y.collect():
    print(t[0], list(t[1]))
    #t[0]: key, t[1]: 그룹핑에 의해 묶인다
```

    MATH [7, 2, 8, 3]
    ENGLISH [7, 4, 9, 4]
    SCIENCE [7, 6, 9, 5]
    

### reduceByKey
- KeyValueRDD.reduecByKey(<func>), numPartitions = None, partitionFunc=(<function portable_hash>)
- 주어진 key를 기준으로 Group을 만들고 합친다
- Transformations 함수
- reduceByKey =  groupByKey + reduce이다.
- groupByKey보다 reduceByKey가 훨씬 빠르다


```python
from operator import add

rdd = sc.parallelize([
    ("짜장면", 15),
    ("짬뽕", 10),
    ("짜장면", 5)
])

rdd.reduceByKey(add).collect()
```




    [('짜장면', 20), ('짬뽕', 10)]




```python
x = sc.parallelize([
    ("MATH", 7), ("MATH", 2), ("ENGLISH", 7),
    ("SCIENCE", 7), ("ENGLISH", 4), ("ENGLISH", 9),
    ("MATH", 8), ("MATH", 3), ("ENGLISH", 4),
    ("SCIENCE", 6), ("SCIENCE", 9), ("SCIENCE", 5)
], 3)
x.reduceByKey(lambda a, b : a+b).collect()
```




    [('MATH', 20), ('ENGLISH', 24), ('SCIENCE', 27)]



### mapValues
- keyValueRDD.mapValues(<func>)
- 함수를 Value에만 적용한다
- 파티션과 key는 원래 위치 그대로 유지한다
- Transformations 작업이다


```python
rdd = sc.parallelize([
    ("하의", ["청바지", "반바지", "치마"]),
    ("상의", ["니트", "반팔", "나시", "긴팔"])
])
    
rdd.mapValues(lambda x : len(x)).collect()
# key가 아닌 value에만 적용할 함수를 만들 수 있기 때문에 데이터의 파티션이 변경될 걱정이 없다
```




    [('하의', 3), ('상의', 4)]



### countByKey
- KeyValueRDD.countByKey(<func>)
- 각 키가 가진 요소들의 개수를 센다
- Action


```python
rdd = sc.parallelize([
    ("하의", ["청바지", "반바지", "치마"]),
    ("상의", ["니트", "반팔", "나시", "긴팔"])  
])

rdd.countByKey()
#key를 기준으로 count
```




    defaultdict(int, {'하의': 1, '상의': 1})



### keys()
- 모든 key를 가진 RDD를 생성
- 파티션을 유지, 키가 많은 경우 Transformations 작업이다


```python
rdd.keys()
```




    PythonRDD[45] at RDD at PythonRDD.scala:53




```python
rdd.keys().collect()
```




    ['하의', '상의']




```python
x = sc.parallelize([
    ("MATH", 7), ("MATH", 2), ("ENGLISH", 7),
    ("SCIENCE", 7), ("ENGLISH", 4), ("ENGLISH", 9),
    ("MATH", 8), ("MATH", 3), ("ENGLISH", 4),
    ("SCIENCE", 6), ("SCIENCE", 9), ("SCIENCE", 5)
], 3)

print(x.keys().count())
# 단순한 키의 총 개수(중복 count)
print(x.keys().distinct().count())
```

    12
    3
    

### Joins
#### inner join 
서로간에 존재하는 키만 합쳐준다
#### outer join
- 기준인 한 쪽에는 데이터, 다른 쪽에는 데이터가 없는 경우
- 설정한 기준에 따라서 기준에 맞는 데이터가 항상 남아있는다
- leftOuterJoin: 왼쪽에 있는 rdd가 기준이 된다(함수를 호출하는 rdd)
- rightOuterJoin: 오른쪽에 있는 rdd가 기준이 된다(함수에 매개변수로 들어가는 쪽)


```python
rdd1= sc.parallelize([
    ("foo", 1),
    ("goo", 2),
    ("hoo", 3)
])

rdd2 = sc.parallelize([
    ("foo", 1),
    ("goo", 2),
    ("goo", 4),
    ("moo", 6)
])

rdd1.join(rdd2).collect()
```




    [('foo', (1, 1)), ('goo', (2, 2)), ('goo', (2, 4))]




```python
#outer join
rdd1.leftOuterJoin(rdd2).collect()
# rdd1을 기준으러 join 따라서 'hoo'가 join 되고 rdd2에 없기 때문에 None으로 처리한다
```




    [('foo', (1, 1)), ('goo', (2, 2)), ('goo', (2, 4)), ('hoo', (3, None))]




```python
rdd1.rightOuterJoin(rdd2).collect()
# rdd2를 기준으로 join하기 때문에 'moo'가 join 되고 rdd1에 없기 때문에 None 처리된다
```




    [('foo', (1, 1)), ('moo', (None, 6)), ('goo', (2, 2)), ('goo', (2, 4))]




```python
sc.stop()
```

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
lines
```




    file:///C:\Users\sonjj\study_spark\data\restaurant_reviews.csv MapPartitionsRDD[3] at textFile at NativeMethodAccessorImpl.java:0




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
header = lines.first()
header
```




    'id,item,cateogry,reviews,'




```python
filtered_lines = lines.filter(lambda row : row != header)
filtered_lines
```




    PythonRDD[6] at RDD at PythonRDD.scala:53




```python
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
    # reviews는 점수로 parse
    reviews = fields[3]
    reviews = int(reviews)
    
    return category, reviews
```


```python
parse('0,짜장면,중식,125')
```




    ('중식', 125)




```python
# RDD 내의 모든 row에 대해 'parse' 함수를 적용 후 추출(map)
category_reviews = filtered_lines.map(parse)
category_reviews
```




    PythonRDD[7] at RDD at PythonRDD.scala:53




```python
category_reviews.collect()
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
#카테고리 별 리부 평균
category_review_count = category_reviews.mapValues(lambda x: (x,1))
# 리뷰의 개수를 구하기 위해 x 함수를 추가
category_review_count.collect()
```




    [('중식', (125, 1)),
     ('중식', (235, 1)),
     ('분식', (32, 1)),
     ('분식', (534, 1)),
     ('일식', (223, 1)),
     ('일식', (52, 1)),
     ('일식', (12, 1)),
     ('아시안', (312, 1)),
     ('패스트푸드', (12, 1)),
     ('패스트푸드', (23, 1))]




```python
reduced = category_review_count.reduceByKey(lambda x, y: (x[0] + y[0], x[1] + y[1]))
reduced.collect()
# 같은 카테고리의 리뷰 건수와, 개수를 각각 x, y로 할당하여 각각 더한다
```




    [('중식', (360, 2)),
     ('분식', (566, 2)),
     ('일식', (287, 3)),
     ('아시안', (312, 1)),
     ('패스트푸드', (35, 2))]




```python
average = reduced.mapValues(lambda x: x[0]/x[1])
average.collect()
```




    [('중식', 180.0),
     ('분식', 283.0),
     ('일식', 95.66666666666667),
     ('아시안', 312.0),
     ('패스트푸드', 17.5)]




```python
sc.stop()
```

