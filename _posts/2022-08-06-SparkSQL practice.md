---
layout: post
title: SparkSQL Practice
subtitle: SparkSQL
categories: BigData_SQL
tags: SparkSQL 
---

### Wordcount

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as func

spark = SparkSession.builder.appName("wordcount").getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "book"

inputDF = spark.read.text(f"file:///{directory}\\{filename}")
inputDF.show()
```

    +--------------------+
    |               value|
    +--------------------+
    |Self-Employment: ...|
    |Achieving Financi...|
    |       By Frank Kane|
    |                    |
    |                    |
    |                    |
    |Copyright � 2015 ...|
    |All rights reserv...|
    |                    |
    |                    |
    |            CONTENTS|
    |          Disclaimer|
    |             Preface|
    |Part I: Making th...|
    |  Overcoming Inertia|
    |     Fear of Failure|
    |Career Indoctrina...|
    |The Carrot on a S...|
    |      Ego Protection|
    |Your Employer as ...|
    +--------------------+
    only showing top 20 rows
    
    


```python
words = inputDF.select(func.split(inputDF.value, "\\W+")).alias("word")
words.show()
```

    +---------------------+
    |split(value, \W+, -1)|
    +---------------------+
    | [Self, Employment...|
    | [Achieving, Finan...|
    |    [By, Frank, Kane]|
    |                   []|
    |                   []|
    |                   []|
    | [Copyright, 2015,...|
    | [All, rights, res...|
    |                   []|
    |                   []|
    |           [CONTENTS]|
    |         [Disclaimer]|
    |            [Preface]|
    | [Part, I, Making,...|
    | [Overcoming, Iner...|
    |  [Fear, of, Failure]|
    | [Career, Indoctri...|
    | [The, Carrot, on,...|
    |    [Ego, Protection]|
    | [Your, Employer, ...|
    +---------------------+
    only showing top 20 rows
    
    


```python
# explode: flatMap과 비슷, 시퀀스를 풀어 하나의 row로 각각 들어갈 수 있게 해준다
words = inputDF.select(func.explode(func.split(inputDF.value, "\\W+")).alias("word"))
words.filter(words.word != "")
words.show()
```

    +----------+
    |      word|
    +----------+
    |      Self|
    |Employment|
    |  Building|
    |        an|
    |  Internet|
    |  Business|
    |        of|
    |       One|
    | Achieving|
    | Financial|
    |       and|
    |  Personal|
    |   Freedom|
    |   through|
    |         a|
    | Lifestyle|
    |Technology|
    |  Business|
    |        By|
    |     Frank|
    +----------+
    only showing top 20 rows
    
    


```python
lowercaseWords = words.select(func.lower(words.word).alias("word"))
lowercaseWords.show()
```

    +----------+
    |      word|
    +----------+
    |      self|
    |employment|
    |  building|
    |        an|
    |  internet|
    |  business|
    |        of|
    |       one|
    | achieving|
    | financial|
    |       and|
    |  personal|
    |   freedom|
    |   through|
    |         a|
    | lifestyle|
    |technology|
    |  business|
    |        by|
    |     frank|
    +----------+
    only showing top 20 rows
    
    


```python
wordCounts = lowercaseWords.groupBy("word").count()
wordCounts.show()
```

    +-------------+-----+
    |         word|count|
    +-------------+-----+
    |       online|   50|
    |          few|   40|
    |         some|  121|
    |  requirement|    1|
    |         hope|    5|
    |        still|   65|
    |        those|   68|
    |      barrier|    2|
    |indoctrinated|    1|
    |       harder|    3|
    |         guts|    1|
    |  interaction|    3|
    |    connected|    2|
    |     medicare|    2|
    |       travel|    5|
    |    traveling|    1|
    |   likelihood|    2|
    |      persist|    1|
    |    involving|    4|
    |          art|    3|
    +-------------+-----+
    only showing top 20 rows
    
    


```python
wordCountSorted = wordCounts.sort("count", ascending = False)
wordCountSorted.show()
```

    +--------+-----+
    |    word|count|
    +--------+-----+
    |     you| 1878|
    |      to| 1828|
    |    your| 1420|
    |     the| 1292|
    |       a| 1191|
    |      of|  970|
    |     and|  934|
    |        |  772|
    |    that|  747|
    |      it|  649|
    |      in|  616|
    |      is|  560|
    |     for|  537|
    |      on|  428|
    |     are|  424|
    |      if|  411|
    |       s|  391|
    |       i|  387|
    |business|  383|
    |     can|  376|
    +--------+-----+
    only showing top 20 rows
    
```python
spark.stop()
```

---

### Friends GroupBy Age

```python
from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql import functions as func

