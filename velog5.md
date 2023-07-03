# 기타 git 명령어 정리(log, revisions)

## `git log`

우선 간단한 표현법부터 정리해보자.

```
$ git log foo bar ^baz
```

이 명령어는 `foo`나 `bar`에서 접근 가능하지만, `baz`에서는 접근 불가능한 모든 커밋을 나타내라는 의미이다.

`<commit1>..<commit2>`는 `^<commit1> <commit2>`와 같다.

```
$ git log origin..HEAD
$ git log HEAD ^origin
```

이 두 명령어는 같은 걸 보여준다.

의미를 생각해보자.

`<commit1>..<commit2>` 형태일 때, `<commit1>`이 `<commit2>`의 조상(ancestor)이면,

조상에선 안되고 자손에서는 접근되는 커밋들은 결국 그 사이에 있는 커밋들이다.

그래서 `origin..HEAD`는 `origin`의 조상이면서 `HEAD`의 자손인 커밋들을 보여준다.

`<commit1>..<commit2>` 표현의 의미에 대해 생각해보자.

결국 두 커밋 사이의 커밋들을 `<commit1> < 커밋들 <= <commit2>` 이런 식으로 보여주려는거니까

앞쪽에서 캐럿(`^`)이 들어가는 게 맞는거다.

만약에 `HEAD..origin` 이런 식으로 주면 당연히 값이 없는거고,

첫 번째 커밋이 조상이 아니면 그냥 두 번째 커밋에서 접근가능한 것들이 나오거나 할 것이다.

## revisions