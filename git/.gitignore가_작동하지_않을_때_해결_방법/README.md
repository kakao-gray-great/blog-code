## 1. 문제 상황

commit을 해야하는데 .gitignore가 제대로 작동되지 않아서 애를 먹은 적이 여러 번 있습니다.

분명 .gitignore에 ignore할 디렉터리, 파일을 작성하였는데 changes에 올라와 있는 경우가 있습니다.

## 2. 해결 방법

git의 캐시가 문제의 원인이기 때문에 git chache를 전부 삭제 후 커밋하면 됩니다.

```
git rm -r --cached .
git add .
git commit -m "cache removed"
```

위 명령을 입력하였으나 캐시가 지워지지 않고 -f 옵션을 요구하는 경우가 있습니다.

이럴 땐 -f 옵션을 사용하시고 모든 파일을 add 해주시면 됩니다.

```
git rm -r -f --cached .
git add .
```

## 참고

* [stackoverflow](https://stackoverflow.com/questions/11451535/gitignore-is-ignored-by-git)