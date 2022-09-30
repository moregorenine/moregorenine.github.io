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
