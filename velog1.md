# git 명령어 정리 (restore, reset, checkout, switch)

HEAD, index(staging area), working tree의 관점에서 정리해보았습니다.

이렇게까지 할 필요는 없는 거 같긴 한데... 혹시나 저처럼 처음부터 끝까지 거의 완벽히 알아야 직성이 풀리는 분들이 계시면 도움이 됐으면 해서 올리게 되었습니다.

[git-scm](https://git-scm.com/doc)의 Book이랑 레퍼런스를 주로 참고했고, 아래 예시들은 대부분 만들어서 돌려봤습니다.

혹시 잘못된 부분 있으면 댓글로 알려주시면 감사하겠습니다

## 명령어로 분류

### `git restore`

1. `git restore <file>`
2. `git restore --source <tree-ish> --staged <file>`

사실상 둘이 같은 포맷.

#### 1. `git restore <file>`

옵션 없는 기본은 
```
index -> working tree
```

#### 2. 옵션이 있는 포맷

`--staged`를 주면, 목적 지점을 index(staging area)로 바꾸는 느낌으로
```
HEAD -> index
```

`--staged --worktree`하면 
```
<tree-ish> -> index, working tree
```
(`--source` 값이 없는 경우 index -> working tree로 됨)

출발지점은 `--source=<tree-ish>`, `--source <tree-ish>` 형태로 줄 수 있다.

되돌리는 역할을 한다고 생각할 수 있다.

### `git reset`

1. `git reset <tree-ish> -- <file>`
2. `git reset <mode> <commit>`

`git add`의 반대 역할을 한다.

`git add`는 worktree -> index로 올린다. 그렇다면 그 반대는 뭘까?

생각해보면 HEAD -> index로 업데이트 하는 게 그 반대이다.

reset의 두 포맷 다 그 역할을 한다.

index를 업데이트 하는 게 메인 역할이라고 볼 수 있다.

#### 1. `git reset <tree-ish> -- <file>` (`<tree-ish>`의 기본값 HEAD)
```
주어진 파일에 대해서
<tree-ish> -> index
```

#### 2. `git reset <mode> <commit>` (`<mode>`의 기본값 `--mixed`, `<commit>`의 기본값 HEAD)

```
HEAD가 가리키는 브랜치를 <commit>으로 옮긴다
<commit> -> index
```

결론적으로 HEAD와 index를 업데이트한 게 된다

이 포맷에서는 **HEAD가 가리키는 브랜치를 항상 `<commit>`으로 옮긴다.** (detached HEAD면 HEAD를 옮김)

그렇다면 `<mode>`의 값에 따라 index와 working tree는 어떻게 되는지 정리해보자

`<mode>`의 값들 :

- `--soft` : 아무것도 안함

- `--mixed` (디폴트값) : `<commit> -> index`

- `--hard` : `<commit> -> index, working tree`

- ...

`git reset <mode> <commit>`은 **HEAD가 가리키는 브랜치**를 옮기게 해주는 명령어이다.

더 자세한 내용은 문서 [Reset 명확히 알고 가기 (git-scm)](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Reset-%EB%AA%85%ED%99%95%ED%9E%88-%EC%95%8C%EA%B3%A0-%EA%B0%80%EA%B8%B0) 참고

### `git checkout`

1. `git checkout <tree-ish> -- <file>`
2. `git checkout -- <file>`
3. `git checkout <branch>`

working tree를 업데이트하는 역할이라고 생각할 수 있다.

#### 1. `git checkout <tree-ish> -- <file>`

```
주어진 파일에 대해서
<tree-ish> -> index, working tree
```

파일을 갖고 온다 라는 느낌이다.

#### 2. `git checkout -- <file>`

```
주어진 파일에 대해서
index -> working tree
```

1과 2는 사실 같은 포맷인데,

레퍼런스 문서에서, `<tree-ish>`가 없으면 `index -> working tree`로 하고 `<tree-ish>`가 있으면 `<tree-ish> -> index, working tree`로 한다고 되어있다. ([참고](https://git-scm.com/docs/git-checkout#Documentation/git-checkout.txt-emgitcheckoutem-f--ours--theirs-m--conflictltstylegtlttree-ishgt--pathspec-from-fileltfilegt--pathspec-file-nul))

아는 건 없지만 이런 식으로 명령어가 만들어진 이유를 고민해보자면,

`git add <file>`은 `working tree -> index`로 파일을 올리고, `git reset -- <file>`은 `HEAD -> index`로 파일을 내린다.

그래서 `index -> working tree`로 내릴 수 있는 명령어로 비슷한 형태인 `git checkout -- <file>`을 만든 게 아닐까 싶다. 

물론 이젠 `git restore <file>`로도 똑같은 기능을 할 수 있다. (아래 **기능으로 분류** 참고)

#### 3. `git checkout <branch>` (`<branch>`의 기본값 HEAD)

**HEAD를 이동**하게 해주는 명령어인데, 일반적으로 다른 브랜치로 이동하는 데에 쓰인다.

`<branch>`에는 정확히는 브랜치 뿐만 아니라 커밋, 태그도 줄 수 있다.

대상이 될 커밋이 존재하기만 하면 되는 느낌이다.

```
HEAD를 주어진 브랜치 또는 커밋(<branch>)으로 이동
<branch> -> index, working tree
```

HEAD를 이동하고, 그 HEAD를 index와 working tree에 덮어씌운다.

이렇게 index와 working tree를 모두 덮어씌우는 명령어들은, 작업했지만 커밋하지 않은 것들을 **모두 날려버릴 수 있으므로** 그런 local changes를 어떻게 대하는지가 중요하다.

### local changes

HEAD, index, working tree 사이에서 나올 수 있는 파일이 바뀌는 경우의 수는 생각보다 그렇게 많지 않다.

파일은 추가, 변경, 삭제 이렇게 3종류가 있고, 단계 이동은 `HEAD <-> index`, `index <-> working tree` 이렇게 2개가 있다.

이걸 파일로 하나하나 만들어보자.

working tree에서 추가됐으면 AddWorktree, index에서 삭제됐으면 DelIdx 이런 식으로 이름을 붙여주자.

그리고 Modify는 HEAD, index, working tree를 모두 다르게 해줘서 한 파일이지만 2번의 변경이 있게 해주자.

그렇게 이름에 맞게 파일을 만들고 index에 올릴 것들을 `git add`로 올려주고 `git status -s`를 해주면 아래처럼 된다.

```
A  AddIdx.txt
D  DelIdx.txt
 D DelWorktree.txt
MM Modify.txt
?? AddWorktree.txt
```

각 단계별로 존재하는 파일들을 정리해보면

```
HEAD : DelIdx, Modify, DelWorktree
index : AddIdx, Modify, DelWorktree
working tree : AddIdx, Modify, AddWorktree
```

이런 식으로 존재한다. (HEAD는 커밋)

Modify는 셋 다 존재하지만 파일 내용이 HEAD에는 '1', index에는 '2', working tree에는 '3'이라고 적혀있는 식으로 모두 다른 상태이다.

이 상태에서 다른 커밋이나 브랜치로 `checkout`, `switch` 또는 `reset --hard` 하면 index와 working tree의 파일들 중 어떤 게 날아가고 어떤 게 남을까?

---

가장 일반적인 시나리오는 어떤 브랜치에서 작업하다가, 어떤 파일들은 index에 올리고 어떤 파일들은 안올리고 수정만 하고 있는 상태에서 다른 브랜치로 이동하려는 상황일 것이다.

직관적으로 생각해볼 때 날아가도 문제없는 것과 날아가면 안되는 게 뭔지 생각해보자.

```
1) 커밋에 존재하지만 삭제된 파일. 
working tree에만 삭제되었거나, `git add`로 삭제됐다는 사실을 index에 올림.
(DelWorktree, DelIdx)

2) 커밋에 없었는데 추가된 파일. 
working tree에만 추가되었거나, `git add`로 추가되었다는 사실을 index에 올림.
(AddWorktree, AddIdx)

3) 커밋에 있었고 index에서 바뀌었거나 working tree에서 바뀐 파일. 
(Modify)
```

1)은 날아가도 상관없다. 커밋(HEAD)에서 restore, reset 등을 통해 되돌릴 수 있다.

2)와 3)은 날아가면 안되는 내용들이고 이게 유지되어야 할 local changes임을 알 수 있다.

