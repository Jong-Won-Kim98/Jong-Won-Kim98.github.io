---
layout: post
title: Spark using SQL
subtitle: SQL
categories: BigData_SQL
tags: Reduction Operations
---

### Spark SQL
- 스파크를 기반으로 구현된 하나의 패키지
- 스파크 프로그래밍 내부에서 관계형 처리(Join)
- 스키마의 정보를 이용해 자동으로 최적화가 가능하다
- 외부 데이터 세트(CSV, JSON, ...)를 사용하기 쉽게 한다
  #### 3가지 주된 API 존재
    - SQL
    - DataFrame
    - Datasets
  #### 2개의 백엔드 컴포넌트
    - Catalyst: 쿼리 최적화 엔진(파티션 및 셔플링 최적화)
    - Tungsten: 시리얼라이저(데이터 용량 최적화)

### DataFrame
- Spark Core에 RDD = Spark SQL DataFrame
- 테이블 데이터 세트
- RDD에 스키마가 적용된 데이터 세트
- RDD에 스키마를 정의한 다음 DataFrame으로 변형 가능
- CSV, JSON 등의 데이터를 받아 DataFrame으로 만들어 낼 수 있다

### SparkSession
- Spark 구동 환경을 만든다
- Spark Core의 SparkContext = SPark SQL의 SparkSession

### createOrReplaceTempView
- 데이터 프레임을 RDBMS의 테이블 처럼 사용하기 위해 위 함수를 이용해 tempoary view를 만들어줘야 한다
- Spark SQL을 사용할 수 있게 된다

### SQL 구조
1. SELECT: 컬럼 조회 하기 위한 쿼리 절
2. FROM: 테이블, 어떤 데이터프레임에서 데이터를 가져오는가
3. WHERE: 데이터가 조회되기 위한 조건

```python
from pyspark import SparkConf, SparkContext
conf = SparkConf().setMaster("local").setAppName("spark_sql_basic")
sc = SparkContext(conf=conf)
```


```python
#RDD 데이터 추출
movies_rdd = sc.parallelize([
    (1, ("어벤져스", "마블")),
    (2, ("슈퍼맨", "DC")),
    (3, ("배트맨", "DC")),
    (4, ("겨울왕국", "디즈니")),
    (5, ("아어언맨", "마블"))
])

attendances_rdd = sc.parallelize([
    (1, (13934592, "KR")),
    (2, (2182227, "KR")),
    (3, (4226242, "KR")),
    (4, (10303058, "KR")),
    (5, (4300365, "KR"))
])
```


```python
'''
마블 영화 중 관객수 500만 이상 가져오기
1. Inner Join -> Filter by moive -> Filter By attendance
2. Filter By Movie, Filter By attendance -> inner Join(추천)
'''
```




    '\n마블 영화 중 관객수 500만 이상 가져오기\n1. Inner Join -> Filter by moive -> Filter By attendance\n2. Filter By Movie, Filter By attendance -> inner Join(추천)\n'




```python
# 1. Inner Join -> Filter by moive -> Filter By attendance
movie_att = movies_rdd.join(attendances_rdd)
movie_att.filter(
    lambda x: x[1][0][1] == "마블" and x[1][1][0] > 5000000
).collect()
```




    [(1, (('어벤져스', '마블'), (13934592, 'KR')))]




```python
# 2. Filter By Movie, Filter By attendance -> inner Join(추천)
filtered_movies = movies_rdd.filter(lambda x : x[1][1] == "마블")
filtered_att = attendances_rdd.filter(lambda x : x[1][0] > 5000000)
filtered_movies.join(filtered_att).collect()
```




    [(1, (('어벤져스', '마블'), (13934592, 'KR')))]




```python
# Spark SQL Session 만들기
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local").appName("spark_sql").getOrCreate()
```


```python
# RDD로 DataFrame 생성
movies = [
    (1, "어벤져스", "마블", 2012, 4, 26),
    (2, "슈펴맨", "DC", 2013, 6, 13),
    (3, "배트맨", "DC", 2008, 8, 6),
    (4, "겨울왕국", "디즈니", 2014, 1, 16),
    (5, "아이언맨", "마블", 2008, 4, 30)
]
```


