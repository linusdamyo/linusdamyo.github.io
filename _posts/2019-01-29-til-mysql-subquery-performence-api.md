---
layout: post
title:  "TIL : MySQL 서브쿼리 성능"
categories: database
---

### 문제
- 사용자 목록 조회 화면에, 로그인 로그에서 가져와야할 데이터가 있었습니다.
- typeorm 의 `createQueryBuilder().getMany()` 메서드로 사용자 목록을 가지고 오고 있었는데,   
  사용자 목록을 가져온 후, 각 사용자별로 로그인 로그 테이블에서 다시 조회를 하는 형태로 작업했습니다.
- 배포이후, 속도가 현저히 느려지는 현상이 발생했습니다.

### 추정
- 막연히 로그인 로그 테이블의 10만건 정도의 데이터가 문제라고 생각했습니다.
- 고작 10만건으로? 라는 의문이 좀 있었지만, 그냥 그럴 수도 있다고 생각했습니다.

### 해결방안 1
- 사용자 테이블에 조회해야할 정보를 미리 넣어두면 될 거라 생각해서, 컬럼을 새로 추가할 생각이었습니다.

### 해결방안 2
- 팀 동료분이 서브쿼리로 조회를 해보시고, 서브쿼리로는 속도가 느리지 않다고 알려주셨습니다.
- 막연하게, 로그 테이블과 조인이나 서브쿼리로 가져오면 문제가 될 거라고 생각했었는데,   
  실제로는 그렇지 않았습니다.
- 서브쿼리를 고려하지 않았던 이유 중 하나는, `getMany()` 메서드 때문이었습니다.   
  `UserEntity[]`를 리턴하고 있었는데, 서브쿼리 결과를 끼워넣기가 애매해서 처음 방식대로 만들었습니다.
- 하지만, `getRawAndEntities()` 메서드가 있었습니다.   
  아니면, `getRawMany()` 메서드와, `getMany()` 메서드를 엮어서 사용할 수 도 있었고요.

### 결론 
- 기존에 사용하던 `UserEntity[]` 는 `getMany()` 메서드의 리턴값을 사용하고,   
  서브쿼리로 가져온 항목들은 `getRawMany()` 메서드의 리턴값을 사용하는 것으로 해결하였습니다.   
  `getRawAndEntities()` 메서드를 사용하지 않은 이유는 `getCount()` 도 필요했기 때문이었고요.
  원하는 결과를 얻을 수 있었습니다.

### 배운 것
- 결국 추정만 하고, 실제로 어떤 결과를 얻으려 하지 않았던 점이 문제였습니다.   
  조인을 해보거나, 서브쿼리를 써보거나, 사용자별 데이터를 조회할때의 시간을 측정해보거나,  
  어느 것 하나 하지않고 추정만으로 일을 진행하고 있었습니다.
- MySQL 서브쿼리 성능이.. 생각보다 나쁘지 않다는 것도 알게되었습니다.
- 자주 사용하는 라이브러리의 API 는 틈날때마다 열심히 봐야겠습니다.