그리고 index와 working tree는 두 공간으로 존재하지만 사실 올렸냐 안올렸냐의 차이일 뿐, 어차피 커밋이 되지 않은 건 마찬가지다.

그래서 index와 worktree 둘을 묶어서 local이라고 볼 수 있다.

따라서 정리해보면

Local Changes = 로컬에서 추가되었거나 변경된 파일들

그리고 Modify를 두 개로 나누어서 생각해보면 결론적으로 이런 파일들이 local changes이다.
```
AddIdx, AddWorktree, ModIdx, ModWorktree
```
이제 각 명령어들의 실행 결과가 어떻게 되는지 알아보자.

index와 working tree를 모두 바꾸는(덮어씌우는) 명령어에는 이런 게 있다.

```
git checkout <tree-ish> -- <file>
git checkout <commit>
git switch <branch>
git reset --hard <commit>
git restore --source <tree-ish> --staged --worktree <file>
```

HEAD를 옮기는 명령어와 안 옮기는 명령어의 행동은 조금 다르다.

그리고 `<commit>`나 `<tree-ish>`, `<branch>`가 **HEAD**일 때와 **다른 커밋**일 때가 조금 다른데, 다른 커밋일 때엔 생각해야될 게 하나 늘어난다.

HEAD와 `<commit>`를 비교해볼 수 있다는 점인데, 사실 그렇게 복잡한 건 아니고 똑같이 파일의 추가, 변경, 삭제가 있다.

