---
title: frequently used git command
author: Wendy
categories: [Blogging, 개발환경]
tags: [개발환경]
---


### 자주 사용하는 git command 모음

자주 사용하는 git command를 적어놓으려고한다.
git 이 익숙치 않아서 매번 흩어진 자료에서 찾는데, 
혹시 기억하지 못하거나 순서 상 빼먹는것이 없도록~ 적어보자~

#### git 정보 확인

#### 코드 commit 과 push 까지


### commit 관련 


git add 하여 staging area 에 있는 파일을 다시 unstage로 변경

git reset HEAD {file}



### 로컬 저장소의 코드 업데이트
다수와 함께 작업할때, 또는 혼자 작업하더라도 환경을 옮겨가며 작업시에 github server의 데이터로 로컬 내용을 업데이트가 필요하게 된다.

#### 로컬저장소에 변경된게 아무것도 없을때


git pull <remote> <branch> 와 같이 쓸수 있다.


``` console
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
`````

이경우에는 단순히 git pull만 해주면 됨
``` console
$git pull
```


#### 로컬저장소에 수정사항에 대한 변경이 있을때


```
jia@DESKTOP-3SLNH3A:~/workspace/uftrace$ git pull origin tests
From https://github.com/wendyisdream/uftrace
 * branch              tests      -> FETCH_HEAD
Auto-merging utils/dwarf.c
CONFLICT (content): Merge conflict in utils/dwarf.c
Automatic merge failed; fix conflicts and then commit the result.
```

이떄, remote(origin)에 있는 것으로 덮어 쓰고 싶을때

어떤 부분이 충돌되었는지 확인

```console
git diff
```

충돌한 파일이 diff syntax에 의해 변경된 것을 볼수 있다.
이분을 수정후에 다시 commit해주면 merge로 변경됨.



2. 


#### 이미 PR한 commit 내용 수정하기

##### commit 메시지 변경시

아래 링크를 따라 이용함

1. git log로 현재 수정할 commit 위치 확인
 
```console
git log
```

2. 수정할 commit이 HEAD로 부터 몇번째인지 확인
보통 젤위에것이면 HEAD~1이 된다.
pick을 reword로 변경


```console
git rebase -i HEAD~1



```

3. 수정 후 반영
```console
git push -f origin {branch-name}
```

##### code 변경시



1. pick을 edit로 변경

```console
$ git rebase -i HEAD~1
Stopped at c01b6eb9...  tests: Fix applying zero offset to null pointer in unittest
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```


2. 변경 내용 반영 후 git add 하고 변경 내용 저장하기

```console
git add {변경파일}
git commit --amend
```
3. commit 작업이 완료되었음을 알림
```console
git rebase --continue
```
4. 코드를 원격 git 서버로 푸시~~
```console
git push -f origin {branch-name}
```


### 참고 링크

<https://velog.io/@ongddree/git-%EC%9D%B4%EB%AF%B8-push%ED%95%9C-commit-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0>
<https://backlog.com/git-tutorial/kr/stepup/stepup7_6.html>