spark = SparkSession.builder.appName("FriendsByAge").getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "fakefriends-header.csv"
```


```python
lines = spark.read.option("header", "true").option("inferSchema", "true").csv(f"file:///{directory}\\{filename}")
lines.collect()[:3]
```




    [Row(userID=0, name='Will', age=33, friends=385),
     Row(userID=1, name='Jean-Luc', age=26, friends=2),
     Row(userID=2, name='Hugh', age=55, friends=221)]




```python
friendsByAge = lines.select("age", "friends")
friendsByAge.show()
```

    +---+-------+
    |age|friends|
    +---+-------+
    | 33|    385|
    | 26|      2|
    | 55|    221|
    | 40|    465|
    | 68|     21|
    | 59|    318|
    | 37|    220|
    | 54|    307|
    | 38|    380|
    | 27|    181|
    | 53|    191|
    | 57|    372|
    | 54|    253|
    | 56|    444|
    | 43|     49|
    | 36|     49|
    | 22|    323|
    | 35|     13|
    | 45|    455|
    | 60|    246|
    +---+-------+
    only showing top 20 rows
    
    


```python
friendsByAge.groupBy("age").avg("friends").show()
```

    +---+------------------+
    |age|      avg(friends)|
    +---+------------------+
    | 31|            267.25|
    | 65|             298.2|
    | 53|222.85714285714286|
    | 34|             245.5|
    | 28|             209.1|
    | 26|242.05882352941177|
    | 27|           228.125|
    | 44| 282.1666666666667|
    | 22|206.42857142857142|
    | 47|233.22222222222223|
    | 52| 340.6363636363636|
    | 40| 250.8235294117647|
    | 20|             165.0|
    | 57| 258.8333333333333|
    | 54| 278.0769230769231|
    | 48|             281.4|
    | 19|213.27272727272728|
    | 64| 281.3333333333333|
    | 41|268.55555555555554|
    | 43|230.57142857142858|
    +---+------------------+
    only showing top 20 rows
    
    


```python
friendsByAge.groupBy("age").avg("friends").sort("age").show()
```

    +---+------------------+
    |age|      avg(friends)|
    +---+------------------+
    | 18|           343.375|
    | 19|213.27272727272728|
    | 20|             165.0|
    | 21|           350.875|
    | 22|206.42857142857142|
    | 23|             246.3|
    | 24|             233.8|
    | 25|197.45454545454547|
    | 26|242.05882352941177|
    | 27|           228.125|
    | 28|             209.1|
    | 29|215.91666666666666|
    | 30| 235.8181818181818|
    | 31|            267.25|
    | 32| 207.9090909090909|
    | 33| 325.3333333333333|
    | 34|             245.5|
    | 35|           211.625|
    | 36|             246.6|
    | 37|249.33333333333334|
    +---+------------------+
    only showing top 20 rows
    
    


```python
# func round 함수 이용해 숫자들 소수 둘 째 자리까지 표시
friendsByAge.groupBy("age").agg(func.round(func.avg("friends"),2)).sort("age").show()
```

    +---+----------------------+
    |age|round(avg(friends), 2)|
    +---+----------------------+
    | 18|                343.38|
    | 19|                213.27|
    | 20|                 165.0|
    | 21|                350.88|
    | 22|                206.43|
    | 23|                 246.3|
    | 24|                 233.8|
    | 25|                197.45|
    | 26|                242.06|
    | 27|                228.13|
    | 28|                 209.1|
    | 29|                215.92|
    | 30|                235.82|
    | 31|                267.25|
    | 32|                207.91|
    | 33|                325.33|
    | 34|                 245.5|
    | 35|                211.63|
    | 36|                 246.6|
    | 37|                249.33|
    +---+----------------------+
    only showing top 20 rows
    
    


```python
friendsByAge.groupBy("age").agg(func.round(func.avg("friends"),2).alias("friends_avg")).sort("age").show()
```

    +---+-----------+
    |age|friends_avg|
    +---+-----------+
    | 18|     343.38|
    | 19|     213.27|
    | 20|      165.0|
    | 21|     350.88|
    | 22|     206.43|
    | 23|      246.3|
    | 24|      233.8|
    | 25|     197.45|
    | 26|     242.06|
    | 27|     228.13|
    | 28|      209.1|
    | 29|     215.92|
    | 30|     235.82|
    | 31|     267.25|
    | 32|     207.91|
    | 33|     325.33|
    | 34|      245.5|
    | 35|     211.63|
    | 36|      246.6|
    | 37|     249.33|
    +---+-----------+
    only showing top 20 rows
    
    


```python
spark.stop()
```

---

### Counting friends who meet the requirements

#### SpakrSession 생성


```python
from pyspark.sql import SparkSession
from pyspark.sql import Row
```


```python
spark = SparkSession.builder.appName("SparkSQL").getOrCreate()
```

#### fakefriends.csv 로딩(헤더가 없는 경우)


```python
# RDD에서 로딩하는 방법
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "fakefriends.csv"
# colum의 이름이 없는 데이터
lines = spark.sparkContext.textFile(f"file:///{directory}\\{filename}")
lines.collect()[:3]
```




    ['0,Will,33,385', '1,Jean-Luc,26,2', '2,Hugh,55,221']




```python
# colum의 이름을 만들어 준다
def mapper(line):
    fields = line.split(",")
    return Row(ID = int(fields[0]), name = str(fields[1].encode("utf-8")), age = int(fields[2]), numFriends = int(fields[3]))

