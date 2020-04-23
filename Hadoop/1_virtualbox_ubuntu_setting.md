# virtualbox의 ubuntu를 외부인 로컬에서 접속

## 0. ubuntu 18.04 base 기반
- https://ubuntu.com/download/desktop/thank-you?version=18.04.4&architecture=amd64#download
## 1. virtual box에 세팅
- 메모리 : 4096Mb
- 저장소 : 30Gb
- 네트워크 수정 (기본 설정은 NAT)
    - default setting
    ![](http://drive.google.com/uc?id=1Cg7cylPAbkdX-AvbcTyupeqlauIEZMEb)
    
    - change setting
    ![](http://drive.google.com/uc?id=1ydx0oIPrYIuZXfAMnVg0Bnnd8uK-L7_p)

- iso 파일 임포트
    - setting > 저장소 > 네모 부분 눌러서 iso 파일 선택
    ![](http://drive.google.com/uc?id=1p1mTOqGnBm6qGwb40tIlxlgqTAYlXY5M)
    
### 처음 만들어주는 계정이름을 추후에 사용하니 기억할 것!
## 2. 실행 후 세팅
- Ctrl + alt + t : 터미널 창 열기

- ip주소 확인

~~~
# 업데이트
sudo apt update -y
# 서버 주소 확인용 라이브러리
sudo apt install net-tools
# 서버 ip주소 확인
ifconfig
# 127.0.0.1 => 이 주소는 게이트웨이
# 192.168.0.xxx => 이 주소를 기록&기억
~~~

- 외부 연결위한 세팅

~~~
# ssh 프로그램 설치
sudo apt install ssh -y
# ssh 구동  
sudo service ssh start
# 방화벽 활성화
sudo ufw enable
# 방화벽 개방 22번 port - 외부 연결용도, 외부에서 이 리눅스를 컨트롤할 것이다.
sudo ufw allow 22
# 방화벽 확인
sudo ufw status
# hostname 확인
hostname
# hostname 변경 => 접속시 편하게 하기 위함, 위의 IP와 같게 해주도록
sudo hostnamectl set-hostname 192.168.0.xxx
# 재부팅!
sudo reboot

~~~

## 3. mobaxterm
- 외부의 ip주소에 접속하여 컨트롤하기 위함
- https://mobaxterm.mobatek.net/ => Free 버전으로 설치

- Session을 연결!
- Session > SSH > insert host & username
![](http://drive.google.com/uc?id=1YFZN2AJCe7yir1bLgAsX03GizJdiX0WB)

<hr>
![](http://drive.google.com/uc?id=1Rpylo-y4HPyovHyvpn3qTNI31pXy2zbW)
- username에 위의 이름을 입력


### 왜 하는가?
- 하둡을 쓴다는 가정 하에 현재 리눅스는 서버이다.
- 서버는 직접 사용하지 않고 외부에서 조정하는 것이 일반적이므로  
  mobaxterm을 통해 외부인 Ubuntu를 컨트롤한다.
  
## 4. 개발 환경설정 - python & jupyter

- Anaconda 설치
~~~
# wget [url] : url을 통해 파일을 다운로드
wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
# 설치한 파일을 실행
bash Anaconda3-2020.02-Linux-x86_64.sh
# 설치시 경로는 default로!
# 마지막 질문은 자동으로 환경변수로 잡을건지에 대한 질문
# yes입력!, 기본값이 no이기 때문에 조심

# no로 넘겼을 경우
rm -rf ~/anaconda3 # bash ~~~부터 다시 실행
# yes로 넘겼을 경우
source ~/.bashrc # 환경변수 변경 적용
# ~/.bashrc => 환경변수들이 담겨있는 파일
# 변경이 생기면 source 명령을 통해 적용을 시켜주어야 한다!
~~~

- jupyter notebook 설정

~~~
jupyter notebook --generate-config
# 에러 뜰 경우 pip로 설치

# 암호 생성
ipython
: from notebook.auth import passwd
: passwd()
Enter password: [password] # 알아서 설정(기억해야 함)
Verify password: [password]
Out[2]: 'sha1:xxx...' # 비밀번호 암호화(복사해둘 것)
exit() # python 모드 탈출
~~~
    
~~~
# 작업경로 생성
mkdir ~/jupyter-workspace

# 설정 파일 수정
nano ~/.jupyter/jupyter_notebook_config.py

    048 line: c.NotebookApp.allow_origin = '*'  # 외부 접속 허용하기
    204 line: c.NotebookApp.ip = '192.168.0.XXX'  # 본인 아이피 설정
    266 line: c.NotebookApp.notebook_dir = u'/home/[username]/jupyter-workspace' #작업경로 설정
    272 line: c.NotebookApp.open_browser = False # 시작 시 서버PC에서 주피터 노트북 창이 열릴 필요 없음
    281 line: c.NotebookApp.password = u'sha1로 시작하는 암호...' #비밀번호 설정
    292 line: c.NotebookApp.port = 8888   #포트 설정

# 설정 확인
jupyter notebook --config ~/.jupyter/jupyter_notebook_config.py
~~~

- jupyter 접속

~~~
jupyter notebook

[I 16:39:25.637 NotebookApp] JupyterLab extension loaded from /home/[username]/anaconda3/lib/python3.7/site-packages/jupyterlab
[I 16:39:25.637 NotebookApp] JupyterLab application directory is /home/[username]/anaconda3/share/jupyter/lab
[I 16:39:25.643 NotebookApp] Serving notebooks from local directory: /home/[username]/jupyter-workspace
[I 16:39:25.644 NotebookApp] The Jupyter Notebook is running at:
[I 16:39:25.644 NotebookApp] http://192.168.0.xxx:8888/
[I 16:39:25.645 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
~~~
- chrome에 http://192.168.0.xxx:8888/ 로 접속
    - 비밀번호는 위의 jupyter 설정에서 한 비밀번호