#### `git checkout <tree-ish> -- <file>`

HEAD를 안 옮긴다.

즉, 다른 커밋과 HEAD를 비교할 필요는 없다.

이 명령어는 생각해보면 다른 커밋으로부터 파일을 가져오겠다, 덮어쓰겠다 라는 목적으로 생각할 수 있다.

1) `git checkout HEAD -- .`

로컬(index, worktree)에서 추가(AddIdx, AddWorktree) : 안건드림
(특히 AddWorktree는 Untracked 파일이다.)

로컬에서 변경(ModIdx, ModWorktree) : HEAD에서 덮어씀
이 명령어의 목적이 주어진 파일들을 `<tree-ish>`로 부터 덮어쓰겠다는 거였다.

로컬에서 삭제(DelIdx, DelWorktree) : HEAD에서 덮어씀 (돌아옴)

**결론 : 로컬에서 추가된 건 놔둠, 나머지는 덮어씀**

2) `git checkout <commit> -- .`

HEAD와 다른 커밋(`<commit>`)으로부터 가지고 온다.

따라서 그 커밋과 비교해서 어떻게 되는지 생각해볼 필요가 있다.

그 커밋과 비교했을 때 HEAD에 추가됐으면 AddCommit, HEAD에서 삭제됐으면 DelCommit, 둘 다 존재하고 내용이 바뀌었으면 Modify라고 하자.

- HEAD에서 추가(AddCommit) : 안건드림

즉, HEAD에 있고 `<commit>`에 없을 때, 그걸 덮어씌워서 없애는 게 아니라 **놔둔다**.

로컬과 `<commit>`을 비교할 때에도 로컬에서 생긴 건 모두 안건드리고 놔둔다.

- `<commit>`에서 삭제(DelCommit) : 덮어씀

즉, HEAD에 없고 `<commit>`에 있을 때, 갖고 온다.

