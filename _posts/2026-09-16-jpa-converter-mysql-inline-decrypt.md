---
title: "JPA Converter로 자동 암호화하고 DB 툴에서 SQL로 바로 복호화하기 (AES-256-CBC 고정 IV & 단일 컬럼 설계)"
excerpt: "단일 컬럼으로 검색과 암호화를 동시에 해결하고, DB에 함수(UDF) 생성 없이 DBeaver 등 DB 툴에서 MySQL 기본 내장 함수로 즉시 복호화하는 실무 아키텍처 가이드."
categories:
  - springboot
tags:
  - springboot
  - jpa
  - mysql
  - encryption
  - dbeaver
  - privacy
  - aes256
last_modified_at: 2026-09-16T15:25:00+09:00
toc: true
toc_sticky: true
---

개인정보(전화번호, 계좌번호, 이메일 등)를 암호화할 때 가장 흔히 겪는 고민 중 하나는 **"테이블 스키마가 지저분해지는 문제"**입니다.

보통 난수 IV(Initialization Vector)를 사용하는 AES-GCM이나 일반 CBC 방식을 쓰면 매번 암호문이 바뀌기 때문에, `WHERE email = ?` 검색을 지원하려면 `email_index` 같은 **별도의 해시 인덱스 컬럼**을 테이블마다 추가로 만들어야 합니다.

하지만 실무에서는 다음과 같은 이유로 인덱스 컬럼 생성이 꺼려집니다:

1. **테이블 스키마 오염**: 암호화할 개인정보 컬럼이 5개면 인덱스 컬럼도 5개가 추가되어 컬럼 수가 2배로 폭증합니다.
2. **엔티티 복잡도 증가**: 엔티티마다 `email`과 `emailIndex`를 쌍으로 관리해야 하고, 비즈니스 로직에서도 매번 해시를 생성해 주입해야 합니다.
3. **DB 용량 및 인덱스 오버헤드**: 64자리 해시 인덱스가 테이블마다 추가되어 스토리지 용량과 INSERT/UPDATE 오버헤드가 증가합니다.
4. **DBA/운영팀의 복호화 요구**: 장애 분석이나 고객 대응 시 DB 툴(DBeaver, DataGrip 등)에서 직관적으로 복호화 조회가 되어야 합니다.

이번 포스팅에서는 **`email_index` 같은 추가 컬럼을 전혀 만들지 않고**, **`email` 단일 컬럼만으로 JPA 자동 암호화 + 고속 인덱스 검색 + DB 툴 인라인 복호화(함수 생성 0개)**를 모두 해결하는 **AES-256-CBC 결정적 암호화(Fixed IV)** 아키텍처를 정리합니다.

---

## 1. 아키텍처 개요 및 설계 원리

### 1.1 핵심 아이디어
1. **결정적 암호화 (Deterministic Encryption):**  
   매번 바뀌는 난수 IV 대신, 사전에 정의된 **16바이트 고정 IV(Fixed IV)**를 사용합니다.  
   ➡️ 동일한 평문은 항상 동일한 암호문으로 변환되므로, `email` 컬럼 자체에 일반 B-Tree 인덱스를 걸어 **별도 인덱스 컬럼 없이 `WHERE email = ?` 검색이 가능**해집니다.
2. **저장 포맷 간소화:**  
   IV를 암호문 앞에 따로 붙여 저장할 필요가 없으므로 DB에는 순수 **`Base64( AES-256-CBC 암호문 )`**만 깔끔하게 저장됩니다 (스토리지 용량 절감).
3. **애플리케이션 투명성:**  
   JPA `AttributeConverter`를 통해 엔티티 입출력 시 자동으로 암복호화가 일어납니다.
4. **DB 툴 즉시 복호화 (Zero DDL):**  
   DB 서버에 어떠한 함수(`CREATE FUNCTION`)나 뷰(`VIEW`)도 만들지 않고, MySQL 내장 함수인 **`AES_DECRYPT(FROM_BASE64(col), @KEY, @IV)`** 한 줄로 즉시 평문 조회가 가능합니다.

