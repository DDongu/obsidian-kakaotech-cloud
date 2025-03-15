---
sticker: emoji//270d-fe0f
---
# 기초
Git: 컴퓨터 파일의 변경사항을 추적하고 여러 명의 사용자들 간의 작업을 조율하기 위한 분산 버전 관리 시스템
GitHub: Git 저장소의 호스팅을 지원하는 오픈소스 웹 서비스

### 중앙 집중식 버전 관리 시스템 VS 분산 버전 관리 시스템
**중앙 집중식 버전 관리 시스템**
- 중앙에서 관리
- + 여러 사람이 동시에 동일한 정보를 볼 수 있음
- + 작업 순서를 알 수 있음
- + 로컬 컴퓨터의 저장소 少
- - 중앙의 서버가 날라가거나 인터넷이 되지 않으면 사용할 수 없다
- - 서버가 날라가면 복구 힘듦

**분산 버전 관리 시스템**
- Git
-  로컬과 서버에서 관리
- + 인터넷의 연결이 없어도 사용 가능
- + 서버가 날라가도 로컬 컴퓨터를 통해 복구할 수 있음.
- - 여러 명이서 같은 작업을 할 때 관리가 어려움


# 명령어

**git 동작의 도식화**
![[Pasted image 20250126203859.png]]

**명령어**
- git init: Working Directory를 생성하는 것
- git remote: 로컬 저장소와 연결된 원격 저장소를 관리하는 명령어
```bash
# git remote add <원격저장소명> <저장소주소>

# 해당 저장소를 origin이라고 원격저장소명을 저장한거임
git remote add origin "저장소 주소"

# 등록된 원격저장소 확인
git remote -v
```
- git status: 작업 중인 저장소의 상태를 확인하는 명령어
- git add: 수정된 파일을 stage에 파일을 올리는 것  (Working Directory -> Stage)
- git commit: 파일의 변경 내용을 확정하는 명령어(Stage -> History)
- git push: 로컬 저장소에서 변경된 파일을 원격 저장소에 올리는 명령어(로컬저장소 -> 원격저장소)
- git clone: 원격 저장소의 repository를 로컬에 복제하는 명령어(원격에서 처음 받을 때)
- git pull: (`git fetch+git merge`) 원격 저장소의 repository의 최신 내용은 로컬로 가져오는 명령어(원격에서 처음 이후에 받을 때)
- git diff : 파일의 변경사항을 확인할 때 사용하는 명령어
- git difftool : git diff를 GUI 버전으로 보는 명령어
- git show: 특정 commit 정보를 볼 때 사용하는 명령어
- git blame: 특정 파일의 수정 내역을 확인할 때 사용하는 명령어
- git stash: unstaged Files 위치의 파일의 변경사항을 임시공간으로 옮기는 명령어

#### diff vs status
`diff`
-> 마지막 commit 이후의 코드 변경사항을 보는 명령어
`status`
-> 파일(코드)가 stage에 있는지 unstage에 있는지 확인하는 명령어

# Branch
-> branch는 commit의 복사본이다

- git branch
```bash
# git branch <브랜치명>
git branch feature

# merge 완료된 브랜치 삭제
git branch -d "브랜치명"

# merge 안된 브랜치 삭제
git branch -D "브랜치명"
```
- git checkout: 브랜치 전환 명령어

#  Merge 및 커밋 복구

Merge의 네 종류
- 3-way merge
- fast-forward merge
- squash and merge
- rebase&merge

### 3-way merge
병합할 두 branch와 공통의 조상 커밋을 이용하여 병합하는 커밋 방식
- 공통 조상(base)를 기준으로 각 각의 브랜치에 새로 올라온 커밋을 하나로 합쳐, main에 새로운 병합 커밋을 올림.

![[Pasted image 20250126235544.png]]
**사용법**
```bash
git merge
# 현재 브랜치와 병합 대상의 관계가 Fast-forward관계 여부와 상관없이 
# Merge 커밋을 생성하여 병합한다.
git merge —-no-ff
```

### fast-forward merge
새로운 branch를 만들고 main 브랜치에 커밋이 없을 때, 만들어진 새 branch를 main으로 바꿔버리는 방식

![[Pasted image 20250127000044.png]]

**사용법**
```bash
git merge

# 현재 브랜치와 병합할 브랜치가 Fast-forward 관계일 경우 Fast-forward 병합을 진행하고, 
# 그렇지 않은 경우에는 3-way-merge를 진행하여 Merge 커밋을 생성하여 병합합니다.

# git merge --ff <브랜치명>
git merge --ff feature

#현재 브랜치와 병합할 브랜치가 Fast-forward 관계인 경우에만 병합을 진행하고, 
# 그렇지 않은 경우에는 병합을 진행하지 않는다는 명령입니다

# git merge --ff-only {병합할 브랜치 명}
git merge --ff-only  feature
```

