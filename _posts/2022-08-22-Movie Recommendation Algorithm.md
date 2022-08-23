---
layout: post
title: Movie Recommendation Algorithm
subtitle: Machine Learning Practice
categories: Machine_Learning
tags: machine_learning
---

### 영화 추천 알고리즘

#### 원리

- A와 B가 동일한 영화에 대해 비슷한 별점을 주었을 경우 영화 취향이 비슷하다 판단 후 상대방이 보지 않은 영화에 대해 높은 별점의 영화가 있다면 그것을 추천해 준다.

- 한 사람이 모든 영화를 볼 수 없기 때문에 모든 영화에 대한 평점을 알 수 없다, 따라서 비슷한 영화 취향을 가지고 있는 사람들을 비교하여 서로 평점을 주지 않은 영화에 대한 평점을 예측하여 이를 바탕으로 추천한다.

#### Matrix

![matrix](https://user-images.githubusercontent.com/77920565/185779618-96d8c2c6-6811-491d-9a4d-6a05f7e486b0.png)

- User Matrix: 영화에 대한 Matrix로 특정 영화를 본 user
- Item Matrix: User에 대한 Matrix로 특정 사람이 본 영화
- Rating Matrix: User Matrix X Item Matrix로 표현할 수 있다
- User Matrix의 값과 Item Matrix의 값은 랜덤하게 채워지고, Item 행렬을 고정 시키고 User 행렬을 최적화 한다, 반대로 User 행렬을 고정 시키고 Item 행렬을 최적화 시킨다.

=> 결과적으로 User Matrix의 값과 Item Matrix의 결과물을 합쳐 Rating Matrix와 최대한 가까운 Matrix가 생성되게 되고, 이 Matrix는 빈 칸에 있는 값들이 모두 채워진 형식이 된다.

### Practice

```python
from pyspark.sql import SparkSession
```

### OutOfMemory

많은 양의 데이터를 사용하다 보면 가끔 발생하는 오류를 방지하기 위해
OutOfMemory 오류가 발생하면 각종 설정을 추가적으로 해줄 수 있다


```python
MAX_MEMORY = '5g'
spark = SparkSession.builder.appName("movierecommendation")\
    .config("spark.executor.memory", MAX_MEMORY)\
    .config("spark.driver.memory", MAX_MEMORY)\
    .getOrCreate()
```


```python
directory = "C:\\Users\\sonjj\\study_spark\\data\\ml-25m"
filename = "ratings.csv"
```


```python
ratings_df = spark.read.csv(f"file:///{directory}\\{filename}", inferSchema = True, header = True)
ratings_df.show()
```

    +------+-------+------+----------+
    |userId|movieId|rating| timestamp|
    +------+-------+------+----------+
    |     1|    296|   5.0|1147880044|
    |     1|    306|   3.5|1147868817|
    |     1|    307|   5.0|1147868828|
    |     1|    665|   5.0|1147878820|
    |     1|    899|   3.5|1147868510|
    |     1|   1088|   4.0|1147868495|
    |     1|   1175|   3.5|1147868826|
    |     1|   1217|   3.5|1147878326|
    |     1|   1237|   5.0|1147868839|
    |     1|   1250|   4.0|1147868414|
    |     1|   1260|   3.5|1147877857|
    |     1|   1653|   4.0|1147868097|
    |     1|   2011|   2.5|1147868079|
    |     1|   2012|   2.5|1147868068|
    |     1|   2068|   2.5|1147869044|
    |     1|   2161|   3.5|1147868609|
    |     1|   2351|   4.5|1147877957|
    |     1|   2573|   4.0|1147878923|
    |     1|   2632|   5.0|1147878248|
    |     1|   2692|   5.0|1147869100|
    +------+-------+------+----------+
    only showing top 20 rows
    
    


```python
#timestamp 제거
ratings_df = ratings_df.select(["userid", "movieId", "rating"])
ratings_df.printSchema()
```

    root
     |-- userid: integer (nullable = true)
     |-- movieId: integer (nullable = true)
     |-- rating: double (nullable = true)
    
    


```python
ratings_df.select('rating').describe().show()
```

    +-------+------------------+
    |summary|            rating|
    +-------+------------------+
    |  count|          25000095|
    |   mean| 3.533854451353085|
    | stddev|1.0607439611423508|
    |    min|               0.5|
    |    max|               5.0|
    +-------+------------------+
    
    

### train, test 데이터

train, test 데이터 세트 분리

=> 모델에 대한 학습을 위해 train 데이터와 제대로 훈련이 되었는지 확인하기 위한 train 데이터를 분리하여 세트로 묶는다


```python
train_ratio = 0.8
test_ratio = 0.2

train_df, test_df = ratings_df.randomSplit([0.8, 0.2])
# 총 데이터를 train, test 데이터로 랜덤으로 섞어서 배분
```

### ALS 추천 알고리즘

- maxter: 훈련 횟수
- regParma: 정규화 파라미터
- userCol: user column
- itemCol: item에 대한  column


```python
from pyspark.ml.recommendation import ALS
```


```python
als = ALS(
    maxIter = 5,
    regParam = 0.1,
    userCol = "userid",
    itemCol = "movieId",
    ratingCol = "rating",
    coldStartStrategy = "drop"
)
```


```python
model = als.fit(train_df)
```

### 예측(model)


```python
predictions = model.transform(test_df)
predictions.show()
```

    +------+-------+------+----------+
    |userid|movieId|rating|prediction|
    +------+-------+------+----------+
    |    31|   1580|   3.0| 2.3154936|
    |   159|  54190|   5.0| 3.8682168|
    |   322|    463|   3.0| 3.2169788|
    |   375|   1580|   2.5| 3.5502603|
    |   385|   1088|   3.0|  3.129886|
    |   460|  44022|   3.0|  3.985426|
    |   481|   1580|   4.0|  3.559844|
    |   497|   2366|   4.0| 3.7733438|
    |   596|   1580|   3.0| 3.5715413|
    |   597|   3997|   1.0| 2.0389478|
    |   606|   5803|   4.5| 3.6832404|
    |   606|  44022|   4.5| 4.0517616|
    |   613|   1580|   3.0| 3.3293386|
    |   626|   2366|   3.0|  3.169262|
    |   626|   2866|   3.0| 3.3851285|
    |   626|   3175|   4.0| 3.4651163|
    |   772|   1580|   3.0|  3.179849|
    |   772|   2122|   2.0| 2.0400379|
    |   830|   1591|   2.0|  2.905604|
    |   833|   3175|   5.0| 3.2890477|
    +------+-------+------+----------+
    only showing top 20 rows
    
    

### Model 분석
위 표를 볼 경우 실제 rating(실제값)과 prediction(예측값)에 대한 오차의 범위를 파악해 볼 필요가있다.


```python
predictions.select("rating", "prediction").describe().show()
```

    +-------+------------------+------------------+
    |summary|            rating|        prediction|
    +-------+------------------+------------------+
    |  count|           4998157|           4998157|
    |   mean| 3.534814732710477|3.3985339229473572|
    | stddev|1.0604504076393308|0.6398912114190655|
    |    min|               0.5|        -1.0638217|
    |    max|               5.0|         6.2587276|
    +-------+------------------+------------------+
    
    

위 표를 볼 경우 stddev(분표도)가 prediection이 더 작은 것을 볼 수 있다. 또한 최소값과 최댓값의 경우 최소 0점에서 최대 5점의 범위를 벗어 난 것을 볼 수 있다. 이는 예측값이 제일 잘 어울리고 어울리지 않는 것을 보여 준다고 해섥할 수 있다. 이에 우리는 예측값과 실제값의 오차 범위를 확인해 볼 것이다.

### RMSE Evaluation

MSE: 평균 제곱 오차
$$
MSE: \frac{1}{N}\sum_{i=1}^{N}(y_i - t_i)^2
$$
- $y_i$: 예측값 ($\hat{y}$)
- $t_i$: 실제 값

RMSE: 평균 제곱 오차의 제곱근
$$
RMSE: \sqrt{\frac{1}{N}\sum_{i=1}^{N}(y_i - t_i)^2}
$$


```python
from pyspark.ml.evaluation import RegressionEvaluator
#영화 평점 예측(회귀)를 진행했기 때문에 RegressionEvaluator를 이용한다

evaluator = RegressionEvaluator(metricName='rmse', labelCol='rating', predictionCol='prediction')
```


```python
rmse = evaluator.evaluate(predictions)
print(rmse)
```

    0.8140739669886965
    

평균적으로 예측값에 대해 약 0.82 정도의 오차가 있는 것을 알 수 있다.


```python
# 각 사람에게 top3 영화를 추천해준다
model.recommendForAllUsers(3).show()
```

    C:\Users\sonjj\anaconda3\lib\site-packages\pyspark\sql\context.py:125: FutureWarning: Deprecated in 3.0.0. Use SparkSession.builder.getOrCreate() instead.
      warnings.warn(
    

    +------+--------------------+
    |userid|     recommendations|
    +------+--------------------+
    |    26|[{194434, 5.84182...|
    |    27|[{203086, 6.28317...|
    |    28|[{194434, 7.95826...|
    |    31|[{194434, 4.04793...|
    |    34|[{203086, 5.17719...|
    |    44|[{203086, 6.85985...|
    |    53|[{200930, 6.60731...|
    |    65|[{194434, 6.85464...|
    |    76|[{194434, 6.16198...|
    |    78|[{200930, 7.14571...|
    |    81|[{207824, 5.16132...|
    |    85|[{193369, 6.00314...|
    |   101|[{203086, 5.26059...|
    |   103|[{194434, 6.46885...|
    |   108|[{203086, 5.68607...|
    |   115|[{203086, 6.46553...|
    |   126|[{203086, 6.52667...|
    |   133|[{194434, 5.47059...|
    |   137|[{203086, 6.07446...|
    |   148|[{194434, 6.20442...|
    +------+--------------------+
    only showing top 20 rows
    
    


```python
# 각 movie에 어울리는 top3 user를 추천
model.recommendForAllItems(3).show()
```

    +-------+--------------------+
    |movieId|     recommendations|
    +-------+--------------------+
    |     12|[{87426, 5.099797...|
    |     26|[{105801, 4.91600...|
    |     27|[{14103, 5.172527...|
    |     28|[{3195, 5.374069}...|
    |     31|[{18230, 5.024632...|
    |     34|[{32202, 5.167577...|
    |     44|[{87426, 5.214043...|
    |     53|[{67565, 5.241554...|
    |     65|[{87426, 5.038391...|
    |     76|[{87426, 5.203023...|
    |     78|[{67467, 4.687769...|
    |     81|[{38766, 4.629398...|
    |     85|[{67565, 4.772734...|
    |    101|[{142811, 4.86805...|
    |    103|[{87426, 5.031621...|
    |    108|[{86709, 5.410115...|
    |    115|[{67565, 5.594385...|
    |    126|[{87426, 4.573233...|
    |    133|[{32202, 4.98517}...|
    |    137|[{148502, 4.90414...|
    +-------+--------------------+
    only showing top 20 rows
    
    


```python
from pyspark.sql.types import IntegerType
# user_list를 통해 예측
user_list = [65, 78, 81]
users_df = spark.createDataFrame(user_list, IntegerType()).toDF("userId")

users_df.show()
```

    +------+
    |userId|
    +------+
    |    65|
    |    78|
    |    81|
    +------+
    
    


```python
# recommendForUserSubset을 이용해 데이터 프레임으로 예측
user_recs = model.recommendForUserSubset(users_df, 5)
user_recs.show()
```

    +------+--------------------+
    |userid|     recommendations|
    +------+--------------------+
    |    65|[{194434, 6.85464...|
    |    78|[{200930, 7.14571...|
    |    81|[{207824, 5.16132...|
    +------+--------------------+
    
    

특정 userid 사용자를 위한 추천 영화 목록 만들기


```python
movies_list = user_recs.collect()[0].recommendations
movies_list
```




    [Row(movieId=194434, rating=6.854640007019043),
     Row(movieId=203086, rating=6.829344272613525),
     Row(movieId=205277, rating=6.7727460861206055),
     Row(movieId=98221, rating=6.582407474517822),
     Row(movieId=167106, rating=6.499454498291016)]




```python
recs_df = spark.createDataFrame(movies_list)
recs_df.show()
```

    +-------+------------------+
    |movieId|            rating|
    +-------+------------------+
    | 194434| 6.854640007019043|
    | 203086| 6.829344272613525|
    | 205277|6.7727460861206055|
    |  98221| 6.582407474517822|
    | 167106| 6.499454498291016|
    +-------+------------------+
    
    


```python
# 영화 Id가 아닌 이름 추천
movies_file = "movies.csv"
```


```python
# movieId에 따른 title 확인
movies_df = spark.read.csv(f"file:///{directory}\\{movies_file}", inferSchema=True, header=True)
movies_df.show()
```

    +-------+--------------------+--------------------+
    |movieId|               title|              genres|
    +-------+--------------------+--------------------+
    |      1|    Toy Story (1995)|Adventure|Animati...|
    |      2|      Jumanji (1995)|Adventure|Childre...|
    |      3|Grumpier Old Men ...|      Comedy|Romance|
    |      4|Waiting to Exhale...|Comedy|Drama|Romance|
    |      5|Father of the Bri...|              Comedy|
    |      6|         Heat (1995)|Action|Crime|Thri...|
    |      7|      Sabrina (1995)|      Comedy|Romance|
    |      8| Tom and Huck (1995)|  Adventure|Children|
    |      9| Sudden Death (1995)|              Action|
    |     10|    GoldenEye (1995)|Action|Adventure|...|
    |     11|American Presiden...|Comedy|Drama|Romance|
    |     12|Dracula: Dead and...|       Comedy|Horror|
    |     13|        Balto (1995)|Adventure|Animati...|
    |     14|        Nixon (1995)|               Drama|
    |     15|Cutthroat Island ...|Action|Adventure|...|
    |     16|       Casino (1995)|         Crime|Drama|
    |     17|Sense and Sensibi...|       Drama|Romance|
    |     18|   Four Rooms (1995)|              Comedy|
    |     19|Ace Ventura: When...|              Comedy|
    |     20|  Money Train (1995)|Action|Comedy|Cri...|
    +-------+--------------------+--------------------+
    only showing top 20 rows
    
    


```python
# spark SQL을 사용하기 위해 TempView 등록
recs_df.createOrReplaceTempView("recommendations")
movies_df.createOrReplaceTempView("movies")
```


```python
# 추천 영화와 추천 영화 제목, 장르 데이터프레임과 추천 완료한 테이블 join
query = """
    SELECT *
    
    FROM movies 
    JOIN recommendations ON movies.movieId = recommendations.movieId
    
    ORDER BY rating desc
"""

recommended_movies = spark.sql(query)
recommended_movies.show()
```

    +-------+--------------------+--------------------+-------+------------------+
    |movieId|               title|              genres|movieId|            rating|
    +-------+--------------------+--------------------+-------+------------------+
    | 194434|   Adrenaline (1990)|  (no genres listed)| 194434| 6.854640007019043|
    | 203086|Truth and Justice...|               Drama| 203086| 6.829344272613525|
    | 205277|   Inside Out (1991)|Comedy|Drama|Romance| 205277|6.7727460861206055|
    |  98221|Year One, The (L'...|              Comedy|  98221| 6.582407474517822|
    | 167106|Breaking a Monste...|         Documentary| 167106| 6.499454498291016|
    +-------+--------------------+--------------------+-------+------------------+
    
    


```python
# user_id: 찾고자 하는 user의 id
# num_recs: 추천 받을 영화의 개수
def get_recommendations(user_id, num_recs):
    user_df = spark.createDataFrame([user_id], IntegerType()).toDF("userId")
    user_recs_df = model.recommendForUserSubset(user_df, num_recs)
    
    recs_list = user_recs_df.collect()[0].recommendations
    recs_df = spark.createDataFrame(recs_list)
    
    recommended_movies = recs_df.join(movies_df, "movieId")
    return recommended_movies
```


```python
recs = get_recommendations(456, 10)
recs.toPandas()
```

    C:\Users\sonjj\anaconda3\lib\site-packages\pyspark\sql\context.py:125: FutureWarning: Deprecated in 3.0.0. Use SparkSession.builder.getOrCreate() instead.
      warnings.warn(
    




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
      <th>movieId</th>
      <th>rating</th>
      <th>title</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>203086</td>
      <td>6.883613</td>
      <td>Truth and Justice (2019)</td>
      <td>Drama</td>
    </tr>
    <tr>
      <th>1</th>
      <td>194434</td>
      <td>6.769002</td>
      <td>Adrenaline (1990)</td>
      <td>(no genres listed)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>77344</td>
      <td>6.560772</td>
      <td>Chizuko's Younger Sister (Futari) (1991)</td>
      <td>Drama</td>
    </tr>
    <tr>
      <th>3</th>
      <td>203882</td>
      <td>6.446171</td>
      <td>Dead in the Water (2006)</td>
      <td>Horror</td>
    </tr>
    <tr>
      <th>4</th>
      <td>199187</td>
      <td>6.396005</td>
      <td>Hoaxed (2019)</td>
      <td>(no genres listed)</td>
    </tr>
    <tr>
      <th>5</th>
      <td>183475</td>
      <td>6.263090</td>
      <td>Abe &amp; Phil's Last Poker Game (2018)</td>
      <td>Comedy|Drama</td>
    </tr>
    <tr>
      <th>6</th>
      <td>144202</td>
      <td>6.229063</td>
      <td>Catch That Girl (2002)</td>
      <td>Action|Children</td>
    </tr>
    <tr>
      <th>7</th>
      <td>157791</td>
      <td>6.191031</td>
      <td>.hack Liminality In the Case of Kyoko Tohno</td>
      <td>(no genres listed)</td>
    </tr>
    <tr>
      <th>8</th>
      <td>157789</td>
      <td>6.191031</td>
      <td>.hack Liminality In the Case of Yuki Aihara</td>
      <td>(no genres listed)</td>
    </tr>
    <tr>
      <th>9</th>
      <td>200930</td>
      <td>6.180189</td>
      <td>C'est quoi la vie? (1999)</td>
      <td>Drama</td>
    </tr>
  </tbody>
</table>
</div>




```python
spark.stop()
```