# RDD의 map을 이용해 ROW 형식으로 변환
people = lines.map(mapper)
people.collect()[:3]
```




    [Row(ID=0, name="b'Will'", age=33, numFriends=385),
     Row(ID=1, name="b'Jean-Luc'", age=26, numFriends=2),
     Row(ID=2, name="b'Hugh'", age=55, numFriends=221)]



#### 매핑된 people RDD를 활용해 데이터 프레임 생성


```python
# cache(): 원본 데이터 프레임을 메모리에 저장해 query를 통해 데이터를 변경해도 원본은 보관해 둔다
schemaPeople = spark.createDataFrame(people).cache()
schemaPeople.createOrReplaceTempView("people")
schemaPeople.show()
```

    +---+-----------+---+----------+
    | ID|       name|age|numFriends|
    +---+-----------+---+----------+
    |  0|    b'Will'| 33|       385|
    |  1|b'Jean-Luc'| 26|         2|
    |  2|    b'Hugh'| 55|       221|
    |  3|  b'Deanna'| 40|       465|
    |  4|   b'Quark'| 68|        21|
    |  5|  b'Weyoun'| 59|       318|
    |  6|  b'Gowron'| 37|       220|
    |  7|    b'Will'| 54|       307|
    |  8|  b'Jadzia'| 38|       380|
    |  9|    b'Hugh'| 27|       181|
    | 10|     b'Odo'| 53|       191|
    | 11|     b'Ben'| 57|       372|
    | 12|   b'Keiko'| 54|       253|
    | 13|b'Jean-Luc'| 56|       444|
    | 14|    b'Hugh'| 43|        49|
    | 15|     b'Rom'| 36|        49|
    | 16|  b'Weyoun'| 22|       323|
    | 17|     b'Odo'| 35|        13|
    | 18|b'Jean-Luc'| 45|       455|
    | 19|  b'Geordi'| 60|       246|
    +---+-----------+---+----------+
    only showing top 20 rows
    
    

#### 데이터 조회(SQL)

createOrReplaceTempView를 이용해 등록된 테이블에 쿼리를 사용


```python
# 13~19세의 모든 정보 조회
query = """
SELECT *
    FROM people
    WHERE age >= 13 AND age <= 19
"""
spark.sql(query).show()
```

    +---+----------+---+----------+
    | ID|      name|age|numFriends|
    +---+----------+---+----------+
    | 21|  b'Miles'| 19|       268|
    | 52|b'Beverly'| 19|       269|
    | 54|  b'Brunt'| 19|         5|
    |106|b'Beverly'| 18|       499|
    |115|  b'Dukat'| 18|       397|
    |133|  b'Quark'| 19|       265|
    |136|   b'Will'| 19|       335|
    |225|   b'Elim'| 19|       106|
    |304|   b'Will'| 19|       404|
    |341|   b'Data'| 18|       326|
    |366|  b'Keiko'| 19|       119|
    |373|  b'Quark'| 19|       272|
    |377|b'Beverly'| 18|       418|
    |404| b'Kasidy'| 18|        24|
    |409|    b'Nog'| 19|       267|
    |439|   b'Data'| 18|       417|
    |444|  b'Keiko'| 18|       472|
    |492|  b'Dukat'| 19|        36|
    |494| b'Kasidy'| 18|       194|
    +---+----------+---+----------+
    
    


```python
schemaPeople.groupBy("age").count().orderBy("age").show()
# 나이대별 정렬 후 사람 수 count
```

    +---+-----+
    |age|count|
    +---+-----+
    | 18|    8|
    | 19|   11|
    | 20|    5|
    | 21|    8|
    | 22|    7|
    | 23|   10|
    | 24|    5|
    | 25|   11|
    | 26|   17|
    | 27|    8|
    | 28|   10|
    | 29|   12|
    | 30|   11|
    | 31|    8|
    | 32|   11|
    | 33|   12|
    | 34|    6|
    | 35|    8|
    | 36|   10|
    | 37|    9|
    +---+-----+
    only showing top 20 rows
    
    


```python
spark.stop()
```

#### 헤더가 있는 csv 파일에서 실습


```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("SparkSQL").getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "fakefriends-header.csv"
```


```python
# 자동으로 데이터 타입 유추(inferSchema, true)
people = spark.read.option("header", "true").option("inferSchema", "true").csv(f"file:///{directory}\\{filename}")
people.show()
```

    +------+--------+---+-------+
    |userID|    name|age|friends|
    +------+--------+---+-------+
    |     0|    Will| 33|    385|
    |     1|Jean-Luc| 26|      2|
    |     2|    Hugh| 55|    221|
    |     3|  Deanna| 40|    465|
    |     4|   Quark| 68|     21|
    |     5|  Weyoun| 59|    318|
    |     6|  Gowron| 37|    220|
    |     7|    Will| 54|    307|
    |     8|  Jadzia| 38|    380|
    |     9|    Hugh| 27|    181|
    |    10|     Odo| 53|    191|
    |    11|     Ben| 57|    372|
    |    12|   Keiko| 54|    253|
    |    13|Jean-Luc| 56|    444|
    |    14|    Hugh| 43|     49|
    |    15|     Rom| 36|     49|
    |    16|  Weyoun| 22|    323|
    |    17|     Odo| 35|     13|
    |    18|Jean-Luc| 45|    455|
    |    19|  Geordi| 60|    246|
    +------+--------+---+-------+
    only showing top 20 rows
    
    


```python
# 스키마 정보
people.printSchema()
```

    root
     |-- userID: integer (nullable = true)
     |-- name: string (nullable = true)
     |-- age: integer (nullable = true)
     |-- friends: integer (nullable = true)
    
    


```python
# select 활용하기
people.select("name").show()
```

    +--------+
    |    name|
    +--------+
    |    Will|
    |Jean-Luc|
    |    Hugh|
    |  Deanna|
    |   Quark|
    |  Weyoun|
    |  Gowron|
    |    Will|
    |  Jadzia|
    |    Hugh|
    |     Odo|
    |     Ben|
    |   Keiko|
    |Jean-Luc|
    |    Hugh|
    |     Rom|
    |  Weyoun|
    |     Odo|
    |Jean-Luc|
    |  Geordi|
    +--------+
    only showing top 20 rows
    
    


```python
# 21살 미만(조건) - where 절
people.filter(people.age<21).show()
```

    +------+-------+---+-------+
    |userID|   name|age|friends|
    +------+-------+---+-------+
    |    21|  Miles| 19|    268|
    |    48|    Nog| 20|      1|
    |    52|Beverly| 19|    269|
    |    54|  Brunt| 19|      5|
    |    60| Geordi| 20|    100|
    |    73|  Brunt| 20|    384|
    |   106|Beverly| 18|    499|
    |   115|  Dukat| 18|    397|
    |   133|  Quark| 19|    265|
    |   136|   Will| 19|    335|
    |   225|   Elim| 19|    106|
    |   304|   Will| 19|    404|
    |   327| Julian| 20|     63|
    |   341|   Data| 18|    326|
    |   349| Kasidy| 20|    277|
    |   366|  Keiko| 19|    119|
    |   373|  Quark| 19|    272|
    |   377|Beverly| 18|    418|
    |   404| Kasidy| 18|     24|
    |   409|    Nog| 19|    267|
    +------+-------+---+-------+
    only showing top 20 rows
    
    


```python
# 그룹핑
people.groupBy("age").count().show()
```

    +---+-----+
    |age|count|
    +---+-----+
    | 31|    8|
    | 65|    5|
    | 53|    7|
    | 34|    6|
    | 28|   10|
    | 26|   17|
    | 27|    8|
    | 44|   12|
    | 22|    7|
    | 47|    9|
    | 52|   11|
    | 40|   17|
    | 20|    5|
    | 57|   12|
    | 54|   13|
    | 48|   10|
    | 19|   11|
    | 64|   12|
    | 41|    9|
    | 43|    7|
    +---+-----+
    only showing top 20 rows
    
    


```python
# 연산(모든 사람 나이 10살 더하기)
people.select(people.name, people.age+10).show()
```

    +--------+----------+
    |    name|(age + 10)|
    +--------+----------+
    |    Will|        43|
    |Jean-Luc|        36|
    |    Hugh|        65|
    |  Deanna|        50|
    |   Quark|        78|
    |  Weyoun|        69|
    |  Gowron|        47|
    |    Will|        64|
    |  Jadzia|        48|
    |    Hugh|        37|
    |     Odo|        63|
    |     Ben|        67|
    |   Keiko|        64|
    |Jean-Luc|        66|
    |    Hugh|        53|
    |     Rom|        46|
    |  Weyoun|        32|
    |     Odo|        45|
    |Jean-Luc|        55|
    |  Geordi|        70|
    +--------+----------+
    only showing top 20 rows
    
    


```python
spark.stop()
```

---

### Obtaining the minimum temperature

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as func
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, FloatType

spark = SparkSession.builder.appName("MinTemperatures").getOrCreate()

```