---

### 1.2 저장 구조 비교

#### 기존 방식 (난수 IV + 인덱스 컬럼 분리)
* `email`: `Base64( 16바이트 난수 IV + 암호문 )`
* `email_index`: `HMAC-SHA256 해시값 (64바이트)` ➡️ **컬럼 2개 필요, 스키마 복잡**

#### 본 포스팅 방식 (고정 IV 단일 컬럼)
* `email`: `Base64( 암호문 )` ➡️ **컬럼 딱 1개로 끝! 인덱스도 이 컬럼에 직접 생성**

---

## 2. Spring Boot 애플리케이션 구현

### 2.1 개발 환경
* **Spring Boot:** 2.4.3
* **Java:** 11 (LTS)
* **JPA / Hibernate:** 5.4.27.Final (`javax.persistence.*`)
* **Database:** MySQL 5.7+ / 8.0+

### 2.2 application.yml 설정
AES-256 비밀키(32바이트)와 고정 IV(16바이트)를 정의합니다.

```yaml
crypto:
  # 정확히 32바이트 대칭키 (256비트)
  secret-key: "my-secure-aes-256-secret-key-32"
  # 정확히 16바이트 고정 IV (128비트 블록 크기)
  fixed-iv: "my-fixed-16b-iv!"
```

> **보안 팁:** 운영 환경에서는 위 키 값들을 형상관리(Git)에 올리지 않고, 환경 변수(`CRYPTO_SECRET_KEY`, `CRYPTO_FIXED_IV`)나 AWS Secrets Manager, Vault 등을 통해 안전하게 주입해야 합니다.

---

### 2.3 고정 IV 기반 암호화 컴포넌트 (`AesCbcCrypto.java`)

```java
package com.example.crypto;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

@Component
public class AesCbcCrypto {

    private static final String ALGORITHM = "AES/CBC/PKCS5Padding";

    private final SecretKeySpec secretKeySpec;
    private final IvParameterSpec fixedIvSpec;

    public AesCbcCrypto(@Value("${crypto.secret-key}") String secretKey,
                        @Value("${crypto.fixed-iv}") String fixedIv) {
        
        byte[] keyBytes = secretKey.getBytes(StandardCharsets.UTF_8);
        if (keyBytes.length != 32) {
            throw new IllegalArgumentException("AES-256 비밀키는 반드시 32바이트여야 합니다. (현재: " + keyBytes.length + "바이트)");
        }

        byte[] ivBytes = fixedIv.getBytes(StandardCharsets.UTF_8);
        if (ivBytes.length != 16) {
            throw new IllegalArgumentException("AES IV는 반드시 16바이트여야 합니다. (현재: " + ivBytes.length + "바이트)");
        }

        this.secretKeySpec = new SecretKeySpec(keyBytes, "AES");
        this.fixedIvSpec = new IvParameterSpec(ivBytes);
    }

    /**
     * 암호화: 고정 IV를 사용하므로 동일 평문은 항상 동일한 Base64 암호문을 반환
     */
    public String encrypt(String plainText) {
        if (plainText == null || plainText.isEmpty()) {
            return plainText;
        }

        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, fixedIvSpec);
            byte[] encryptedBytes = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));
            return Base64.getEncoder().encodeToString(encryptedBytes);
        } catch (Exception e) {
            throw new IllegalStateException("데이터 암호화 중 오류가 발생했습니다.", e);
        }
    }

    /**
     * 복호화: Base64 디코딩 후 고정 IV로 복호화
     */
    public String decrypt(String cipherText) {
        if (cipherText == null || cipherText.isEmpty()) {
            return cipherText;
        }

        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, fixedIvSpec);
            byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(cipherText));
            return new String(decryptedBytes, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new IllegalStateException("데이터 복호화 중 오류가 발생했습니다.", e);
        }
    }
}
```

