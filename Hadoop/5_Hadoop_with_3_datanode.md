# 3 대의 Ubuntu를 통해 Hadoop의 분산 저장 시스템 확인

## 1. Ubuntu 이미지 복제
- Virtual box 프로그램에서 1개 있는 ubuntu 이미지를 복제한다.

![](http://drive.google.com/uc?id=1r6DxTZobx88ySn9vAsSHtYMUG7I7eu8a)
- 이미지를 종료한 후 > 우클릭 > 복제 > 이름 변경 후 > 완전한 복제
- 이미지가 실행 중에는 복제가 안된다.

<br>

- 위의 이미지처럼 3개 만들어주도록 하자, 메모리가 부족하다면 2대로도 실습은 가능
- 여기서 하고자 하는 것은 파일이 분산되는 모습을 보고자 하는 것

## 2. 복제한 이미지 세팅
- 이제 총 3대의 우분투를 순서대로 u1, u2, u3로 지칭하겠다.
### 2.1. hostname 변경

~~~
# u1
sudo hostnamectl set-hostname hadoop1
# u2
sudo hostnamectl set-hostname hadoop2
# u3
sudo hostnamectl set-hostname hadoop3
# 재부팅해주어야 적용된다.
sudo reboot
~~~
- hostname이 도메인에 사용되기 때문에 위처럼 설정해주도록 하자.

### 2.2. ip주소 확인

~~~
ifconfig
~~~
- 이는 복제된 u2, u3에서 확인한다.

### 2.3. host 추가

~~~
# u1, u2, u3
192.168.0.[u1]  (탭)   hadoop1
192.168.0.[u2]  (탭)   hadoop2
192.168.0.[u3]  (탭)   hadoop3
# 편집 적용
sudo service networking restart
~~~

### 2.4. 환경변수 추가

~~~
# u1, u2, u3
nano ~/.bashrc
# 추가
export HADOOP_HOME=/home/user1/hadoop-3.1.3
export HADOOP_COMMON_HOME=/home/user1/hadoop-3.1.3
export HADOOP_MAPRED_HOME=${HADOOP_HOME}    => 추가
export HADOOP_HDFS_HOME=${HADOOP_HOME}      => 추가
export YARN_HOME=${HADOOP_HOME}             => 추가
export HDFS_NAMENODE_USER="user1"
export HDFS_DATANODE_USER="user1"
export HDFS_SECONDARYNAMENODE_USER="user1"
export YARN_RESOURCEMANAGER_USER="user1"
export YARN_NODEMANAGER_USER="user1"
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin
# 환경변수 적용
source ~/.bashrc
~~~

### 2.5. hadoop 설정
- u1, u2, u3 모두 해주면 된다.

#### 2.5.1. nano ~/hadoop-3.1.3/etc/hadoop/core-site.xml

~~~
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.0.1:9000</value>
    </property>
</configuration>

~~~
- hadoop의 코어는 u1 쪽으로 붙어야 한다.

#### 2.5.2. nano ~/hadoop-3.1.3/etc/hadoop/hdfs-site.xml

~~~
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/user1/hadoop-3.1.3/data/nameNode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/user1/hadoop-3.1.3/data/dataNode</value>
    </property>

    <property>
        <name>dfs.replication</name>
        <value>3</value> <!-- 데이터 노드 개수만큼 -->
    </property>
</configuration>
~~~


<hr>

- output

~~~
print(g.vertices.count())
print(g.edges.count())

7
12


g.degrees.show()  

+---+------+
| id|degree|
+---+------+
|  3|     7|
| 98|     2|
| 99|     2|
|  5|     3|
|  1|     4|
|  4|     3|
|  2|     3|
+---+------+


g.bfs(fromExpr="id='1'", toExpr="id='4'", maxPathLength=30).show(truncate=False)

+------------------------+--------------+--------------------+--------------+---------------------+
|from                    |e0            |v1                  |e1            |to                   |
+------------------------+--------------+--------------------+--------------+---------------------+
|[1, Carter, Derrick, 50]|[1, 3, friend]|[3, Mills, Jeff, 80]|[3, 4, friend]|[4, Hood, Robert, 65]|
+------------------------+--------------+--------------------+--------------+---------------------+
~~~