```python
movie_schema = ["id", "name", "company", "year", "month", "day"]
```


```python
# DataFrame 생성
df = spark.createDataFrame(data=movies, schema=movie_schema)
```


```python
# 스키마 타입 확인
df.dtypes
```




    [('id', 'bigint'),
     ('name', 'string'),
     ('company', 'string'),
     ('year', 'bigint'),
     ('month', 'bigint'),
     ('day', 'bigint')]




```python
# 전체 데이터 프레임 내용 확인(show())
df.show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  3|  배트맨|     DC|2008|    8|  6|
    |  4|겨울왕국| 디즈니|2014|    1| 16|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
'''
- 'SELCT': 컬럼 조회 하기위한 쿼리 절
- 'FROM': 테이블, (어떤 데이터프레임(테이블)에서 데이터를 가져오는가)
- 'where': 데이터가 조회되기 위한 조건
'''
# DataFrame temporary view에 등록해야 spark sql을 사용할 수 있다
df.createOrReplaceTempView("movies")
```


```python
# 영화 이름만 가져오기
query = """
SELECT name 
    FROM movies
"""
spark.sql(query).show()
```

    +--------+
    |    name|
    +--------+
    |어벤져스|
    |  슈펴맨|
    |  배트맨|
    |겨울왕국|
    |아이언맨|
    +--------+
    
    


```python
# 2010년 이후에 개봉한 영화
query = """
SELECT *
#*: 모든 컬럼
    FROM movies
    WHERE year >= 2010
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  4|겨울왕국| 디즈니|2014|    1| 16|
    +---+--------+-------+----+-----+---+
    
    


```python
# 2012년도 이전에 개봉한 영화의 이름과 회사 출력
query = """
SELECT name, company
    FROM movies
    WHERE year <= 2012
    
"""
spark.sql(query).show()
```

    +--------+-------+
    |    name|company|
    +--------+-------+
    |어벤져스|   마블|
    |  배트맨|     DC|
    |아이언맨|   마블|
    +--------+-------+
    
    


```python
# like 문자열 데이터에서 특정 단어, 문장을 포함한 데이터
# % 기호를 사용해 문장이 매칭되는지 확인 가능
# 제목이 ~~맨으로 끝나는 데이터의 모든 정보 조회
query = """
SELECT *
    FROM movies
    WHERE name Like '%맨'
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  3|  배트맨|     DC|2008|    8|  6|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# 제목에 '이'가 들어간 영화 찾기
query = """
SELECT *
    FROM movies
    WHERE name Like '%이%'
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# BETWEEN 특정 데이터와 데이터 사이 조회
# 개봉 월이 4~8월 사이 4월 <= 개봉월 <= 8월
query = """
SELECT * 
    FROM movies
    WHERE month BETWEEN 4 AND 8
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  3|  배트맨|     DC|2008|    8|  6|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# 이름이 ~맨으로 끝나고, 개봉연도가 2015년 이하인 영화
query = """
SELECT *
    FROM movies
    WHERE name Like '%맨'
    AND year <= 2010
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  3|  배트맨|     DC|2008|    8|  6|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# 영화의 회사가 마블, DC인 영화
query = """
SELECT *
    FROM movies
    WHERE company = '마블' OR company = 'DC'
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  3|  배트맨|     DC|2008|    8|  6|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# in 연산 활용
# 컬럼명 in(값1, 값2, ...)
# 데이터 값이 정확히 매칭이 되는 경우 사용

query = """
SELECT *
    FROM movies
    WHERE company in ('마블', 'DC')
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  3|  배트맨|     DC|2008|    8|  6|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# 회사 명이 "마로"로 시작하거나, "즈"로 끝나는 영화
query = """
SELECT *
    FROM movies
    WHERE company Like '마%' OR company Like '%니'
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  4|겨울왕국| 디즈니|2014|    1| 16|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# 회사 명이 "마로"로 시작하거나, "즈"로 끝나는 영화 중 2010년 이후로 개봉
query = """
SELECT *
    FROM movies
    WHERE company Like '마%' OR company Like '%니'
    AND year >= 2010
