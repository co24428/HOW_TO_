# 스파크 구동

- Hadoop 구동 : start-all.sh

## 1. 스칼라 및 스파크 설치

- 라이브러리 다운로드

~~~
# 스칼라 -> 기본적으로 스파크를 사용하기 위해서 필요하다.
sudo apt install scala -y

# 스파크
pip install pyspark
    OR
conda install -c conda-forge pyspark
~~~

- 환경을 위한 파일 다운로드

~~~
# 버전은 이전 경로에서 선택하도록!
# 상당한 시간이 걸린다.
wget http://apache.tt.co.kr/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz

# 압축해제
tar -xzvf spark-2.4.5-bin-hadoop2.7.tgz
~~~

## 2. 스파크 환경설정

- 환경변수 설정

~~~
# 환경변수 폴더 진입
nano ~/.bashrc

# 경로 추가
export SPARK_HOME=/home/user1/spark-2.4.5-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

# 적용
source ~/.bashrc
~~~

- 설정파일 생성
    - 압축해제한 폴더 내의 설정파일 template를 복사해주어야 한다.
    - 잘 못 설정할 경우 바로 고칠 수 있게 이렇게 해놓은 것으로 보인다.  
      또한 필요한 설정만 잡을 수 있어 편리하다.
    ~~~
    # 내부의 경로로 이동
    cd ~/spark-2.4.5-bin-hadoop2.7/conf
    # template 파일 복사
    cp spark-env.sh.template spark-env.sh
    cp spark-defaults.conf.template spark-defaults.conf
    ~~~

- 설정파일 수정
    - ***nano spark-env.sh***
    ~~~
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    export HADOOP_HOME=/home/user1/hadoop-3.1.3
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native
    export SPARK_LOCAL_IP="192.168.0.XXX"  # => local ip
    export PYSPARK_PYTHON=/home/user1/anaconda3/bin/python3
    export PYSPARK_DRIVER_PYTHON=/home/user1/anaconda3/bin/python3
    export SPARK_MASTER_HOST="192.168.0.XXX"  # => 원격 접속ip
    export SPARK_WORKER_INSTANCES=1
    export SPARK_WORKER_MEMORY=4096m
    export SPARK_WORKER_CORES=8
    export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=5"
    ~~~
    
    - ***nano spark-defaults.conf***
    ~~~
    spark.master                spark://127.0.0.1:7077
    spark.executor.instances    1
    spark.executor.cores        3
    spark.executor.memory       4g
    spark.driver.cores          1
    spark.driver.memory         4g
    ~~~

- 스파크를 위한 방화벽 포트(7077) 개방

~~~
sudo ufw allow 7077
~~~

## 3. 프로세스 구동
- Spark master & slave 구동

~~~
start-master.sh
start-slave.sh spark://127.0.0.1:7077
# 프로세스 중지 시
stop-master.sh
stpp-slave.sh

# 확인
jps
3009 SecondaryNameNode
4262 Master            => 추가 됨.
2727 DataNode
4314 Worker            => 추가 됨.
3434 NodeManager
2538 NameNode
3243 ResourceManager
4349 Jps
~~~

# 4. 스파크 실습

- jupyter notebook 실행
    - mobaxterm에서 한 IP에 여러 터미널로 접속할 수 있다.
    - jupyter notebook 실행을 위한 터미널을 만들어주도록 하자.
    
~~~
jupyter notebook

# 크롬에서 진행
http://192.168.0.XXX:8888
~~~

### example code 1
- Spark 단순 구동

~~~
from pyspark.sql import SparkSession
import pandas as pd

# 파이썬 설치 위치 지정
import os
os.environ['PYSPARK_PYTHON']='/home/user1/anaconda3/bin/python3'
os.environ['PYSPARK_DRIVER_PYTHON']='/home/user1/anaconda3/bin/python3'

# 스파크 객체 생성 
# virtual box 성능이 약해서 local을 끌어 사용했다.
# spark = SparkSession.builder.master("spark://192.168.0.xxx:7077") \
spark = SparkSession.builder.master("local[*]") \
    .enableHiveSupport().appName("hive01") \
    .getOrCreate()

