---
layout: post
title:  "Docker로 Gitlab 설치하기"
categories: docker
---

## 설치환경
- CentOS 7

## 준비
- git, docker는 yum 으로 간단히 설치
- yum install git
- yum install docker
- service docker start

## 실패
- gitlab 설치를 위해 구글링 결과 여기저기 따라해봤지만 대부분 실패했다.
- 대부분 아래 url 에 있는 내용을 기반으로 설치하고 있었는데, 결과적으로 몇몇 옵션이 달라서 설치가 잘 안됐다.
- https://github.com/sameersbn/docker-gitlab

## 결론
- 아래 순서대로 설치하면 잘된다.
- 디비 패스워드, IP 주소, 키로 사용할 문자열 부분만 변경해주면 된다.
: {% highlight bash %}
docker run --name gitlab-mysql -d \
    --env 'DB_NAME=gitlabhq_production' \
    --env 'DB_USER=gitlab' --env 'DB_PASS=디비 패스워드' \
    --volume /srv/docker/gitlab/mysql:/var/lib/mysql \
    sameersbn/mysql:latest
docker run --name gitlab-redis -d \
    --volume /srv/docker/gitlab/redis:/var/lib/redis \
    sameersbn/redis:latest
docker run --name gitlab -d \
    --link gitlab-mysql:mysql --link gitlab-redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_HOST=사용할 도메인 or IP 주소' \
    --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=키로 사용할 문자열' \
    --env 'GITLAB_SECRETS_SECRET_KEY_BASE=키로 사용할 문자열' \
    --env 'GITLAB_SECRETS_OTP_KEY_BASE=키로 사용할 문자열' \
    --volume /srv/docker/gitlab/gitlab:/home/git/data \
    sameersbn/gitlab:latest
{% endhighlight %}
- 한가지 더 해줘야 할게
: {% highlight bash %}
chcon -Rt svirt_sandbox_file_t  /srv/docker/gitlab
{% endhighlight %}
- 이제 http://IP:10080 으로 접속하면 된다.

## 참고 URL
- https://p_yg7.gitbooks.io/today-i-learned/content/gitlab.html
- http://digndig.kr/gitlab/801/
- http://www.damagehead.com/docker-gitlab/
