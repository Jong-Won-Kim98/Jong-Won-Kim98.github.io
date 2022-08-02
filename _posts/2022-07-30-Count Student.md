---
layout: post
title: Counting students
subtitle: Using spark_element
categories: BigData
tags: Spark pyspark
---

Pyspark: 

```python
from pyspark import SparkConf, SparkContext
```

```python
conf = SparkConf().setMaster("local").setAppName("country-student-counts")

sc = SparkContext(conf=conf)
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "xAPI-Edu-Data.csv"
```


```python
#데이터 로밍 및 RDD 생성
lines = sc.textFile("file:///{}\\{}".format(directory, filename))
lines
```




    file:///C:\Users\sonjj\study_spark\data\xAPI-Edu-Data.csv MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0




```python
header = lines.first() # 첫번째 줄을 출력하는(Action)
header
```




    'gender,NationalITy,PlaceofBirth,StageID,GradeID,SectionID,Topic,Semester,Relation,raisedhands,VisITedResources,AnnouncementsView,Discussion,ParentAnsweringSurvey,ParentschoolSatisfaction,StudentAbsenceDays,Class'




```python
datas = lines.filter(lambda row : row != header)
datas #데이터의 변환(필요한 부분만 보고싶다)이 바로 일어나지 않는다(Transformation)
```




    PythonRDD[3] at RDD at PythonRDD.scala:53




```python
# collect(): 실제 데이터 확인
datas.collect()[:3]
```




    ['M,KW,KuwaIT,lowerlevel,G-04,A,IT,F,Father,15,16,2,20,Yes,Good,Under-7,M',
     'M,KW,KuwaIT,lowerlevel,G-04,A,IT,F,Father,20,20,3,25,Yes,Good,Under-7,M',
     'M,KW,KuwaIT,lowerlevel,G-04,A,IT,F,Father,10,7,0,30,No,Bad,Above-7,L']




```python
#국적만 추출하기
countries = datas.map(lambda row : row.split(',')[2])
countries 
```




    PythonRDD[4] at RDD at PythonRDD.scala:53



## RDD의 변화 과정 
1. lines 
2. datas 
3. countries

### lines
```python
countries.collect()[:3]
```




    ['KuwaIT', 'KuwaIT', 'KuwaIT']



### datas, countries
```python
#국적 count
result = countries.countByValue()
result
```




    defaultdict(int,
                {'KuwaIT': 180,
                 'lebanon': 19,
                 'Egypt': 9,
                 'SaudiArabia': 16,
                 'USA': 16,
                 'Jordan': 176,
                 'venzuela': 1,
                 'Iran': 6,
                 'Tunis': 9,
                 'Morocco': 4,
                 'Syria': 6,
                 'Iraq': 22,
                 'Palestine': 10,
                 'Lybia': 6})



## 시각화


```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
series = pd.Series(result, name='countries')
series
```




    KuwaIT         180
    lebanon         19
    Egypt            9
    SaudiArabia     16
    USA             16
    Jordan         176
    venzuela         1
    Iran             6
    Tunis            9
    Morocco          4
    Syria            6
    Iraq            22
    Palestine       10
    Lybia            6
    Name: countries, dtype: int64




```python
plt.figure(figsize=(15, 10))
sns.barplot(x=series.index, y=series.values)
plt.show()
```

![output_14_0](https://user-images.githubusercontent.com/77920565/182392465-e48f78d5-490d-4055-af02-f7e2a018ca4e.png)


```python
#spark 사용을 종료한다는 명령어
sc.stop()
```