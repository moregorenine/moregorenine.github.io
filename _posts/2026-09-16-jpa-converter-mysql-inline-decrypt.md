---
title: "JPA Converter로 자동 암호화하고 DB 툴에서 SQL로 바로 복호화하기 (AES-128-ECB & MySQL 기본 모드 호환)"
excerpt: "16자리 대칭키와 MySQL 기본 암호화 모드(AES-128-ECB)를 활용하여 IV 없이 단일 컬럼으로 검색과 암호화를 처리하고, DB 툴에서 AES_DECRYPT(column, @KEY)로 즉시 복호화하는 실무 가이드."
categories:
  - springboot
tags:
  - springboot
  - jpa
  - mysql
  - encryption
  - dbeaver
  - privacy
  - aes128
last_modified_at: 2026-09-17T15:35:00+09:00
toc: true
toc_sticky: true
---

개인정보(전화번호, 계좌번호, 이메일 등)를 암호화할 때 가장 흔히 겪는 고민 중 하나는 **"테이블 스키마가 지저분해지는 문제"**와 **"DB 툴에서의 복호화 쿼리가 너무 길고 복잡해지는 문제"**입니다.

난수 IV(Initialization Vector)를 사용하는 CBC나 GCM 방식을 쓰면 매번 암호문이 달라져 `email_index` 같은 **별도의 해시 인덱스 컬럼**을 테이블마다 만들어야 하고, DB 툴에서 복호화할 때도 IV를 바이트 단위로 쪼개거나 세션 암호화 모드를 일일이 변경해야 합니다.

하지만 실무에서는:
1. **스키마 오염 방지**: `email_index` 같은 보조 컬럼 없이 **단일 컬럼**으로 검색과 암호화를 모두 끝내고 싶고,
2. **IV 파라미터 없는 초간단 쿼리**: DB 툴(DBeaver, DataGrip 등)에서 `AES_DECRYPT(column, @KEY)` 형태로 **IV 없이 딱 2개 파라미터만 전달하여 즉시 복호화**하고 싶은 요구가 매우 큽니다.

이번 포스팅에서는 **16자리 비밀키(AES-128)와 ECB 모드**를 활용하여, **MySQL 기본 엔진 설정 그대로 IV 없이 `AES_DECRYPT(column, @KEY)`로 즉시 복호화하는 JPA Converter 아키텍처**를 정리합니다.

---

## 1. 아키텍처 개요 및 설계 원리

### 1.1 왜 16자리 키 + No-IV (ECB)인가?

MySQL/MariaDB의 기본 블록 암호화 설정(`block_encryption_mode`)은 별도로 변경하지 않는 한 **`'aes-128-ecb'`**입니다.

* **MySQL 기본값과 100% 일치:**  
  MySQL 서버에서 `SET block_encryption_mode = ...` 같은 세션 환경변수를 바꿀 필요조차 없습니다.
* **16자리(16바이트 = 128비트) 키 사용:**  
  AES-128의 규격 블록 크기 및 키 크기와 정확하게 맞아떨어집니다.
* **IV(Initialization Vector) 불필요:**  
  ECB 모드는 블록 단위로 독립 암호화되므로 IV가 존재하지 않습니다. 따라서 `AES_DECRYPT(암호문, 키)` 형태로 단 2개의 인자만 전달하면 바로 복호화됩니다.
* **단일 컬럼 검색 (`WHERE email = ?`):**  
  동일한 평문은 항상 동일한 암호문으로 변환되므로(결정적 암호화), 별도의 `email_index` 컬럼 없이 `email` 컬럼 자체에 B-Tree 인덱스를 걸어 고속 검색이 가능합니다.

---

### 1.2 컬럼 저장 방식에 따른 2가지 선택지

| 방식 | DB 컬럼 타입 | JPA 변환 타입 | DB 툴 복호화 쿼리 |
|---|---|---|---|
| **옵션 A (가장 보편적)** | `VARCHAR(255)` | `String` (Base64 문자열) | `CAST(AES_DECRYPT(FROM_BASE64(email), @KEY) AS CHAR)` |
| **옵션 B (극단적 간결함)** | `VARBINARY(255)` | `byte[]` (순수 바이너리) | `CAST(AES_DECRYPT(email, @KEY) AS CHAR)` |

