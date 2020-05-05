# 정리(일반화)
~~~
git remote add [name] [폴더경로] // git remote로 [name] 확인
git fetch [name]
git merge --allow-unrelated-histories [name]/master // ls로 파일 확인
git push // commit log에서 확인 : Merge remote-tracking branch '[name]/master'
~~~

# 예시 테스트

## 디렉토리
~~~
git_merge_test
    L merge1
        L from_merge1.md
        L README.md
    L merge2
        L from merge2.md
    L merge3
        L output
            L some image file
~~~

## merge2를 merge1에 병합
~~~
git remote add merge2 ../merge2

git remote
// 현재 remote중인 경로 확인
merge2 // 추가한 경로
origin // 기본적인 origin, 현재폴더 기준

~~~
- ../merge2 폴더 경로를 "merge2"라는 이름으로 설정

~~~
git fetch merge2

warning: no common commits
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 8 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (8/8), 1.44 KiB | 22.00 KiB/s, done.
From ../merge2
 * [new branch]      master     -> merge2/master

~~~
- 뭔가 로딩이 된다..?
- staging or commit과 같은 개념으로 생각된다.
- 파일들을 현재 폴더(origin)에 넣을 준비까지 한 상태

~~~
git merge --allow-unrelated-histories merge2/master

Merge made by the 'recursive' strategy.
 from merge2.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 from merge2.md
~~~
- merge2/master 것을 "history관계없이" 병합한다.
- push와 같은 개념으로 추측된다.

~~~
'from merge2.md'   from_merge1.md   README.md
~~~
- ls 확인 => merge2의 것이 왔다.

~~~
git remote remove merge2
~~~
- 다 쓴 경로는 삭제!

~~~
git commit -m 'Merge : merge2 into merge1' // 안해도 됨.
git push
~~~
- 이제 commit & push 진행
- merge시에 자동으로 commit을 한다.  
  Merge remote-tracking branch 'merge2/master'
- commit list 확인 시 병합된 repository의 commit history도 추가되어 있다.  
  시간순으로 정렬이 된다!!

## 주의점
- 같은 이름의 파일이 있으면 충돌이 일어난다.
    - 미리 제거해주자.
- master 경로를 가져오게 된다.
    - 깔끔한 push까지 마무리해둔 상태에서 진행할 것
## 출처
- https://mansoo-sw.blogspot.com/2017/08/git-repository-merge.html