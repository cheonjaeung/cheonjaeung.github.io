---
title: "Git Worktree : 여러 개의 작업 트리 관리하기"
summary: "동시에 여러 개의 작업 상황을 다루어야 하는 경우 각 작업 브랜치로 이동하는 것이 번거롭다. 여러 개의 작업 트리를 두어 편하게 스위칭할 수 있는 git worktree를 활용해보자."
coverAlt: "tree"
coverCaption: "Photo by author"
date: 2025-01-09
categories: ["Git"]
---

Git으로 버전 관리를 하며 작업을 하다 보면 동시에 여러 개의 작업 트리를 이동해야 할 일이 생긴다.
예를 들어 A 브랜치와 B 브랜치 두개를 동시 작업중이면서 갑자기 버그가 발견되어 핫픽스 해야 할 일이 생길 수도 있는 것이다.
보통은 `git stash`를 쓰거나 임시 커밋 같은 방식으로 하던 일을 저장해두고 브랜치를 이동한다.

작업을 잠시 중단하고 다른 작업으로 바꾼다는 것은 아주 번거로운 일이다.
안드로이드의 경우 각 브랜치에서 새로운 모듈이 생겼을 수도 있고 라이브러리 업데이트가 발생했을 수도 있다.
때문에 Gradle Sync를 자주 해줘야 하는 불편함이 추가된다.
그 밖에도 IDE의 indexing을 기다리는 시간 등 시간적 소요도 많고 일이 끝난 후 돌아왔을 때 어디까지 했는지 다시 기억을 되살려야 하는 것도 번거롭다.
Stash를 여러 개 해둔 경우 어떤게 무슨 작업이였는지 헷갈리는 경우도 많다.

## git worktree

`git worktree`는 이런 상황에서 유용하게 사용할 수 있는 명령어로, 여러 개의 작업 디렉토리(a.k.a. 워킹 트리 또는 워크 트리)를 관리할 수 있도록 돕는다.

{{< alert "circle-info" >}}
여기서부터는 워크 트리(Work Tree)는 작업 공간, 작업 디렉토리 등과 동의어로 사용한다.
{{< /alert >}}

일반적으로 git을 사용하면서 어떠한 작업의 단위로 branch를 나누어 개발하게 된다.
git worktree는 특정한 작업을 별도의 워크 트리로 분리해 별도로 관리할 수 있게 해준다.

예를 들어 일반적으로 개발할 때 아래와 같은 파일 구조를 가진다고 해보자.

```
Development
 ┗ project
   ┣ .git
   ┗ src
     ┗ main.kt
```

`project`라는 repository를 `Development`라는 디렉토리 아래에 clone 해둔 상태다.
여기서 `main.kt`를 수정했다고 하자.
그렇다면 `project`에서 status를 확인하면 아직 커밋되지 않은 변경사항이 보일 것이다.

```
❯ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   main.kt

no changes added to commit (use "git add" and/or "git commit -a")
```

하지만 `main.kt`를 수정하던 도중 급하게 무언가를 수정해야 한다고 해보자.
`git worktree` 없이는 `git stash`를 쓰거나 임시로 커밋을 해야 할 것이다.
대신 `git worktree`를 사용해 워크 트리를 분리해보자.

```
❯ git worktree add ../project-hotfix
Preparing worktree (new branch 'project-hotfix')
HEAD is now at cbfc60b Initial commit
```

`git worktree add ../project-hotfix` 명령을 하나 하나 분석해보자.
`git worktree add`는 새로운 워크 트리를 생성하는 명령이고 그 뒤의 매개변수인 `../project-hotfix`는 상위 디렉토리에 `project-hotfix`라는 이름의 디렉토리에 새 워크 트리를 만든다는 뜻이다.
해당 위치로 이동해 보면 똑같은 파일 트리를 가진 새로운 워크 트리가 만들어져 있을 것이다.

```
❯ cd ../project-hotfix
❯ la
total 16
-rw-r--r--@ 1 username  staff    75B Jan  9 11:46 .git
-rw-r--r--@ 1 username  staff    40B Jan  9 11:46 main.kt
```

또한 branch 이름이 워크 트리 이름과 똑같이 설정되어 있을 것이다.
별도 지정하지 않으면 기본 값으로 똑같이 가져가기 때문이다.

이 상태의 파일 구조를 보면 아래와 같다.

