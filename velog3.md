# git 명령어 정리 (merge, rebase, cherry-pick)

### `git merge`

다른 브랜치를 현재 브랜치로 합친다.

1. Fast Forward Merge
2. True Merge

명령어 종류

1. `git merge <option> <commit>...`
2. `git merge (--continue | --abort | --quit)`

1이 시작할 때 쓰는 명령어고, ..~~

#### 1. Fast Forward Merge

현재 브랜치(`HEAD`)가 주어진 `<commit>`의 조상일 때.

HEAD가 그 커밋으로 이동되고 종료된다.

`--no-ff`를 줄 경우 fast forward에도 커밋을 만들겠다는 옵션이므로 커밋이 생긴다.

보통 `git pull`할 때

#### 2. True Merge

일반적인 상황으로 충돌이 발생하는 경우이다.

자세히 알아보기 위해 우선 예시를 만들어보자.

두 커밋 사이에는 Add, Modify, Del이 있을 수 있다.

`common -> topic`

`common -> main(HEAD)`

```
common]
DelMain
DelTopic
Modify (common ancestor)

topic]
AddTopic
DelMain
Modify (topic)

main]
AddMain
DelTopic
Modify (main)
``` 

일단 로컬의 변화들에 대해서는, 하나라도 있으면 Abort 되는듯? 확인 필요

```
$ git merge topic
```

돌리면

```
A  AddTopic
D  DelTopic
UU Modify

AddMain AddTopic Modify
```

결론

둘 중 한 쪽이라도 있으면 없앰
둘 중 한 쪽이라도 추가됐으면 냅둠
바뀐 건 Unmerged (conflict)

merge를 할 때

```
common ancestor
HEAD (ORIG_HEAD)
MERGE_HEAD (합칠려는 브랜치)
```

를 비교한다고 했다.

명령어는 한 브랜치에서 `$ git merge <다른 브랜치>` 하지만,

사실 목적은 두 브랜치를 합치는거다. 변화들을 합치는거다.

언제부터? common ancestor로부터.

(CA, Topic, Main 모두가 서로를 비교할 때 추가, 삭제, 변화가 있다는 그림)

CA(common ancestor)로부터의 변화를 합친다.

AddTopic, AddMain은 CA기준 추가
DelTopic, DelMain은 CA기준 삭제
이걸 모두 적용하여 합치면 Add만 둘다 남고 Del은 둘다 지워지는거다.

그런데 Modify는 두 브랜치에서 모두 바뀌었을 때 UU 뜨면서 <<< === >>> 이게 나오는데, 한 쪽만 바뀌면 어떨까?

CA 기준 변화를 합친다 라는 관점으로 보면 당연히 바뀐 쪽이 적용되게 된다.

### `git rebase`

1. `git rebase <options> [<upstream> [<branch>]]`
2. `git rebase <options> [--root [<branch>]]`
3. `git rebase (--continue | --skip | --abort | --quit | --edit-todo)`

#### 1. `git rebase <options> [<upstream> [<branch>]]`



#### 2. `git rebase <options> [--root [<branch>]]`



#### 3. `git rebase (--continue | --skip | --abort | --quit | --edit-todo)`




### `git cherry-pick`

