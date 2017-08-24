---
layout: post
title:  "Event bublling: div onClick 처리시 child a tag 처리"
categories: javascript
---

div로 row를 하나 만들고 click 이벤트를 처리해야 하는데,   
내부에 a tag를 넣어야 해서, 하고나니 링크를 누를때 div 의 onClick handler가 같이 호출된다.
```javascript
  <div id='row' onClick={(e) => handleRowClick}>
    <a href='#'>LINK</a>
  </div>
```

일단 검색을 해야겠는데, 도무지 키워드를 모르겠어서, 이상한모임에 질문.   
많은 분들이 힌트와 답을 주셨다!  

그래서 알게된 event bubbling.  
DOM event가 상위로 전파된다는 것도 처음 알았다.  
막는 방법도 간단하다. 해당 DOM onClick handler 에 stopPropagation() 만 넣어주면 끝.
```javascript
  <div id='row' onClick={(e) => handleRowClick}>
    <a href='#' onClick={(e) => e.stopPropagation()}>LINK</a>
  </div>
```

이상한모임 없었으면 어떻게 개발했을까 싶다. 감사!!
