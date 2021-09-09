---
title: "Spring Data"
excerpt: "Spring Data"
categories: 
  - springboot
tags: 
  - springdata
  - springboot
last_modified_at: 2021-9-10T00:00:00+09:00
toc: true
toc_sticky: true
---

## fetch size
https://stackoverflow.com/questions/20284503/set-the-fetch-size-with-spring-data  
대용량의 데이터 조회시 fetch size에 따라 속도 차이가 많이 납니다.  
fetch size는 전역 설정과 특정 쿼리에 hint로 설정할 수 있는 방법을 제공합니다.  

특정 쿼리에 대해 지정하려면 Query 개체에서 JPA의 쿼리 힌트를 사용하십시오.  
```java
TypedQuery<Foo> q = em.createTypedQuery("여기에 일부 hql", Foo.class);
q.setHint("org.hibernate.fetchSize", "100");
```
또는 Spring Data JPA를 사용할 때 인터페이스 메소드에 @QueryHints 주석을 사용하십시오.  
@Query가 있는 방법과 없는 방법 모두에 적용할 수 있습니다.
```java
@QueryHints(@javax.persistence.QueryHint(name="org.hibernate.fetchSize", value="50"))
List<Foo> findAll();
```

## query method
https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods  
spring data 사용시 쿼리메소드를 많이 사용하게 됩니다.