> 본 가이드에서는 범용적인 **옵션 A (VARCHAR + Base64)**를 기본으로 설명하며, 바이너리 컬럼을 사용하는 옵션 B 쿼리도 함께 안내합니다.

---

## 2. Spring Boot 애플리케이션 구현

### 2.1 개발 환경
* **Spring Boot:** 2.4.3
* **Java:** 11 (LTS)
* **JPA / Hibernate:** 5.4.27.Final (`javax.persistence.*`)
* **Database:** MySQL 5.7+ / 8.0+

### 2.2 application.yml 설정
16자리(16바이트 UTF-8 = 128비트) 대칭키를 정의합니다. **고정 IV 설정은 완전히 제거**되었습니다.

```yaml
crypto:
  # 정확히 16글자(16바이트) 대칭키 (AES-128)
  secret-key: "my-16byte-secret"
```

---

### 2.3 No-IV 기반 암호화 컴포넌트 (`AesEcbCrypto.java`)

Java의 `AES/ECB/PKCS5Padding`은 MySQL의 `AES_ENCRYPT()`, `AES_DECRYPT()` 기본 동작과 바이트 단위로 완벽히 호환됩니다.

```java
package com.example.crypto;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

@Component
public class AesEcbCrypto {

    private static final String ALGORITHM = "AES/ECB/PKCS5Padding";
    private final SecretKeySpec secretKeySpec;

    public AesEcbCrypto(@Value("${crypto.secret-key}") String secretKey) {
        byte[] keyBytes = secretKey.getBytes(StandardCharsets.UTF_8);
        if (keyBytes.length != 16) {
            throw new IllegalArgumentException("AES-128 비밀키는 반드시 16바이트여야 합니다. (현재: " + keyBytes.length + "바이트)");
        }
        this.secretKeySpec = new SecretKeySpec(keyBytes, "AES");
    }

    /**
     * 평문 암호화 (IV 없음, 16바이트 키만 사용)
     */
    public String encrypt(String plainText) {
        if (plainText == null || plainText.isEmpty()) {
            return plainText;
        }

        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            // IV 없이 키 스펙만 전달
            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
            byte[] encryptedBytes = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));
            return Base64.getEncoder().encodeToString(encryptedBytes);
        } catch (Exception e) {
            throw new IllegalStateException("데이터 암호화 중 오류가 발생했습니다.", e);
        }
    }

    /**
     * 암호문 복호화 (IV 없음)
     */
    public String decrypt(String cipherText) {
        if (cipherText == null || cipherText.isEmpty()) {
            return cipherText;
        }

        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);
            byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(cipherText));
            return new String(decryptedBytes, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new IllegalStateException("데이터 복호화 중 오류가 발생했습니다.", e);
        }
    }
}
```

---

### 2.4 JPA AttributeConverter 구현 (`PrivacyEcbConverter.java`)

```java
package com.example.crypto;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter
@Component
@RequiredArgsConstructor
public class PrivacyEcbConverter implements AttributeConverter<String, String> {

    private final AesEcbCrypto aesEcbCrypto;

    @Override
    public String convertToDatabaseColumn(String attribute) {
        return aesEcbCrypto.encrypt(attribute);
    }

    @Override
    public String convertToEntityAttribute(String dbData) {
        return aesEcbCrypto.decrypt(dbData);
    }
}
```

> **Hibernate 5 Bean 컨테이너 설정:**  
> Hibernate 5.4 환경에서 `@Converter` 클래스에 생성자 주입을 정상 사용하려면 `HibernatePropertiesCustomizer` 설정이 필요합니다.
> 
> ```java
> @Configuration
> public class HibernateConfig implements HibernatePropertiesCustomizer {
>     private final ConfigurableListableBeanFactory beanFactory;
>     public HibernateConfig(ConfigurableListableBeanFactory beanFactory) { this.beanFactory = beanFactory; }
>     @Override
>     public void customize(Map<String, Object> hibernateProperties) {
>         hibernateProperties.put(AvailableSettings.BEAN_CONTAINER, new SpringBeanContainer(beanFactory));
>     }
> }
> ```

---

### 2.5 단일 컬럼 엔티티 (`Member.java`)

`email_index` 같은 추가 컬럼이 전혀 없습니다. `email` 컬럼 자체에 인덱스를 걸어 중복 체크와 일치 검색을 수행합니다.