```
Development
 ┣ project
 ┃ ┣ .git
 ┃ ┗ src
 ┃   ┗ main.kt
 ┗ project-hotfix
   ┣ .git
   ┗ src
     ┗ main.kt
```

이제 두 개의 워크 트리가 분리된 것이다.
원본인 `project` 디렉토리에 가보면 커밋하지 않은 변경사항이 그대로 남아있고, `project-hotfix`에서 다른 변경사항이 일어나도 `project`에 영향을 주지 않는다.
즉, 두 디렉토리는 물리적으로 다른 공간이고 한 쪽에서 파일을 바꿔도 다른 워크 트리에 영향을 주지 않는다.
그리고 두 워크 트리는 서로 다른 branch이기 때문에 커밋을 해도 서로에게 직접적으로 영향을 주지 않는다.
`git stash`나 임시 커밋 등을 하지 않고도 하던 작업을 그대로 둔 채 다른 작업으로 넘어갈 수 있게 되었다.

## 다른 명령어들

`git worktree`는 몇 개의 명령어가 더 있다.
그 중 자주 사용할 만한 명령어를 몇 개 더 소개하고자 한다.

### git worktree add 고급

위에서는 단순히 `git worktree add <path>`로 워크 트리를 생성했지만 사실 특정 branch를 지정하거나 새로운 branch 이름을 직접 지정할 수도 있다.

- `git worktree add <path> <branch>`를 사용하면 워크 트리를 생성하면서 이미 만들어져 있는 branch를 쓰도록 지정할 수 있다.
- `git worktree add <path> -b <branch>`를 사용하면 워크 트리를 생성하면서 같이 만들 branch 이름을 지정할 수 있다.
- `git worktree add <path> <commit-ish>`를 사용하면 현재 HEAD 커밋 대신 특정 커밋으로 워크 트리를 만들 수도 있다.

### git worktree list

`git worktree list`를 사용하면 지금 만들어져 있는 워크 트리의 목록과 브랜치 이름, 상태 등을 볼 수 있다.

```
❯ git worktree list
/Users/sample/Development/project             cbfc60b [main]
/Users/sample/Development/project-hotfix  cbfc60b [project-hotfix] prunable
```

워크 트리의 경로와 현재 어떤 커밋과 브랜치를 바라보고 있는지 등을 확인할 수 있다.

### git worktree remove

`git worktree remove`는 더 이상 필요 없어진 워크 트리를 지우기 위한 명령어다.

```
❯ git worktree remove project-hotfix
❯ git worktree list
/Users/sample/Development/project  cbfc60b [main]
```

명령어로 지우고 싶은 워크 트리의 이름이나 경로를 입력해 주면 워크 트리를 제거할 수 있다.
이 때 브랜치는 지워지지 않는다.

## 특이사항

### 워크 트리끼리는 git 상태를 공유한다

지금까지만 보자면 같은 repository를 한번 더 clone 한 것과 큰 차이가 없어 보인다.
하지만 실제로는 조금 다르다.

`git worktree`로 만든 워크 트리는 원본과 연결되어 있다.
commit log나 branch 목록 등 히스토리나 상태를 공유한다.

실제로 워크 트리의 `.git` 파일을 열어 보면 원본 워크 트리를 가리키도록 되어 있다.

```
❯ cat .git
gitdir: /Users/sample/Development/project/.git/worktrees/project-hotfix
```

상태를 공유하기 때문에 커밋 로그, 브랜치 목록 등을 똑같이 가져갈 수 있다.
한 쪽에서 새로운 커밋을 생성하면 다른 쪽에서 별다른 행동 없이도 그 커밋을 읽을 수 있다.
브랜치 생성도 마찬가지이고 `git fetch`나 `git pull` 등으로 원격 repository를 읽는 것 또한 마찬가지이다.

### 하나의 branch는 하나의 워크 트리만 사용할 수 있다

```
❯ git checkout project-hotfix
fatal: 'project-hotfix' is already used by worktree at '/Users/sample/Development/project-hotfix'
```

원본 `project` 워크 트리에서 `project-hotfix`가 점유 중인 브랜치로 checkout을 시도해 보면 위와 같이 "이미 사용중이다"라는 메세지와 함께 실패한다.
덕분에 여러 개의 워크 트리에서 접근해 혼란스러워 지는 일은 일어나지 않을 것이다.

## 레퍼런스

- https://git-scm.com/docs/git-worktree
