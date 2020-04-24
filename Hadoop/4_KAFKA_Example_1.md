# kafka 통신으로 로컬컴퓨터와 통신
- vscode에서 진행.
- 기본적인 kafka는 하둡이나 스파크가 선행되지 않아도 된다.

## 1. kafka 설치(로컬)

~~~
pip install kafka-python
~~~

## 2. python code

### 2.1. import Library

~~~
import time, threading, multiprocessing
from kafka import KafkaConsumer, KafkaProducer
~~~
- thread
    - 한번에 Consumer, Producer 등을 실행시키려면 한개의 프로세스로는 불가능
    - 한 프로세스에서 여러 thread가 백그라운드에 돌게 사용
    
### 2.2. Consumer class

~~~
class Consumer(multiprocessing.Process):
    def __init__(self):
        multiprocessing.Process.__init__(self)
        self.stop_event = multiprocessing.Event()

    def stop(self):
        self.stop_event.set()

    def run(self):
        consumer = KafkaConsumer(bootstrap_servers='192.168.0.xxx', # 본인 IP주소
            auto_offset_reset='latest', consumer_timeout_ms=1000)
        consumer.subscribe(['testTopic2'])    
        while not self.stop_event.is_set():
            for msg in consumer:
                # str = msg.value # raw data로 받아오기
                str = (msg.value).decode('utf-8') # 한글 인코딩
                print(str)

                if self.stop_event.is_set():
                    break
        consumer.close()
~~~
- KafkaConsumer() 함수를 통해서 서버를 지정해준다.
    - auto_offset_reset
        - latest(마지막), 입력하는 것만 출력, 일반적인 선택
        - earliest(처음부터), 이전에 받았던 것까지 모두 남는다.
- 어떤 토픽에 연결할지 선택 (subcribe)
- utf-8 인코딩을 해주지 않으면 b'' 처럼 byte text로 오게 된다.

### 2.3. Consumer class

~~~
class Producer(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)
        self.stop_event = threading.Event()

    def stop(self):
        self.stop_event.set()

    def run(self): 
        producer = KafkaProducer(bootstrap_servers='192.168.0.xxx:9092') # 본인 IP주소
        while not self.stop_event.is_set():
            str = input('send msg : ')
            producer.send('testTopic2', str.encode()) #string to byte
            # time.sleep(3)
            
        producer.close() 
~~~
- Producer의 경우 어떤 포트에 연결할지 지정해야 한다. (9092)
- 데이터를 보낼 때는 byte text로 인코딩해서 보내주어야 한다.

### 2.4. main method

~~~
def main():
    tasks = [Consumer(), Producer()]
    # tasks = [Producer()]
    # tasks = [Consumer()]
    
    for tmp in tasks:
        tmp.start()
    time.sleep(1000)    

    for tmp in tasks:
        tmp.stop()

    for tmp in tasks:
        tmp.join()    
~~~
- 위에서 만든 클래스를 List형태로 선언하여 사용할 것이다.
- 여러 개 있을 경우 안전하게 하나씩 접속하여 생성하게 된다. ( time.sleep )


### 2.5. run code

~~~
if __name__ == '__main__':
    main()        
~~~
- 보안(?)을 위해서 코드를 외부 코드에서 import가 불가능하게 설정
- 외부에서 들어올 경우 __name__이 __main__이 아니라, 파일명으로 되어 실행이 안된다.


## 3. output

- Consumer()만 사용

![](http://drive.google.com/uc?id=1yuQjEeNfbm6x7PKZMriB8IJaPivxc7Kj)


- Consumer(), Producer() 사용
    - 시간 차를 주기 위해 Producer class에 time.sleep(1) 추가

![](http://drive.google.com/uc?id=16cbOjPLfPXsLYY8HHqxRnXYHQ55lXiDh)

- 한글 인코딩을 걸어두었기에 한글도 전송이 잘 된다.