"""
spark.sql(query).show()
'''
and가 or보다 우선순위가 높기 때문에
company Like '%니' AND year >= 2010 실행 후
company Like '마%' OR 가 실행되기 때문에
year >= 2010 조건이 충족되지 않는다 
'''
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  4|겨울왕국| 디즈니|2014|    1| 16|
    |  5|아이언맨|   마블|2008|    4| 30|
    +---+--------+-------+----+-----+---+
    
    


```python
# 회사 명이 "마로"로 시작하거나, "즈"로 끝나는 영화 중 2010년 이후로 개봉
query = """
SELECT *
    FROM movies
    WHERE (company Like '마%' OR company Like '%니')
    AND year >= 2010
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  1|어벤져스|   마블|2012|    4| 26|
    |  4|겨울왕국| 디즈니|2014|    1| 16|
    +---+--------+-------+----+-----+---+
    
    


```python
'''
'ORDER BY'절: 정렬
    - asc(ascending): 오름차순(기본값, 생략 가능)
    - desc(descending): 내림차순
'''
# 개봉 연도 오름차순으로 정렬
query = """
SELECT *
    FROM movies
    ORDER BY year asc
"""
spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+
    | id|    name|company|year|month|day|
    +---+--------+-------+----+-----+---+
    |  3|  배트맨|     DC|2008|    8|  6|
    |  5|아이언맨|   마블|2008|    4| 30|
    |  1|어벤져스|   마블|2012|    4| 26|
    |  2|  슈펴맨|     DC|2013|    6| 13|
    |  4|겨울왕국| 디즈니|2014|    1| 16|
    +---+--------+-------+----+-----+---+
    
    


```python
'''
- 'count': 개수 세기
- 'mean': 평균 구하기
- 'sum': 총 합
'''
query = """
SELECT count(*) as movies_count
    FROM movies
    WHERE company = "DC"
"""
spark.sql(query).show()
```

    +------------+
    |movies_count|
    +------------+
    |           2|
    +------------+
    
    


```python
# JOIN 구현하기
attendances = [
    (1, 13934592, "KR"),
    (2, 2182227, "KR"),
    (3, 4226242, "KR"),
    (4, 10303058, "KR"),
    (5, 4300365, "KR")
]
```


```python
# 스키마 지정
# Struct: 컬럼, StructField: 컬럼의 데이터
from pyspark.sql.types import StringType, FloatType, IntegerType, StructType, StructField
```


```python
att_schema = StructType([
# 모든 컬럼의 타입을 통칭 - 컬럼 데이터의 집합
    StructField("id", IntegerType(), True),
    StructField("att", IntegerType(), True),
    StructField("theater_country", StringType(), True)
    # StructFiled: 컬럼
])
```


```python
att_df = spark.createDataFrame(
    data = attendances,
    schema = att_schema
)

att_df.dtypes
```




    [('id', 'int'), ('att', 'int'), ('theater_country', 'string')]




```python
att_df.createOrReplaceTempView("att")
```


```python
# 쿼리를 사용하지 않고 모든 데이터 확인
# DataFrame API 사용
att_df.select("*").show()
```

    +---+--------+---------------+
    | id|     att|theater_country|
    +---+--------+---------------+
    |  1|13934592|             KR|
    |  2| 2182227|             KR|
    |  3| 4226242|             KR|
    |  4|10303058|             KR|
    |  5| 4300365|             KR|
    +---+--------+---------------+
    
    


```python
query = """
SELECT *
    FROM movies
    JOIN att ON movies.id = att.id
"""

