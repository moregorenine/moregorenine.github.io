---
title: "Spring Boot JSP 변경사항 새로고침(Refresh) 되지 않을 때"
excerpt: "Spring Boot JSP 변경사항 새로고침 되지 않을 때, 캐쉬를 삭제해도 변경사항 반영되지 않을 때"
categories: 
  - springboot
tags: 
  - springboot
last_modified_at: 2018-11-20T00:28:00+09:00
toc: true
toc_sticky: true
---

## Intro
Spring Boot JSP화면의 변경 사항이 새로고침을 해도 반영되지 않는 문제가 발생했다. JSP 수정시마다 서버 재시작을 할 수는 없는 법, 캐쉬를 삭제하고 새로고침을 시도해도 현상은 마찬가지.  
검색 결과 `server.jsp-servlet.init-parameters.development=true` 설정을 추가하는 방법도 있었으나 spring boot reference에서 찾을 수 없는 설정값인데다 해당 설정값이 어디에서 관리되고 있는지에 대한 설명 글을 찾을 수 없어서 다른 방법을 찾던 중  
`spring-boot-devtools`에서 JSP 변경 사항 동적으로 새로고침 기능을 지원해준다고 하여 적용하였다. `spring-boot-devtools`은 평소 spring boot 프로젝트 생성시 궁금해하던 기능이였는데 이런 기능을 지원해주는구나.

## dependency 추가
- Spring Boot에서 버젼관리를 해주는 dependency라 따로 Version은 기술하지 않았다.
{% highlight java linenos %}
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
</dependency>
{% endhighlight %}

## application.properties 추가
{% highlight java linenos %}
# DEVTOOLS (DevToolsProperties)
spring.devtools.livereload.enabled=true
{% endhighlight %}
