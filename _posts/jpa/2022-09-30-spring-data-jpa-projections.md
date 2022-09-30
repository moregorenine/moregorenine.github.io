---
title: "Spring Data JPA Projections"
excerpt: "Spring Data JPA Projections"
categories: 
  - JPA
tags: 
  - JPA
  - Projections
last_modified_at: 2022-09-30T00:00:00+09:00
toc: true
toc_sticky: true
---
## 1. projections
JPA에서 가져온 entity를 보통 그대로 가져다 쓰지는 않는다.  
호환성, 확장성 같은 이유가 있기도 하고  
FetchType.LAZY나 필요없는 정보까지 다 끌어오기 때문이기도 하다.  
한국에서 projections에 대한 설명 대부분이 interface를 사용한 방법들이 검색이 된다.  
하지만 그냥 DTO 객체를 이용하는 편이 여러모로 나아보인다.  
[https://www.baeldung.com/spring-data-jpa-projections](https://www.baeldung.com/spring-data-jpa-projections)  

## 2. findById with projection in Spring Data
그럼 findById 같이 기본적으로 JpaRepository에서 제공하는 query method를 어떻게 Dto로 받을까?  
한국에서는 mapper를 이용한 convert에 대한 설명이 대부분이긴 한데.  
이미 JpaRepository에서는 Generic 까지 지원해줘서 간편하게 원하는 Dto로 받아올 수 있다.  
[stackoverflow.com 답변](https://stackoverflow.com/questions/49669511/how-to-override-findbyid-with-projection-in-spring-data)  
[docs.spring.io 공식문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#projection.dynamic)
