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

## 스프링 부트 어떻게 시작할까요?
스프링 부트 프로젝트 생성을 어떻게 시작할지 아래 3가지 방법에 대해서 알아본다.
* STS IDE를 통한 방법
  * 예제 [링크](https://javabrains.io/courses/spring_bootquickstart/lessons/Using-the-STS-IDE/)
* 공식 스프링 사이트에서 프로젝트 생성 다운로드 [https://start.spring.io](https://start.spring.io)
  * 예제 [링크](https://javabrains.io/courses/spring_bootquickstart/lessons/Using-Spring-Initializr/)
* 스프링 부트 CLI 명령어를 통한 방법
  * 예제 [링크](https://javabrains.io/courses/spring_bootquickstart/lessons/Using-Spring-Boot-CLI/)

## application.properties
스프링 설정파일 정보만 해도 어마어마하게 방대하다. 이걸 일일이 외우는 사람은 없겠죠? 자신의 버젼에 맞는 스프링 API 문서에서 찾을 수 있습니다.  
current의 항상 최신 버젼의 스프링 API를 예로 들면  
current version의 [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/index.html)  
문서양만 해도 어마무시한데요 무려 10번째 항목 Common application properties 에서 찾을 수 있으니 참고세요..
* X. Appendices
  * A. Common application properties

이런 방대한 양의 설정들을 `application.properties` or `application.yml` 에서 간단하게 설정할 수 있도록 도와주니 고마운 일이죠.

## Related Posts
Spring Boot Quick Start #2 [link](https://moregorenine.github.io/springboot/springboot-2/ "Spring Boot Quick Start #2")
