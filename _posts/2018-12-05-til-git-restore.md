---
layout: post
title:  "TIL: git commit을 날렸을 때, git reflog 로 복구하기"
categories: git
---

* 얼마전에 git stash 에 넣었던 하루치 코드를 날렸다.
* [구글 검색에서 찾은 아주 고마운 링크](https://hashcode.co.kr/questions/4650/git%EC%97%90%EC%84%9C-stash-%EC%8B%A4%EC%88%98%EB%A1%9C-%EC%A7%80%EC%9B%A0%EC%9D%84-%EB%95%8C-%EB%B3%B5%EA%B5%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
```
git fsck --no-reflog | awk '/dangling commit/ {print $3}'
```

* 오늘은 commit을 해놓고, reset 으로 이전 커밋으로 돌아갔다가, 다시 돌아오려고 하니, log 에서 보이지 않았다.    
  당황하지 않고, 구글링을 해보니, 간단한 명령어가 나왔다.
```
git reflog
```

* reset 직전 커밋이 있었고, `git reset --hard COMMIT_ID` 로 복구할 수 있었다.

* 예전에도 몇번 작업내용 날린 적 있었는데.. 그땐 그냥 다시했는데.. 구글에 찾아볼 걸 그랬다.

* 명령어와 기록을 남겨주신 모든 분들께 감사합니다. SO 와 구글에게도 항상 감사합니다.