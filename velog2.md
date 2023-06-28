# git 명령어 정리 (stash, clean, reflog)

### `git stash`

index와 worktree를 저장해주는 기능이다.

다른 브랜치로 이동하거나 할 때 로컬에서 작업한 걸 한꺼번에 저장해두게 해준다.

그렇게 저장한 뒤 HEAD -> index, worktree로 모두 덮어쓴다.

1. `git stash list`
2. `git stash push <option> -- <pathspec>`
3. `git stash apply [--index] <stash>`
4. `git stash pop [--index] <stash>`
5. `git stash drop <stash>`
6. 다른 옵션들 (show, clear, ...)

- 참고 > [Git 도구 - Stashing과 Cleaning (git-scm)](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Stashing%EA%B3%BC-Cleaning)

#### 1. `git stash list`

현재 stash entry를 보여줌

```
$ git stash list
stash@{0}: WIP on main: c7e8283 2
stash@{1}: WIP on main: c7e8283 2
```

마지막에 생긴 게 0번에 들어가고 나머지의 번호가 밀려난다.

#### 2. `git stash push <option> -- <pathspec>`

index, working tree를 저장한다.

이전에 쓰이던 명령어로 `git stash save`가 있다고 한다.

똑같은 기능을 하는데, 차이점은 `push`는 각 파일에 대해 적용할 수 있게 `<pathspec>`을 줄 수 있게 되었다는 것이다.

```
A  AddIdx.txt
D  DelIdx.txt
 D DelWorktree.txt
MM Modify.txt
?? AddWorktree.txt
```

이 상태에다가, HEAD^와 HEAD 사이에 AddCommit, DelCommit, Modify도 있다고 치자.

즉, Modify는 HEAD^에서 1, HEAD에서 2, index에서 3, worktree에서 4.