spark.sql(query).show()
```

    +---+--------+-------+----+-----+---+---+--------+---------------+
    | id|    name|company|year|month|day| id|     att|theater_country|
    +---+--------+-------+----+-----+---+---+--------+---------------+
    |  1|어벤져스|   마블|2012|    4| 26|  1|13934592|             KR|
    |  2|  슈펴맨|     DC|2013|    6| 13|  2| 2182227|             KR|
    |  3|  배트맨|     DC|2008|    8|  6|  3| 4226242|             KR|
    |  4|겨울왕국| 디즈니|2014|    1| 16|  4|10303058|             KR|
    |  5|아이언맨|   마블|2008|    4| 30|  5| 4300365|             KR|
    +---+--------+-------+----+-----+---+---+--------+---------------+
    
    


```python
query = """
SELECT movies.id, movies.name, movies.company, att.att
    FROM movies
    JOIN att ON movies.id = att.id
"""

spark.sql(query).show()
```

    +---+--------+-------+--------+
    | id|    name|company|     att|
    +---+--------+-------+--------+
    |  1|어벤져스|   마블|13934592|
    |  2|  슈펴맨|     DC| 2182227|
    |  3|  배트맨|     DC| 4226242|
    |  4|겨울왕국| 디즈니|10303058|
    |  5|아이언맨|   마블| 4300365|
    +---+--------+-------+--------+
    
    


```python
'''
DataFrame API
    - Transformations 작업, collect(), shwo()를 통해서 데이터 확인
    - select: DataFrame에 보고싶은 데이터만 select
    - agg: Aggregate, 그룹핑 후 데이터를 하나로 합쳐주는 역할
    
'''
df.select("*").collect()
```




    [Row(id=1, name='어벤져스', company='마블', year=2012, month=4, day=26),
     Row(id=2, name='슈펴맨', company='DC', year=2013, month=6, day=13),
     Row(id=3, name='배트맨', company='DC', year=2008, month=8, day=6),
     Row(id=4, name='겨울왕국', company='디즈니', year=2014, month=1, day=16),
     Row(id=5, name='아이언맨', company='마블', year=2008, month=4, day=30)]




```python
df.select("name", "company").collect()
```




    [Row(name='어벤져스', company='마블'),
     Row(name='슈펴맨', company='DC'),
     Row(name='배트맨', company='DC'),
     Row(name='겨울왕국', company='디즈니'),
     Row(name='아이언맨', company='마블')]




```python
df.select(df.name, (df.year-2000).alias("year")).show()
```

    +--------+----+
    |    name|year|
    +--------+----+
    |어벤져스|  12|
    |  슈펴맨|  13|
    |  배트맨|   8|
    |겨울왕국|  14|
    |아이언맨|   8|
    +--------+----+
    
    


```python
df.agg({"id": "count"}).collect()
```




    [Row(count(id)=5)]




```python
from pyspark.sql import functions as F

df.agg(F.min(df.year)).collect()
```




    [Row(min(year)=2008)]




```python
df.groupBy().avg().collect()
```




    [Row(avg(id)=3.0, avg(year)=2011.0, avg(month)=4.6, avg(day)=18.2)]




```python
# 회사별 개봉 월 평균
df.groupBy('company').agg({"month": "mean"}).collect()
```




    [Row(company='디즈니', avg(month)=1.0),
     Row(company='마블', avg(month)=4.0),
     Row(company='DC', avg(month)=7.0)]




```python
# 회사 별 월 별 영화 개수 정보
df.groupBy([df.company, df.month]).count().collect()
```




    [Row(company='디즈니', month=1, count=1),
     Row(company='DC', month=8, count=1),
     Row(company='DC', month=6, count=1),
     Row(company='마블', month=4, count=2)]




```python
# join: 다른 데이터 프레임, 사용자가 지정한 컬럼을 기준으로 합친다
df.join(att_df, 'id').select(df.name, att_df.att).show()
```

    +--------+--------+
    |    name|     att|
    +--------+--------+
    |어벤져스|13934592|
    |  슈펴맨| 2182227|
    |  배트맨| 4226242|
    |겨울왕국|10303058|
    |아이언맨| 4300365|
    +--------+--------+
    
    


```python
# select, where, orderBy 절 사용
marvel_df = df.select("name", "company", "year").where("company=='마블'").orderBy("id")
marvel_df.collect()
```




    [Row(name='어벤져스', company='마블', year=2012),
     Row(name='아이언맨', company='마블', year=2008)]




```python
spark.stop()
sc.stop()
```
