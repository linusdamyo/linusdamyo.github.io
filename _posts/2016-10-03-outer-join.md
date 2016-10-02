---
layout: post
title:  "잘못알고있던 OUTER JOIN"
categories: database
---

그동안 outer join 구문, left join 혹은 right join, 을 쓰면서 잘못 생각했던 부분이 있었다.
우선 sample table 2개를 만들어보면..

> | table A |
> |:-----|:-:|:-----|:-:|
> | id   | 1 | name | 1 |
> | id   | 2 | name | 2 |

> | table B |
> |:-----|:-:|:-----|:-:|
> | id   | 1 | name | 1 |
> | id   | 3 | name | 3 |

이 경우, 그냥 A join B를 하면, id=1 만 나올것이고, A left join B를 하면, id=1 / id=2 에 B의 id=1 이 붙을 것이고, A right join B를 하면, id=1 / id=3 에 A의 id=1 이 붙을 거고..

여기까진 당연한 얘기인데.. 기준 테이블보다 많은 row를 가진 경우에..

> | table A |
> |:-----|:-:|:-----|:-:|
> | id   | 1 | name | 1 |
> | id   | 2 | name | 2 |

> | table B |
> |:-----|:-:|:-----|:---:|
> | id   | 1 | name | 1-1 |
> | id   | 1 | name | 1-2 |
> | id   | 1 | name | 1-3 |
> | id   | 2 | name | 2   |

내가 그동안 생각한 A left join B 의 결과는..

> | A left join B |
> | A.id | 1 | A.name | 1 | B.id | 1    | B.name | 1-1  |
> | A.id | 2 | A.name | 2 | B.id | NULL | B.name | NULL |

이렇게 생각했었는데.. 결과는 아래처럼 여러개의 row 가 모두 나온다.

> | A left join B |
> | A.id | 1 | A.name | 1 | B.id | 1    | B.name | 1-1  |
> | A.id | 1 | A.name | 1 | B.id | 1    | B.name | 1-2  |
> | A.id | 1 | A.name | 1 | B.id | 1    | B.name | 1-3  |
> | A.id | 2 | A.name | 2 | B.id | NULL | B.name | NULL |

그동안 left join 구문을 쓸때 보통 겹치는 경우가 없어서 완전 잘못 알고 있었네;;