- 변경 (Modify) : 덮어씀 (커밋으로부터 파일을 덮어쓰는 게 목적)

정리하면 **갖고오고 덮어쓰는데, 추가된 건 놔둔다.**

- 로컬에서 변경 : 모두 덮어씀

- 로컬에서 추가 : 모두 놔둠

즉, `<commit>`이랑 비교할 때 index나 worktree에 추가된 건 모두 놔둔다.

- 로컬에서 삭제 : 각각 생각해봐야 함

DelIdx -> HEAD O, index X, worktree X

`<commit>`에서 있으면, 그냥 갖고오는 파일인거니까 OOO로 덮어씀
없으면, XXX니까 그냥 없는 파일임

DelWorktree -> HEAD O, index O, worktree X

`<commit>`에서 있으면, 이것도 갖고오는 파일이니까 OOO로 덮어쓰면 됨
없으면, index에서 추가된거니까 '로컬에서 추가'니까 놔둠


**결론 : `<commit>`으로부터 갖고 오고 덮어쓴다. 추가된 건 모두 놔둔다.**

```
AddCommit, AddIdx, AddWorktree,
그리고 <commit>에 없을 때의 DelWorktree (XOX여서 AddIdx가 됨)
를 놔둔다.
```

포맷(`git checkout <commit> -- <file>`) 자체가 파일을 덮어쓰겠다는 의도이므로 Modify는 모두 말없이 적용함


#### `git checkout <commit>`

HEAD를 옮긴다.

즉, 그 커밋으로 이동하겠다는 의미가 된다.

따라서 직관적으로 생각해볼 때,

커밋 - 커밋의 비교 = HEAD와 `<commit>`의 비교 에서의 바뀐 건 모두 적용시킨다.

AddCommit, DelCommit, Modify 모두 덮어쓴다.

로컬에서 추가되거나 바뀐 건 어떻게 되야할까?

```
AddIdx, AddWorktree, ModIdx, ModWorktree
```

의도적으로 로컬을 덮어쓰려는 게 아니다.

따라서 Modify는 덮어씌우지 않는 게 맞다. 그래서 checkout 자체가 막히고 Abort라고 뜨게 된다.

추가된 건 그냥 놔두면 될 것 같다.

1) `git checkout HEAD`

아무것도 안 건드린다.

2) `git checkout <tree-ish>`

Modify가 있을 시 Abort 됨.

로컬에서 추가된 건 놔둠.(AddIdx, AddWorktree)

나머지는 모두 적용(DelIdx, DelWorktree, Modify(커밋-커밋))

**결론 : Modify 있으면 Abort, 로컬 추가는 놔둠**

#### `git reset --hard <commit>`

1) `git reset --hard (HEAD)`

HEAD는 기본값이므로 생략 가능

로컬 삭제(DelIdx, DelWorktree)는 날아가도 상관없다. 날아간다. = 적용됨

로컬 변경(ModIdx, ModWorktree)는 HEAD로부터 덮어쓴다. = 적용됨

로컬 추가(AddIdx, AddWorktree)는 위에선 다 놔뒀는데, 여기선 날린다.

그런데 AddWorktree는 Untracked여서 애초에 건드리질 않는다.

AddIdx를 날린다. (HEAD에서 덮어씀)

정리하면 **Untracked 빼고 모두 적용시킴**

2) `git reset --hard <commit>`

HEAD(가 가리키는 브랜치)를 이동시킨다.

커밋-커밋간 비교 3개(AddCommit, DelCommit, Modify)에 대해 생각해보자.

이동하겠다는 의도이므로 모두 적용시키는 게 직관적으로 맞고 실제로 그렇다.

로컬은 어떻게 될까?

Del은 어차피 날려도 상관없다.

Modify : HEAD를 이동하는 다른 종류의 명령어인 `checkout <branch>`에서는 하나라도 있으면 Abort 시키지만, 그냥 덮어씀

