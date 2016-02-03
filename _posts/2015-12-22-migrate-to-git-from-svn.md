---
layout: post
title:  "SVN에서 GIT으로 옮기기"
categories: git
---

## 들어가며

몇년동안 마음만 먹었던 git으로의 이주를 실행했다. svn을 사용하기 시작한건 2009년 4월부터인데, git을 알게 되고 관심을 갖게 된건 아마도 github 에 가입한 2012년 7월이었던 것 같다. 3년동안이나 실행에 옮기지 못한 이유는 아마 게으름때문이겠고, svn도 제대로 활용을 못하는데 git까지 쓸 필요가 있을까란 생각도 조금 있었다. 그래도 관련 자료를 이것저것 모으고 읽고 조금 써보고 하다가 어렴풋이 이해가 될듯한 지점이 왔고, ProGit을 찬찬히 읽기 시작했다. 읽다보니 svn에서 git으로 옮기는 챕터가 나왔고, 그래서 git으로의 이주를 결행했다.

## GIT으로 옮기기

#### 실행환경
> - CentOS 6.7 64bit
> - git 1.7.1
> - svn 1.6.11 (r934486)

svn에서 git으로 옮기기는 생각보다 간단했다. git svn 이란 툴이 있어서 손쉽게 옮길 수 있다. git svn 명령이 실행되지 않으면 `yum install git-svn`으로 간단히 설치할 수 있다.

