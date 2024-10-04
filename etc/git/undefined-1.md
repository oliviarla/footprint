# 내가 자주 사용하는 명령어 모음

## 외부 Repository의 브랜치 pull하기

```
git pull <git_url> <branch> --allow-unrelated-histories
git pull <git_url> <remote branch>:<local branch>
```

## Upstream 등록하기

* 작업을 할 때 저장소를 그대로 pull하여 코드를 수정할 수도 있지만, 외부 저장소의 코드라면 Fork한 후에 내 저장소에  서 작업할 수 있다.
* 이 때 upstream을 등록하여 외부 저장소와 연결할 수 있다. 외부 저장소 브랜치로부터 pull, push 작업을 할 수 있다.

```
git remote add upstream https://github.com/jam2in/arcus-hubble-v3.git
git pull upstream {branch name}
git push upstream {branch name}
```

## 브랜치 이동하기

* git switch 를 통해 브랜치를 이동할 수 있다. 새로운 브랜치 생성 시에는 `-c` 인자를 주어야 한다.

```
git switch {branch name}
git switch -c {branch name}
```

* 브랜치를 새로 생성할 때에는 master, develop 등 기준이 되는 브랜치에서 생성하는 것이 좋다.
  * task1 진행중인 브랜치에서 또다른 작업을 하기 위해 task2 브랜치를 생성하면, task1의 커밋이력을 기준으로 새로운 브랜치가 생성된다. 이렇게 되면 develop 브랜치에 PR을 보냈을 때 task2의 작업만 보내지는 것이 아니라 task1의 작업까지 보내진다.
* 현재 브랜치에서 특정 커밋을 기준으로 새로운 브랜치를 생성할 수 있다.

```
git switch -c {pr용 새로운 브랜치} {Commit Hash}
```

## Merge 하기

* develop브랜치에 fix 브랜치의 커밋 머지하기

```
git switch develop
git merge fix
```

## Rebase 하기

* Rebase란&#x20;
  * `commit2`
  * `commit1`
  * 순서로 commit이 쌓였을 때 commit2의 결과를 commit1로 합치는 것

```
git rebase -i HEAD^^
```

```
pick abcedfg commit1
pick abcedfh commit2
```

* 여기서 아랫줄 `pick`을 `fixup` 혹은 `f`로 변경해 저장한다.

```
pick abcedfg commit1
f abcedfh commit2
```

* 다시 git push할 때에는 `-f` 옵션을 주어야 한다.

```
git push origin {branch name} -f
```

## Commit 명 변경하기

```
git commit --amend
```

## Stash하기

* 현재까지의 변경사항을 임시 저장하고, 원하는 작업을 한 다음에 임시저장한 내용을 불러올 수 있다.

```
git stash
git stash apply
```

* Stash의 내용을 확인할 수 있다.

```
git stash show -p stash@{0}
```

## 사라진 Stash 찾기

```
git fsck --no-reflog | awk '/dangling commit/ {print $3}' | xargs -L 1 git --no-pager show -s --format="%ci %H" | sort`
```

* 날짜를 기준으로 사라진 Stash를 추측해 해당 hash값을 이용해 하나씩 `git stash apply` 해보기

## PR 커밋 로컬로 가져오기

```
git pull origin pull/{pr번호}/head:{pr branch}
```

### ✅ 특정 커밋을 현재 브랜치로 떼오기

```
git cherry-pick {commit hash}
```

* 커밋 순서를 바꾸고 싶다면 git rebase -i HEAD^^ 로 들어가 순서 변경 가능

## 다른 사람 레포지토리 커밋에 내용 덮어써서 커밋하기

```bash
git remote add <사용자명 등 식별할 수 있는 이름> <github 주소>
git switch <커밋이 있는 브랜치명>
git pull <사용자명 등 식별할 수 있는 이름> <브랜치 명>
git commit -m "..." # 내가 한 작업을 커밋한다.
git rebase -i HEAD^^ # 새로운 작업으로 추가한 커밋을 기존 커밋에 rebase한다.
git push <사용자명 등 식별할 수 있는 이름> <브랜치 명> -f
```

* 예를 들어 oliviarla 사용자의 arcus-java-client 레포지토리에 있는 cleanup 브랜치의 `CLEANUP: tidy codes` 커밋에서 내가 직접 코드를 변경하고 싶다면 아래와 같이 사용하면 된다.

```bash
git remote add oliviarla https://github.com/oliviarla/arcus-java-client.git
git switch -c cleanup
git pull oliviarla cleanup
git commit -m "my modification" # 내가 한 작업을 커밋한다.
git rebase -i HEAD^^ # rebase하여 기존 커밋으로 합친다.
git push oliviarla cleanup -f
```