---

### 2.4 JPA AttributeConverter 구현 (`PrivacyCbcConverter.java`)

```java
package com.example.crypto;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter
@Component
@RequiredArgsConstructor
public class PrivacyCbcConverter implements AttributeConverter<String, String> {

    private final AesCbcCrypto aesCbcCrypto;

    @Override
    public String convertToDatabaseColumn(String attribute) {
        return aesCbcCrypto.encrypt(attribute);
    }

    @Override
    public String convertToEntityAttribute(String dbData) {
        return aesCbcCrypto.decrypt(dbData);
    }
}
```

> **Hibernate 5 스프링 빈 주입 설정:**  
> Hibernate 5.4 환경에서 `@Converter` 클래스에 스프링 빈 의존성을 주입하려면 `HibernatePropertiesCustomizer`를 등록해야 합니다.
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

`email` 컬럼 자체에 유니크 인덱스를 걸어 중복 가입을 방지하고 고속 검색을 지원합니다.

```java
package com.example.domain;

import com.example.crypto.PrivacyCbcConverter;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Entity
@Table(name = "member", indexes = {
    // 별도 해시 컬럼 없이 암호화 컬럼 자체에 인덱스 설정!
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
    @Convert(converter = PrivacyCbcConverter.class)
    @Column(name = "email", nullable = false, length = 255)
    private String email;

    @Convert(converter = PrivacyCbcConverter.class)
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
암호화된 `email` 컬럼을 그대로 파라미터로 넘겨 조회할 수 있습니다.

```java
package com.example.repository;

import com.example.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface MemberRepository extends JpaRepository<Member, Long> {

    // email 컬럼 자체로 직접 조회 (B-Tree 인덱스 탐색)
    Optional<Member> findByEmail(String email);

    // 이메일 중복 체크
    boolean existsByEmail(String email);

    // 전화번호로 회원 조회
    Optional<Member> findByPhoneNumber(String phoneNumber);
}
```

> **Spring Data JPA 파라미터 자동 변환 동작:**  
> Spring Data JPA의 파인드 메서드(`findByEmail(String email)`)에 평문 `"hong@example.com"`을 전달하면, 매핑된 `PrivacyCbcConverter`가 쿼리 생성 시 자동으로 `convertToDatabaseColumn()`을 호출하여 암호문으로 치환한 뒤 바인딩합니다.  
> 따라서 서비스 코드에서는 암호화를 전혀 신경 쓰지 않고 **평문 그대로 전달**하면 됩니다.

#### Service (`MemberService.java`)
비즈니스 로직 어디에도 암호화나 해시 생성 코드가 침투하지 않아 극도로 깔끔합니다.

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
        // 평문으로 중복 체크 (JPA Converter가 내부적으로 암호화하여 EXISTS 쿼리 실행)
        if (memberRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("이미 가입된 이메일입니다.");
        }

        // 엔티티 생성 시 평문 전달 -> DB INSERT 시 자동 암호화
        Member member = Member.create(name, email, phoneNumber);
        return memberRepository.save(member).getId();
    }

    @Transactional(readOnly = true)
    public Member findByEmail(String email) {
        // 평문으로 검색 -> Converter가 암호문으로 변환하여 WHERE email = ? B-Tree 인덱스 탐색
        return memberRepository.findByEmail(email)
                .orElseThrow(() -> new EntityNotFoundException("회원을 찾을 수 없습니다."));
    }
}
```

---

## 3. DB 툴(DBeaver, DataGrip)에서 함수 없이 복호화하기

고정 IV 방식을 사용하면 암호문 앞에 16바이트를 자를 필요가 없기 때문에, **DB 툴에서의 쿼리가 훨씬 더 직관적이고 단순해집니다.**

