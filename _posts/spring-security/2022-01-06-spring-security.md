---
title: "Spring Security"
excerpt: "인증, 인가, 보안을 책임지는 Spring Security"
categories: 
  - springsecurity
tags: 
  - springsecurity
last_modified_at: 2022-01-06T00:00:00+09:00
toc: true
toc_sticky: true
---

## Spring Security 이란?
Spring Security는 `사용자`의 **인증, 인가** 및 악의적인 공격에 대한 **보호** 등 포괄적인 지원을 제공합니다.  

## 인증 & 인가
인증은 특정 리소스에 액세스하려는 사람의 신원을 확인하는 방법입니다.  
사용자를 인증하는 일반적인 방법은 사용자에게 `사용자`의 `ID`와 `Password`를 입력하도록 요구하는 것입니다.  
인증이 완료되면 `사용자`의 신원을 확인하고 자원에 대한 접근권한인 `인가`을 수행할 수 있습니다.  

## 보안
Spring Security는 악의적인 공격에 대한 보안 기능을 제공합니다.  
가능한 한 보안 기능은 별도의 설정 없이도 기본적으로 활성화됩니다.  

1. Cross Site Request Forgery (CSRF)  
CSRF(교차 사이트 요청 위조)  는 악성 웹 사이트, 이메일, 블로그, 인스턴트 메시지 또는 프로그램이  
사용자가 인증될 때 사용자의 웹 브라우저가 신뢰할 수 있는 사이트에서 원치 않는 작업을 수행하도록 할 때 발생하는 공격 유형입니다.  
CSRF 공격은 브라우저 요청이 세션 쿠키를 포함한 모든 쿠키를 자동으로 포함하기 때문에 작동합니다.  
따라서 사용자가 사이트에 대해 인증된 경우 사이트는 합법적으로 승인된 요청과 위조된 인증된 요청을 구별할 수 없습니다.  
Spring은 CSRF 공격으로부터 보호하려면 사용자의 요청이 정상적인 요청인지 악의적인 공격인지를 구분할 수 있도록 두가지 메커니즘을 제공합니다.  
- The Synchronizer Token Pattern
  - CSRF 공격으로부터 보호하는 가장 보편적이고 포괄적인 방법
  - 서버에서 `Token`을 넘겨주고 클라이언트에서 요청시 해당 `Token`을 사용하여 인증받습니다. 그렇기 때문에 `Token`은 쿠기를 사용하여 전송되어서는 안됩니다.
- Specifying the SameSite Attribute on your session cookie
  - 서버는 쿠키를 설정할 때 `SameSite` attribute을 지정하며 외부 사이트에서 오는 request 경우 쿠키를 보내지 않습니다.