#### 스키마 정의


```python
schema = StructType([
    StructField("stationID", StringType(), True),
    StructField("date", IntegerType(), True),
    StructField("measure_type", StringType(), True),
    StructField("temperature", FloatType(), True)
])
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "1800.csv"
```


```python
df = spark.read.schema(schema).csv(f"file:///{directory}\\{filename}")
df.printSchema()
```

    root
     |-- stationID: string (nullable = true)
     |-- date: integer (nullable = true)
     |-- measure_type: string (nullable = true)
     |-- temperature: float (nullable = true)
    
    


```python
df.show()
```

    +-----------+--------+------------+-----------+
    |  stationID|    date|measure_type|temperature|
    +-----------+--------+------------+-----------+
    |ITE00100554|18000101|        TMAX|      -75.0|
    |ITE00100554|18000101|        TMIN|     -148.0|
    |GM000010962|18000101|        PRCP|        0.0|
    |EZE00100082|18000101|        TMAX|      -86.0|
    |EZE00100082|18000101|        TMIN|     -135.0|
    |ITE00100554|18000102|        TMAX|      -60.0|
    |ITE00100554|18000102|        TMIN|     -125.0|
    |GM000010962|18000102|        PRCP|        0.0|
    |EZE00100082|18000102|        TMAX|      -44.0|
    |EZE00100082|18000102|        TMIN|     -130.0|
    |ITE00100554|18000103|        TMAX|      -23.0|
    |ITE00100554|18000103|        TMIN|      -46.0|
    |GM000010962|18000103|        PRCP|        4.0|
    |EZE00100082|18000103|        TMAX|      -10.0|
    |EZE00100082|18000103|        TMIN|      -73.0|
    |ITE00100554|18000104|        TMAX|        0.0|
    |ITE00100554|18000104|        TMIN|      -13.0|
    |GM000010962|18000104|        PRCP|        0.0|
    |EZE00100082|18000104|        TMAX|      -55.0|
    |EZE00100082|18000104|        TMIN|      -74.0|
    +-----------+--------+------------+-----------+
    only showing top 20 rows
    
    