### squash and merge
새로 만든 branch를 그냥 없애버리고 main에 커밋을 새로 하나 올리는 방식

![[스크린샷 2025-01-27 오전 12.00.54.png]]

**사용법**
```bash
# 병합 대상 브랜치의 모든 커밋을 하나의 커밋으로 squash(압축)해서 현재 브랜치로 가져옵니다.
# 이때 Merge 커밋 없이 실제 작업이 이루어지는 따로의 커밋은 존재하지 않습니다.

# git merge --squash <브랜치명>
git merge --squash feature
```


### rebase & merge
새로운 branch의 commit을 다른 브랜치의 커밋으로 넣어버리는 방식
- 그리고 새로운 branch의 마지막 커밋이 base가 됨(rebase 과정)
![[스크린샷 2025-01-27 오전 12.01.03.png]]

**사용법**
```bash
# 먼저, 옮길 commit이 있는 브랜치로 이동
git checkout 새로운브랜치
# 새로운 브랜치의 base를 main 맨 끝의 commit으로 rebase 하는 동작
git rebase "브랜치명" 
# 이때 conflict가 생긴다면 충돌을 해결한다.
# 해결한 파일을 `git add ...` 명령어를 통해 stage로 올린다.
# 다음 명령어로 rebase를 마무리
git rebase --continue

# 다시 merge를 수행할 브랜치로 이동
git checkout "브랜치명"
# merge 진행
git merge 새로운브랜치
# 이때도 마찬가지로 conflict가 발생한다면, 해결한 뒤에 git add로 stage에 올리고 다음 명령어를 수행
git merge --continue
```

#### "3-way merge" vs "rebase&merge"

✅ `3-way merge`는 base 브랜치를 기점으로, 각 브랜치의 새로 올라온 commit을 병합해 새로운 commit(병합 커밋)을 생성하여 넣는 것(*새로운 commit 생성 O*)
- 뻗어난 가지가 이어졌지만 분리된 모습이 그대로 존재함

✅ `rebase&merge`는 한 브랜치의 커밋을 다른 브랜치의 끝에 넣어, 하나의 가지로 만드는 것(*새로운 commit 생성 X*)
- 뻗어난 가지가 하나로 이어져서 완전히 하나의 가지만 남음.

**두 방법 모두 두 브랜치를 병합하는 과정인데, 그 과정에서 같은 파일에 대한 수정이 있으면 conflict가 일어남. 따라서 conflict에 대해 개발자가 직접 수정해줘야 됨.**


#### .gitignore 파일
git에 반영하고 싶지 않은 파일의 정보를 설정하는 파일
- IDE나 특정 개발환경마다 세팅 파일이 자동으로 생성되는데, google에 "python pycharm .gitignore file example"이라고 검색하면 적당한 .gitignore 파일을 구할 수 있음


# Git 복구
- git restore: 작업 디렉토리의 파일 상태를 복구할 때 사용하는 명령어
```bash
git restore 파일명

# 특정 commit 시점으로 해당파일을 복구한다
git restore --source <commit id> <파일명>

# 스테이지 취소
git restore --staged <파일명>
```
- git revert: 특정  commit 시점으로 되돌아감.(`기록 O`)
```bash
# 지정한 commit에서 변경된 내용을 되돌립니다. 이를 통해 이전 커밋으로 되돌아갈 수 있습니다.
git revert <커밋아이디> 

# 여러 개의 커밋에서 변경된 내용을 모두 되돌립니다.
git revert <commit id 1> <commit id 2>

# 직전의 commit 변경 내용을 되돌립니다.
git revert HEAD

# Merge 커밋을 취소합니다.
git revert HEAD
```
- git reset: 작업하는 곳을 특정 commit 상태로 완전히 되돌리는 기능.(`기록 X`)
```bash
# 업 디렉토리, 스테이징 영역, 커밋을 모두 지우고 지정한 커밋으로 돌아가는 방법
git reset —hard 커밋아이디

# 변경 지우지 않고 커밋아이디이후 변경한것은 stageing area에 올려놓은 상태로 돌아가는 방법
git reset —soft 커밋아이디

# 변경 지우지 않고 커밋아이디이후 변경한 것은 unstage해놓은 상태로 돌아가는 방법
git reset —mixed 커밋아이디
```


# origin, upstream

![[Pasted image 20250127124038.png]]
fork를 할 때 쓰이는 개념으로, fork의 대상이 되는 레포를 upstream, fork해서 내 계정에 복사해온 레포를 origin이라고 함.

