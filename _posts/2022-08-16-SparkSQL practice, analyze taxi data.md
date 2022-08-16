---
layout: post
title: SparkSQL practice, analyze taxi data
subtitle: SparkSQL practice
categories: BigData_SQL
tags: SparkSQL analyze data
---


```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("trip_count_sql").getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
# 한 폴더에 있는 trip 데이터 전부 로딩
trip_files = "\\trips\\"
zone_file = "taxi+_zone_lookup.csv"
```


```python
trips_df = spark.read.csv(f"file:///{directory}\\{trip_files}", inferSchema=True, header=True)
zone_df = spark.read.csv(f"file:///{directory}\\{zone_file}", inferSchema=True, header=True)
```


```python
# 데이터프레임 확인
trips_df.printSchema()
zone_df.printSchema()
```

    root
     |-- VendorID: integer (nullable = true)
     |-- tpep_pickup_datetime: string (nullable = true)
     |-- tpep_dropoff_datetime: string (nullable = true)
     |-- passenger_count: integer (nullable = true)
     |-- trip_distance: double (nullable = true)
     |-- RatecodeID: integer (nullable = true)
     |-- store_and_fwd_flag: string (nullable = true)
     |-- PULocationID: integer (nullable = true)
     |-- DOLocationID: integer (nullable = true)
     |-- payment_type: integer (nullable = true)
     |-- fare_amount: double (nullable = true)
     |-- extra: double (nullable = true)
     |-- mta_tax: double (nullable = true)
     |-- tip_amount: double (nullable = true)
     |-- tolls_amount: double (nullable = true)
     |-- improvement_surcharge: double (nullable = true)
     |-- total_amount: double (nullable = true)
     |-- congestion_surcharge: double (nullable = true)
    
    root
     |-- LocationID: integer (nullable = true)
     |-- Borough: string (nullable = true)
     |-- Zone: string (nullable = true)
     |-- service_zone: string (nullable = true)
    
    


```python
trips_df.createOrReplaceTempView("trips")
zone_df.createOrReplaceTempView("zone")
```

### Join

사용할 두개의 데이터프레임을 join한 후 이를 계속 사용한다


```python
query = """
SELECT
    t.VendorID as vendor_id,
    TO_DATE(t.tpep_pickup_datetime) as pickup_date,
    TO_DATE(t.tpep_dropoff_datetime) as dropoff_date,
    HOUR(t.tpep_pickup_datetime) as pickup_time,
    HOUR(t.tpep_dropoff_datetime) as dropoff_time,
    t.passenger_count,
    t.trip_distance,
    t.payment_type,
    t.fare_amount,
    t.tip_amount,
    t.tolls_amount,
    t.total_amount,
    pz.Zone as pickup_zone,
    dz.Zone as dropoff_zone

FROM
    trips t
LEFT JOIN zone pz ON t.PULocationID = pz.LocationID
LEFT JOIN zone dz ON t.DOLocationID = dz.LocationID
"""

comb_df = spark.sql(query)
comb_df.createOrReplaceTempView("comb")
```


```python
comb_df.printSchema()
```

    root
     |-- vendor_id: integer (nullable = true)
     |-- pickup_date: date (nullable = true)
     |-- dropoff_date: date (nullable = true)
     |-- pickup_time: integer (nullable = true)
     |-- dropoff_time: integer (nullable = true)
     |-- passenger_count: integer (nullable = true)
     |-- trip_distance: double (nullable = true)
     |-- payment_type: integer (nullable = true)
     |-- fare_amount: double (nullable = true)
     |-- tip_amount: double (nullable = true)
     |-- tolls_amount: double (nullable = true)
     |-- total_amount: double (nullable = true)
     |-- pickup_zone: string (nullable = true)
     |-- dropoff_zone: string (nullable = true)
    
    


```python
comb_df.show(5)
```

    +---------+-----------+------------+-----------+------------+---------------+-------------+------------+-----------+----------+------------+------------+-----------------+--------------+
    |vendor_id|pickup_date|dropoff_date|pickup_time|dropoff_time|passenger_count|trip_distance|payment_type|fare_amount|tip_amount|tolls_amount|total_amount|      pickup_zone|  dropoff_zone|
    +---------+-----------+------------+-----------+------------+---------------+-------------+------------+-----------+----------+------------+------------+-----------------+--------------+
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.0|           2|        3.0|       0.0|         0.0|         4.3|               NV|            NV|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.0|           2|        2.5|       0.0|         0.0|         3.8|   Manhattanville|Manhattanville|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.0|           2|        3.5|       0.0|         0.0|         4.8|   Manhattanville|Manhattanville|
    |        1| 2021-03-01|  2021-03-01|          0|           0|              0|         16.5|           1|       51.0|     11.65|        6.12|       70.07|LaGuardia Airport|            NA|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         1.13|           1|        5.5|      1.86|         0.0|       11.16|     East Chelsea|            NV|
    +---------+-----------+------------+-----------+------------+---------------+-------------+------------+-----------+----------+------------+------------+-----------------+--------------+
    only showing top 5 rows
    
    


