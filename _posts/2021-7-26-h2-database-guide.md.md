---
title: "H2 Database 연결 방식 및 JDBC URL 설정 가이드"
excerpt: "H2 데이터베이스의 세 가지 주요 모드(File, TCP, In-Memory)의 특징과 환경별 JDBC URL 설정 방법을 상세히 정리합니다."
categories:
  - Database
tags:
  - h2-database
  - embedded-db
  - in-memory-db
  - jdbc
  - spring-boot
  - backend
last_modified_at: 2021-07-26T17:13:41+09:00
toc: true
toc_sticky: true
---

## JDBC URL

H2는 크게 **file**, **mem**, **tcp** 방식을 지원합니다. 각 방식은 데이터의 영속성과 동시 접속 가능 여부에 따라 선택하여 사용합니다.

### 1. 로컬(Embedded 모드)

- **URL:** `jdbc:h2:~/test`
- **특징:** 데이터베이스가 'test'로 시작하는 파일로 사용자 홈 디렉토리에 저장됩니다.
- **경로 설정:** `jdbc:h2:/data/db/test`와 같은 절대 경로와 `jdbc:h2:./data/test`(현재 작업 디렉토리 기준)를 지원합니다.
- **제한:** 애플리케이션과 동일한 프로세스 내에서 실행되며, **한 번에 하나의 프로세스만** 접근 가능합니다. 데이터베이스가 없으면 권한이 있는 경우 자동으로 생성됩니다.

> **Tip:** 운영 환경에 따라 `~` 또는 절대 경로를 사용하는 것이 경로 혼선을 줄이는 데 유리합니다.

### 2. 원격(Server 모드 / Client-Server)

- **URL:** `jdbc:h2:tcp://localhost/~/test`
- **특징:** TCP/IP를 통해 연결하며, 별도로 실행 중인 H2 TCP 서버에 접속합니다.
- **장점:** **여러 클라이언트가 동시에** 동일한 데이터베이스에 연결할 수 있습니다. 내장 데이터베이스와 동일한 위치 규칙이 적용되지만, 반드시 서버를 먼저 구동해야 합니다.

### 3. 인메모리(In-Memory 모드)

- **URL:** `jdbc:h2:mem:test`
- **특징:** 'test'라는 이름의 데이터베이스를 메모리상에 생성합니다.
- **영속성:** 데이터가 디스크에 저장되지 않으므로, **마지막 연결이 닫히면 모든 데이터가 소멸**됩니다.
- **용도:** 테스트 코드 실행이나 휘발성 데이터 처리에 최적화되어 있습니다. 동일 프로세스 내 여러 스레드에서 접근 가능합니다.

---

더 상세한 정보는 공식 문서의 https://namu.wiki/w/%EA%B0%9C%EC%9A%94(http://www.h2database.com/html/features.html#database_url)에서 확인하실 수 있습니다.
