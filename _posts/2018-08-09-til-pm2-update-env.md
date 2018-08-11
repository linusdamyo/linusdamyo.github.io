---
layout: post
title:  "TIL: pm2 --update-env 옵션 사용시 주의점"
categories: node
---


* pm2로 서버랑 배치잡들을 관리하고 있습니다.  
* 배치잡들은 auto-restart:false 로 실행 후에 stopped 상태로 남아있게 됩니다.  


* 배치잡중 하나에 임시로 args를 넣어야 해서 pm2.json에 넣고 실행하고, 끝난 후에 args 옵션을 제거했습니다.  
* pm2 start 명령에 `—update-env` 옵션을 주고 실행하고 있었기 때문에 별 문제없을 것으로 생각했으나...
> `—update-env`가 **제거된 옵션**의 경우 값을 갱신하지 않고 있었습니다.


* 해당 옵션을 `""`으로 초기화한 경우엔 갱신이 됩니다.  
* 옵션을 제거한 경우에는 pm2 delete를 통해 지워준 후 재실행해야 적용됩니다.