- [egoing/gistory (github)](https://github.com/egoing/gistory)

이 프로그램은 생활코딩의 이고잉님이 만드신건데, index, stash, logs(reflog) 등에 어떤 내용이 들어있는지 볼 수 있게 해준다.

이걸로 `.git/refs/stash`에 어떤 내용이 들어있는지 분석하면서 정리해보자.

일단 stash가 만들어지면 `.git/refs/stash`에는 커밋을 가리키는 해시가 내용으로 저장된다.

즉 stash는 커밋 형식으로 저장된다.

---

push 명령어를 옵션 없이 하면

```
?? AddWorktree.txt

AddCommit.txt  AddWorktree.txt  DelIdx.txt  DelWorktree.txt  Modify.txt
```

즉 Untracked 빼고 모두 저장되고 HEAD로부터 덮어씌워진다. (`git reset --hard HEAD`와 마찬가지로 AddIdx까지 날림)

`.git/refs/stash`의 내용인 해시가 가리키는 커밋을 보면 내용이 이런 식으로 되어있다.

```
tree 130400b6e49e1b051e34479cf31bc1b242f617e6
parent c7e828300c8c94ab031b7820567d7f72a5d3e859
parent bec3c52e3761078711017ea8b153a77429cad214
author vhv3y8 <vhv3y8@gmail.com> 1687697153 +0900
committer vhv3y8 <vhv3y8@gmail.com> 1687697153 +0900

WIP on main: c7e8283 2
```

tree 1개와 parent 2개가 있다.

tree = working tree의 내용을 저장

첫 번째 parent = stash가 만들어질 때 속했던 커밋

두 번째 parent = index

index의 내용은 대충 이런 식으로 되어있다.

```
tree d9b93fb4bcb4827367c65c79b0c2f1154b7cd5b6
parent c7e828300c8c94ab031b7820567d7f72a5d3e859
author vhv3y8 <vhv3y8@gmail.com> 1687697153 +0900
committer vhv3y8 <vhv3y8@gmail.com> 1687697153 +0900

index on main: c7e8283 2
```

tree의 내용

```
100644 blob 755d0092b230c95d537e19613d92eac246864841	AddCommit.txt
100644 blob 0010bbc76234a11b9e47869da01de2d3dd9a3ac9	AddIdx.txt
100644 blob e54a0d90bf7cc015e1615f309c602f24ff3446f4	DelWorktree.txt
100644 blob e440e5c842586965a7fb77deda2eca68612b1f53	Modify.txt
```

`-u`, `--include-untracked`를 하면

```

AddCommit.txt  DelIdx.txt  DelWorktree.txt  Modify.txt
```

이렇게 Untracked까지 모두 저장하게 된다.

`.git/refs/stash`에 적힌 커밋의 내용을 보면

```
tree 130400b6e49e1b051e34479cf31bc1b242f617e6
parent c7e828300c8c94ab031b7820567d7f72a5d3e859
parent 3401217643b024a31de2d4c55e4930926469cd52
parent e9736728d7511ed638a68a73b3c7a27eafdf7e55
author vhv3y8 <vhv3y8@gmail.com> 1687698308 +0900
committer vhv3y8 <vhv3y8@gmail.com> 1687698308 +0900

WIP on main: c7e8283 2
```

이런 식으로 마지막에 parent가 하나 더 추가되서 3개가 된다.

3번째 parent의 내용은 이렇다.

```
tree c10b02685bda93d6d8daf663a52d11134156875e
author vhv3y8 <vhv3y8@gmail.com> 1687698308 +0900
committer vhv3y8 <vhv3y8@gmail.com> 1687698308 +0900

untracked files on main: c7e8283 2
```

tree의 내용

```
100644 blob 5f727a1e500099001e1b4cb2ae689fddd3e35fff	AddWorktree.txt
```

`-k`, `--keep-index`를 하면

```
A  AddIdx.txt
D  DelIdx.txt
M  Modify.txt

AddCommit.txt  AddIdx.txt  DelWorktree.txt  Modify.txt
```

이런 식으로 index는 놔두고 저장하게 된다.

하지만 stash 내용을 보면 똑같이 두 번째 parent에 index는 기록되어있음

`-m`, `--message`로 `git commit -m "메시지"` 하듯이 메시지를 적을 수 있다.

정리하면

- 옵션 없음 : Untracked 빼고 모두 저장
- `-k`, `--keep-index` : index 놔두고 저장
- `-u`, `--include-untracked` : 모두 저장

stash 저장 내용을 정리하면

- tree : working tree 저장
- parent 1 : 자신이 기록됐던 커밋
- parent 2 : index 저장
- parent 3 (untracked 저장했을 때) : untracked만

#### 3. `git stash apply [--index] <stash>`

stash entry에 있는 걸 현재 index와 working tree에 적용한다.

옵션 안주면

```
A  AddIdx.txt
 D DelIdx.txt
 D DelWorktree.txt
 M Modify.txt
?? AddWorktree.txt

AddCommit.txt  AddIdx.txt  AddWorktree.txt  Modify.txt
```

Modify의 내용은 4. (HEAD는 2)

git status 기준에서 비어있던 상태에서 불러온 뒤에 모두 다 바뀐건데,

왜 AddIdx는 index에 올라갔는데 DelIdx는 안올라간걸까?

정리해보면 DelIdx나 Modify나 모두 index에 있는 건 안불러오고 **worktree만** 불러왔는데,

AddIdx는 index에 안올려두면 worktree에만 추가된, 즉 untracked랑 마찬가지가 되기 때문인 것 같다.

결론적으로 **가능한 최소의 조건으로 worktree만 업데이트했다** 라고 볼 수 있다.

`--index`를 주면

```
A  AddIdx.txt
D  DelIdx.txt
 D DelWorktree.txt
MM Modify.txt
?? AddWorktree.txt

AddCommit.txt  AddIdx.txt  AddWorktree.txt  Modify.txt
```

즉 원래 있던 그대로 돌아옴.

똑같은 stash에서 불러와도 이렇게 둘 다 가능한 이유는, 

앞서 파일 내용을 봤듯이 index와 worktree를 모두 커밋 형태로 해서 담아서 저장해뒀기 때문인듯하다.

정리하면 **index, worktree를 있는 그대로 모두 저장하고 불러오고 싶다면**

- `git stash push -u`
- `git stash apply --index <stash>`

이렇게 하면 push는 클린한 로컬로 만들면서 모두 저장하고,

apply(나 pop)은 원래 있던대로 그대로 불러와준다.

#### 4. `git stash pop [--index] <stash>`

`git stash apply`와 같은데, 적용한 stash를 버린다.

#### 5. `git stash drop <stash>`

주어진 stash를 버린다.

0번을 버리면, 전에 1번에 있던 게 0번으로 온다.

(번호를 0부터 순서대로 채움)

#### 6. 나머지

`git stash show [-u|--include-untracked|--only-untracked] <stash>`

stash와 자신이 만들어졌던 커밋의 비교를 보여줌

`git stash clear`

모든 stash를 버림


### `git clean`

1. `git clean <option> (-- <pathspec>)`

working tree의 Untracked 파일들을 지운다.

좁은 범위의 역할이고 옵션도 몇 개 없고 심플하다.

#### 1. `git clean <option> -- <pathspec>`

기본값, `-f`일 때

Untracked files를 지운다.

현재 폴더에 있는 것만 지운다.

config의 clean.requireForce가 true면 `-f`를 줘야 실제로 지움

`-d`를 주면

하위 디렉토리에도 적용한다.

파일이 싹 다 Untracked인 디렉토리에 대해서 그렇다.

하나라도 stage되있으면 그 폴더 파일들은 기본으로 clean 대상임

`-n`을 주면

dry run을 할 수 있다.

실제로 지우지 않고 뭘 지울지 미리 알려준다.

```
$ git clean -n
Would remove AddWorktree.txt
```

`-x`를 주면

ignore 파일들도 모두 지운다.

`-X`를 주면

ignore 파일들만 지운다.

`-e`라는 옵션도 있는데, 

이건 패턴으로 ignore할 파일들을 추가할 수 있다고 하는데 쓸 일이 잘 없을 것 같다.

`-x`나 `-X` 같은 옵션 쓸 때 제외할 파일들을 직접 지정할 수 있는 옵션이라고 하는듯하다.

#### `git clean` 예시

실제로 어떻게 돌아가는지 이해하기 위해 예시를 만들어보자.

```
.gitignore (내용 : Ignore.txt)
Ignore.txt
AddWorktree.txt
(DelWorktree.txt) (로컬에서 지워짐)
t1/
  added.txt
  test1.txt
  test2.txt

  tt1/
      added.txt
      ttest1.txt
      ttest2.txt
t2/
  (없음)
```

커밋은 하나 뿐이고 커밋에는 `.gitignore` 파일과 `DelWorktree.txt`를 추가하자.

나머지 파일은 모두 커밋 이후 로컬에서 추가해주고 `git status -s`를 돌리면

```
 D DelWorktree.txt
?? AddWorktree.txt
?? t1/
```

이런 식으로 된다.

t2는 폴더는 있지만 파일이 없어서 추가되지 않은듯하다.

이 상태에서 각 옵션들을 주면 어떻게 되는지 dry run(`-n`)으로 확인해보자.

기본값은 현재 폴더의 Untracked 파일만 지우므로, 돌려보면

```
$ git clean -n
Would remove AddWorktree.txt
```

이제 `-d` 옵션과 기본값에서 선택되는 파일들에 대해 알아보자.

```
$ git clean -dn
Would remove AddWorktree.txt
Would remove t1/
Would remove t2/
```

그런데 `git add t1/added.txt`를 하고나서 기본값을 돌려보면

```
$ git clean -n
Would remove AddWorktree.txt
Would remove t1/test1.txt
Would remove t1/test2.txt
```

즉 하나라도 stage 된 파일이 있으면 그 폴더의 파일들은 기본적으로 clean 대상이 된다.

`t1/added.txt` 말고 `t1/tt1/added.txt` 파일만 stage 해주고나서 돌려보면

```
$ git clean -n
Would remove AddWorktree.txt
Would remove t1/added.txt
Would remove t1/test1.txt
Would remove t1/test2.txt
Would remove t1/tt1/ttest1.txt
Would remove t1/tt1/ttest2.txt
```

즉 stage 된 파일이 속한 폴더와 상위 폴더까지 모두 적용한다는 걸 알 수 있다.

만약에 `t1/tt2` 폴더를 만들고 그 안에 `ttest1.txt` 같은 파일을 만들더라도 이건 기본 삭제대상은 아니게 된다.

`-d` 옵션을 주면 하위 폴더까지 싹 다 적용한다.

```
$ git clean -dn
Would remove AddWorktree.txt
Would remove t1/
Would remove t2/
```

빈 폴더를 포함해서 폴더들까지 싹 다 지우게 된다.

기본 값은 현재 폴더의 파일들을 지운다고 했었다.

그걸 확인하기 위해 이번에는 `t1` 폴더로 이동하고 돌려보자.

```
$ cd t1
$ git clean -n
Would remove ./
```

`-f`로 실제로 날려보면, 하위 폴더(`tt1`, `tt2`)까지 싹 다 날린다. (???)

이번엔 stage에 `t1/added.txt`만 추가해주고 돌려보자.

```
$ git add added.txt
```

```
$ git clean -n
Would remove test1.txt
Would remove test2.txt
```

```
$ git clean -dn
Would remove test1.txt
Would remove test2.txt
Would remove tt1/
Would remove tt2/
```

이런 식으로 된다.

`-x` 옵션을 주면 ignore 파일들까지 지운다.

```
$ git clean -xn
Would remove AddWorktree.txt
Would remove Ignore.txt
```

`-X` 옵션을 주면 ignore 파일들만 지운다.

```
$ git clean -Xn
Would remove Ignore.txt
```

실제로 지울 땐 `clean.requireForce`가 true면 `-f`도 같이 줘야한다.

ex) `git clean -fd` 등등

