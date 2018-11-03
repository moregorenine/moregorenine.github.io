---
title: "Spring Boot Quick Start #1"
excerpt: "Spring Boot에 대해 알아보자 설정에서 부터"
categories: 
  - springboot
tags: 
  - springboot
last_modified_at: 2018-11-03T21:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
Spring Boot에 대해 알아보자 설정에서 부터  
[Spring Boot Quick Start 강좌 - Java Brains](https://javabrains.io/courses/spring_bootquickstart/ "Spring Boot Quick Start 강좌 Link")  
[연습예제 github](https://github.com/moregorenine/study/tree/master/spring-boot-quick-start "연습예제 github Link")

## Spring Boot의 시작은?
처음 spring boot를 접했을 땐 tomcat 설정도 필요없이 혼자 알아서 작동되는 걸 보고 신기했었다. spring에서 처음 설정해주던 .xml도 보이지 않았으니깐 마법의 시작은 아래 한줄이였다.

> SpringApplication.run(App.class, args);

이 한줄이 하는 역활이 무엇일까?
- 기본 configuration 설정
- spring application 시작
- classpath 스캔
- 톰캣서버 시작

## Embedded 톰캣 특징
- 편리하다
- Servlet Container의 설정 파일이 application의 설정 파일로 대체
- 독립적인 applicaiton
- microservices 아키텍쳐에 유용함.
