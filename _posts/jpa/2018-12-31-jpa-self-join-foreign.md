---
title: "JPA Self Join Foreign-Key 셀프 조인시 외래키 참조"
excerpt: "JPA Self Join 시 Foreign-Key를 어떻게 참조할까?"
categories: 
  - jpa
tags: 
  - jpa
last_modified_at: 2018-12-31T00:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
JPA에서 Self Join시에 @JoinColumn(name="foreignKey")으로 생성시 DB에 FOREIGN_KEY 컬럼으로 데이터가 생성되나 Java에서 객체의 foreignKey는 존재하는 않는 parameter라 접근 방법이 모호하다. 이에 대한 해결 방법을 알아보자.

## 3가지 방법
[3가지 방법](https://stackoverflow.com/questions/3941797/how-can-i-retrieve-the-foreign-key-from-a-jpa-manytoone-mapping-without-hitting "3가지 방법 link")

b) Use read-only fields for the FKs
방법이 괜찮아 보이나. 해당 설명만으론 부족하다 @JoinColumn(name="foreignKey")랑 동일한 foreignKey값으로 parameter 생성시 insertable = false, updatable = false 설정을 할경우 foreignKey값이 DB에 insert, update가 되지 않는 문제가 발생한다.
아래는 이에 대한 자세한 해결법이다.

## b) 방법에 대한 자세한 해결법
[자세한 해결법](https://stackoverflow.com/questions/6311776/hibernate-foreign-keys-instead-of-entities "자세한 해결법 link")

즉 foreignKey값을 참조하는 다른 명칭의 parameter명의 생성한다.