**결론 : `git clean` 하기 전엔 `-n` 옵션으로 어떤 게 지워질지 보고나서 돌리자.**

### `git reflog`

RefLog는 Reference Log의 약자로, 브랜치나 레퍼런스들이 업데이트 될 때 끝부분이 어디 있었는지 기록한다.

위에서 썼던 gistory로 파일들의 내용을 보면 쉽게 이해할 수 있다.

`.git/logs/HEAD`
```
0000000000000000000000000000000000000000 f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 vhv3y8 <vhv3y8@gmail.com> 1687259264 +0900	commit (initial): 1
f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 0000000000000000000000000000000000000000 vhv3y8 <vhv3y8@gmail.com> 1687259275 +0900	Branch: renamed refs/heads/master to refs/heads/main 
0000000000000000000000000000000000000000 f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 vhv3y8 <vhv3y8@gmail.com> 1687259275 +0900	Branch: renamed refs/heads/master to refs/heads/main 
f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 c7e828300c8c94ab031b7820567d7f72a5d3e859 vhv3y8 <vhv3y8@gmail.com> 1687259376 +0900	commit: 2
...(생략)
```

`.git/logs/refs/stash`
```
0000000000000000000000000000000000000000 6f12b30b94c8d93194c9d92d2a6007a741bdc055 vhv3y8 <vhv3y8@gmail.com> 1687700564 +0900	WIP on main: c7e8283 2
6f12b30b94c8d93194c9d92d2a6007a741bdc055 9f39d6442daa79023b99d8a4e4998ad74417ffb9 vhv3y8 <vhv3y8@gmail.com> 1687700576 +0900	WIP on main: c7e8283 2
```

