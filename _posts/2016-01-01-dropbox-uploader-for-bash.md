---
layout: post
title:  "LINUX 용 DROPBOX BASH SCRIPT"
categories: linux
---

vultr에서 작은 서버를 하나 운영하고 있는데, 소스야 github이나 bitbucket으로 관리하고 있으니 상관없으나, DB 백업이 문제였다.
마땅히 백업할 곳이 없어서 고민하다가, 언젠가 리눅스용 드롭박스 스크립트가 있단 글을 본 기억이 있어서 구글에 검색하니 쉽게 찾을 수 있었다.

> https://github.com/andreafabrizi/Dropbox-Uploader

이 곳인데, 업/다운로드, 리스트 등등 거의 모든 기능을 지원하는 것 같다.
설명도 잘 되어있고, 사용법도 쉬워서 금방 적응할 수 있었다.

근데, 문제가 한글로 된 폴더에 업로드하려니 에러. 리스트도 안되고.. 역시나 이슈가 등록되어 있었는데, 한글만 안되는게 아니고 유니코드 문제였다. 지금은 close 상태.

> https://github.com/andreafabrizi/Dropbox-Uploader/issues/139

중간에 보니, 업로드만 수정하신 분도 있었는데..

> https://github.com/andreafabrizi/Dropbox-Uploader/pull/202

urlencode 부분만 수정을 하셔서, 다운로드나 리스트 등에서는 여전히 문제가 있었다.
열심히 구글링하다가 아래에서 perl을 사용해서 해결하는 방법을 찾아서 적용했다.

> http://stackoverflow.com/questions/296536/how-to-urlencode-data-for-curl-command

리스트/다운로드 등에서 유니코드 hexa로 표시되는 부분도 perl을 사용해서 바꿔봤더니, 잘 동작한다.
아래는 수정된 코드의 링크이다.

> curl "https://raw.githubusercontent.com/linusdamyo/Dropbox-Uploader/fix-unicode/dropbox_uploader.sh" -o dropbox_uploader.sh

perl이 없는 환경에서는 쓸 수 없다는 게 문제인데.. 일반적으로 사용하는 리눅스나 맥에서는 문제가 없는 것 같다.

아, 그래서 DB 백업은, dump를 뜨고 dropbox_uploader.sh을 사용해서 업로드되도록 script를 만들고, cron에 등록했다.
그리고 perl도 공부해야 하는가 하는 생각이 들었는데.. 과연..