```python
spark.sql("SELECT pickup_date, pickup_time FROM comb WHERE pickup_time > 0").show()
```

    +-----------+-----------+
    |pickup_date|pickup_time|
    +-----------+-----------+
    | 2021-02-28|         23|
    | 2021-02-28|         23|
    | 2021-02-28|         23|
    | 2021-02-28|         23|
    | 2021-02-28|         23|
    | 2021-02-28|         23|
    | 2021-02-28|         23|
    | 2021-03-01|         22|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    | 2021-03-01|          1|
    +-----------+-----------+
    only showing top 20 rows
    
    

### 데이터 cleaning

2021년도 데이터 분석을 위해 2020 이하 데이터의 존재 유무를 확인 후 제거

택시 데이터의 경우 2021년 1월 1일 자정을 넘기기 전에 택시를 탑승해 2021년에 하차한 경우, 잘못된 데이터와 같은 목적에 적합하지 않는 데이터가 포함되어 있을 수 있기 때문에 자신의 목적과 적합하지 않는 데이터가 속해 있는지 반드시 확인해야 한다


```python
query = """
    SELECT pickup_date, pickup_time
    FROM comb
    WHERE pickup_date < '2020-12-31'
"""
spark.sql(query).show()
```

    +-----------+-----------+
    |pickup_date|pickup_time|
    +-----------+-----------+
    | 2009-01-01|          0|
    | 2008-12-31|         23|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    | 2009-01-01|          1|
    | 2009-01-01|          0|
    | 2008-12-31|         23|
    | 2008-12-31|         23|
    | 2008-12-31|         23|
    | 2008-12-31|         23|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    | 2009-01-01|         16|
    | 2009-01-01|         16|
    | 2009-01-01|          0|
    | 2009-01-01|          0|
    +-----------+-----------+
    only showing top 20 rows
    
    


```python
# 택시 요금(total_amount) 데이터 확인
comb_df.select("total_amount").describe().show()
```

    +-------+-----------------+
    |summary|     total_amount|
    +-------+-----------------+
    |  count|         15000700|
    |   mean|18.75545205708744|
    | stddev|145.7442452805979|
    |    min|           -647.8|
    |    max|         398469.2|
    +-------+-----------------+
    
    

### 데이터 cleaner(total_amount)

택시비의 최솟값 min < 0 경우,

택시비의 최댓값 max가 약 4억원인 경우


```python
# 택시 이동 거리(trip_distance) 확인
comb_df.select("trip_distance").describe().show()
```

    +-------+-----------------+
    |summary|    trip_distance|
    +-------+-----------------+
    |  count|         15000700|
    |   mean|6.628629402627818|
    | stddev|671.7293482115828|
    |    min|              0.0|
    |    max|        332541.19|
    +-------+-----------------+
    
    

### 데이터 cleaner(trip_distance)

max 거리의 경우 너무 큰 33만 마일

min 거리가 0인 경우


```python
# 월별(trips) 확인
query = """
     SELECT
        DATE_TRUNC("MM", c.pickup_date) AS month,
        count(*) AS trips
     FROM comb c
     
     GROUP BY month
     ORDER BY month desc
"""

spark.sql(query).show()
```

    +-------------------+-------+
    |              month|  trips|
    +-------------------+-------+
    |2029-05-01 00:00:00|      1|
    |2021-12-01 00:00:00|      5|
    |2021-11-01 00:00:00|      5|
    |2021-10-01 00:00:00|      3|
    |2021-09-01 00:00:00|      3|
    |2021-08-01 00:00:00|     36|
    |2021-07-01 00:00:00|2821430|
    |2021-06-01 00:00:00|2834204|
    |2021-05-01 00:00:00|2507075|
    |2021-04-01 00:00:00|2171215|
    |2021-03-01 00:00:00|1925130|
    |2021-02-01 00:00:00|1371688|
    |2021-01-01 00:00:00|1369749|
    |2020-12-01 00:00:00|     16|
    |2009-01-01 00:00:00|    111|
    |2008-12-01 00:00:00|     26|
    |2004-04-01 00:00:00|      1|
    |2003-01-01 00:00:00|      1|
    |2002-12-01 00:00:00|      1|
    +-------------------+-------+
    
    


