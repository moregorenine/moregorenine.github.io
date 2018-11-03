---
title: "Spring Boot Quick Start #1"
excerpt: "Spring Boot에 대해 알아보자 설정에서 부터"
categories: 
  - Java
tags: 
  - Java
  - Lambda
last_modified_at: 2018-11-03T21:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
Spring Boot에 대해 알아보자 설정에서 부터
[Spring Boot Quick Start 강좌 - Java Brains](https://javabrains.io/courses/spring_bootquickstart/ "Spring Boot Quick Start 강좌 Link")  
[연습예제 github](https://github.com/moregorenine/study/tree/master/spring-boot-quick-start "연습예제 github Link")

## Spring Boot의 시작은?
처음 spring boot를 접했을 땐 tomcat 설정도 필요없이 혼자 알아서 작동되는 걸 보고 신기해했었다. spring에서 처음 설정해주던 .xml도 보이지 않았으니깐 그 비밀의 시작은 아래 한줄이였다.  
> SpringApplication.run(App.class, args);

