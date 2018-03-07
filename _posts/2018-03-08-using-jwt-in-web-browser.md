---
layout: post
title:  "웹브라우저에서 JWT 사용"
categories: javascript
---

### JWT

> JWT의 기본적인 설명과 장점, 한계점 등은 [Outsider님의 블로그 글](https://blog.outsider.ne.kr/1160)에 잘 설명되어 있습니다. 댓글에 있는 내용도 꼭 살펴보시길 권해드립니다.

### App에서 JWT 사용

> App에서 사용할때는 비교적 간단합니다. 로그인시 JWT를 발급하고, App은 KeyChain 같은 곳에 JWT를 안전하게 보관합니다.  
이후, API 요청시 Authrization Header 에 JWT를 설정하고, API 서버는 Header의 JWT를 검증하면 됩니다.

### 웹브라우저에서 JWT 사용

> 웹브라우저에서 JWT를 사용하는 경우는 고려할 부분이 좀 더 많습니다. App에서 사용할 수 있는 KeyChain같은 것이 없기때문에, JWT를 어디에 보관해야 하는가? 가 중요한 이슈가 됩니다.  

* http://lazyhoneyant.tistory.com/7
* https://medium.com/@OutOfBedlam/jwt-%EC%9E%90%EB%B0%94-%EA%B0%80%EC%9D%B4%EB%93%9C-53ccd7b2ba10
* https://swalloow.github.io/implement-jwt
* https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage

1. JWT를 Web Storage에 보관하는 경우, XSS 공격으로 JWT가 유출될 수 있습니다.
1. 대안으로, Cookie 보관하는 경우, (http only 옵션과 secure 옵션 사용은 필수), CSRF 공격에 대비해야 합니다.
1. 웹프레임워크의 템플릿 구조를 사용해서 렌더링 결과를 보내는 경우, 사용자 입력이 필요한 부분에 CSRF 토큰을 포함하는 기능이 제공됩니다.
1. 하지만, API 서버가 별도로 분리되어 있는 경우, CSRF 방어를 별도로 해줄 필요가 있습니다.

### 결론

> 결국 아래 URL에서 제시한 방식으로 구현했습니다.

* https://stackoverflow.com/questions/27067251/where-to-store-jwt-in-browser-how-to-protect-against-csrf

1. Cookie 에 http only, secure 옵션을 주고, JWT를 발급합니다.
1. 이때, CSRF 토큰도 생성하고, JWT 안에 CSRF 토큰을 넣어둡니다.
1. 로그인에 대한 응답으로 CSRF 토큰을 보냅니다.
1. 브라우저에서는 응답으로 받은 CSRF 토큰을 임시 보관하고, API 요청시 Header 에 넣어, (ex: X-CSRF-TOKEN) 요청을 보냅니다.
1. API 서버는 Cookie를 통해 받은 JWT와, Header로 받은 CSRF 토큰을 각각 검증합니다. CSRF 토큰은 JWT의 유효성만 검증하고, JWT로 일반적인 인증/권한 처리를 합니다.

* CSRF 토큰이 유출되는 경우는 어쩌나? 하는 의문이 들었는데, JWT를 보관하듯이 Web Storage나 Cookie를 사용하지 않고, 임시로만 가지고 있으면 크게 문제가 되지 않을 것 같습니다.  
* CSRF 토큰을 잃어버린 경우에는, 임시저장이므로 메모리에서 사라진 경우, JWT만 가지고 CSRF 토큰을 갱신해주는 처리를 하면 될 것 같습니다.  
이 부분에 큰 위험요소는 없다고 판단했습니다.

### 그 외에..

#### access/refresh

> JWT를 발급할때, 주기가 짧은 access token 과 주기가 좀 더 긴 refresh token 을 따로 발급해서 유출에 대한 피해를 최소화할 수 있는 방안에 대한 얘기도 많았습니다.

* https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/

> 하지만, 결국 refresh token을 안전한 곳에 보관해야 된다는 이슈는 마찬가지여서, 두개로 발급하는 구조는 제외했습니다.

#### revoke

> JWT의 단점은 파기가 불가능하다는 점인데, 이를 해결하려면 결국 JWT를 세션관리 하듯이 별도로 DB등으로 관리해야 합니다. 그럴거면 그냥 세션을 쓰지? 그래도 JWT안에 간단한 정보들을 넣어둘 수도 있고, 브라우저나 App에서 자체적으로 만료여부를 판별할 수도 있어서, 세션보다 장점이 많다고 생각합니다.