```python
# 최저 온도
minTemps = df.filter(df.measure_type == "TMIN")
minTemps.show()
```

    +-----------+--------+------------+-----------+
    |  stationID|    date|measure_type|temperature|
    +-----------+--------+------------+-----------+
    |ITE00100554|18000101|        TMIN|     -148.0|
    |EZE00100082|18000101|        TMIN|     -135.0|
    |ITE00100554|18000102|        TMIN|     -125.0|
    |EZE00100082|18000102|        TMIN|     -130.0|
    |ITE00100554|18000103|        TMIN|      -46.0|
    |EZE00100082|18000103|        TMIN|      -73.0|
    |ITE00100554|18000104|        TMIN|      -13.0|
    |EZE00100082|18000104|        TMIN|      -74.0|
    |ITE00100554|18000105|        TMIN|       -6.0|
    |EZE00100082|18000105|        TMIN|      -58.0|
    |ITE00100554|18000106|        TMIN|       13.0|
    |EZE00100082|18000106|        TMIN|      -57.0|
    |ITE00100554|18000107|        TMIN|       10.0|
    |EZE00100082|18000107|        TMIN|      -50.0|
    |ITE00100554|18000108|        TMIN|       14.0|
    |EZE00100082|18000108|        TMIN|      -31.0|
    |ITE00100554|18000109|        TMIN|       23.0|
    |EZE00100082|18000109|        TMIN|      -46.0|
    |ITE00100554|18000110|        TMIN|       31.0|
    |EZE00100082|18000110|        TMIN|      -75.0|
    +-----------+--------+------------+-----------+
    only showing top 20 rows
    
    