### 3.1 세션 설정 및 변수 선언 (필수 1회 실행)
DBeaver나 DataGrip의 SQL 에디터 창에서 아래 명령어를 실행합니다:

```sql
-- 1. 현재 세션의 암호화 모드를 AES-256-CBC로 설정 (내 세션에만 적용)
SET block_encryption_mode = 'aes-256-cbc';

-- 2. 키와 고정 IV를 세션 변수로 선언 (application.yml 값과 동일)
SET @KEY = 'my-secure-aes-256-secret-key-32';
SET @IV  = 'my-fixed-16b-iv!';
```

---

### 3.2 순수 내장 함수 인라인 복호화 쿼리

`SUBSTRING`으로 바이트를 쪼갤 필요 없이, MySQL 기본 내장 함수 `FROM_BASE64()`와 `AES_DECRYPT()`만으로 복호화합니다:

```sql
SELECT 
    id,
    name,
    -- 이메일 즉시 복호화
    CAST(AES_DECRYPT(FROM_BASE64(email), @KEY, @IV) AS CHAR CHARSET utf8mb4) AS email,
    -- 전화번호 즉시 복호화
    CAST(AES_DECRYPT(FROM_BASE64(phone_number), @KEY, @IV) AS CHAR CHARSET utf8mb4) AS phone_number
FROM member;
```

#### 실행 결과 예시
| id | name | email | phone_number |
|---|---|---|---|
| 1 | 홍길동 | gdhong@example.com | 010-1234-5678 |
| 2 | 이순신 | sunshin@example.com | 010-9876-5432 |

---

### 3.3 DB 툴에서 특정 이메일로 바로 검색하기

운영 중 특정 고객의 이메일(`gdhong@example.com`)로 회원을 찾아야 할 때, 고정 IV 방식이므로 **DB 툴에서도 MySQL 내장 암호화 함수를 이용해 인덱스 검색**을 바로 수행할 수 있습니다!

```sql
-- DB 툴에서 평문으로 조건 검색 (인덱스를 타고 O(1)로 조회됨)
SELECT 
    id, 
    name, 
    CAST(AES_DECRYPT(FROM_BASE64(email), @KEY, @IV) AS CHAR CHARSET utf8mb4) AS email
FROM member
WHERE email = TO_BASE64(AES_ENCRYPT('gdhong@example.com', @KEY, @IV));
```

이 쿼리는 `email` 컬럼의 B-Tree 인덱스를 정확하게 타기 때문에, 수백만 건의 회원 테이블에서도 즉시 결과를 반환합니다.

---

### 3.4 DBeaver 쿼리 템플릿(Snippet) 등록

DBeaver를 자주 쓴다면 템플릿을 등록해 두면 더욱 편리합니다.

1. 메뉴: `창(Window)` -> `설정(Preferences)` -> `편집기(Editors)` -> `SQL 편집기` -> `템플릿(Templates)`
2. `새로 만들기(New)`:
   * **이름:** `dec`
   * **패턴:**
     ```sql
     CAST(AES_DECRYPT(FROM_BASE64(${column}), @KEY, @IV) AS CHAR CHARSET utf8mb4)
     ```
3. 에디터에서 `dec` 입력 후 `Tab`을 누르면 컬럼명만 채워 즉시 복호화할 수 있습니다.

---

## 4. (선택) 권한이 있는 경우: 복호화 전용 VIEW 생성

만약 읽기 전용 복제본(Replica)에 DDL 권한이 있거나 개발/스테이징 DB라면, VIEW를 생성해 두면 쿼리 작성이 더 쉬워집니다.

```sql
CREATE OR REPLACE VIEW v_member_plain AS
SELECT 
    id,
    name,
    CAST(AES_DECRYPT(FROM_BASE64(email), 'my-secure-aes-256-secret-key-32', 'my-fixed-16b-iv!') AS CHAR CHARSET utf8mb4) AS email,
    CAST(AES_DECRYPT(FROM_BASE64(phone_number), 'my-secure-aes-256-secret-key-32', 'my-fixed-16b-iv!') AS CHAR CHARSET utf8mb4) AS phone_number
FROM member;

-- DB 툴에서는 평소처럼 심플하게 SELECT
SELECT * FROM v_member_plain WHERE email = 'gdhong@example.com';
```

