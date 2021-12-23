---
title: "Developer Tools"
excerpt: "spring boot 자동 재시작 및 브라우저 자동 refesh 기능"
categories: 
  - springboot
tags: 
  - springboot
  - devtools
last_modified_at: 2021-12-23T00:00:00+09:00
toc: true
toc_sticky: true
---

## Developer Tools 이란?
Spring Boot에서 지원하는 여러 라이브러리는 `캐시`를 사용하여 성능을 향상시킵니다.  
예를 들어 템플릿 엔진은 템플릿 파일을 반복적으로 구문 분석하지 않도록 컴파일된 템플릿을 캐시합니다.  
또한 Spring MVC는 정적 리소스를 제공할 때 응답에 HTTP 캐싱 헤더를 추가할 수 있습니다.  
  
캐싱은 운영에 배포되는 production에서는 매우 유용하지만 **개발 중에는 비생산적이어서 애플리케이션에서 방금 변경한 내용을 볼 수 없습니다.**  
이러한 이유로 spring-boot-devtools는 기본적으로 캐싱 옵션을 비활성화합니다.  

개발자 도구는 완전히 패키지된 응용 프로그램을 실행할 때 자동으로 비활성화됩니다.  
응용 프로그램이 java -jar특수 클래스 로더에서 시작되거나 시작되는 경우 "production 프로그램"으로 간주됩니다.
{: .notice--info}
  
캐시 옵션은 일반적으로 application.properties파일의 설정에 의해 구성됩니다.  
예를 들어, Thymeleaf는 spring.thymeleaf.cache속성을 제공합니다.  
이러한 속성을 수동으로 설정할 필요 없이 spring-boot-devtools 모듈은 합리적인 개발 시간 구성을 자동으로 적용합니다.  
  
Spring MVC 및 Spring WebFlux 애플리케이션을 개발하는 동안 웹 요청에 대한 추가 정보가 필요하기 때문에  
개발자 도구는 로깅 그룹 DEBUG에 대한 web로깅 을 활성화할 것을 제안합니다.  
그러면 들어오는 요청, 처리 중인 처리기, 응답 결과 및 기타 세부 정보에 대한 정보가 제공됩니다.  
모든 요청 세부 정보(잠재적으로 민감한 정보 포함)를 기록하려면 spring.mvc.log-request-details 또는 spring.codec.log-request-details 구성 속성을 켤 수 있습니다.  

## Developer Tools 설치
maven
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

gradle
```css
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

## 자동 재시작
spring-boot-devtools은 클래스 경로의 파일이 변경될 때마다 자동으로 다시 시작됩니다.  
이것은 코드 변경에 대한 매우 빠른 피드백 루프를 제공하므로 IDE에서 작업할 때 유용한 기능이 될 수 있습니다.  
기본적으로 디렉토리를 가리키는 클래스 경로의 모든 항목은 변경 사항에 대해 모니터링됩니다.  
하지만 static, view templates 같은 특정 리소스는 애플리케이션을 다시 시작할 필요가 없습니다.  

### 재시작 트리거  
DevTools는 클래스 경로 리소스를 모니터링하므로 다시 시작을 트리거하는 유일한 방법은 클래스 경로를 업데이트하는 것입니다. 클래스 경로를 업데이트하는 방법은 사용 중인 IDE에 따라 다릅니다.  
- Eclipse에서 수정된 파일을 저장하면 클래스 경로가 업데이트되고 다시 시작됩니다.  
- IntelliJ에서 `Build → Build Project`(단축키:Ctrl+F9)를 빌드하는 것은 동일한 효과를 가집니다.  
- 빌드 플러그인을 사용하는 경우 mvn compileMaven 또는 gradle buildGradle을 실행하면 다시 시작됩니다.  


## LiveReload
별다른 설정 없이 `spring-boot-devtools` library를 추가하는 것 만으로 devtools 자동 재시작이 동작하고  
java -jar로 production 배포시에는 자동적으로 devtools를 disable 될 것입니다.  
하지만 node.js, webpack, react, vue 에서 처럼 코드 변경시 브라우져가 자동으로 refresh 되지는 않습니다.  
자동으로 refresh 기능은 Chrome, Firefox, Safari 에서 [livereload.com](http://livereload.com/extensions/) 확장 프로그램을 설치해야 합니다.  
intellij 를 사용하신다면 `Build → Build Project`(단축키:Ctrl+F9) 실행하는 순간 브라우저가 자동적으로 refresh 되는걸 확인하실 수 있습니다.  

한 번에 하나의 LiveReload 서버만 실행할 수 있습니다. 응용 프로그램을 시작하기 전에 다른 LiveReload 서버가 실행되고 있지 않은지 확인하십시오. IDE에서 여러 응용 프로그램을 시작하는 경우 첫 번째 응용 프로그램만 LiveReload를 지원합니다.
{: .notice--info}