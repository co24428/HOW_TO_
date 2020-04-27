# kafka를 통해 spark를 구동
- spark를 사용할 것이기 때문에 모든 start 코드를 실행한다.
- 주피터 노트북을 사용  
  => 다른 터미널에서 구동

~~~
start-all.sh
start-master.sh
start-slave.sh spark://127.0.0.1:7077
zookeeper-server-start.sh ~/kafka_2.11-2.2.0/config/zookeeper.properties &
kafka-server-start.sh ~/kafka_2.11-2.2.0/config/server.properties &
~~~

## 1. kafka library install (.jar)
- kafka 또한 java를 기반으로 하기 때문에
- java 라이브러리를 python 코드를 경로를 잡아주어야 한다.
- 특정 폴더에 넣는 것이 정리하는 데 좋다. (여기서는 그냥 진행)
~~~
wget https://repo1.maven.org/maven2/org/apache/spark/spark-sql-kafka-0-10_2.11/2.4.5/spark-sql-kafka-0-10_2.11-2.4.5.jar
wget https://repo1.maven.org/maven2/org/apache/kafka/kafka-clients/0.11.0.0/kafka-clients-0.11.0.0.jar
~~~

## 2. python code (ubuntu, jupyter notebook)

### 2.1. import library

~~~
from pyspark.sql import SparkSession
import pandas as pd
import os
~~~

### 2.2. python path setting

~~~
os.environ['PYSPARK_PYTHON']='/home/user1/anaconda3/bin/python3'
os.environ['PYSPARK_DRIVER_PYTHON']='/home/user1/anaconda3/bin/python3'
~~~

### 2.3. spark setting

~~~
spark = SparkSession.builder.master("local[*]") \
    .appName("exam01") \
    .config('spark.driver.extraClassPath','/home/[user_name]/spark-sql-kafka-0-10_2.11-2.4.5.jar' ) \
    .config('spark.driver.extraClassPath','/home/[user_name]/kafka-clients-0.11.0.0.jar' ) \
    .config('spark.jars.packages','org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.5' ) \
    .getOrCreate()
~~~
- 이전과 같이 메모리는 local의 모든 것을 끌어 사용한다.
- 아까 받은 jar 파일을 연결시켜준다.

### 2.4. kafka setting

~~~
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers","192.168.0.xxx:9092") \
    .option("subscribe", "testTopic2") \
    .option("startingOffsets", "latest") \
    .load()
~~~
- 본인 IP를 잡아준다.
- subcribe : 연결한 토픽을 설정
- startingOffsets : latest or earliest

### 2.5. DataFrame 생성

~~~
df1 = df.selectExpr("CAST (key AS STRING)", "CAST(value AS STRING)")
~~~
- df의 값을 변경해서 df1에 보관

### 2.6. 실시간 입력을 df에 저장

~~~
df1.writeStream.outputMode("append") \
    .format("console") \
    .option("truncate", "false") \
    .trigger(processingTime="10 seconds") \
    .start() \
    .awaitTermination() 
~~~
- df1의 실시간 값을 console에 출력함
- .writeStream.outputMode("append")
    - 어떤 동작을 할지 -> append
- .awaitTermination()
    - 안 끝나도록, 무한 반복
- .trigger(processingTime="10 seconds")
    - n seconds 동안 대기 후 한꺼번에 동작, 여러 데이터를 한번에 받기 위함.
    - 이 옵션을 없애면 바로 한 개의 데이터를 담은 df 보여준다.

## 3. output

- trigger 미사용

![](http://drive.google.com/uc?id=1cCKPlfZgXZZSPiqL_FCGGlCmjd5LeeaA)


- trigger 사용

![](http://drive.google.com/uc?id=1tmTrCpJC0RrBB5U9a22q4r4qvHc-qYYW)

- vscode에서든 mobaxterm 터미널에서든 Producer가 있다면 계속해서 받을 수 있다.
- append 대신 다른 명령을 주면 spark의 다른 sql도 사용가능할 것으로 생각된다.
