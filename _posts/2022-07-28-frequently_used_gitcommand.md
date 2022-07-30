---
title: frequently used git commands
author: Wendy
categories: [Blogging, 개발환경]
tags: [개발환경]
---


## 자주 사용하는 git command 모음

git 이 익숙치 않아 사용할때 마다 정보를 찾곤 한다. 
사용하게 되는 git command를 적어 보자

### 작업 준비

- 현재 소스 상태 확인  
```git status
```

- 현재 branch 확인  
```git branch
```

- branch 생성  
``` git checkout -b {branch name}
```

- branch 삭제  
```git branch -D {branch name} 
```

- fork 한 것에서 clone하여 가져온 repo인경우 fork를 생성한 원 repository를 remote에 upstream으로 등록해줘야한다.  
```git remote add upstream {fork가 생성된 원 repository}
```
- remote 정보 확인  
```git remote -v
```

### 코드 stage -> commit -> PR 

한개의 반영 단위를 commit 인데, 반영에 포함될 파일을 먼저 staged 상태로 만든 후 commit하면 된다. 

1. 파일을 staging 상태로 만들기  
```git add {file}
```

2. staging area의 파일을 commit하기  
```git commit -sm {commit message}
```
이떄 -m 옵션은 메시지, -s 옵션은 singed-off 정보(git config의 정보)가 commit message 끝에 함게 기입된다.

#### 원격서버에 반영하기 
orgin은 git clone으로 가져온  repository이며, upstream은 fork가 수행된 원 repository이다.

3. 원 소스와 싱크를 맞추기  
git fetch는 target branch로 부터 commit history를 가져온다. 이때 로컬 코드에 대한 수정이 이루어지진 않는다.  
```git fetch upstream master
```

4. fetch로 가져온 최신을 현재 로컬에 반영  
```git rebase upstream/master
```

5. remote에 코드 올리기  
```git push origin {branch}
```

6. github 사이트에서 업데이트된 commit에 대해 원 repo로 pull request를 보내면 됨!


#### 작업 중 수정이 필요할 때

- commit 메시지 수정하기  
```git command --amend
```

- staging area 에 있는 파일을 다시 unstage로 변경  
```git reset HEAD {file}
```

- commit을 취소 하고 싶을때  
  - commit을 취소 & 파일을 staged 상태로 두기  
  ```git reset --soft HEAD^
  ```

  - commit을 취소 & 파일을 unstaged 상태로 두기  
  ```  git reset --mixed HEAD^
  ```

### 이미 PR한 commit 내용 수정하기

#### commit 메시지 변경시

1. git log로 현재 수정할 commit 위치 확인  
```git log
```

2. rebase -i HEAD~<수정할 commit의 순서> (interactive 모드로 reword 모드로 변경) 
처음 commit 수정의 경우 HEAD~1

※ pick을 reword로 변경하고 저장하면 변경하는 화면 나옴
```console
 $git rebase -i HEAD수정할 commit의 순서
 pick a5c3307f tests: Fix applying zero offset to null pointer in unittest
 ...
 r, reword = use commit, but edit the commit message
 ...
```

3. 수정 후 push  
```git push -f origin {branch-name}
```

##### code 변경시

1. git log로 현재 수정할 commit 위치 확인  
```git log
```

2. pick을 edit로 변경

※ pick을 edit로 변경하고 저장 

```console
$ git rebase -i HEAD~1
pick a5c3307f tests: Fix applying zero offset to null pointer in unittest

# Rebase 07332935..a5c3307f onto 07332935 (1 command)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#


Stopped at c01b6eb9...  tests: Fix applying zero offset to null pointer in unittest
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

2. 로컬에서 변경 진행하고 해당 파일을 git add 후 commit --amend 하기

```console
git add {변경파일}
git commit --amend
```

3. commit 작업이 완료되었음을 알림  
```git rebase --continue
```

4. 코드를 원격 git 서버로 푸시~~  
```git push -f origin {branch-name}
```


### 참고 링크

<https://velog.io/@ongddree/git-%EC%9D%B4%EB%AF%B8-push%ED%95%9C-commit-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0>
<https://backlog.com/git-tutorial/kr/stepup/stepup7_6.html>
