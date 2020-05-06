# 3 대의 Ubuntu를 통해 Hadoop의 분산 저장 시스템 확인

## 1. Ubuntu 이미지 복제
- Virtual box 프로그램에서 1개 있는 ubuntu 이미지를 복제한다.

![](http://drive.google.com/uc?id=1r6DxTZobx88ySn9vAsSHtYMUG7I7eu8a)
- 이미지를 종료한 후 > 우클릭 > 복제 > 이름 변경 후 > 완전한 복제
- 이미지가 실행 중에는 복제가 안된다.

<br>

- 위의 이미지처럼 3개 만들어주도록 하자, 메모리가 부족하다면 2대로도 실습 가능
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
# u1, u2, u3 
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
# u1, u2, u3 => 상단에 추가(우선순위)
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
        <value>hdfs://192.168.0.[u1_IP]:9000</value>
    </property>
</configuration>

~~~
- hadoop의 코어는 u1 쪽으로 붙어야 한다.

#### 2.5.2. nano ~/hadoop-3.1.3/etc/hadoop/hdfs-site.xml

~~~
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/[user_name]/hadoop-3.1.3/data/nameNode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/[user_name]/hadoop-3.1.3/data/dataNode</value>
    </property>

    <property>
        <name>dfs.replication</name>
        <value>3</value> <!-- 데이터 노드 개수만큼 -->
    </property>
</configuration>
~~~

#### 2.5.3 nano ~/hadoop-3.1.3/etc/hadoop/mapred-site.xml

~~~
<configuration>
    <property>
        <name>mapreduce.jobtracker.address</name>
        <value>192.168.0.[u1_IP]:54311</value><!-- 첫번째 PC로 지정 -->
    </property>

    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
~~~

#### 2.5.4 nano ~/hadoop-3.1.3/etc/hadoop/yarn-site.xml

~~~
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>

    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>192.168.0.[u1_IP]</value><!-- 첫번째 PC로 지정 -->
    </property>
</configuration>
~~~

### 2.6 ssh 인증없이 연결
- ubuntu1의 것을 복제했기 때문에 이 부분은 확인만 하고 패스해도 된다.
- 만약에, 새로했다면 진행 코드는 아래를 참고하면 된다.

#### 2.6.1 u1에서 인증키 생성

~~~
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
~~~

#### 2.6.2 u2, u3에 같은 키 복사 

~~~
# u2, u3 -> .ssh 폴더를 생성
mkdir ~/.ssh

# u1 -> u1의 인증키를 u2, u3에 복사
scp ~/.ssh/authorized_keys 192.168.0.[u2]:/home/user1/.ssh/authorized_keys
scp ~/.ssh/authorized_keys 192.168.0.[u3]:/home/user1/.ssh/authorized_keys
~~~

#### 2.6.3 확인
- u1에서 u2, u3에 접속이 되는지 확인

~~~
ssh 192.168.0.[u2]
ifconfig # 현재 접속중인 ip주소 확인
exit
ssh 192.168.0.[u3]
ifconfig # 현재 접속중인 ip주소 확인
exit
~~~

## 3. u1에서 하둡 세팅
- 이제 u1에서만 컨트롤해주면 된다.
- 단, 실행 시 u2, u3가 모두 켜져 있어야 한다.

### 3.1. mapreduce를 위한 포트 개방

~~~
sudo ufw allow 54311
~~~

### 3.2. master & worker ip설정

~~~
nano ~/hadoop-3.1.3/etc/hadoop/masters
192.168.0.[u1]

nano ~/hadoop-3.1.3/etc/hadoop/workers
192.168.0.[u1]
192.168.0.[u2]
192.168.0.[u3]
~~~

## 4. namenode 포맷
- 현재 1대로 하둡 실습한 것을 복제했기 때문에 추가 작업이 필요하다.
- 생성된 dataNode, nameNode 폴더를 삭제 후 재생성. ( 실습한 데이터도 날아간다. )

### 4.0. (이번 경우만) dataNode, nameNode 폴더를 삭제 후 재생성

~~~
# u1, u2, u3 모두 
# 하위 폴더까지 다 지우고
rm -rf ~/hadoop-3.1.3/data/nameNode
rm -rf ~/hadoop-3.1.3/data/dataNode
# 폴더 생성
mkdir -p ~/hadoop-3.1.3/data/nameNode
mkdir -p ~/hadoop-3.1.3/data/dataNode
~~~

### 4.1 포맷

~~~
# u1에서만
hdfs namenode -format
~~~

## 5. 하둡 실행!
- 이제 진짜 u1에서만 실행하면 된다.

~~~
start-all.sh
# 자동으로 u2, u3의 node가 실행된다.
~~~

### 5.1. 확인
- jps 확인
    - u1
    ~~~
    29555 Jps
    28804 SecondaryNameNode
    29205 NodeManager
    29031 ResourceManager
    28362 NameNode
    28540 DataNode
    ~~~
    - u2
    ~~~
    3394 NodeManager
    3230 DataNode
    3519 Jps
    ~~~
    - u3
    ~~~
    3161 DataNode
    3437 Jps
    3325 NodeManager
    ~~~

- chrome에서 확인
    - 192.168.0.[u1]:9870 접속 > Datanodes > 3개 확인
    ![](http://drive.google.com/uc?id=1EFD8YxjdU69BQz61oLeZrDyPYrjQMXkx)

### 5.2. 데이터 분산 확인
- 외부에서 데이터를 임포트할 수 있게 권한을 변경한다

~~~
hdfs dfs -chmod 777 /
~~~

- Utilities > Browse the file system > 데이터 임포트 버튼으로 삽입
- 아래처럼 Replication이 3개면 정상작동
![](http://drive.google.com/uc?id=1dHskFSFYGxoYTDTMViMGdS5u5KF06_H9)

- 다시 Datanodes로 와서 표에서 **Blocks** 확인
- 3 node 모두 같은 숫자이면 정상작동, 한 파일을 나눠서 가지고 있는 것같다.
![](http://drive.google.com/uc?id=105410kWUyThpWmBYEWVULGjSd_OgI6la)

- 위에서 밑줄친 링크를 누르면 각각 datanode를 확인할 수 있다.
- datanode 각각은 **9866** 포트를 사용한다.
![](http://drive.google.com/uc?id=1oEMg4oxUL5hcqqKR22SP63Dc7HX91gHc)



