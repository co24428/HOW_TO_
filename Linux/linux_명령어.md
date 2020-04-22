- 관리자 모드 진입

~~~ 
sudo -i # password 입력

exit # 나갈때
~~~

- 현재 경로 확인

~~~
pwd
~~~

~~~
/home/user1
~~~

- 현재 경로에서 파일 확인

~~~
ls : 이름만
ls -a : 숨김파일까지
ls -l : 상세정보(권한, 용량, 생성일 등등)
ls -al : 위 두 옵션을 동시에
~~~

~~~
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 공개
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 다운로드
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 문서
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 바탕화면
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 비디오
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 사진
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 음악
drwxr-xr-x  2 user1 user1      4096  4월 21 09:45 템플릿
~~~


- 경로 이동

~~~
cd [절대경로] : 절대경로로 이동
cd .. : 이전 폴더로 이동

# 상대경로 사용시에 시작부분만 치고 tab을 누르면 자동완성된다.
# tab을 여러번 누르면 시작부분으로 시작하는 모든 폴더를 가이드해준다.
~~~

- 파일 생성 및 삭제

~~~
touch [file_name] : 파일 생성
rm [file_name] : 파일 삭제
~~~

- 폴더 생성 및 삭제

~~~
mkdir [folder_name]
mkdir -p [folder_path] : 하위 폴더까지 싹 만들어준다.

rmdir [folder_name] : 빈 폴더 삭제!!
rm -rf [folder_name] : 하위 폴더까지 싹 삭제.
~~~

- 파일 복사

~~~
cp [원본 file name] [복사 file name]
~~~

- 파일 이동

~~~
mv [file_name] [folder_name]
# 권한 문제로 안될 경우
sudo mv [file_name] [folder_name]
~~~

- 파일 및 폴더 권한 변경
    - rwxrwxrwx
        - r : read
        - w : write
        - x : execute
        - 3개 묶음으로 소유자, 그룹, 기타 사용자
    - 일반적으로 쓰는 경우는
        - 777 : 모두에게 모든 권한 허용
        - 755 : 소유자는 모든 권한, 이외에게는 r, x 권한만 부여
        
~~~
chmod 777 [file_name]
chmod 755 [folder_name]
# 권한 문제로 안될 경우
sudo chmod 777 [name]
~~~