# 데이터 프레임 생성
df1 = spark.createDataFrame([(1,'a',10),(2,'b',20),(3,'c',30)]).toDF("id","name","age")
df1.printSchema()
df1.show()
~~~
- Hive => 하둡과 스파크를 이어주는 플랫폼 역할
- Pandas => 데이터프레임을 사용하기 위해서 import  
  다음 예시에서 쓸 예정

- Outputs
    - df1.printSchema() output
    - 데이터베이스의 Schema
    ~~~
    root
     |-- id: long (nullable = true)
     |-- name: string (nullable = true)
     |-- age: long (nullable = true)
    ~~~

    - df1.show() output
    ~~~
    +---+----+---+
    | id|name|age|
    +---+----+---+
    |  1|   a| 10|
    |  2|   b| 20|
    |  3|   c| 30|
    +---+----+---+
    ~~~

### example code 2
- Spark 객체를 Hadoop에 적재 및 수정

~~~
[example code 1]
...
#스파크 객체 생성 
spark = SparkSession.builder.master("local[*]") \
    .enableHiveSupport().appName("hive01") \
    .config("spark.sql.warehouse.dir","/user/hive/warehouse") \
    .config("spark.datasource.hive.metastore.uris","hdfs://192.168.0.19:9000") \
    .getOrCreate()
# config("spark.sql.warehouse.dir","/user/hive/warehouse")
# => spark data를 저장할 Hadoop 경로
...
# df1을 table1으로 설정, DB에서의 "Table"
df1.createOrReplaceTempView("table1")
# 간단한 SQL문
spark.sql("SELECT id, name FROM table1").show()

# 데이터프레임을 pandas로 변경
pd1 = df1.select("*").toPandas()
print(type(pd1))

# 하둡 : DB 생성
spark.sql("create database db01")

# 데이터 프레임으로 테이블 생성
spark.sql("create table db01.t01 as select * from table1")

# 데이터 추가하기
spark.sql("insert into db01.t01 values(4,'a1',40)")

# 테이블 내용 가져오기
spark.sql("select * from db01.t01").show()

# DELETE / UPDATE는 hadoop에서 지원하지 않음.
~~~

- Outputs
    
    ~~~
    # 데이터프레임을 pandas로 변경
    pd1 = df1.select("*").toPandas()
    print(type(pd1))
    
    -> <class 'pandas.core.frame.DataFrame'>
    
    ~~~
    
    ~~~
    # 하둡 : DB 생성
    spark.sql("create database db01") 
    
    -> DataFrame[]
    -> 9987 port(chrome)에서 utility에서 /user/hive/warehouse/db01.db 확인
    ~~~
    
    ~~~
    # 데이터 프레임으로 테이블 생성
    spark.sql("create table db01.t01 as select * from table1")

    -> DataFrame[]
    -> 9987 port(chrome)에서 utility에서 /user/hive/warehouse/db01.db/t01 확인
    -> 파일이 하나 있을 것이다.
    ~~~
    
    ~~~
    # 데이터 추가하기
    spark.sql("insert into db01.t01 values(4,'a1',40)")
    
    -> DataFrame[]
    -> 9987 port(chrome)에서 utility에서 /user/hive/warehouse/db01.db/t01 확인
    -> 새로운 파일이 하나 생성 됨.
    -> 추가 시마다 각각의 파일로 저장되는 것같다.
    ~~~
    
    ~~~
    # 테이블 내용 가져오기
    spark.sql("select * from db01.t01").show()
    
    -> 기존 저장한 것에 추가된 것 확인 가능
    -> 파일이 분산되어 있어도 하나의 테이블로 인식한다.
    +---+----+---+
    | id|name|age|
    +---+----+---+
    |  1|   a| 10|
    |  2|   b| 20|
    |  3|   c| 30|
    |  4|  a1| 40|
    +---+----+---+
    
    ~~~

- 최종 example 결과 hdfs
![](http://drive.google.com/uc?id=1v16TyEDK72MtkMLCbJXlAfPQIjt8VSF4)

### !! 잘 끄고 마무리하자

~~~
stop-all.sh
stop-master.sh
stpp-slave.sh
# 확인
jps
~~~