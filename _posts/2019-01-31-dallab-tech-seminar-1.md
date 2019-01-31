---
layout: post
title:  "달랩 기술세미나 1 후기"
categories: seminar
---

주제는 레일즈와 뷰 시연회

* Rails
  - `gem install rails`
  - `rails new new-app`
  -  레일즈는 처음 접했는데, 엄청 편해보였다. 테스트코드 작성도 쉬운거 같고.. 뭔가 엄청 빠르게 만들 수 있을 것 같다.  
  - test, fixture 를 쉽게 다룰 수 있는 거 같다.   
  - 테스트가 돌때, transaction 을 이용해서 끝나면 rollback 시킨다고 하는데, 되게 좋은 방식인 거 같다.

* 되새길 것들
  - `모델 테스트는, 해당 모델을 생성하는 가이드가 될 수 있다. `
  - `확신을 얻고 싶은 만큼 테스트를 작성한다.`
  - 더 알아봐야할 것 : Active Record

* pow.cx
  - 이거도 엄청 편해보였다. 로컬 /etc/hosts 를 변경하는 듯?
  - `gem install powder`
  - `powder link|unlink`

* npx
  - global로 설치하면 관리가 귀찮기 때문에, npx 를 이용해서 실행하면, 따로 설치하지 않고 사용할 수 있다. 바로 실행.

* Vue.js
  - 테스트는 jest / 셀레늄 / headless chrome
  - 테스트 코드 작성을 안보여주셔서 아쉬웠다.
  - vue는 예전에 잠깐 해봤을때 꽤 맘에 들었었는데, 다시 봐도 뭔가 편하고 쉬운 느낌이 있다.

> 프론트 개발에 뭔가 힌트를 얻을 수 있지않을까 싶었는데, 오히려 레일즈가 더 인상적이었다. 엄청 쉬워보임. vue도 다시 한번 해볼까 싶다. 작은 toy app 하나 만들어 보고 싶다.