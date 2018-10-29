---
title: "Querydsl"
categories: 
  - JPA
tags: 
  - JPA
  - Querydsl
last_modified_at: 2018-10-29T14:30:00+09:00
toc: true
toc_sticky: true
---

## Intro

Spring Framework + Mybatis + H2DB 로 개발은 진행 중이였다.  
SI 업무상 여러 사이트에 투입되게 되니  
Default성 프레임워크를 만들고 싶은 욕구가 생겨나서 작업을 시작했다.  
분석/설계 단계에 투입될 경우 개발서버와 개발DB가 구축되지 않았을 경우가 있다.  
이경우 메모리 DB를 사용하면 (ex : H2) 개발자 로컬에서 자유로이 개발이 가능한 장점이 있다.  
그렇게 H2 DB로 개발을 진행 후 
실제 설치된 Maria DB로 변경을 할려고 했더니 H2에서 사용한 Sequence가 문제가 되었다.  
Maria DB에서는 H2 시퀀스 SQL 문이 실행되지 않았기 때문이다.  

이를 보완하기 위해 JPA를 적용해보려한다.
예전 JPA를 사용했던 경험상
JPA는 DB에 비종속적이어서 DB 변경작업시 호환성이 뛰어나고
쿼리문을 따로 작성하지 않아 쿼리오류 발생을 미연에 방지해 준다.

위의 경우는 단순한 CRUD 쿼리에 해당하는 내용이였고
inline view, union all, 통계성 쿼리... 등에서는
JPA를 이용하기에 그 당시 어려움이 있었다.(2013년)

JPA를 좀더 효율적으로 사용하게 지원해주는 Querydsl이 있다고 하니
한번 적용하면서 그 기록을 남기고자 한다.