```python
comb_df.select("passenger_count").describe().show()
```

    +-------+------------------+
    |summary|   passenger_count|
    +-------+------------------+
    |  count|          14166672|
    |   mean|1.4253783104458126|
    | stddev|  1.04432704905968|
    |    min|                 0|
    |    max|                 9|
    +-------+------------------+
    
    


```python
query = """
SELECT
    c.*
FROM
    comb c
WHERE
    c.total_amount < 5000
    AND c.total_amount > 0
    AND c.trip_distance < 100
    AND c.passenger_count < 4
    AND c.pickup_date >= '2021-01-01'
    AND c.pickup_date < '2021-08-01'
"""

cleaned_df = spark.sql(query)
cleaned_df.createOrReplaceTempView("cleaned")
```


```python
cleaned_df.show()
```

    +---------+-----------+------------+-----------+------------+---------------+-------------+------------+-----------+----------+------------+------------+--------------------+--------------------+
    |vendor_id|pickup_date|dropoff_date|pickup_time|dropoff_time|passenger_count|trip_distance|payment_type|fare_amount|tip_amount|tolls_amount|total_amount|         pickup_zone|        dropoff_zone|
    +---------+-----------+------------+-----------+------------+---------------+-------------+------------+-----------+----------+------------+------------+--------------------+--------------------+
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.0|           2|        3.0|       0.0|         0.0|         4.3|                  NV|                  NV|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.0|           2|        2.5|       0.0|         0.0|         3.8|      Manhattanville|      Manhattanville|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.0|           2|        3.5|       0.0|         0.0|         4.8|      Manhattanville|      Manhattanville|
    |        1| 2021-03-01|  2021-03-01|          0|           0|              0|         16.5|           1|       51.0|     11.65|        6.12|       70.07|   LaGuardia Airport|                  NA|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         1.13|           1|        5.5|      1.86|         0.0|       11.16|        East Chelsea|                  NV|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         2.68|           1|       10.5|      4.29|         0.0|       18.59|Upper West Side S...|      Yorkville East|
    |        1| 2021-03-01|  2021-03-01|          0|           0|              1|         12.4|           1|       40.0|       0.0|         0.0|        43.8|Penn Station/Madi...|           Flatlands|
    |        1| 2021-03-01|  2021-03-01|          0|           1|              2|          9.7|           2|       31.0|       0.0|         0.0|        32.3|         JFK Airport|                  NA|
    |        1| 2021-03-01|  2021-03-01|          0|           0|              1|          9.3|           1|       26.5|      7.25|        6.12|       43.67|   LaGuardia Airport|     Lenox Hill West|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         9.58|           1|       28.5|      7.68|        6.12|        46.1|   LaGuardia Airport|        Clinton West|
    |        1| 2021-03-01|  2021-03-01|          0|           0|              1|         16.2|           2|       44.0|       0.0|         0.0|        45.3|         JFK Airport|           Homecrest|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         3.58|           1|       13.5|       2.0|         0.0|        19.3|     Lenox Hill East|             Astoria|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         0.91|           1|        6.0|       5.0|         0.0|        14.8|Upper West Side S...|Upper West Side N...|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              2|         2.57|           2|       11.5|       0.0|         0.0|        12.8|    Hamilton Heights|      Central Harlem|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          0.4|           2|        4.0|       0.0|         0.0|         5.3|   East Harlem North|      Central Harlem|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         3.26|           2|       13.5|       0.0|         0.0|        17.3|Upper West Side S...| Little Italy/NoLiTa|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|        13.41|           1|       36.5|      9.45|         0.0|       47.25|         JFK Airport|           Flatlands|
    |        1| 2021-03-01|  2021-03-01|          0|           0|              2|         18.3|           1|       52.0|       0.0|        6.12|       61.42|         JFK Airport|Times Sq/Theatre ...|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|         1.53|           1|        8.0|      2.36|         0.0|       14.16|Sutton Place/Turt...|        Clinton East|
    |        2| 2021-03-01|  2021-03-01|          0|           0|              1|          2.0|           2|        8.0|       0.0|         0.0|        11.8|        Clinton East|        East Chelsea|
    +---------+-----------+------------+-----------+------------+---------------+-------------+------------+-----------+----------+------------+------------+--------------------+--------------------+
    only showing top 20 rows
    
    

### 시각화 분석


```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
```