```java
package com.example.domain;

import com.example.crypto.PrivacyEcbConverter;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Entity
@Table(name = "member", indexes = {
    // 별도 인덱스 컬럼 없이 암호화 컬럼 자체에 B-Tree 인덱스 설정
    @Index(name = "idx_member_email", columnList = "email", unique = true),
    @Index(name = "idx_member_phone", columnList = "phone_number")
})
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private String name;

    // 단일 컬럼: DB에는 암호문 저장, 자바에서는 평문
    @Convert(converter = PrivacyEcbConverter.class)
    @Column(name = "email", nullable = false, length = 255)
    private String email;

    @Convert(converter = PrivacyEcbConverter.class)
    @Column(name = "phone_number", nullable = false, length = 255)
    private String phoneNumber;

    public static Member create(String name, String email, String phoneNumber) {
        Member member = new Member();
        member.name = name;
        member.email = email;
        member.phoneNumber = phoneNumber;
        return member;
    }

    public void updateContact(String email, String phoneNumber) {
        this.email = email;
        this.phoneNumber = phoneNumber;
    }
}
```

---

### 2.6 Repository 및 Service 구현

#### Repository (`MemberRepository.java`)
```java
package com.example.repository;

import com.example.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface MemberRepository extends JpaRepository<Member, Long> {

    // email 컬럼 자체로 직접 검색 (B-Tree 인덱스 탐색)
    Optional<Member> findByEmail(String email);

    // 이메일 중복 체크
    boolean existsByEmail(String email);
}
```

#### Service (`MemberService.java`)
비즈니스 로직에서는 암호화를 전혀 의식하지 않고 평문만 다룹니다.

```java
package com.example.service;

import com.example.domain.Member;
import com.example.repository.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityNotFoundException;

@Service
@RequiredArgsConstructor
@Transactional
public class MemberService {

    private final MemberRepository memberRepository;

    public Long register(String name, String email, String phoneNumber) {
        // 평문으로 중복 체크 -> Converter가 암호문으로 변환하여 EXISTS 실행
        if (memberRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("이미 가입된 이메일입니다.");
        }

        Member member = Member.create(name, email, phoneNumber);
        return memberRepository.save(member).getId();
    }

    @Transactional(readOnly = true)
    public Member findByEmail(String email) {
        // 평문으로 검색 -> Converter가 암호문으로 변환 후 WHERE email = ? B-Tree 인덱스 탐색
        return memberRepository.findByEmail(email)
                .orElseThrow(() -> new EntityNotFoundException("회원을 찾을 수 없습니다."));
    }
}
```

---

## 3. DB 툴(DBeaver, DataGrip)에서 초간단 복호화하기

이제 복잡한 IV 설정 없이, **순수하게 키 하나만으로 복호화**할 수 있습니다.

### 3.1 세션 변수 선언 (16자리 키)
MySQL 기본 블록 모드가 이미 `aes-128-ecb`이므로, `SET block_encryption_mode`를 실행할 필요가 없습니다!

```sql
-- 16자리 비밀키 선언 (application.yml 값과 일치)
SET @KEY = 'my-16byte-secret';
```

---

### 3.2 복호화 쿼리 실행 (`AES_DECRYPT(column, @KEY)`)

#### 1) VARCHAR(Base64) 컬럼인 경우
```sql
SELECT 
    id,
    name,
    -- FROM_BASE64 디코딩 후 키만 전달하여 즉시 복호화 (IV 불필요)
    CAST(AES_DECRYPT(FROM_BASE64(email), @KEY) AS CHAR CHARSET utf8mb4) AS email,
    CAST(AES_DECRYPT(FROM_BASE64(phone_number), @KEY) AS CHAR CHARSET utf8mb4) AS phone_number
FROM member;
```

#### 2) VARBINARY(바이너리) 컬럼인 경우
만약 엔티티 컬럼을 `VARBINARY`로 매핑하여 저장했다면 `FROM_BASE64`조차 필요 없이 완전히 `AES_DECRYPT(column, @KEY)`로 직결됩니다:
```sql
SELECT 
    id,
    name,
    CAST(AES_DECRYPT(email, @KEY) AS CHAR CHARSET utf8mb4) AS email,
    CAST(AES_DECRYPT(phone_number, @KEY) AS CHAR CHARSET utf8mb4) AS phone_number
FROM member;
```