`.git/logs/refs/heads/main`
```
0000000000000000000000000000000000000000 f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 vhv3y8 <vhv3y8@gmail.com> 1687259264 +0900	commit (initial): 1
f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 vhv3y8 <vhv3y8@gmail.com> 1687259275 +0900	Branch: renamed refs/heads/master to refs/heads/main 
f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 c7e828300c8c94ab031b7820567d7f72a5d3e859 vhv3y8 <vhv3y8@gmail.com> 1687259376 +0900	commit: 2
c7e828300c8c94ab031b7820567d7f72a5d3e859 f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 vhv3y8 <vhv3y8@gmail.com> 1687346595 +0900	reset: moving to first
f7d375d8ab97caae1b6f0b0e6d450b0cbfa15ef7 c7e828300c8c94ab031b7820567d7f72a5d3e859 vhv3y8 <vhv3y8@gmail.com> 1687346619 +0900	reset: moving to second
...(생략)
```

이런 식으로 `.git/logs` 폴더 안에 각각 파일로 브랜치, HEAD 등이 바뀔 때마다 시간, 메시지 등과 함께 기록을 해두고 있다.

`HEAD@{2}`, `master@{one.week.ago}` 이런 식으로 위치를 찾아볼 수 있다고 한다.

시간, 기록 등이 모두 텍스트 형태로 그대로 남아있기 때문에 가능한듯하다.

`reset`, `switch` 등의 명령어에서도 이런 표현들을 쓸 수 있다.

1. `git reflog show <ref>`
2. `git reflog expire <options>`
3. `git reflog delete`