여기서 `-u` 옵션은 `--set-upstream`의 약어로 push 할 원격저장소(upstream)를 설정함.
```bash
git push -u origin main
```

# Git 브랜치 전략 (git-flow)
-> 브랜치는 중요도에 따라 세 가지 주요 유형으로 구분하는 전략

1. Master / Main : 배포 브랜치
    - 안정성이 보장되며, 운영 중 발생할 수 있는 치명적인 문제에 대비해 롤백할 수 있는 커밋 집합을 유지함.

1. Develop (Staging) : 테스트 브랜치
    - 로컬에서의 테스트와 개발을 마친 후 최종적인 테스트가 이루어지는 브랜
    - 별도의 Staging Zone용 브랜치를 만드는 것은 가능하지만, 과도한 조치일 수 있음

1. Feature : 개발 브랜치 - Develop 브랜치를 기점으로 새 브랜치를 만들어 개발
    1. `feature/**jira-001**` : 기획과 디자인에 따라 개발자에게 발행된 티켓으로 브랜치명
    2. `feature/**login-with-session**` : 어떤 작업인지 바로 알 수 있도록, 기능 내용에 대한 요약 브랜치명
    3. `void/**login-with-session**` : 브랜치명에 개발자 명을 붙여넣기도함
    
    - 로컬에서 개발을 완료한 경우 Develop 브랜치에 PR 요청을 하여 마지막 테스트를 준비한다.
        - PR 요청을 통해 로컬에서 완료한 개발 코드를 개발자들에게 최초 노출하여 코드 리뷰를 받을 수 있음
        - PR 요청을 통해 Github Workflow 로 CI (테스팅) 파이프라인을 연결하였다면, 코드 유효성 테스트 가능


# Git - 한 줄 정리
| 단어                     | 뜻                                                                                                 | 비고                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 분산 버전 관리 시스템           | 로컬과 서버 모두에서 코드나 프로그램을 관리하는 시스템                                                                    |                                                                                  |
| commit                 | stage에 올라와 있는 파일의 변경 이력을 History에 올리는 것                                                           |                                                                                  |
| working directory      | 개발자가 작업 중인 코드나 파일이 있는 디렉토리                                                                        |                                                                                  |
| repository             | git에서 프로젝트의 코드 및 버전 등을 올리고 관리할 수 있도록 보관하는 장소                                                      |                                                                                  |
| branch                 | 프로젝트의 개발 라인을 복사 및 분리하여 독립적으로 작업할 수 있도록 하는 것                                                       |                                                                                  |
| organization           | 협업을 위해 여러 개인 계정을 묶은 그룹 계정                                                                         |                                                                                  |
| origin                 | Fork를 수행할 때, 복사하여 내 계정에 만들어진 Repository                                                          |                                                                                  |
| upstream               | Fork를 수행할 때, 복사의 대상이 되는 Repository                                                                |                                                                                  |
| merge                  | 어떤 브랜치의 commit 상황을 다른 브랜치에 병합하는 것                                                                 |                                                                                  |
| fork                   | 원본 Repository에 영향을 주지 않고 작업을 독립적으로 개발하기 위해, 원본 Repository를 개인 계정의 Repository로 복사하여 개발할 수 있도록 하는 것 |                                                                                  |
| pull request           | fork를 하여 복사해온 원본 repository에, 개인 계정에 복사된 repository의 수정사항을 반영해달라고 요청을 보내는 것                       |                                                                                  |
| 3-way merge<br>        | 병합을 진행하는 두 브랜치의 base를 기준으로 병합을 하는 방식                                                              | merge commit이 새로 생성됨                                                             |
| fast-forward merge<br> | 기준 브랜치에는 새로운 commit이 없고, 새 브랜치에서만 commit이 있을 때, 가지를 새 브랜치로 이어 한 줄로 만드는 방식                         | 기준 branch의 HEAD와 대상 branch의 base가 같을 때, 기준 branch의 HEAD를 대상 branch의 HEAD로 바꾸는 방식 |
| squash and merge<br>   | 새 브랜치의 commit을 모두 1개의 commit으로 만들어서 기준 branch에 올리는 방식                                             | 브랜치 선이 끊어지고, 기준 브랜치에는 1개의 commit이 생성됨                                            |
| rebase&merge           | 새 브랜치의 base를 rebase하여, 새 브랜치의 모든 commit을 기준 branch에 옮기는 방식                                        | 브랜치 선이 끊어지고, 기준 브랜치에는 새 브랜치의 모든 commit 생성됨                                       |
| base                   | 브랜치 분기의 기준이 되는 commit                                                                             | 가장 최근의 공통 조상(commit)                                                             |
|                        |                                                                                                   |                                                                                  |
