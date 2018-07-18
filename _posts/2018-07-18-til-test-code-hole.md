---
layout: post
title:  "TIL: 테스트 코드 구멍으로 발생한 문제"
categories: tdd
---

Test First는 못하고 있지만(역량의 문제로 TDD는 비효율적이라), 가능한 주요 로직에는 테스트 코드를 짜고 있습니다.  
그럼에도 불구하고 당연히 빠지는 케이스들이 많이 발생하고 있고, 그 부분은 충분히 인지하고 있었는데, 오늘은 테스트 코드도 있었고, 테스트도 통과했지만, 오류가 발생한 상황입니다.  

오류가 발생한 부분은 회원가입 부분이고, 간단한 테스트 코드가 작성되어 있습니다.  
회원가입 API를 호출하고, user 테이블을 조회해서, 몇몇 값들(이름/휴대폰번호 등)을 테스트 데이터와 비교하는 단순한 코드입니다.  
문제가 된 부분은 가입일인데, 가입일은 user 테이블에 reg_date라는 필드로 되어있고, typeorm의 `@CreateDateColumn` 기능으로 정의되어 있었습니다.

### 왜 문제가 발생했는가?
며칠전 기존에 사용하던 typeorm 0.1.12 버전을 0.2.7로 업데이트 했었고, 테스트 코드 오류도 고친 상황이었습니다.  
하지만, typeorm의 `@CreateDateColumn` 기능에 변경이 있었는데, 기존 버전에서는 INSERT 시에 `실제 시간값`을 넣어주던 반면에,
업데이트된 버전에서는 `DEFAULT` 구문으로 변경되어 있었습니다.  
```
변경 전
INSERT INTO user(name, reg_date) VALUES ('아이유', '2018-05-16 xxxxxx');
변경 후
INSERT INTO user(name, reg_date) VALUES ('아이유', DEFAULT);
```
하지만, user 테이블에 reg_date 필드의 DEFAULT 값이 제대로 정의되어 있었다면, 변경에 관련없이 문제가 발생하지 않았을 겁니다.  
문제는 user 테이블에 DEFAULT 값이 설정되어 있지 않았고, 따라서 null로 설정되고 있었습니다.  
대략 아래와 같은 구조입니다.  
```
CREATE TABLE user (
  id int unsigned NOT NULL auto_increment,
  name varchar(32) NOT NULL,
  reg_date datetime,
  PRIMARY KEY (id)
);

ALTER TABLE reg_date MODIFY datetime NOT NULL DEFAULT CURRENT_TIMESTAMP;
```

### 어떻게 막을 수 있었을까?
테스트 코드를 좀 더 촘촘히 구성했으면, typeorm 업데이트시에 바로 발견할 수 있었습니다.  
user 테이블을 조회한 후에, reg_date 값이 null 이 아닌지 체크하는 코드만 있었어도 피할 수 있는 문제였습니다.  
```
expect(user.regDate).to.not.be.null;
```
위와 같은 코드입니다.

네.. 같은 상황을 막기 위해서, 위의 줄을 추가해뒀습니다.  
그리고 user 테이블에 DEFAULT 값을 설정했습니다.

### 해결
기존에 가입일이 들어가지 않은 사용자들은 일단 다른 테이블의 정보로 유추해서 업데이트를 해주고,  
코드상에 수정이 필요하진 않아서 user 테이블 변경만으로 처리된 상황입니다만, 그래도 미리 알고 막을 수 있었으면 더 좋았을텐데 아쉬움이 남습니다.