#### 셔플링 최소화
지역별 최저 온도를 구하기 위한 그룹핑 전,
필요 없는 데이터는 미리 삭제(filter)

-> 셔플링 최소화


```python
# 최적화, Shuffle이 많이 일어나는 것 미연 방지(필요한 데이터 추출)
stationTemps = minTemps.select("stationID", "temperature")
stationTemps.show()
```

    +-----------+-----------+
    |  stationID|temperature|
    +-----------+-----------+
    |ITE00100554|     -148.0|
    |EZE00100082|     -135.0|
    |ITE00100554|     -125.0|
    |EZE00100082|     -130.0|
    |ITE00100554|      -46.0|
    |EZE00100082|      -73.0|
    |ITE00100554|      -13.0|
    |EZE00100082|      -74.0|
    |ITE00100554|       -6.0|
    |EZE00100082|      -58.0|
    |ITE00100554|       13.0|
    |EZE00100082|      -57.0|
    |ITE00100554|       10.0|
    |EZE00100082|      -50.0|
    |ITE00100554|       14.0|
    |EZE00100082|      -31.0|
    |ITE00100554|       23.0|
    |EZE00100082|      -46.0|
    |ITE00100554|       31.0|
    |EZE00100082|      -75.0|
    +-----------+-----------+
    only showing top 20 rows
    
    


```python
minTempsByStations = stationTemps.groupBy("stationID").min("temperature")
minTempsByStations.show()
```

    +-----------+----------------+
    |  stationID|min(temperature)|
    +-----------+----------------+
    |ITE00100554|          -148.0|
    |EZE00100082|          -135.0|
    +-----------+----------------+
    
    