```python
# 일 별 trips 수 확인 및 분석
query = """
SELECT
    c.pickup_date,
    COUNT(*) AS trips
FROM
    cleaned c
GROUP BY
    c.pickup_date
"""

pd_df = spark.sql(query).toPandas()
pd_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pickup_date</th>
      <th>trips</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-03-22</td>
      <td>52415</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-03-07</td>
      <td>34374</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-03-21</td>
      <td>39659</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-05-27</td>
      <td>83211</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-03-14</td>
      <td>38738</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>207</th>
      <td>2021-03-30</td>
      <td>58625</td>
    </tr>
    <tr>
      <th>208</th>
      <td>2021-03-27</td>
      <td>59735</td>
    </tr>
    <tr>
      <th>209</th>
      <td>2021-03-29</td>
      <td>50624</td>
    </tr>
    <tr>
      <th>210</th>
      <td>2021-04-27</td>
      <td>67953</td>
    </tr>
    <tr>
      <th>211</th>
      <td>2021-03-28</td>
      <td>35535</td>
    </tr>
  </tbody>
</table>
<p>212 rows × 2 columns</p>
</div>




```python
fig, ax = plt.subplots(figsize = (16, 6))
sns.lineplot(x="pickup_date", y="trips", data=pd_df)
plt.show()
# 그래프를 볼 경우 8월달 데이터를 포함시켜 위 query에서 8월 데이터를 삭제해 준다
```


    
![output_23_0](https://user-images.githubusercontent.com/77920565/184827132-42994f4e-0d1f-4681-b213-cd891173ca00.png)    



```python
# 요일 별 trips 중간값 확인
query = """
SELECT
    c.pickup_date,
    DATE_FORMAT(c.pickup_date, 'EEEE') as day_of_week,
    COUNT(*) AS trips
FROM cleaned c

GROUP BY c.pickup_date, day_of_week

"""

pd_df2 = spark.sql(query).toPandas()
pd_df2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pickup_date</th>
      <th>day_of_week</th>
      <th>trips</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-03-24</td>
      <td>Wednesday</td>
      <td>60382</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-03-03</td>
      <td>Wednesday</td>
      <td>55222</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-03-05</td>
      <td>Friday</td>
      <td>60984</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-03-09</td>
      <td>Tuesday</td>
      <td>54085</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-03-26</td>
      <td>Friday</td>
      <td>64810</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>207</th>
      <td>2021-03-30</td>
      <td>Tuesday</td>
      <td>58625</td>
    </tr>
    <tr>
      <th>208</th>
      <td>2021-04-24</td>
      <td>Saturday</td>
      <td>70466</td>
    </tr>
    <tr>
      <th>209</th>
      <td>2021-03-28</td>
      <td>Sunday</td>
      <td>35535</td>
    </tr>
    <tr>
      <th>210</th>
      <td>2021-04-25</td>
      <td>Sunday</td>
      <td>44048</td>
    </tr>
    <tr>
      <th>211</th>
      <td>2021-03-29</td>
      <td>Monday</td>
      <td>50624</td>
    </tr>
  </tbody>
</table>
<p>212 rows × 3 columns</p>
</div>




```python
data = pd_df2.groupby("day_of_week")["trips"].median().to_frame().reset_index()
data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>day_of_week</th>
      <th>trips</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Friday</td>
      <td>73094.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Monday</td>
      <td>56063.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Saturday</td>
      <td>61471.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunday</td>
      <td>43131.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Thursday</td>
      <td>72190.5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Tuesday</td>
      <td>66821.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wednesday</td>
      <td>69398.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 요일별 정렬(요일별 숫자 부여 후 정렬)
data["sort_dow"] = data["day_of_week"].replace({
    "Sunday": 0,
    "Monday": 1,
    "Tuesday": 2,
    "Wednesday": 3,
    "Thursday": 4,
    "Friday": 5,
    "Saturday": 6,
})
data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>day_of_week</th>
      <th>trips</th>
      <th>sort_dow</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Friday</td>
      <td>73094.0</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Monday</td>
      <td>56063.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Saturday</td>
      <td>61471.0</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunday</td>
      <td>43131.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Thursday</td>
      <td>72190.5</td>
      <td>4</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Tuesday</td>
      <td>66821.0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wednesday</td>
      <td>69398.5</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
data_sorted = data.sort_values(by="sort_dow")
data_sorted
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>day_of_week</th>
      <th>trips</th>
      <th>sort_dow</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>Sunday</td>
      <td>43131.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Monday</td>
      <td>56063.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Tuesday</td>
      <td>66821.0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wednesday</td>
      <td>69398.5</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Thursday</td>
      <td>72190.5</td>
      <td>4</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Friday</td>
      <td>73094.0</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Saturday</td>
      <td>61471.0</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(12,5))
sns.barplot(
    x='day_of_week',
    y='trips',
    data=data_sorted
)
plt.show()
```


    
![output_28_0](https://user-images.githubusercontent.com/77920565/184827145-9e33e3bd-cf4b-4fa6-af59-a847545ceb04.png)    



```python
spark.stop()
```
