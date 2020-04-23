# kafka 구동

## 1. 카프카 설치 및 설정

~~~
# 오래 걸린다.
wget https://archive.apache.org/dist/kafka/2.2.0/kafka_2.11-2.2.0.tgz

tar -zxvf kafka_2.11-2.2.0.tgz => 압축풀기

nano ~/.bashrc  
# 추가
export KAFKA_HOME=/home/user1/kafka_2.11-2.2.0
export PATH=$PATH:$KAFKA_HOME/bin
# 적용
source ~/.bashrc
# 방화벽 포트 개방
sudo ufw allow 2181  => zookeeper
sudo ufw allow 9092  => kafka
~~~

## 2. zookeeper 서버 구동
- kafka 기본적으로 데이터베이스를 가지고 있지 않는다.
- 이를 보조하기 위한 베이스가 zookeeper
- 즉, kafka를 실행하기 전에 zookeeper를 먼저 실행해야 한다.

<br>

- 설정 확인 (대개 적용되어있지만 확인)

~~~
nano ~/kafka_2.11-2.2.0/config/zookeeper.properties

dataDir=/tmp/zookeeper
clientPort=2181
maxClientCnxns=0
~~~

- zookeeper 서버 구동

~~~
# "&"를 붙여주면 백그라운드에서 서버를 구동한다.
zookeeper-server-start.sh ~/kafka_2.11-2.2.0/config/zookeeper.properties &
< 종료 >
zookeeper-server-stop.sh
# 서버 확인
jps
-> 16494 QuorumPeerMain
~~~

## 3. kafka 서버 구동
- zookeeper를 실행했으니 이제 kafka를 실행시켜보자

~~~
nano ~/kafka_2.11-2.2.0/config/server.properties

31 라인 : listeners=PLAINTEXT://0.0.0.0:9092
36 라인 : advertised.listeners=PLAINTEXT://192.168.0.xxx:9092
123 라인 : zookeeper.connect=127.0.0.1:2181
가장마지막라인에 추가 : delete.topic.enable=true
~~~

- kafka 서버 구동

~~~
# "&"를 붙여주면 백그라운드에서 서버를 구동한다.
kafka-server-start.sh ~/kafka_2.11-2.2.0/config/server.properties &
< 종료 >
kafka-server-stop.sh
# 서버 확인
jps
-> 1234 Kafka
~~~

## 4. kafka 동작 확인

- 토픽( 브로커 ) 생성  
  => testTopic2 => 채널

~~~
# 토픽을 만들어서 2181포트(zookeeper)에 연결? 저장?
kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic [Topic name]

# 토픽 확인
kafka-topics.sh --list --zookeeper 127.0.0.1:2181
# 가운데 잘 보면 [Topic name] 있을 것이다.
~~~

- Producer & Consumer
    - Producer : data를 주는 곳
    - Consumer : data를 받는 곳
    - 이들은 9092포트(kafka)를 사용한다

~~~
# 각각 다른 터미널에 해주어야 한다 !
Producer생성
kafka-console-producer.sh --broker-list 192.168.0.xxx:9092 --topic testTopic2

Consumer생성
kafka-console-consumer.sh --bootstrap-server 192.168.0.xxx:9092 --topic testTopic2 --from-beginning
~~~

![](http://drive.google.com/uc?id=12P4IObW9PKMqrpY8-IQvwTvSMRo63xCs)

- 좌측이 Consumer / 우측이 Producer
- Producer에서 입력을 하면 Consumer에서 출력이 된다.