```python
from pyspark import SparkConf, SparkContext
```


```python
conf = SparkConf().setMaster("local").setAppName("tranformaionts_actions")
sc = SparkContext(conf=conf)
```


```python
#스파크 환경 설정 확인
sc.getConf().getAll()
```




    [('spark.master', 'local'),
     ('spark.app.id', 'local-1659421591906'),
     ('spark.rdd.compress', 'True'),
     ('spark.app.startTime', '1659421587394'),
     ('spark.serializer.objectStreamReset', '100'),
     ('spark.submit.pyFiles', ''),
     ('spark.executor.id', 'driver'),
     ('spark.submit.deployMode', 'client'),
     ('spark.driver.host', 'DESKTOP-FQL8QUA'),
     ('spark.app.name', 'tranformaionts_actions'),
     ('spark.driver.port', '12456'),
     ('spark.ui.showConsoleProgress', 'true')]



1. RDD 생성
2. 일반 파이썬 리스트를 이용, RDD 생성
3. parallelize([item1, item2, item3, ...])


```python
foods = sc.parallelize(["짜장면", "마라탕", "짬뽕", "떡볶이", "쌀국수", "짬뽕", "짜장면"])
foods
```




    ParallelCollectionRDD[0] at readRDDFromFile at PythonRDD.scala:274




```python
foods.collect()
```




    ['짜장면', '마라탕', '짬뽕', '떡볶이', '쌀국수', '짬뽕', '짜장면']



- 각 음식 별 개수 count
    - countByValue()


```python
foods.countByValue()
```




    defaultdict(int, {'짜장면': 2, '마라탕': 1, '짬뽕': 2, '떡볶이': 1, '쌀국수': 1})



- 각 음식 별 개수 count
    - countByValue()


```python
foods.take(3)
```




    ['짜장면', '마라탕', '짬뽕']



- 처음 1개의 데이터 가져오기
    - first()


```python
foods.first()
```




    '짜장면'



- RDD 내 전체 데이터의 개수 세기
    - count()


```python
foods.count()
```




    7



- 중복 데이터 제거
    - distinct()
    - :중복 데이터를 제거한 RDD를 새로 생성 하는 transformation



```python
fd = foods.distinct()
fd
```




    PythonRDD[13] at RDD at PythonRDD.scala:53




```python
fd.collect()
```




    ['짜장면', '마라탕', '짬뽕', '떡볶이', '쌀국수']




```python
#중복을 제외한 데이터 개수 세기
fd.count()
```




    5



- 요소들을 하나 씩 꺼내서 함수에 저장
    - action 이지만 return을 하지 않는다
    - foreach(<func>)
    - worker 노드에서 실행된다
    - Driver Program(SparkContext)에서 실행하는 것이 아니기 때문에 SparkContext에서 확인할 수 없다
    - RDD에 연산을 하고 난 후 log를 저장할 때 유용하다


```python
foods.foreach(lambda x : print(x))
# x에 RDD 원소 하나하나가 들어간다
# 결과값을 print했기 때문에 터미널에서만 확인 가능하다(worker 공간)
```

- Narrow Transformations
    - 1:1 변환을 의미한다
    - 하나의 열을 조작하기 위해 다른 열 및 파티션의 데이터를 사용할 필요가 없다
    - filter(), map(), flatMap(), sample(), union()


```python
sample_rdd = sc.parallelize([1, 2, 3])
sample_rdd
```




    ParallelCollectionRDD[17] at readRDDFromFile at PythonRDD.scala:274




```python
sample_rdd2 = sample_rdd.map(lambda x : x + 2)
sample_rdd2
```




    PythonRDD[18] at RDD at PythonRDD.scala:53




```python
sample_rdd2.collect()
```




    [3, 4, 5]




```python
movies = [
    "토이 스토리",
    "어벤져서",
    "닥터 스트레인지",
    "헐크",
    "아이언맨",
    "해운대",
    "아바타"
]
```


```python
moviesRDD = sc.parallelize(movies)
moviesRDD
```




    ParallelCollectionRDD[19] at readRDDFromFile at PythonRDD.scala:274