---

## 5. 보안성 분석 및 트레이드오프

결정적 암호화(고정 IV)를 채택하기 전에, 장점과 함께 보안적 트레이드오프를 명확히 인지하고 있어야 합니다.

| 비교 항목 | 난수 IV 방식 (기존) | 고정 IV 방식 (본 포스팅) |
|---|---|---|
| **필요 컬럼 수** | 컬럼당 2개 (`email`, `email_index`) | **컬럼당 1개 (`email` 단일 컬럼)** |
| **B-Tree 인덱스 검색** | 별도 해시 인덱스 컬럼 이용 | **암호문 컬럼 자체로 직접 검색** |
| **DB 툴 복호화 쿼리** | `SUBSTRING`으로 IV 분리 필요 | **`FROM_BASE64` + `AES_DECRYPT`로 초간단** |
| **스토리지 용량** | IV(16B) + 해시(64B) 추가 소모 | **최소 용량 (순수 암호문만 저장)** |
| **패턴 노출 위험** | 없음 (동일 평문도 매번 다른 암호문) | **있음 (동일 평문 = 동일 암호문)** |
| **개인정보보호법 적합성** | ✅ 적합 (AES-256 사용) | ✅ 적합 (AES-256 사용) |

### ⚠️ 고정 IV 방식 도입 시 보안 체크포인트

1. **빈도 분석(Frequency Analysis) 공격 주의:**  
   동일한 이메일이나 전화번호는 항상 동일한 암호문으로 치환됩니다.  
   * 주민등록번호의 성별 식별 자리처럼 "경우의 수가 적은 값"에는 고정 IV를 쓰면 패턴이 쉽게 노출될 수 있으므로 주의해야 합니다.  
   * 반면 이메일, 전화번호, 계좌번호처럼 **엔트로피(경우의 수)가 매우 높은 고유 식별값**에는 실무에서 널리 허용되는 방식입니다.
2. **패딩 오라클(Padding Oracle) 방어:**  
   CBC 모드는 복호화 시 패딩 에러를 이용한 공격에 노출될 수 있습니다. 복호화 실패 시 세부 예외(`BadPaddingException`)를 클라이언트 API 응답으로 노출하지 말고, 항상 통일된 `500 Server Error`나 `400 Bad Request`로 마스킹하여 반환해야 합니다.
3. **DB 쿼리 로그 키 노출 방지:**  
   DB 툴에서 쿼리를 실행할 때 `general_log`에 대칭키나 IV가 남지 않도록 세션 변수를 사용하거나 감사 로그 정책을 확인해야 합니다.

---

## 6. 마무리 요약

1. **스키마 단순화**: `email_index` 같은 불필요한 보조 컬럼 없이 **단일 컬럼**으로 깔끔하게 테이블과 엔티티를 유지할 수 있습니다.
2. **JPA 완벽 호환**: 엔티티와 비즈니스 로직은 평문만 다루며, JPA Converter가 저장/조회 시 자동으로 암복호화 및 파라미터 변환을 처리합니다.
3. **DB 툴 즉시 조회**: DB에 함수를 만들 필요 없이, MySQL 기본 내장 함수(`AES_DECRYPT`)로 DBeaver에서 즉시 복호화 및 조건 검색을 수행할 수 있습니다.
4. **실무 권장**: 스키마 복잡도를 줄이고 DBA/운영 생산성을 극대화해야 하는 비즈니스 환경이라면, **AES-256-CBC 고정 IV 방식**이 가장 실용적이고 균형 잡힌 해답이 될 수 있습니다.
