# 하둡 설치
- mobaxterm에서 모든 것을 진행한다!


## 1. Java 설치

~~~
sudo apt install openjdk-8-jre-headless -y
sudo apt install openjdk-8-jdk-headless -y
# 설치 확인
java -version
~~~

### why?
- Hadoop은 기본적으로 자바를 기반으로 설치된다.
- 이를 위해 Java를 설치!

## 2. Hadoop 설치

- 파일 다운로드

~~~
# 다른 버전 확인
http://apache.tt.co.kr/hadoop/common/hadoop-3.1.3/
# wget을 통해 다운로드
wget http://apache.tt.co.kr/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz
# 압축해제
tar -zxvf hadoop-3.1.3.tar.gz
~~~

## 3. Hadoop 환경설정

- 텍스트 에디터는 nano를 사용한다.
- nano 사용법
    - 마우스 휠 사용가능
    - 기본적으로 키보드는 인풋모드
    - alt + shift + 3 : 좌측 코드라인 띄우기
    - 아래의 명령어
        - ^ : Ctrl + 해당키
    - Ctrl + s : save / Ctrl + x : exit

- 환경변수 설정

~~~
nano ~/.bashrc
~~~
~~~
export HADOOP_HOME=/home/[user_name]/hadoop-3.1.3
export HADOOP_COMMON_HOME=/home/[user_name]/hadoop-3.1.3
export HDFS_NAMENODE_USER="[user_name]"
export HDFS_DATANODE_USER="[user_name]"
export HDFS_SECONDARYNAMENODE_USER="[user_name]"
export YARN_RESOURCEMANAGER_USER="[user_name]"
export YARN_NODEMANAGER_USER="[user_name]"
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin
~~~
~~~
# 환경변수 적용
source ~/.bashrc
~~~

- 이하 Hadoop을 위한 환경설정
    - nano ~/hadoop-3.1.3/etc/hadoop/core-site.xml
    
    ~~~
    <configuration>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://192.168.0.XXX:9000</value>
        </property>
    </configuration>
    ~~~
    - nano ~/hadoop-3.1.3/etc/hadoop/hdfs-site.xml
    
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
            <value>1</value>
        </property>
    </configuration>
    ~~~
    - nano ~/hadoop-3.1.3/etc/hadoop/mapred-site.xml
    
    ~~~
    <configuration>
        <property>
            <name>map.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    ~~~
    - nano ~/hadoop-3.1.3/etc/hadoop/yarn-site.xml
    
    ~~~
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
    ~~~

- namenode & datanode 디렉토리 생성

~~~
mkdir -p ~/hadoop-3.1.3/data/nameNode
mkdir -p ~/hadoop-3.1.3/data/dataNode
~~~

## 4. Hadoop 동작 확인

- 인증키 등록
    - 외부에서 localhost 접속시 암호 확인 없애기 위함.

~~~
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa => 파일생성
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys => 파일내용복사
chmod 0600 ~/.ssh/authorized_keys  => 0600권한 변경
~~~

- localhost 확인

~~~
ssh localhost
~~~

- namenode 포맷(초기화)

~~~
hdfs namenode -format
# 에러가 난다면 확인 후 해당 폴더의 설정을 확인
# 똑같아보이더라도 새로 복사붙여넣기를 하면 되는 경우가 더러 있음..
~~~

- Hadoop 실행!

~~~
start-all.sh
# 기다리면 한 줄씩 실행될 것이다.

# 확인
jps
# 성공적인 output
7377 NameNode
8537 Jps
8042 ResourceManager
7580 DataNode
7822 SecondaryNameNode
8222 NodeManager
~~~

- Hadoop을 위한 방화벽 포트 개방

~~~
sudo ufw allow 9000
sudo ufw allow 9870
sudo ufw allow 9864
sudo ufw allow 9866
~~~

- 로컬 컴퓨터에서 Hadoop 확인
    - 192.168.0.XXX:9870 로 접속해보면 초록창이 나와야 한다!
    
    ![](http://drive.google.com/uc?id=1jthNwmOcMpepNPf9dO0J4Qw7SW1CRqJd)
    
    - Datanodes를 확인해보면 활성화된 것을 확인할 수 있다.
    
    ![](http://drive.google.com/uc?id=1Ne2ejQ76rBKcZWwTQVZWRZdchAG5qVUp)
    
    
### Hadoop 등 가상환경 시 중요한 점!!
- 실행한 것을 안끄고 강제로 꺼버리면 추후에 문제우려가 있다.
    - 문을 열었으면 잘 닫아주어야 한다.

~~~
stop-all.sh

# 확인
jps
# jps 한개만 나와야 한다.
6791 Jps
~~~