```python
mapMovies = moviesRDD.map(lambda x : x.split(" "))
mapMovies
```




    PythonRDD[20] at RDD at PythonRDD.scala:53




```python
mapMovies.collect()
```




    [['토이', '스토리'], ['어벤져서'], ['닥터', '스트레인지'], ['헐크'], ['아이언맨'], ['해운대'], ['아바타']]




```python
flatMovies = moviesRDD.flatMap(lambda x : x.split(" "))
flatMovies.collect()
```




    ['토이', '스토리', '어벤져서', '닥터', '스트레인지', '헐크', '아이언맨', '해운대', '아바타']




```python
#특정 데이터를 제거한 RDD 생성
filteredMovie = flatMovies.filter(lambda x : x != "닥터")
filteredMovie.collect()
```




    ['토이', '스토리', '어벤져서', '스트레인지', '헐크', '아이언맨', '해운대', '아바타']



- 집합 Transformation
    - 교집합(intersection)
    - 합집합(union)
    - 차집합(subtract)


```python
num1 = sc.parallelize([1, 2, 3, 4, 5])
num2 = sc.parallelize([4, 5, 6, 7, 8, 9, 10])
```


```python
num1.intersection(num2).collect()
# 교집합
```




    [4, 5]




```python
num1.union(num2).collect()
# 합집합
```




    [1, 2, 3, 4, 5, 4, 5, 6, 7, 8, 9, 10]




```python
num1.subtract(num2).collect()
# 차집합s
```




    [2, 1, 3]



데이터 랜덤 추출
- sample(withReplacement, graction, seed = None)
- 샘플링: 데이터에서 일부분 추출
- withReplacement: 한 번 추출 된 샘플을 다시 샘플링 대상을 삼을 것인가 여부
    - True: 한 번 샘플링 된 데이터가 다시 대상
    - False: 한 번 샘플링 된 데이터가 다시 대상이 되지 않는다
- fraction: 샘플링 된 데이터의 기댓값(확률)
    - 각각의 데이터가 추출될 확률
    - 높아지면 높아질 수록 원본에서 샘플링되는 원소의 개수가 많아진다
- seed: 랜덤을 고정해서 항상 같은 결과가 나올 수 있도록
'''


```python
numUnion = num1.union(num2)
numUnion.collect()
```




    [1, 2, 3, 4, 5, 4, 5, 6, 7, 8, 9, 10]




```python
numUnion.sample(True, 0.5).collect()
```




    [4, 6, 7]




```python
numUnion.sample(False, 0.8).collect()
```




    [2, 3, 4, 4, 5, 6, 7, 8, 10]




```python
numUnion.sample(True, 0.5, seed=42).collect()
```




    [5, 6, 6]




```python
'''
Wide Transformations
- groupBy(<func>)
'''
foods = sc.parallelize(["짜장면", "마라탕", "짬뽕", "떡볶이", "쌀국수", "짬뽕", "짜장면"])
foods
```




    ParallelCollectionRDD[94] at readRDDFromFile at PythonRDD.scala:274




```python
# 그룹핑의 기준을 문자열의 첫 번째 글자를 설정
foodsGroup = foods.groupBy(lambda x : x[0])
foodsGroup
```




    PythonRDD[99] at RDD at PythonRDD.scala:53




```python
res = foodsGroup.collect()
res
```




    [('짜', <pyspark.resultiterable.ResultIterable at 0x28233039b50>),
     ('마', <pyspark.resultiterable.ResultIterable at 0x28232ce83a0>),
     ('짬', <pyspark.resultiterable.ResultIterable at 0x2823304f130>),
     ('떡', <pyspark.resultiterable.ResultIterable at 0x2823304ff40>),
     ('쌀', <pyspark.resultiterable.ResultIterable at 0x2823304ff10>)]




```python
for(k, v) in res:
    print(k, list(v))
```

    짜 ['짜장면', '짜장면']
    마 ['마라탕']
    짬 ['짬뽕', '짬뽕']
    떡 ['떡볶이']
    쌀 ['쌀국수']
    


```python
sc.stop()
```
