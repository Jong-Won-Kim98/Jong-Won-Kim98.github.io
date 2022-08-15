---
layout: post
title: SparkSQL Optimization UDF
subtitle: SparkSQL
categories: BigData_SQL
tags: SparkSQL Optimization UDF
---

### UDF(User Defined function)

사용자 정의 함수로 SQL의 Query에서 사용할 함수를 개발자가 직접 만들어 줄 수 있는 기능으로 타입을 지정하지 않으면 String 형식으로 리턴한다

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("udf").getOrCreate()
```


```python
datas = [
    ("A", "2022-08-14", 31231),
    ("B", "2022-08-14", 31223),
    ("C", "2022-08-14", 31432),
    ("D", "2022-08-14", 30323),
    ("E", "2022-08-14", 29234)
]
columns = ["product", "date", "price"]
```


```python
df = spark.createDataFrame(data=datas, schema=columns)
df.show()
```

    +-------+----------+-----+
    |product|      date|price|
    +-------+----------+-----+
    |      A|2022-08-14|31231|
    |      B|2022-08-14|31223|
    |      C|2022-08-14|31432|
    |      D|2022-08-14|30323|
    |      E|2022-08-14|29234|
    +-------+----------+-----+
    
    


```python
df.createOrReplaceTempView("product")
```


```python
spark.sql("select * from product").show()
```

    +-------+----------+-----+
    |product|      date|price|
    +-------+----------+-----+
    |      A|2022-08-14|31231|
    |      B|2022-08-14|31223|
    |      C|2022-08-14|31432|
    |      D|2022-08-14|30323|
    |      E|2022-08-14|29234|
    +-------+----------+-----+
    

```python
from pyspark.sql.types import LongType

# 파이썬 함수
def squared(n):
    return n*n

spark.udf.register("squared", squared, LongType())
```




    <function __main__.squared(n)>




```python
spark.sql("SELECT price, squared(price) FROM product").show()
```

    +-----+--------------+
    |price|squared(price)|
    +-----+--------------+
    |31231|     975375361|
    |31223|     974875729|
    |31432|     987970624|
    |30323|     919484329|
    |29234|     854626756|
    +-----+--------------+
    
    


```python
def read_number(n):
    units = ["", "십", "백", "천", "만"]
    nums = "일이삼사오육칠팔구"
    result = []
    i = 0
    while n > 0:
        n, r = divmod(n, 10)
        if r > 0:
            result.append(nums[r-1]+units[i])
        i += 1
    return "".join(reversed(result))
    
print(read_number(33000))
```

    삼만삼천
    


```python
spark.udf.register("read_number", read_number)
```




    <function __main__.read_number(n)>




```python
spark.sql("select price, read_number(price) from product").show()
```

    +-----+------------------+
    |price|read_number(price)|
    +-----+------------------+
    |31231|삼만일천이백삼십일|
    |31223|삼만일천이백이십삼|
    |31432|삼만일천사백삼십이|
    |30323|    삼만삼백이십삼|
    |29234|이만구천이백삼십사|
    +-----+------------------+
    
    


```python
def get_weekday(date):
    import calendar
    return calendar.day_name[date.weekday()]

spark.udf.register("get_weekday", get_weekday)
```




    <function __main__.get_weekday(date)>




```python
query = """
select
    product,
    date,
    get_weekday(to_date(date)),
    read_number(price)
from product
"""

spark.sql(query).show()
```

    +-------+----------+--------------------------+------------------+
    |product|      date|get_weekday(to_date(date))|read_number(price)|
    +-------+----------+--------------------------+------------------+
    |      A|2022-08-14|                    Sunday|삼만일천이백삼십일|
    |      B|2022-08-14|                    Sunday|삼만일천이백이십삼|
    |      C|2022-08-14|                    Sunday|삼만일천사백삼십이|
    |      D|2022-08-14|                    Sunday|    삼만삼백이십삼|
    |      E|2022-08-14|                    Sunday|이만구천이백삼십사|
    +-------+----------+--------------------------+------------------+
    
    


```python
spark.stop()
```


```python

```