#### 실행 결과 예시
| id | name | email | phone_number |
|---|---|---|---|
| 1 | 홍길동 | gdhong@example.com | 010-1234-5678 |
| 2 | 이순신 | sunshin@example.com | 010-9876-5432 |

---

### 3.3 DB 툴에서 특정 이메일로 인덱스 검색하기

운영 중 특정 고객의 이메일(`gdhong@example.com`)을 찾을 때도 IV 없이 즉시 조회할 수 있습니다:

```sql
-- VARCHAR(Base64) 컬럼 기준
SELECT 
    id, 
    name, 
    CAST(AES_DECRYPT(FROM_BASE64(email), @KEY) AS CHAR CHARSET utf8mb4) AS email
FROM member
WHERE email = TO_BASE64(AES_ENCRYPT('gdhong@example.com', @KEY));
```

* `email` 컬럼의 B-Tree 인덱스를 타기 때문에 수천만 건 테이블에서도 즉시 반환됩니다.

---

### 3.4 DBeaver 쿼리 템플릿(Snippet) 등록

DBeaver에 등록해 두면 쿼리 작성이 더 쉬워집니다.

1. 메뉴: `창(Window)` -> `설정(Preferences)` -> `편집기(Editors)` -> `SQL 편집기` -> `템플릿(Templates)`
2. `새로 만들기(New)`:
   * **이름:** `dec`
   * **패턴:**
     ```sql
     CAST(AES_DECRYPT(FROM_BASE64(${column}), @KEY) AS CHAR CHARSET utf8mb4)
     ```
3. 에디터에서 `dec` 입력 후 `Tab` 키를 누르면 컬럼명만 채워서 즉시 복호화됩니다.

---

## 4. 보안성 분석 및 실무 가이드

| 비교 항목 | AES-256-CBC (고정 IV) | AES-128-ECB (본 포스팅) |
|---|---|---|
| **키 길이** | 32바이트 (256비트) | **16바이트 (128비트)** |
| **IV(초기화 벡터)** | 16바이트 고정 IV 필요 | **완전 불필요 (No IV)** |
| **MySQL 기본 호환성** | `SET block_encryption_mode` 필요 | **MySQL 기본 모드 (`aes-128-ecb`) 그대로 사용** |
| **DB 툴 쿼리** | `AES_DECRYPT(col, key, iv)` (3개 인자) | **`AES_DECRYPT(col, key)` (2개 인자)** |
| **인덱스 컬럼 수** | 0개 (단일 컬럼) | **0개 (단일 컬럼)** |
| **KISA/컴플라이언스** | 권고 충족 | **AES-128 이상 권고 충족** |

### ⚠️ ECB 모드 도입 시 실무 권고사항

1. **엔트로피가 높은 데이터에 사용:**  
   ECB 모드는 동일 평문이 항상 동일 암호문으로 변환됩니다.  
   * 주민등록번호 뒷자리 첫 번째 숫자(성별 식별 등 경우의 수가 1~4개에 불과한 데이터)에는 ECB 모드를 쓰면 패턴이 쉽게 노출됩니다.  
   * 반면 **이메일, 전화번호, 계좌번호**처럼 고유하고 경우의 수가 무한에 가까운 개인정보 식별자에는 실무에서 검색 편의성과 스키마 단순화를 위해 활발히 채택됩니다.
2. **비밀키 관리:**  
   16바이트 비밀키는 반드시 안전한 Secret Manager나 서버 환경변수로 관리하고, DB 로그에 남지 않도록 세션 변수(`@KEY`)를 활용하세요.

---

## 5. 마무리 요약

1. **키 16자리(AES-128) + No-IV**: MySQL의 기본 암호화 엔진과 100% 바이너리 호환되어 세션 설정 변경 없이 동작합니다.
2. **`AES_DECRYPT(col, @KEY)`**: DB 툴에서 추가 파라미터(IV) 없이 키 하나만 전달하여 가장 간단하게 즉시 복호화할 수 있습니다.
3. **스키마 복잡도 제로**: `email_index` 같은 보조 컬럼 없이 단일 컬럼으로 JPA 자동 암복호화와 B-Tree 인덱스 검색을 모두 달성합니다.