ProGit의 [http://git-scm.com/book/ko/v2/Git으로-이전하기-Git:-범용-Client](http://git-scm.com/book/ko/v2/Git으로-이전하기-Git:-범용-Client) 과 [http://git-scm.com/book/ko/v2/Git으로-이전하기-Git으로-옮기기](http://git-scm.com/book/ko/v2/Git으로-이전하기-Git으로-옮기기) 에 자세히 설명되어 있는데, 요약하자면, 

1. svnadmin 으로 로컬 서버에 svn 저장소를 만들고, svnsync 로 리모트 서버의 svn 저장소를 가져온다.
1. git init 후, git svn clone 명령어로 git으로 옮긴다. svn 의 revision 정보가 필요해서, --no-metadata 옵션은 적용하지 않았다.
 git 으로 가져오기 싫은 디렉토리나 파일은 --ignore-path 옵션으로 빼버릴 수 있다.
```
 git svn clone file:///home/svn --authors-file=users.txt -s my_project --ignore-paths='A|B'
```

1. ProGit 내용과 다른 부분은 svn branch 들이 .git/refs/remotes/ 아래 생기지 않고, .git/packed-refs 파일에 저장되어 있었다.
 파일 안의 내용을 참고해서.git/refs/heads 에 그냥 각각 파일을 만들고, .git/packed-refs를 삭제했더니 똑같이 됐다. 그냥 둬도 무방한 것 같기도 하다.
1. 그 다음, .gitignore 파일을 모든 commit 에 적용하고 싶었는데, [http://stackoverflow.com/questions/7295511/how-to-add-a-file-to-the-first-commit-on-a-git-repo](http://stackoverflow.com/questions/7295511/how-to-add-a-file-to-the-first-commit-on-a-git-repo) 에서 해결. 
1. commit log 에 svn revision 정보가 나오는건 좋은데, 좀 장황해서, 위에서 사용한 git filter-branch 명령으로 간단하게 수정했다.
```
git filter-branch --msg-filter '
sed "s#git-svn-id: file:///home/src/trunk@#r#g"
'
```

1. git remote 저장소를 추가하고 push까지.. 허무하리만큼 간단하다.

## SVN과의 공존

git으로 옮기기는 했지만, 다른 팀원들은 svn 을 계속 사용하고 있으므로, git 과 svn 의 동시 운용이 필요하다. ProGit에 나온 git svn rebase 명령어를 사용해봤는데, 무슨 문제인지 로컬로 가져온 svn 저장소에서 svnsync 명령어가 fail이 났다. git으로 치자면, 로컬 commit은 됐는데, remote push 실패? 아무튼 잘 안되서 결국 같은  commit을 두번씩 하고있다.

대략의 작업흐름은, 다른 팀원 작업내용을 svn에서 가져와 master에 반영하고, 내가 작업한 내용을 master에 merge. 그리고 svn commit. 그리고 git 리모트 저장소에 push, 이 정도다. 같은 내용을 두번씩 기록해야 하지만, 익숙해지니 할만한 것 같다.

{% highlight bash %}
svn up # 작업내용을 가져온다.
git checkout master; git add .; git commit; # master에 반영
git checkout my-branch; git merge; # 내가 작업한 내용 merge
svn add *; svn commit; # svn에 반영
git commit --amend # svn revision 정보 추가
git push --all # remote 저장소로 push
{% endhighlight %}

## 결말

git으로 옮기고나니 소스관리가 훨씬 수월해졌다. 이 좋은걸 이제껏 안썼다니.. 가장 좋은건 역시 branch. svn 쓸 때는 특정 revision을 별도 디렉토리에 checkout하고 수정하고 했었는데.. git branch는 그 과정이 자연스럽게, 그리고 손쉽게 해결된다. 물론 svn에서도 branch를 만드는게 정석이겠으나, 이게 꽤나 번거로워서 별도로 checkout하는 경우가 많았다. 그리고 svn commit은 쉽게 하기 힘들어서 로컬에서 작업한 내용의 history 관리가 좀 불편했는데, git은 일단 로컬로 commit이 되니, 부담없이 쓸 수 있었다. 여러 가지 작업을 할 때도 branch 만들고 각각 적용할 수 있으니 편하고, commit 수정하기도 편하다. 그 외에도 수많은 장점들이 정말 많다. 언젠가 다른 팀원들도 모두 옮겨오면, 정말 편할 것 같지만, 그 날이 과연 올지...

## GIT 명령어들 

마지막으로 자주 사용하는 git 명령어들을 정리해봤다.

- git add NEW_FILE
>> 파일을 stage 상태로 만든다.

- git commit
>> 당연히 commit

- git commit --amend
>> commit 을 했는데, 빼먹은게 있는 경우, git add 하고 git commit --amend 하면 기존 commit과 합쳐준다. 혹은 comment를 고칠때..

- git commit --amend --author "YOU \<YOU&#64;email.com\>"
>> commit 유저명을 바꾼다. 다른 사람이 svn 으로 업데이트한 내용을 받아와서, git 에 적용.

- git checkout -b NEW_BRANCH
>> branch를 생성하고, 그 branch를 checkout 한다.

- git checkout -b NEW_BRANCH remotes/origin/NEW_BRANCH
>> remote branch 를 local 로 checkout

- git merge BRANCH
>> branch 에서 작업한 내용을 master 에 반영할때. git checkout master 먼저 하고.

- git cherry-pick COMMIT-ID
>> 특정 commit 만 골라서 반영해야 할 경우.

- git reset HEAD~1
>> merge를 잘못했거나 했을때, commit 을 되돌린다. stage 상태의 파일도 원복한다.

- git status
>> 현재 상태 보기

- git diff
>> 전체 파일 diff

- git diff --cached
>> add 한 파일만 diff 할때 사용. 현재 수정중인 파일은 제외하고 보여준다.

- git show COMMIT-ID
>> commit 된 내용 확인

- git branch -a
>> 모든 branch 리스트를 본다.

- git push origin --all
>> remote 서버에 push

- git push -f origin
>> rebase 하고 에러났을때, 강제 push. 혼자 쓸때만 사용하자.

- git log --all --grep='trunk@1676'
>> commit 메시지에서 특정 문자 검색

- git remote add origin root@remote:/home/git/project.git
>> remote 서버 추가

- git init --bare
>> remote 서버에 git 저장소를 만든다.
>> mkdir /home/git/project.git; git init --bare

- git reset --hard HEAD , git pull
>> remote 에서 다시 받아서 덮어쓰기. 로컬 작업내용은 날아가므로 주의.

- git stash
>> 작업중인 내용을 임시저장.

- git stash list
>> stash 한 리스트

- git stash apply --index
>> stash 한걸 적용. commit 상태까지.

- git stash drop stash{0}
>> stash 한걸 지운다.

- git filter-branch
>> commit log 의 전체 변경이 필요할때.. 아래는 전체 유저명 변경 예제
  {% highlight bash %}
  git filter-branch -f --commit-filter '
        if [ "$GIT_AUTHOR_NAME" = "OLD" ];
        then
                GIT_AUTHOR_NAME="NEW";
                GIT_AUTHOR_EMAIL="NEW@EMAIL.com";
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD
  {% endhighlight %}

- git update-ref -d refs/original/refs/heads/master
>> filter-branch 실행후 백업된 항목을 지운다.

- git log --graph --all --decorate --color
> 예쁘게 나온다. 한줄로 보고싶으면 --oneline 추가.
