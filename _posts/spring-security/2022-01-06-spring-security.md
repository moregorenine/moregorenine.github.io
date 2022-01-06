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
CSRF 공격은 악의적인 공격의 희생자가 공격자가 되기때문에  
CSRF 공격으로부터 보호하려면 사용자의 요청이 정상적인 요청인지 악의적인 공격인지를 구분할 수 있도록 두가지 메커니즘을 제공합니다.  
- The Synchronizer Token Pattern
  - CSRF 공격으로부터 보호하는 가장 보편적이고 포괄적인 방법
- Specifying the SameSite Attribute on your session cookie
2. 
3. 
4. 
