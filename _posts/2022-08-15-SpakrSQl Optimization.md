---
layout: post
title: SparkSQL Optimization
subtitle: SparkSQL
categories: BigData_SQL
tags: SparkSQL Optimization
---

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("trip_count_sql").getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
trip_file = "fhvhv_tripdata_2020-03.csv"
```


```python
# inferSchema: 자동으로 스키마 예측
data = spark.read.csv(f"file:///{directory}\\{trip_file}", inferSchema=True, header=True)

```

### 불러온 데이터 확인


```python
data.show(5)
```

    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    |hvfhs_license_num|dispatching_base_num|    pickup_datetime|   dropoff_datetime|PULocationID|DOLocationID|SR_Flag|
    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    |           HV0005|              B02510|2020-03-01 00:03:40|2020-03-01 00:23:39|          81|         159|   null|
    |           HV0005|              B02510|2020-03-01 00:28:05|2020-03-01 00:38:57|         168|         119|   null|
    |           HV0003|              B02764|2020-03-01 00:03:07|2020-03-01 00:15:04|         137|         209|      1|
    |           HV0003|              B02764|2020-03-01 00:18:42|2020-03-01 00:38:42|         209|          80|   null|
    |           HV0003|              B02764|2020-03-01 00:44:24|2020-03-01 00:58:44|         256|         226|   null|
    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    only showing top 5 rows
    
    

### SQL 사용해 임시 VIEW 생성


```python
data.createOrReplaceTempView("mobility_data")
# 데이터프레임에 SQL 사용 가능
```


```python
query = """
select *
from mobility_data
limit 5
"""
spark.sql(query).show()
```

    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    |hvfhs_license_num|dispatching_base_num|    pickup_datetime|   dropoff_datetime|PULocationID|DOLocationID|SR_Flag|
    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    |           HV0005|              B02510|2020-03-01 00:03:40|2020-03-01 00:23:39|          81|         159|   null|
    |           HV0005|              B02510|2020-03-01 00:28:05|2020-03-01 00:38:57|         168|         119|   null|
    |           HV0003|              B02764|2020-03-01 00:03:07|2020-03-01 00:15:04|         137|         209|      1|
    |           HV0003|              B02764|2020-03-01 00:18:42|2020-03-01 00:38:42|         209|          80|   null|
    |           HV0003|              B02764|2020-03-01 00:44:24|2020-03-01 00:58:44|         256|         226|   null|
    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    
    

### Spark_SQL 사용 이유

1. 빅데이터 집계
2. 승차 년-월-일 별 counting


```python
query = """
select split(pickup_datetime, ' ')[0] as pickup_date, count(*) as tirps
from mobility_data
group by pickup_date
"""
spark.sql(query).show()
```

    +-----------+------+
    |pickup_date| tirps|
    +-----------+------+
    | 2020-03-03|697880|
    | 2020-03-02|648986|
    | 2020-03-01|784246|
    | 2020-03-05|731165|
    | 2020-03-04|707879|
    | 2020-03-06|872012|
    | 2020-03-07|886071|
    | 2020-03-10|626474|
    | 2020-03-09|628940|
    | 2020-03-08|731222|
    | 2020-03-12|643257|
    | 2020-03-11|628601|
    | 2020-03-13|660914|
    | 2020-03-15|448125|
    | 2020-03-14|569397|
    | 2020-03-16|391518|
    | 2020-03-20|261900|
    | 2020-03-19|252773|
    | 2020-03-17|312298|
    | 2020-03-18|269232|
    +-----------+------+
    only showing top 20 rows
    
    


```python
# 실행 계획
spark.sql(query).show()
```

    +-----------+------+
    |pickup_date| tirps|
    +-----------+------+
    | 2020-03-03|697880|
    | 2020-03-02|648986|
    | 2020-03-01|784246|
    | 2020-03-05|731165|
    | 2020-03-04|707879|
    | 2020-03-06|872012|
    | 2020-03-07|886071|
    | 2020-03-10|626474|
    | 2020-03-09|628940|
    | 2020-03-08|731222|
    | 2020-03-12|643257|
    | 2020-03-11|628601|
    | 2020-03-13|660914|
    | 2020-03-15|448125|
    | 2020-03-14|569397|
    | 2020-03-16|391518|
    | 2020-03-20|261900|
    | 2020-03-19|252773|
    | 2020-03-17|312298|
    | 2020-03-18|269232|
    +-----------+------+
    only showing top 20 rows
    
    


```python
spark.sql(query).explain(True)
```

    == Parsed Logical Plan ==
    'Aggregate ['pickup_date], ['split('pickup_datetime,  )[0] AS pickup_date#225, 'count(1) AS tirps#226]
    +- 'UnresolvedRelation [mobility_data], [], false
    
    == Analyzed Logical Plan ==
    pickup_date: string, tirps: bigint
    Aggregate [split(pickup_datetime#18,  , -1)[0]], [split(pickup_datetime#18,  , -1)[0] AS pickup_date#225, count(1) AS tirps#226L]
    +- SubqueryAlias mobility_data
       +- View (`mobility_data`, [hvfhs_license_num#16,dispatching_base_num#17,pickup_datetime#18,dropoff_datetime#19,PULocationID#20,DOLocationID#21,SR_Flag#22])
          +- Relation [hvfhs_license_num#16,dispatching_base_num#17,pickup_datetime#18,dropoff_datetime#19,PULocationID#20,DOLocationID#21,SR_Flag#22] csv
    
    == Optimized Logical Plan ==
    Aggregate [_groupingexpression#230], [_groupingexpression#230 AS pickup_date#225, count(1) AS tirps#226L]
    +- Project [split(pickup_datetime#18,  , -1)[0] AS _groupingexpression#230]
       +- Relation [hvfhs_license_num#16,dispatching_base_num#17,pickup_datetime#18,dropoff_datetime#19,PULocationID#20,DOLocationID#21,SR_Flag#22] csv
    
    == Physical Plan ==
    AdaptiveSparkPlan isFinalPlan=false
    +- HashAggregate(keys=[_groupingexpression#230], functions=[count(1)], output=[pickup_date#225, tirps#226L])
       +- Exchange hashpartitioning(_groupingexpression#230, 200), ENSURE_REQUIREMENTS, [id=#327]
          +- HashAggregate(keys=[_groupingexpression#230], functions=[partial_count(1)], output=[_groupingexpression#230, count#232L])
             +- Project [split(pickup_datetime#18,  , -1)[0] AS _groupingexpression#230]
                +- FileScan csv [pickup_datetime#18] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/C:/Users/sonjj/study_spark/data/fhvhv_tripdata_2020-03.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<pickup_datetime:string>
    
    

### SQL을 사용해 최적화

Optimized Logical Plan을 살펴볼 경우 데이터에서 필요한 데이터의 위치를 확인 후 가져와 select 기존 Logical Plan에 비해 데이터 filter 과정을 거친 후 이를 바탕으로 데이터 분석(최적화)


```python
spark.stop()
```

---

### join을 이용한 두개 데이터 분석

```python
directory = "C:\\Users\\sonjj\\study_spark\\data"
trip_file = "fhvhv_tripdata_2020-03.csv"
zone_file = "taxi+_zone_lookup.csv"
```


```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("trip_count_sql").getOrCreate()
```


```python
trip_data = spark.read.csv(f"file:///{directory}\\{trip_file}", inferSchema = True, header = True)
zone_data = spark.read.csv(f"file:///{directory}\\{zone_file}", inferSchema = True, header = True)
```


```python
trip_data.show(5)
```

    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    |hvfhs_license_num|dispatching_base_num|    pickup_datetime|   dropoff_datetime|PULocationID|DOLocationID|SR_Flag|
    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    |           HV0005|              B02510|2020-03-01 00:03:40|2020-03-01 00:23:39|          81|         159|   null|
    |           HV0005|              B02510|2020-03-01 00:28:05|2020-03-01 00:38:57|         168|         119|   null|
    |           HV0003|              B02764|2020-03-01 00:03:07|2020-03-01 00:15:04|         137|         209|      1|
    |           HV0003|              B02764|2020-03-01 00:18:42|2020-03-01 00:38:42|         209|          80|   null|
    |           HV0003|              B02764|2020-03-01 00:44:24|2020-03-01 00:58:44|         256|         226|   null|
    +-----------------+--------------------+-------------------+-------------------+------------+------------+-------+
    only showing top 5 rows
    
    


```python
zone_data.show(5)
```

    +----------+-------------+--------------------+------------+
    |LocationID|      Borough|                Zone|service_zone|
    +----------+-------------+--------------------+------------+
    |         1|          EWR|      Newark Airport|         EWR|
    |         2|       Queens|         Jamaica Bay|   Boro Zone|
    |         3|        Bronx|Allerton/Pelham G...|   Boro Zone|
    |         4|    Manhattan|       Alphabet City| Yellow Zone|
    |         5|Staten Island|       Arden Heights|   Boro Zone|
    +----------+-------------+--------------------+------------+
    only showing top 5 rows
    
    


```python
trip_data.createOrReplaceTempView("trip_data")
zone_data.createOrReplaceTempView("zone_data")
```

### Data 스키마 설명
- PULocationID(Pick Up Location ID) : 승차 Location ID

- DOLocationID(Drop Off Location ID) : 하차 Location ID


```python
# 승차 Location(PULocationID)별 개수 세기
# trip_data: 승/하차에 대한 정보(위치 정보: 정수로 표시)
# zone_data: 승/하차 위치에 대한 정보(trip_data에 있는 정수에 zone_data join)
query = """
select borough, count(*) as trips
from(
    select zone_data.Borough as borough
    from trip_data 
    join zone_data on trip_data.PULocationID = zone_data.LocationID
)
group by borough
"""
spark.sql(query).show()
```

    +-------------+-------+
    |      borough|  trips|
    +-------------+-------+
    |       Queens|2437383|
    |          EWR|    362|
    |      Unknown|    845|
    |     Brooklyn|3735764|
    |Staten Island| 178818|
    |    Manhattan|4953140|
    |        Bronx|2086592|
    +-------------+-------+
    
    


```python
# 승차 Location(DOLocationID)별 개수 세기
# trip_data: 승/하차에 대한 정보(위치 정보: 정수로 표시)
# zone_data: 승/하차 위치에 대한 정보(trip_data에 있는 정수에 zone_data join)
query = """
select borough, count(*) as trips
from(
    select zone_data.Borough as borough
    from trip_data 
    join zone_data on trip_data.DOLocationID = zone_data.LocationID
)
group by borough
"""
spark.sql(query).show()
```

    +-------------+-------+
    |      borough|  trips|
    +-------------+-------+
    |       Queens|2468408|
    |          EWR|  65066|
    |      Unknown| 387759|
    |     Brooklyn|3696682|
    |Staten Island| 177727|
    |    Manhattan|4553776|
    |        Bronx|2043486|
    +-------------+-------+
    
    


```python
query = """
select zone_data.Zone, count(*) as trips

from trip_data
join zone_data on trip_data.PULocationID = zone_data.locationID

where trip_data.hvfhs_license_num = "HV0003"

group by zone_data.Zone order by trips desc
"""
spark.sql(query).show()
```

    +--------------------+------+
    |                Zone| trips|
    +--------------------+------+
    | Crown Heights North|163091|
    |       East New York|134198|
    |         JFK Airport|114179|
    |        East Village|112017|
    |      Bushwick South|110150|
    |Central Harlem North|108070|
    |   LaGuardia Airport|104119|
    |Washington Height...| 97324|
    |Flatbush/Ditmas Park| 95724|
    |            Canarsie| 94484|
    |TriBeCa/Civic Center| 94155|
    |             Astoria| 92676|
    |             Bedford| 90352|
    |      Midtown Center| 90261|
    |  Stuyvesant Heights| 88749|
    |            Union Sq| 88372|
    |Times Sq/Theatre ...| 86870|
    |Prospect-Lefferts...| 84347|
    |         Brownsville| 82764|
    |Mott Haven/Port M...| 82396|
    +--------------------+------+
    only showing top 20 rows
    
    


```python
spark.sql(query).explain(True)
```

    == Parsed Logical Plan ==
    'Sort ['trips DESC NULLS LAST], true
    +- 'Aggregate ['zone_data.Zone], ['zone_data.Zone, 'count(1) AS trips#425]
       +- 'Filter ('trip_data.hvfhs_license_num = HV0003)
          +- 'Join Inner, ('trip_data.PULocationID = 'zone_data.locationID)
             :- 'UnresolvedRelation [trip_data], [], false
             +- 'UnresolvedRelation [zone_data], [], false
    
    == Analyzed Logical Plan ==
    Zone: string, trips: bigint
    Sort [trips#425L DESC NULLS LAST], true
    +- Aggregate [Zone#226], [Zone#226, count(1) AS trips#425L]
       +- Filter (hvfhs_license_num#194 = HV0003)
          +- Join Inner, (PULocationID#198 = locationID#224)
             :- SubqueryAlias trip_data
             :  +- View (`trip_data`, [hvfhs_license_num#194,dispatching_base_num#195,pickup_datetime#196,dropoff_datetime#197,PULocationID#198,DOLocationID#199,SR_Flag#200])
             :     +- Relation [hvfhs_license_num#194,dispatching_base_num#195,pickup_datetime#196,dropoff_datetime#197,PULocationID#198,DOLocationID#199,SR_Flag#200] csv
             +- SubqueryAlias zone_data
                +- View (`zone_data`, [LocationID#224,Borough#225,Zone#226,service_zone#227])
                   +- Relation [LocationID#224,Borough#225,Zone#226,service_zone#227] csv
    
    == Optimized Logical Plan ==
    Sort [trips#425L DESC NULLS LAST], true
    +- Aggregate [Zone#226], [Zone#226, count(1) AS trips#425L]
       +- Project [Zone#226]
          +- Join Inner, (PULocationID#198 = locationID#224)
             :- Project [PULocationID#198]
             :  +- Filter ((isnotnull(hvfhs_license_num#194) AND (hvfhs_license_num#194 = HV0003)) AND isnotnull(PULocationID#198))
             :     +- Relation [hvfhs_license_num#194,dispatching_base_num#195,pickup_datetime#196,dropoff_datetime#197,PULocationID#198,DOLocationID#199,SR_Flag#200] csv
             +- Project [LocationID#224, Zone#226]
                +- Filter isnotnull(locationID#224)
                   +- Relation [LocationID#224,Borough#225,Zone#226,service_zone#227] csv
    
    == Physical Plan ==
    AdaptiveSparkPlan isFinalPlan=false
    +- Sort [trips#425L DESC NULLS LAST], true, 0
       +- Exchange rangepartitioning(trips#425L DESC NULLS LAST, 200), ENSURE_REQUIREMENTS, [id=#850]
          +- HashAggregate(keys=[Zone#226], functions=[count(1)], output=[Zone#226, trips#425L])
             +- Exchange hashpartitioning(Zone#226, 200), ENSURE_REQUIREMENTS, [id=#847]
                +- HashAggregate(keys=[Zone#226], functions=[partial_count(1)], output=[Zone#226, count#430L])
                   +- Project [Zone#226]
                      +- BroadcastHashJoin [PULocationID#198], [locationID#224], Inner, BuildRight, false
                         :- Project [PULocationID#198]
                         :  +- Filter ((isnotnull(hvfhs_license_num#194) AND (hvfhs_license_num#194 = HV0003)) AND isnotnull(PULocationID#198))
                         :     +- FileScan csv [hvfhs_license_num#194,PULocationID#198] Batched: false, DataFilters: [isnotnull(hvfhs_license_num#194), (hvfhs_license_num#194 = HV0003), isnotnull(PULocationID#198)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/C:/Users/sonjj/study_spark/data/fhvhv_tripdata_2020-03.csv], PartitionFilters: [], PushedFilters: [IsNotNull(hvfhs_license_num), EqualTo(hvfhs_license_num,HV0003), IsNotNull(PULocationID)], ReadSchema: struct<hvfhs_license_num:string,PULocationID:int>
                         +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint)),false), [id=#842]
                            +- Filter isnotnull(locationID#224)
                               +- FileScan csv [LocationID#224,Zone#226] Batched: false, DataFilters: [isnotnull(LocationID#224)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/C:/Users/sonjj/study_spark/data/taxi+_zone_lookup.csv], PartitionFilters: [], PushedFilters: [IsNotNull(LocationID)], ReadSchema: struct<LocationID:int,Zone:string>
    
    


```python
spark.stop()
```
