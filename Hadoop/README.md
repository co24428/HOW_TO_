# Server start

~~~
# Hadoop
start-all.sh
# Spark
start-master.sh
start-slave.sh spark://127.0.0.1:7077
# Kafka
zookeeper-server-start.sh ~/kafka_2.11-2.2.0/config/zookeeper.properties &
kafka-server-start.sh ~/kafka_2.11-2.2.0/config/server.properties &
~~~

# Server stop

~~~
# Hadoop
stop-all.sh
# Spark
stop-master.sh
stop-slave.sh
# Kafka
zookeeper-server-stop.sh
kafka-server-stop.sh
~~~

# Check

~~~
jps
~~~