```python
# column 함수 추가(섭씨 온도)
minTempsByStationF = minTempsByStations.withColumn("temperature",
                                                 func.round(func.col("min(temperature)") * 0.1 * (9.0/5.0)+32.0, 2)
                                                 )
minTempsByStationF.show()
```

    +-----------+----------------+-----------+
    |  stationID|min(temperature)|temperature|
    +-----------+----------------+-----------+
    |ITE00100554|          -148.0|       5.36|
    |EZE00100082|          -135.0|        7.7|
    +-----------+----------------+-----------+
    
    


```python
minTempsByStationF.select("stationID", "temperature").show()
```

    +-----------+-----------+
    |  stationID|temperature|
    +-----------+-----------+
    |ITE00100554|       5.36|
    |EZE00100082|        7.7|
    +-----------+-----------+
    
    


```python
spark.stop()
```

---

### Calulating the total expenditure of a customer

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as func
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, FloatType
```


```python
# master: 스파크 컨텍스트가 실행될 위치
# local: 해당 컴퓨터에서 실행
# yarn, mesos 같은 클러스터 관리 플랫폼
spark = SparkSession.builder.appName("Total-Spent-By-Customer").master("local[*]").getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "customer-orders.csv"

lines = spark.sparkContext.textFile(f"file:///{directory}\\{filename}")
lines.collect()[:3]
```




    ['44,8602,37.19', '35,5368,65.89', '2,3391,40.64']




```python
schema = StructType([
    StructField("cust_id", IntegerType(), True),
    StructField("item_id", IntegerType(), True),
    StructField("amount_spent", FloatType(), True)
])
```


```python
customersDF = spark.read.schema(schema).csv(f"file:///{directory}\\{filename}")
customersDF.show()
```

    +-------+-------+------------+
    |cust_id|item_id|amount_spent|
    +-------+-------+------------+
    |     44|   8602|       37.19|
    |     35|   5368|       65.89|
    |      2|   3391|       40.64|
    |     47|   6694|       14.98|
    |     29|    680|       13.08|
    |     91|   8900|       24.59|
    |     70|   3959|       68.68|
    |     85|   1733|       28.53|
    |     53|   9900|       83.55|
    |     14|   1505|        4.32|
    |     51|   3378|        19.8|
    |     42|   6926|       57.77|
    |      2|   4424|       55.77|
    |     79|   9291|       33.17|
    |     50|   3901|       23.57|
    |     20|   6633|        6.49|
    |     15|   6148|       65.53|
    |     44|   8331|       99.19|
    |      5|   3505|       64.18|
    |     48|   5539|       32.42|
    +-------+-------+------------+
    only showing top 20 rows
    
    


```python
customersDF = customersDF.select("cust_id", "amount_spent")
customersDF.show()
```

    +-------+------------+
    |cust_id|amount_spent|
    +-------+------------+
    |     44|       37.19|
    |     35|       65.89|
    |      2|       40.64|
    |     47|       14.98|
    |     29|       13.08|
    |     91|       24.59|
    |     70|       68.68|
    |     85|       28.53|
    |     53|       83.55|
    |     14|        4.32|
    |     51|        19.8|
    |     42|       57.77|
    |      2|       55.77|
    |     79|       33.17|
    |     50|       23.57|
    |     20|        6.49|
    |     15|       65.53|
    |     44|       99.19|
    |      5|       64.18|
    |     48|       32.42|
    +-------+------------+
    only showing top 20 rows
    
    


```python
totalByCustomer = customersDF.groupBy("cust_id").agg(func.round(func.sum("amount_spent"), 2).alias("total_spent"))
totalByCustomer.show()
```

    +-------+-----------+
    |cust_id|total_spent|
    +-------+-----------+
    |     31|    4765.05|
    |     85|    5503.43|
    |     65|    5140.35|
    |     53|     4945.3|
    |     78|    4524.51|
    |     34|     5330.8|
    |     81|    5112.71|
    |     28|    5000.71|
    |     76|    4904.21|
    |     27|    4915.89|
    |     26|     5250.4|
    |     44|    4756.89|
    |     12|    4664.59|
    |     91|    4642.26|
    |     22|    5019.45|
    |     93|    5265.75|
    |     47|     4316.3|
    |      1|     4958.6|
    |     52|    5245.06|
    |     13|    4367.62|
    +-------+-----------+
    only showing top 20 rows
    
    


```python
totalByCustomersSorted = totalByCustomer.sort("total_spent")
totalByCustomersSorted.show()
```

    +-------+-----------+
    |cust_id|total_spent|
    +-------+-----------+
    |     45|    3309.38|
    |     79|    3790.57|
    |     96|    3924.23|
    |     23|    4042.65|
    |     99|    4172.29|
    |     75|     4178.5|
    |     36|    4278.05|
    |     98|    4297.26|
    |     47|     4316.3|
    |     77|    4327.73|
    |     13|    4367.62|
    |     48|    4384.33|
    |     49|     4394.6|
    |     94|    4475.57|
    |     67|    4505.79|
    |     50|    4517.27|
    |     78|    4524.51|
    |      5|    4561.07|
    |     57|     4628.4|
    |     83|     4635.8|
    +-------+-----------+
    only showing top 20 rows
    
    


```python
# 전체 데이터 출력
totalByCustomersSorted.show(totalByCustomersSorted.count())
```

    +-------+-----------+
    |cust_id|total_spent|
    +-------+-----------+
    |     45|    3309.38|
    |     79|    3790.57|
    |     96|    3924.23|
    |     23|    4042.65|
    |     99|    4172.29|
    |     75|     4178.5|
    |     36|    4278.05|
    |     98|    4297.26|
    |     47|     4316.3|
    |     77|    4327.73|
    |     13|    4367.62|
    |     48|    4384.33|
    |     49|     4394.6|
    |     94|    4475.57|
    |     67|    4505.79|
    |     50|    4517.27|
    |     78|    4524.51|
    |      5|    4561.07|
    |     57|     4628.4|
    |     83|     4635.8|
    |     91|    4642.26|
    |     74|    4647.13|
    |     84|    4652.94|
    |      3|    4659.63|
    |     12|    4664.59|
    |     66|    4681.92|
    |     56|    4701.02|
    |     21|    4707.41|
    |     80|    4727.86|
    |     14|    4735.03|
    |     37|     4735.2|
    |      7|    4755.07|
    |     44|    4756.89|
    |     31|    4765.05|
    |     82|    4812.49|
    |      4|    4815.05|
    |     10|     4819.7|
    |     88|    4830.55|
    |     20|    4836.86|
    |     89|    4851.48|
    |     95|    4876.84|
    |     38|    4898.46|
    |     76|    4904.21|
    |     86|    4908.81|
    |     27|    4915.89|
    |     18|    4921.27|
    |     53|     4945.3|
    |      1|     4958.6|
    |     51|    4975.22|
    |     16|    4979.06|
    |     30|    4990.72|
    |     28|    5000.71|
    |     22|    5019.45|
    |     29|    5032.53|
    |     17|    5032.68|
    |     60|    5040.71|
    |     25|    5057.61|
    |     19|    5059.43|
    |     81|    5112.71|
    |     69|    5123.01|
    |     65|    5140.35|
    |     11|    5152.29|
    |     35|    5155.42|
    |     40|    5186.43|
    |     87|     5206.4|
    |     52|    5245.06|
    |     26|     5250.4|
    |     62|    5253.32|
    |     33|    5254.66|
    |     24|    5259.92|
    |     93|    5265.75|
    |     64|    5288.69|
    |     90|    5290.41|
    |     55|    5298.09|
    |      9|    5322.65|
    |     34|     5330.8|
    |     72|    5337.44|
    |     70|    5368.25|
    |     43|    5368.83|
    |     92|    5379.28|
    |      6|    5397.88|
    |     15|    5413.51|
    |     63|    5415.15|
    |     58|    5437.73|
    |     32|    5496.05|
    |     61|    5497.48|
    |     85|    5503.43|
    |      8|    5517.24|
    |      0|    5524.95|
    |     41|    5637.62|
    |     59|    5642.89|
    |     42|    5696.84|
    |     46|    5963.11|
    |     97|    5977.19|
    |      2|    5994.59|
    |     71|    5995.66|
    |     54|    6065.39|
    |     39|    6193.11|
    |     73|     6206.2|
    |     68|    6375.45|
    +-------+-----------+
    
    


```python
spark.stop()
```