Add : local changes니까 지켜주는 게 맞아보이지만 Untracked(AddWorktree) 빼고 다 날린다.

즉, AddIdx를 날린다.

**결론 : 말 없이 Modify 덮어씀, Untracked 빼고 모두 적용**


#### `git restore --source <tree-ish> --staged --worktree <file>`

HEAD를 안 옮긴다.

`git reset --hard <commit>`과 똑같다.

**결론 : 말 없이 Modify 덮어씀, Untracked 빼고 모두 적용**

### local changes 정리

정리하면

checkout, switch : 이동하겠다는 의도

reset, restore : 되돌리겠다는 의도

이 의도에 따라 Modify를 어떻게 대하느냐가 달림

되돌리기 명령어들은 덮어씀, 이동 명령어들은 Abort

그래서 `checkout <commit> -- <file>`에서도 `<commit>`에 없는 건 모두 놔둬줌, HEAD에 생긴거든 로컬에 생긴거든.

`-- <file>` 형태의 명령어들은 의도적으로 파일을 덮어쓰겠다는 것이므로 Modify를 말 없이 덮어써도 자연스럽다.

```
1. git checkout <tree-ish> -- <file>
<tree-ish>에 없는 건 모두 놔둠, Modify 덮어씀

2. git checkout <commit>
로컬 추가는 놔둠, Modify 있으면 Abort

3. git switch <branch>
2와 같다.

4. git reset --hard <commit>
Untracked 빼고 다 적용,
HEAD(가 가리키는 브랜치)를 이동하지만 Modify를 말없이 덮어씀

5. git restore --source <tree-ish> --staged --worktree <file>
Untracked 빼고 다 적용, Modify 덮어씀
```
로컬 추가 = AddIdx, AddWorktree
`<tree-ish>`에 없는 건 모두 놔둠 = 로컬 추가 + AddCommit을 놔둠
Untracked 빼고 적용 = 커밋-커밋(AddCommit, DelCommit, Modify) 모두 적용, AddIdx도 날림



### `git switch`

#### 1. 

정리하는중...

git stash도 정리할 예정?

### HEAD의 이동에 대한 정리

#### 1. `git reset <mode> <commit>`

**HEAD가 가리키는 브랜치**를 이동시킨다.

따라서 자연스럽게 HEAD도 따라가게 됨

detached HEAD면 HEAD를 이동

#### 2. `git checkout <branch>`

**HEAD를 이동**한다.

브랜치 주면 브랜치를 가리키게 이동,

커밋을 주면 커밋으로 HEAD를 이동 (detached HEAD로 만듦)

브랜치 이름 주면서 `git checkout --detach 브랜치이름` 이런 식으로 하면 브랜치를 안가리키고 그 커밋을 가리킴

#### 3. `git switch <branch>`

`git checkout <branch>`와 같다.

`--detach`도 똑같이 가능함

## 기능으로 분류

괄호 = 기본 값 (생략 가능)

`.` 대신에 파일 이름을 줄 수 있다.

`HEAD` 대신에 커밋, 브랜치 등을 줄 수 있다.

`--`는 이 뒤에는 파일 이름들만 온다는 뜻

### 1) index -> working tree
```
git restore .
```
```
git checkout -- .
```

### 2) HEAD -> index
```
git restore --staged .
```
```
git reset (--mixed) (HEAD)
```
```
git reset (HEAD) -- .
```

### 3) HEAD -> index, working tree
```
git restore --source HEAD --staged --worktree .
```
```
git reset --hard (HEAD)
```
```
git checkout HEAD -- .
```

여기서 커밋되지 않은 추가 파일들을 모두 놔두는 건 `git checkout HEAD -- .` 뿐이다.

나머지 둘은 Untracked는 안건들지만 stage 된 건 날린다.

파일 내용이 바뀐 건 셋 다 모두 덮어씌운다.

### 4) HEAD -> working tree
```
git restore --source HEAD .
```