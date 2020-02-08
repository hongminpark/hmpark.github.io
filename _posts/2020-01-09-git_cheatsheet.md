---
layout: post
title: "Git cheatsheet"
author: "Hongmin Park"
---
사내 테크쉐어 강의 중 아주 매우 유용한 Git Cheatsheet 강의를 정리해보았다. 요긴히 사용하도록 하자.

### Diff

```bash
git diff # working directory 와 index(현재까지 commit이 완료된 것들의 상태) 비교
git diff (commitA) (commitB) # commit 간 차이 비교
```

### Log

```bash
git log
git log -p # diff와 함께 보기
git log -$Number # history 개수 지정
git log --grep $keyword
git log --oneline
git log --graph
```

### Branch

branch를 변경하면 HEAD(branch 위치), index와 working directory 파일들이 branch 기준으로 바뀐다.<br>
잘 돌아가고 있던 프로젝트를 건드리지 않고, 새로운기능을 추가/실험/버그픽스를 해야할 때 branch를 딴다.

branch 생성 후 변경하는 것을 종종 까먹으므로 `git checkout -b $branch_name` 방법으로 브랜치를 생성하자.

```bash
git branch # branch 목록확인
git branch $branch_name # 해당 이름으로 branch 생성
git branch -d/D $branch_name # merge된 barnch(d)만 또는 그냥 브랜치(D) 삭제
git checkout $branch_name # 해당 branch로 이동
git checkout -b $branch_name # branch를 생성하고 바로 해당 브랜치로 checkout
```

### Merge

코드가 반영될 branch(주로 master)에서 merge를 수행한다.

```bash
git merge $target_branch_name
git branch -d $target_branch_name # merge 후에는 branch를 삭제할 수도 있다.
```

Merge에는 ***Fast forward Merge***와 ***3 way Merge***가 있다.

#### Fast-forward Merge

동일한 base commit 에서 출발한 branch의 경우는 단순히 HEAD를 옮기는 fast-forward Merge를 함, 바로 Merge됨

#### 3 way Merge

Master에서 다른 branch 를 따서 commit을 하는 도중 동료의 commit에 의해 Master가 바껴버릴 수 있다. 이 때에는 Merge Message를 남겨야됨

base commit을 최신으로 유지하는게 좋음. Graph가 더 보기 좋음.

### Rebase

특정 branch의 Base 커밋 위치를 이동시킨다.

commit 주소가 바뀌고, conflict 발생 가능성 농후하다. 시간 순 history는 파괴된다. 일주일 전 커밋이 하루 전 master 커밋에 의해 초기화될 수도 있다. 작업이력이 중요한 경우 부적절하다.

`git rebas `

### remote

```bash
git remote add (remote_alias) (remote_url) # 기존 프로젝트에 원격 저장소를 연결할 때
git remote show (remote alias)
git remote rename (old_name) (new name)

git remote add origin $git_repo
git remote -v
git remote show origin

git push $remote $remote_branch_name
```

### fetch

리모트 저장소에 추가된 object 들을 받아서 local에 저장

### pull

리모트 저장소에 추가된 object 들을 받아서 merge까지 수행 = fetch + merge

### Pull Request

- 프로젝트 내에서 branch를 따거나

- fork한 프로젝트 내에서 branch를 따서 기능 개발 후에 Merge를 요청

### RM

삭제를 안하고 이미 push해버린 경우

```bash
git rm --cached
```

### 그 외 non-fast-forward-push 방지

### commit 되돌리기

```bash
git reset ${commit_id}
```

### reflog

reflog 명령어로 git의 헤드 주소 확인하고 되돌릴 수도 있다.

```bash
git reflog
git reset ${head_id}
```

### branch 생성 후 checkout을 안한 채 변경한 경우
(애초에 이를 방지하기 위해 `git checkout -b $branch_name` 사용을 권고)<br>
생성 브랜치를 삭제 후 다시 만들어서 checkout함. 그리고 master는 원격에서 다시 가져옴.

```bash
git branch -d hongmin-park
git branch hongmin-park
git checkout hongmin-park
git push origin hongmin-park

git branch -D master
git checkout master
```

