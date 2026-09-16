---
title: "Spring Boot + JPA 환경에서 사용자 개인정보 암복호화 및 안전한 검색 구현 (AES-256 & Blind Index)"
excerpt: "Spring Boot 2.4.3, Spring Security, JPA 환경에서 개인정보보호법을 준수하며 AES-256-GCM 암복호화와 Blind Index 검색을 구현하는 실무 가이드."
categories:
  - springboot
tags:
  - springboot
  - springsecurity
  - jpa
  - encryption
  - privacy
  - aes256
last_modified_at: 2026-09-16T11:35:00+09:00
toc: true
toc_sticky: true
---

서비스를 개발하고 운영하다 보면 개인정보보호법 및 보안 컴플라이언스(K-ISMS, ISO 27001 등)에 따라 주민등록번호, 계좌번호, 전화번호, 이메일과 같은 **개인정보의 안전한 암호화 저장**이 필수가 됩니다.

단순히 "DB에 저장하기 전에 암호화하고, 꺼내올 때 복호화하면 되지 않나?"라고 생각하기 쉽지만, 실무에서는 다음과 같은 복잡한 문제들에 직면합니다:

1. **비즈니스 로직 침투 문제**: Service 레이어나 Controller 레이어마다 일일이 암복호화 코드가 들어가면 코드가 지저분해지고 누락 실수가 발생합니다.
2. **보안성(암호화 알고리즘)**: 동일한 평문이 항상 같은 암호문으로 치환되는 취약한 ECB 모드 대신 안전한 **AES-256-GCM**을 적용해야 합니다.
3. **암호화된 데이터 검색 문제 (가장 중요)**: AES-GCM은 무작위 IV(Initialization Vector)를 사용하므로 매번 암호문이 바뀝니다. 따라서 `WHERE phone = :encryptedPhone` 검색이 불가능해집니다.

이번 포스팅에서는 **Spring Boot 2.4.3 + Spring Security + Spring Data JPA** 환경에서 **JPA AttributeConverter**를 통해 투명하게 암복호화를 처리하고, **Blind Index(블라인드 인덱스)** 기법으로 암호화된 개인정보를 빠르게 검색하는 실무 아키텍처를 정리합니다.

> **환경 정보**
> - Spring Boot **2.4.3** / Hibernate **5.4.27.Final** (Java EE 기반 → `javax.*` 패키지 사용)
> - **Java 11** (LTS)
> - `javax.crypto` / `javax.security` — JCA/JCE는 Java SE 영역이므로 Spring Boot 버전에 무관하게 동일합니다.
>
> ⚠️ **Java 버전 주의사항**
> - `record` 키워드는 **Java 16+** 정식 지원 (Java 14~15는 preview). Java 11에서는 사용 불가.
> - `HexFormat` 클래스는 **Java 17+**에서만 사용 가능. 본 포스팅은 Java 11 호환 코드로 작성합니다.
> - Spring Boot 3.x로 전환 시 `javax.*` → `jakarta.*` 패키지 전환이 필요합니다.

---

## 전체 아키텍처 개요

```mermaid
graph TD
    A[CryptoConfig] -->|@Bean| B[AesGcmCrypto]
    A -->|@Bean| C[BlindIndexUtil]
    A -->|@Bean| D[PasswordEncoder]
    E[HibernateConfig] -->|SpringBeanContainer 연결| F[PrivacyEncryptConverter]
    B -->|@Qualifier 생성자 주입| F
    F -->|@Convert 자동 암복호화| G[Member Entity]
    C -->|이메일·전화 인덱스 생성| H[MemberService]
    D -->|비밀번호 해싱| H
    H -->|save / findBy...Index| I[(Database)]
    G --> I
```

> `HibernateConfig`가 없으면 Hibernate 5에서 `PrivacyEncryptConverter`에 Spring 의존성이 주입되지 않습니다.
> `@Qualifier("aesGcmCrypto")`로 빈 이름을 명시하면 키 로테이션 시 다수의 `AesGcmCrypto` 빈이 등록돼도 충돌이 발생하지 않습니다.

---

## 1. 의존성 설정 (pom.xml)

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.3</version>
    <relativePath/>
</parent>

<dependencies>
    <!-- Web MVC + Jackson (JSON 역직렬화에 필수) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA / Hibernate 5.4.27.Final -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Spring Security (PasswordEncoder) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Spring ORM — SpringBeanContainer 포함 (HibernateConfig에서 필요)
         spring-boot-starter-data-jpa가 transitively 포함하므로 보통 자동으로 제공됨.
         명시적으로 선언하면 의존성 의도가 명확해짐. -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-orm</artifactId>
    </dependency>

    <!-- Bean Validation (입력값 검증: @NotBlank, @Email 등) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

---

## 2. 단방향 vs 양방향 암호화 기준 정립

개인정보 필드의 성격에 따라 암호화 방식이 완전히 달라져야 합니다.

| 구분 | 대상 데이터 | 방식 | 권장 알고리즘 | 복호화 필요 여부 |
| :--- | :--- | :--- | :--- | :--- |
| **단방향 암호화** | 비밀번호 | 해시 (Hash + Salt) | BCrypt, Argon2, PBKDF2 | **불가능 (단방향)** |
| **양방향 암호화** | 전화번호, 이메일, 계좌번호, 주민번호 등 | 대칭키 암호화 | AES-256-GCM | **필요 (복호화 가능)** |

- **비밀번호**: 절대 복호화할 수 없어야 하므로 Spring Security의 `PasswordEncoder` (`BCryptPasswordEncoder`)를 사용합니다.
- **개인정보**: SMS 발송, 본인 인증, 이메일 발송 등을 위해 원문 복원이 필요하므로 안전한 **AES-256-GCM** 대칭키 암호화를 사용합니다.

---

## 3. 왜 AES-256-GCM인가?

과거에는 `AES/CBC/PKCS5Padding` 모드를 주로 사용했으나, CBC 모드는 데이터 무결성 검증을 위해 별도의 HMAC을 붙여야 하고 패딩 오라클 공격(Padding Oracle Attack)에 취약할 수 있습니다.

반면 **AES-GCM (Galois/Counter Mode)**은:
- **AEAD (Authenticated Encryption with Associated Data)**를 지원하여 암호화와 무결성/인증 검증(Authentication Tag)을 동시에 수행합니다.
- 패딩을 사용하지 않아 패딩 공격으로부터 안전합니다.
- 매번 12바이트의 무작위 IV(Nonce)를 생성하여 암호문과 함께 보관하므로, 동일한 평문이라도 매번 완전히 다른 암호문이 생성되어 빈도 분석 공격을 원천 차단합니다.

---

## 4. AES-256-GCM 암복호화 컴포넌트 (순수 POJO)

유틸리티 클래스에 `@Component`나 `@Value`를 직접 붙이지 않고 **순수 POJO**로 작성합니다.
이렇게 하면 스프링 컨텍스트를 띄우지 않고도 초경량 단위 테스트를 작성할 수 있습니다.

```java
package com.example.security.crypto;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.security.SecureRandom;
import java.util.Base64;

// 스프링 어노테이션 없는 순수 POJO — CryptoConfig에서 @Bean으로 등록
public class AesGcmCrypto {

    private static final String ALGORITHM = "AES";
    private static final String TRANSFORMATION = "AES/GCM/NoPadding";
    private static final int GCM_TAG_LENGTH = 128; // bit
    private static final int IV_LENGTH = 12;       // 12 bytes recommended for GCM

    private final SecretKey secretKey;
    private final SecureRandom secureRandom = new SecureRandom();

    /**
     * @param secretKeyBase64 Base64 인코딩된 32바이트(256비트) AES 키
     *                        생성: openssl rand -base64 32
     */
    public AesGcmCrypto(String secretKeyBase64) {
        byte[] decodedKey = Base64.getDecoder().decode(secretKeyBase64);
        if (decodedKey.length != 32) {
            throw new IllegalArgumentException(
                "AES-256 키는 반드시 32바이트(256비트)여야 합니다. 현재: " + decodedKey.length + "바이트");
        }
        this.secretKey = new SecretKeySpec(decodedKey, ALGORITHM);
    }

    /**
     * 평문 -> AES-256-GCM 암호화 (IV + CipherText + Tag) -> Base64
     *
     * <p><b>빈 문자열 처리 정책</b>: null은 null로, "" (빈 문자열)은 ""로 그대로 반환합니다.
     * DB에 빈 문자열이 저장될 수 있는 경우, 서비스 레이어에서 빈 문자열 입력을 사전에 차단하세요.
     */
    public String encrypt(String plainText) {
        if (plainText == null || plainText.isEmpty()) {
            return plainText; // null → null, "" → "" (암호화 없이 그대로 반환)
        }
        try {
            byte[] iv = new byte[IV_LENGTH];
            secureRandom.nextBytes(iv);

            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.ENCRYPT_MODE, secretKey, new GCMParameterSpec(GCM_TAG_LENGTH, iv));

            byte[] cipherText = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));

            // IV(12바이트) + 암호문(가변) 결합 후 Base64 인코딩
            ByteBuffer buf = ByteBuffer.allocate(iv.length + cipherText.length);
            buf.put(iv);
            buf.put(cipherText);
            return Base64.getEncoder().encodeToString(buf.array());
        } catch (Exception e) {
            throw new IllegalStateException("데이터 암호화 중 오류 발생", e);
        }
    }

    /**
     * Base64 암호문 -> IV 분리 -> AES-256-GCM 복호화 -> 평문
     *
     * <p><b>빈 문자열 처리 정책</b>: null은 null로, "" (빈 문자열)은 ""로 그대로 반환합니다.
     * encrypt()와 동일한 정책으로 대칭성을 유지합니다.
     */
    public String decrypt(String encryptedBase64) {
        if (encryptedBase64 == null || encryptedBase64.isEmpty()) {
            return encryptedBase64; // null → null, "" → "" (복호화 없이 그대로 반환)
        }
        try {
            byte[] cipherMessage = Base64.getDecoder().decode(encryptedBase64);
            if (cipherMessage.length < IV_LENGTH) {
                throw new IllegalArgumentException("유효하지 않은 암호문 길이입니다.");
            }

            ByteBuffer buf = ByteBuffer.wrap(cipherMessage);
            byte[] iv = new byte[IV_LENGTH];
            buf.get(iv);
            byte[] cipherText = new byte[buf.remaining()];
            buf.get(cipherText);

            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.DECRYPT_MODE, secretKey, new GCMParameterSpec(GCM_TAG_LENGTH, iv));

            return new String(cipher.doFinal(cipherText), StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new IllegalStateException("데이터 복호화 중 오류 발생", e);
        }
    }
}
```

---

## 5. Blind Index 유틸 (순수 POJO)

> **주의**: `Mac` 인스턴스는 스레드 안전하지 않습니다. 매 호출마다 `Mac.getInstance()`로 새 인스턴스를 생성하는 것이 올바른 패턴입니다. `SecretKeySpec`은 불변(Immutable) 객체이므로 필드로 공유해도 안전합니다.

### AES 키 vs HMAC 키 처리 방식 비교

| 키 종류 | 생성 명령 | `CryptoConfig`에서의 처리 | 이유 |
| :--- | :--- | :--- | :--- |
| **AES 키** (`secret-key`) | `openssl rand -base64 32` | `Base64.decode()` → 32바이트 | AES는 정확히 32바이트 필요 |
| **HMAC 키** (`blind-index-key`) | `openssl rand -hex 64` | `getBytes(UTF_8)` → 64바이트 | HMAC-SHA256은 32바이트 이상이면 유효 |

### 이메일과 전화번호의 정규화 규칙은 달라야 합니다

**잘못된 접근**: 단일 메서드에서 점(`.`)까지 제거하면 이메일이 깨집니다.

```
// ❌ 잘못된 정규화 예시
"user@example.com" → "user@examplecom"  (도메인의 점을 제거해버림!)
```

**올바른 접근**: 도메인 유형별 정규화 메서드를 분리합니다.

```java
package com.example.security.crypto;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

// 스프링 어노테이션 없는 순수 POJO — CryptoConfig에서 @Bean으로 등록
public class BlindIndexUtil {

    private static final String HMAC_SHA256 = "HmacSHA256";
    private static final int MIN_KEY_BYTES = 32; // 최소 32바이트(256비트)
    // SecretKeySpec은 불변(Immutable) — 필드 공유 안전
    private final SecretKeySpec keySpec;

    /**
     * @param indexKey 최소 32바이트 이상의 HMAC 키 원문(UTF-8)
     *                 생성: openssl rand -hex 64  (64 hex chars = 64 ASCII bytes)
     *                 주의: AES 키처럼 Base64 디코딩하지 않고 UTF-8 바이트 그대로 사용합니다.
     */
    public BlindIndexUtil(String indexKey) {
        byte[] keyBytes = indexKey.getBytes(StandardCharsets.UTF_8);
        if (keyBytes.length < MIN_KEY_BYTES) {
            throw new IllegalArgumentException(
                "Blind Index 키는 최소 32바이트(256비트) 이상이어야 합니다. 현재: "
                    + keyBytes.length + "바이트. openssl rand -hex 64 로 생성하세요.");
        }
        this.keySpec = new SecretKeySpec(keyBytes, HMAC_SHA256);
    }

    /**
     * 이메일용 블라인드 인덱스 생성.
     * 정규화: 앞뒤 공백 제거 + 소문자 변환만 적용.
     * 이메일 주소의 점(.)과 @ 기호는 도메인 식별자이므로 제거하지 않습니다.
     *
     * <p>예: "User@Example.COM" → "user@example.com"
     */
    public String generateEmailIndex(String email) {
        if (email == null || email.trim().isEmpty()) {
            return null;
        }
        String normalized = email.trim().toLowerCase();
        return computeHmac(normalized);
    }

    /**
     * 전화번호용 블라인드 인덱스 생성.
     * 정규화: 선두의 + 기호(국제번호)는 유지하고, 나머지는 숫자만 남깁니다.
     * 중간·끝의 + 기호는 제거하여 비정상 입력을 방어합니다.
     *
     * <p>예: "010-1234-5678"    → "01012345678"
     * <p>예: "+82-10-1234-5678" → "+821012345678"
     */
    public String generatePhoneIndex(String phoneNumber) {
        if (phoneNumber == null || phoneNumber.trim().isEmpty()) {
            return null;
        }
        String trimmed = phoneNumber.trim();
        // 선두의 + 기호만 유지 (국제번호 형식), 나머지는 숫자만 추출
        boolean hasLeadingPlus = trimmed.startsWith("+");
        String digits = trimmed.replaceAll("[^0-9]", "");
        String normalized = hasLeadingPlus ? "+" + digits : digits;
        return computeHmac(normalized);
    }

    /**
     * HMAC-SHA256 해시 계산 공통 메서드.
     * Mac은 스레드 안전하지 않으므로 호출마다 새 인스턴스를 생성합니다.
     *
     * @return HEX 소문자 64자리 문자열 (HMAC-SHA256 = 32바이트 = HEX 64자)
     */
    private String computeHmac(String normalized) {
        try {
            // Mac은 스레드 안전하지 않으므로 매번 새 인스턴스 생성
            Mac mac = Mac.getInstance(HMAC_SHA256);
            mac.init(keySpec);
            byte[] hashBytes = mac.doFinal(normalized.getBytes(StandardCharsets.UTF_8));

            // Java 11 호환 HEX 변환 (HexFormat은 Java 17+에서만 사용 가능)
            StringBuilder sb = new StringBuilder(hashBytes.length * 2);
            for (byte b : hashBytes) {
                sb.append(String.format("%02x", b));
            }
            return sb.toString();
        } catch (Exception e) {
            throw new IllegalStateException("블라인드 인덱스 생성 실패", e);
        }
    }
}
```

---

## 6. JPA AttributeConverter — Spring Boot 2.x의 핵심 함정

JPA의 `AttributeConverter`를 활용하면 엔티티 필드가 DB에 저장될 때 자동으로 암호화되고, 조회될 때 자동으로 복호화됩니다.

### Hibernate 5 (Spring Boot 2.x) 에서의 주입 문제

Spring Boot 2.x가 사용하는 **Hibernate 5.4.27.Final**에서는 `AttributeConverter` 인스턴스를 기본적으로 **Hibernate가 직접 생성**합니다. 즉, `@Component`를 붙여도 **Spring이 관리하는 인스턴스가 아니라** Hibernate가 별도로 만든 인스턴스가 사용되므로, 일반적인 Spring 생성자 주입이 동작하지 않습니다.

```
// 함정: Spring Bean과 Hibernate가 사용하는 인스턴스가 다른 객체!
[Spring 컨텍스트]               [Hibernate 내부]
@Component Bean → 주입됨         new PrivacyEncryptConverter()  ← 이 인스턴스를 실제로 사용
                                 → crypto = null
                                 → NullPointerException 발생!
```

> **Spring Boot 3.x (Hibernate 6.x)**에서는 Spring의 `BeanContainer` 통합이 개선되어 `HibernateConfig` 없이도 생성자 주입이 기본 동작합니다.

### 해결책: HibernatePropertiesCustomizer로 SpringBeanContainer 연결

`HibernatePropertiesCustomizer`를 통해 Hibernate에 Spring의 `BeanContainer`를 등록하면, Hibernate가 `AttributeConverter` 인스턴스를 생성할 때 Spring 컨텍스트에서 가져오게 되어 **생성자 주입이 정상 동작**합니다.

> **빈 초기화 순서**: `CryptoConfig` → `AesGcmCrypto` Bean 등록 → `HibernateConfig`가 `SpringBeanContainer` 연결 → Hibernate가 `PrivacyEncryptConverter` 생성 시 Spring에서 `AesGcmCrypto`를 가져옴. Spring Boot의 자동 구성이 순서를 보장합니다.

```java
package com.example.security.config;

import org.hibernate.cfg.AvailableSettings;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.boot.autoconfigure.orm.jpa.HibernatePropertiesCustomizer;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.hibernate5.SpringBeanContainer;

import java.util.Map;

@Configuration
public class HibernateConfig implements HibernatePropertiesCustomizer {

    private final ConfigurableListableBeanFactory beanFactory;

    public HibernateConfig(ConfigurableListableBeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    /**
     * Hibernate에 Spring의 BeanContainer를 등록합니다.
     * 이를 통해 AttributeConverter 등 Hibernate 내부 객체에 Spring 의존성 주입이 가능해집니다.
     * spring-orm 모듈의 SpringBeanContainer를 사용합니다.
     */
    @Override
    public void customize(Map<String, Object> hibernateProperties) {
        hibernateProperties.put(
            AvailableSettings.BEAN_CONTAINER,
            new SpringBeanContainer(beanFactory)
        );
    }
}
```

이제 `PrivacyEncryptConverter`에서 **생성자 주입이 정상 동작**합니다:

```java
package com.example.security.crypto;

import javax.persistence.AttributeConverter;  // Spring Boot 2.x: javax.*
import javax.persistence.Converter;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

// autoApply = false (기본값) — 명시적으로 @Convert 어노테이션을 붙인 필드에만 적용
// ⚠️ autoApply = true 로 변경하면 모든 String 타입 필드에 자동 적용되어 의도치 않은 암호화가 발생합니다!
@Converter
@Component
public class PrivacyEncryptConverter implements AttributeConverter<String, String> {

    private final AesGcmCrypto crypto;

    // @Qualifier로 빈 이름을 명시합니다.
    // 키 로테이션 프로파일 활성화 시 AesGcmCrypto 빈이 여러 개 등록되어도
    // "aesGcmCrypto" 빈만 정확히 주입받아 NoUniqueBeanDefinitionException을 방지합니다.
    public PrivacyEncryptConverter(@Qualifier("aesGcmCrypto") AesGcmCrypto aesGcmCrypto) {
        this.crypto = aesGcmCrypto;
    }

    @Override
    public String convertToDatabaseColumn(String attribute) {
        if (attribute == null) return null;
        return crypto.encrypt(attribute);
    }

    @Override
    public String convertToEntityAttribute(String dbData) {
        if (dbData == null) return null;
        return crypto.decrypt(dbData);
    }
}
```

이제 엔티티 클래스에서 암호화가 필요한 필드에 `@Convert(converter = PrivacyEncryptConverter.class)` 어노테이션만 붙여주면 끝납니다.

---

## 7. 보안 설정 집중화: @Configuration + @Bean (CryptoConfig)

> **Q. 유틸 클래스에 `@Component`를 직접 붙이지 않고 왜 `@Configuration + @Bean` 방식을 쓸까요?**
>
> 1. **보안 설정의 가시성**: `CryptoConfig` 클래스 하나만 열면 시스템 내의 모든 암호화 키 주입, 알고리즘, 빈을 한눈에 확인할 수 있습니다. (보안 감사에 유리)
> 2. **순수 단위 테스트**: 유틸이 POJO이므로 `new AesGcmCrypto("base64key...")` 한 줄로 스프링 컨텍스트 없이 단위 테스트가 가능합니다.
> 3. **CGLIB 프록시 오버헤드 방지**: 유틸 클래스 자체에 `@Configuration`을 붙이면 불필요한 프록시가 생성됩니다.

```java
package com.example.security.config;

import com.example.security.crypto.AesGcmCrypto;
import com.example.security.crypto.BlindIndexUtil;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class CryptoConfig {

    /**
     * AES-256 암호화 키 (Base64 디코딩 후 32바이트)
     * 생성: openssl rand -base64 32
     * 빈 이름: "aesGcmCrypto" — PrivacyEncryptConverter가 @Qualifier로 이 이름을 참조합니다.
     */
    @Bean
    public AesGcmCrypto aesGcmCrypto(@Value("${app.crypto.secret-key}") String secretKey) {
        return new AesGcmCrypto(secretKey);
    }

    /**
     * Blind Index HMAC 키 (UTF-8 바이트, 최소 32바이트)
     * 생성: openssl rand -hex 64  (64 hex chars = 64 ASCII bytes)
     * 주의: AES 키와 달리 Base64 디코딩하지 않고 UTF-8 원문 그대로 사용합니다.
     */
    @Bean
    public BlindIndexUtil blindIndexUtil(@Value("${app.crypto.blind-index-key}") String indexKey) {
        return new BlindIndexUtil(indexKey);
    }

    // PasswordEncoder도 한 곳에서 관리 — 보안 관련 빈 집중화
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

## 8. 엔티티(Entity) 설계

비밀번호는 `PasswordEncoder`로 단방향 해싱하고, 전화번호와 이메일은 양방향 암호화 및 Blind Index를 적용합니다.

> **설계 원칙**: JPA 엔티티는 Spring Bean이 아닙니다. `@Autowired`를 엔티티 필드에 붙여도 Spring이 주입해 주지 않습니다.
> 엔티티를 **순수 도메인 객체**로 유지하기 위해 `BlindIndexUtil`을 생성자에 직접 전달하는 대신, **미리 계산된 인덱스 값**을 받습니다.

```java
package com.example.domain.member;

import com.example.security.crypto.PrivacyEncryptConverter;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;   // Spring Boot 2.x: javax.*

@Entity
@Getter
@Table(
    name = "member",
    uniqueConstraints = {
        @UniqueConstraint(name = "uk_member_email_index", columnNames = "email_index"),
        @UniqueConstraint(name = "uk_member_phone_index", columnNames = "phone_index")
    },
    indexes = {
        @Index(name = "idx_member_email_index", columnList = "email_index"),
        @Index(name = "idx_member_phone_index", columnList = "phone_index")
    }
)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    // 단방향 암호화 (Spring Security BCrypt) — 평문 절대 저장 금지
    @Column(nullable = false)
    private String password;

    // 양방향 암호화 (JPA Converter 자동 암복호화)
    @Convert(converter = PrivacyEncryptConverter.class)
    @Column(nullable = false, length = 500)
    private String email;

    // 검색용 블라인드 인덱스 (HMAC-SHA256) — VARCHAR(64) = HEX(32바이트)
    @Column(name = "email_index", nullable = false, length = 64)
    private String emailIndex;

    // 양방향 암호화 (JPA Converter 자동 암복호화)
    @Convert(converter = PrivacyEncryptConverter.class)
    @Column(nullable = false, length = 500)
    private String phoneNumber;

    // 검색용 블라인드 인덱스 (HMAC-SHA256)
    @Column(name = "phone_index", nullable = false, length = 64)
    private String phoneIndex;

    /**
     * 정적 팩터리 메서드 — 순수 도메인 객체 생성.
     *
     * <p><b>중요</b>: 이 메서드는 {@code Member} 클래스 내부에 선언되어 있으므로
     * {@code private} 필드에 직접 접근할 수 있습니다. 외부 클래스에서 이 패턴을 복사하면
     * 컴파일 에러가 발생합니다.
     *
     * <p>BlindIndexUtil 의존성 없이 미리 계산된 인덱스 값을 받아 순수 도메인 객체를 유지합니다.
     * 인덱스 계산은 Service 레이어의 책임입니다.
     */
    public static Member create(String name, String encodedPassword,
                                String email, String emailIndex,
                                String phoneNumber, String phoneIndex) {
        Member member = new Member(); // 같은 클래스 내부이므로 protected 생성자 접근 가능
        member.name = name;
        member.password = encodedPassword;
        member.email = email;
        member.emailIndex = emailIndex;
        member.phoneNumber = phoneNumber;
        member.phoneIndex = phoneIndex;
        return member;
    }

    /**
     * 연락처 변경 — 미리 계산된 인덱스 값을 받아 업데이트합니다.
     * 인덱스 계산은 Service 레이어의 책임입니다.
     */
    public void updateContact(String email, String emailIndex,
                              String phoneNumber, String phoneIndex) {
        this.email = email;
        this.emailIndex = emailIndex;
        this.phoneNumber = phoneNumber;
        this.phoneIndex = phoneIndex;
    }
}
```

---

## 9. DTO 및 Repository, Service, Controller 구현

### DTO

> **Java 11 + Jackson 주의**: `@RequiredArgsConstructor`로 생성된 `final` 필드 전체 생성자는 Jackson이 인식하지 못합니다. Jackson은 기본적으로 인수 없는 기본 생성자로 역직렬화하기 때문입니다.  
> Java 11에서는 `@JsonCreator` + `@JsonProperty`로 Jackson이 사용할 생성자를 명시합니다.

```java
package com.example.domain.member;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Getter;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.Size;

// Java 11 호환 DTO — @JsonCreator로 Jackson 역직렬화 명시
// Java 16+ 사용 시 record로 교체 가능 (record는 @JsonCreator 불필요)
@Getter
public class MemberRegisterRequest {

    @NotBlank(message = "이름은 필수입니다.")
    @Size(max = 100)
    private final String name;

    @NotBlank(message = "비밀번호는 필수입니다.")
    @Size(min = 8, message = "비밀번호는 최소 8자 이상이어야 합니다.")
    private final String password;

    @NotBlank(message = "이메일은 필수입니다.")
    @Email(message = "유효한 이메일 형식이어야 합니다.")
    private final String email;

    @NotBlank(message = "전화번호는 필수입니다.")
    @Pattern(regexp = "^[+]?[0-9\\-() ]+$", message = "유효한 전화번호 형식이어야 합니다.")
    private final String phoneNumber;

    // Jackson이 @RequestBody 역직렬화 시 이 생성자를 사용
    // @RequiredArgsConstructor는 @JsonProperty를 포함하지 않으므로 직접 선언합니다.
    @JsonCreator
    public MemberRegisterRequest(
            @JsonProperty("name")        String name,
            @JsonProperty("password")    String password,
            @JsonProperty("email")       String email,
            @JsonProperty("phoneNumber") String phoneNumber) {
        this.name = name;
        this.password = password;
        this.email = email;
        this.phoneNumber = phoneNumber;
    }
}
```

```java
package com.example.domain.member;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

// 응답 DTO는 Jackson이 getters로 직렬화하므로 @RequiredArgsConstructor 그대로 사용 가능
@Getter
@RequiredArgsConstructor
public class MemberResponse {
    private final Long id;
    private final String name;
    private final String email;       // 복호화된 평문
    private final String phoneNumber; // 복호화된 평문

    // ⚠️ 개인정보 로깅 방지 — toString()에서 이메일·전화번호 마스킹 필수
    @Override
    public String toString() {
        return "MemberResponse{id=" + id + ", name='" + name + "', "
            + "email='***', phoneNumber='***'}";
    }
}
```

### MemberRepository

```java
package com.example.domain.member;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface MemberRepository extends JpaRepository<Member, Long> {

    // 블라인드 인덱스로 B-Tree 인덱스 활용 초고속 조회
    Optional<Member> findByEmailIndex(String emailIndex);
    Optional<Member> findByPhoneIndex(String phoneIndex);

    // 중복 가입 사전 검증용
    boolean existsByEmailIndex(String emailIndex);
    boolean existsByPhoneIndex(String phoneIndex);
}
```

### MemberService

```java
package com.example.domain.member;

import com.example.security.crypto.BlindIndexUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final BlindIndexUtil blindIndexUtil;

    @Transactional
    public Long register(MemberRegisterRequest request) {
        // ① 이메일·전화번호 각각 도메인에 맞는 정규화로 인덱스 생성
        String emailIndex = blindIndexUtil.generateEmailIndex(request.getEmail());
        String phoneIndex = blindIndexUtil.generatePhoneIndex(request.getPhoneNumber());

        // ② 블라인드 인덱스로 중복 사전 검증 (DB UniqueConstraint에만 의존하지 않음)
        if (memberRepository.existsByEmailIndex(emailIndex)) {
            throw new IllegalArgumentException("이미 사용 중인 이메일입니다.");
        }
        if (memberRepository.existsByPhoneIndex(phoneIndex)) {
            throw new IllegalArgumentException("이미 사용 중인 전화번호입니다.");
        }

        String encodedPassword = passwordEncoder.encode(request.getPassword());

        // ③ 미리 계산된 인덱스를 엔티티에 전달 — 엔티티는 BlindIndexUtil 의존성 없음
        //    email, phoneNumber는 save() 시점에 PrivacyEncryptConverter가 자동 암호화
        Member member = Member.create(
            request.getName(),
            encodedPassword,
            request.getEmail(),
            emailIndex,
            request.getPhoneNumber(),
            phoneIndex
        );

        try {
            return memberRepository.save(member).getId();
        } catch (DataIntegrityViolationException e) {
            // Race Condition 대응: 사전 검증 통과 후 다른 스레드가 먼저 저장한 경우
            // DB의 UniqueConstraint가 최후의 방어선으로 동작하며 이 예외로 잡아 친절한 메시지 반환
            throw new IllegalArgumentException("이미 사용 중인 이메일 또는 전화번호입니다.");
        }
    }

    public MemberResponse findByEmail(String plainEmail) {
        // 평문 이메일 → 블라인드 인덱스 변환 → B-Tree 인덱스 검색
        String emailIndex = blindIndexUtil.generateEmailIndex(plainEmail);

        Member member = memberRepository.findByEmailIndex(emailIndex)
            .orElseThrow(() -> new IllegalArgumentException("회원을 찾을 수 없습니다."));

        // getEmail() 호출 시 PrivacyEncryptConverter가 자동 복호화하여 평문 반환
        return new MemberResponse(
            member.getId(),
            member.getName(),
            member.getEmail(),
            member.getPhoneNumber()
        );
    }

    @Transactional
    public void updateContact(Long memberId, String newEmail, String newPhoneNumber) {
        Member member = memberRepository.findById(memberId)
            .orElseThrow(() -> new IllegalArgumentException("회원을 찾을 수 없습니다."));

        String newEmailIndex = blindIndexUtil.generateEmailIndex(newEmail);
        String newPhoneIndex = blindIndexUtil.generatePhoneIndex(newPhoneNumber);

        // 변경 사항이 없으면 조기 반환 — 불필요한 DB 쓰기 방지
        if (newEmailIndex.equals(member.getEmailIndex())
                && newPhoneIndex.equals(member.getPhoneIndex())) {
            return;
        }

        // 변경 전 중복 검증 (본인 제외)
        memberRepository.findByEmailIndex(newEmailIndex)
            .filter(found -> !found.getId().equals(memberId))
            .ifPresent(found -> { throw new IllegalArgumentException("이미 사용 중인 이메일입니다."); });
        memberRepository.findByPhoneIndex(newPhoneIndex)
            .filter(found -> !found.getId().equals(memberId))
            .ifPresent(found -> { throw new IllegalArgumentException("이미 사용 중인 전화번호입니다."); });

        // 미리 계산된 인덱스와 평문을 전달 — Converter가 save 시점에 자동 암호화
        member.updateContact(newEmail, newEmailIndex, newPhoneNumber, newPhoneIndex);

        // Hibernate 5에서 AttributeConverter가 붙은 필드의 더티 체킹은
        // 구현에 따라 신뢰성이 다를 수 있으므로 명시적으로 save()를 호출합니다.
        memberRepository.save(member);
    }
}
```

### Controller + @Valid + 전역 예외 처리

`@Valid`는 Controller 파라미터에 붙이고, 검증 실패 및 비즈니스 예외는 `@RestControllerAdvice`에서 일괄 처리합니다.

```java
package com.example.web;

import com.example.domain.member.MemberRegisterRequest;
import com.example.domain.member.MemberResponse;
import com.example.domain.member.MemberService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/members")
public class MemberController {

    private final MemberService memberService;

    // @Valid — MemberRegisterRequest의 @NotBlank, @Email 등을 활성화
    @PostMapping
    public ResponseEntity<Long> register(@RequestBody @Valid MemberRegisterRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(memberService.register(request));
    }

    @GetMapping("/search")
    public ResponseEntity<MemberResponse> findByEmail(@RequestParam String email) {
        return ResponseEntity.ok(memberService.findByEmail(email));
    }

    @PatchMapping("/{memberId}/contact")
    public ResponseEntity<Void> updateContact(
            @PathVariable Long memberId,
            @RequestParam String email,
            @RequestParam String phoneNumber) {
        memberService.updateContact(memberId, email, phoneNumber);
        return ResponseEntity.noContent().build();
    }
}
```

```java
package com.example.web;

import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.LinkedHashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // @Valid 검증 실패 — 각 필드별 오류 메시지 반환
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }

    // Race Condition 등 DB UniqueConstraint 위반 — 최후의 방어선
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<Map<String, String>> handleDuplicateKey(DataIntegrityViolationException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body(Map.of("error", "이미 사용 중인 이메일 또는 전화번호입니다."));
    }

    // 비즈니스 규칙 위반 (중복 가입, 회원 없음 등)
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleIllegalArg(IllegalArgumentException ex) {
        return ResponseEntity.badRequest().body(Map.of("error", ex.getMessage()));
    }
}
```

---

## 10. 환경 설정 (application.yml) 및 키 관리

암호화 키와 블라인드 인덱스 키는 절대 소스코드나 Git 저장소에 커밋되면 안 되며, 환경변수나 Secret Manager(AWS Secrets Manager, HashiCorp Vault 등)를 통해 주입받아야 합니다.

### application.yml

```yaml
app:
  crypto:
    # AES-256 암호화 키 (Base64, 32바이트) — 환경변수로 주입
    # 생성: openssl rand -base64 32
    secret-key: ${CRYPTO_SECRET_KEY}

    # Blind Index HMAC 키 (UTF-8 원문, 최소 32바이트) — 환경변수로 주입
    # 생성: openssl rand -hex 64
    # 주의: AES 키와 달리 Base64 디코딩하지 않고 UTF-8 원문 그대로 사용합니다.
    blind-index-key: ${BLIND_INDEX_KEY}
```

### 암호화 키 생성 방법 (Terminal / OpenSSL)

```bash
# AES-256용 32바이트 랜덤 키 생성 (Base64 → CRYPTO_SECRET_KEY에 사용)
openssl rand -base64 32

# Blind Index용 HMAC 키 생성 (HEX 64자 = UTF-8 64바이트 → BLIND_INDEX_KEY에 사용)
# AES 키와 달리 Base64 디코딩 없이 문자열 그대로 사용하므로 hex 출력을 사용합니다.
openssl rand -hex 64
```

---

## 11. 키 로테이션(Key Rotation) 전략

실무에서 암호화 키는 보안 정책에 따라 주기적으로 교체해야 합니다. 이미 암호화된 DB 데이터가 대용량으로 쌓여있다면 다음 전략을 고려합니다.

### 기본 전략: 배치 재암호화 (Batch Re-encryption)

```
[구 키로 암호화된 데이터] → 복호화 → [평문] → 새 키로 암호화 → [DB Update]
```

1. 새 키를 환경변수에 추가(`CRYPTO_SECRET_KEY_NEW`)하되, **구 키는 아직 유지**합니다.
2. **마이그레이션 배치 Job**을 작성하여 레코드를 청크(chunk) 단위로 읽어와 구 키로 복호화 → 새 키로 재암호화 → Update합니다.
3. 배치 완료 후 환경변수를 새 키로 교체하고 서버를 재시작합니다.

### @Qualifier로 두 개의 암호화 빈 구분하기

배치 처리 시 구 키와 새 키를 동시에 사용하려면 `@Qualifier`로 빈 이름을 구분합니다.

> ⚠️ `key-rotation` 프로파일 활성화 시 `AesGcmCrypto` 빈이 `aesGcmCrypto`, `oldCrypto`, `newCrypto` 세 개가 됩니다.  
> `PrivacyEncryptConverter`는 `@Qualifier("aesGcmCrypto")`로 정확한 빈만 주입받으므로 `NoUniqueBeanDefinitionException` 충돌이 발생하지 않습니다.

```java
package com.example.security.config;

import com.example.security.crypto.AesGcmCrypto;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("key-rotation") // 키 로테이션 배치 실행 시에만 활성화
public class KeyRotationConfig {

    @Bean("oldCrypto")
    public AesGcmCrypto oldAesGcmCrypto(
            @Value("${app.crypto.old-secret-key}") String oldKey) {
        return new AesGcmCrypto(oldKey);
    }

    @Bean("newCrypto")
    public AesGcmCrypto newAesGcmCrypto(
            @Value("${app.crypto.new-secret-key}") String newKey) {
        return new AesGcmCrypto(newKey);
    }
}
```

```java
package com.example.batch;

import com.example.domain.member.Member;
import com.example.domain.member.MemberRepository;
import com.example.security.crypto.AesGcmCrypto;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

// @RequiredArgsConstructor 미사용 — @Qualifier는 Lombok이 처리하지 못하므로 생성자를 직접 선언합니다.
@Service
public class KeyRotationService {

    private final AesGcmCrypto oldCrypto;
    private final AesGcmCrypto newCrypto;
    private final MemberRepository memberRepository;
    private final JdbcTemplate jdbcTemplate;

    public KeyRotationService(
            @Qualifier("oldCrypto") AesGcmCrypto oldCrypto,
            @Qualifier("newCrypto") AesGcmCrypto newCrypto,
            MemberRepository memberRepository,
            JdbcTemplate jdbcTemplate) {
        this.oldCrypto = oldCrypto;
        this.newCrypto = newCrypto;
        this.memberRepository = memberRepository;
        this.jdbcTemplate = jdbcTemplate;
    }

    /**
     * JPA Pageable로 페이지 단위 조회 — DB 방언에 무관하게 동작합니다.
     * (Hibernate가 각 DB 방언에 맞는 페이징 SQL을 자동 생성)
     *
     * <p>PrivacyEncryptConverter가 JPA 조회 시 구 키(aesGcmCrypto)로 자동 복호화합니다.
     * 복호화된 평문을 새 키로 재암호화한 뒤 JdbcTemplate으로 직접 업데이트합니다.
     * (JPA save()는 Converter가 다시 암호화하므로 여기서는 JdbcTemplate 직접 업데이트 사용)
     */
    @Transactional
    public int rotateChunk(int page, int size) {
        // JPA Pageable — DB 방언 무관 (MySQL LIMIT/OFFSET 아님)
        Page<Member> members = memberRepository.findAll(PageRequest.of(page, size));

        for (Member member : members.getContent()) {
            // member.getEmail(): Converter가 구 키로 자동 복호화하여 평문 반환
            String newEncEmail = newCrypto.encrypt(member.getEmail());
            String newEncPhone = newCrypto.encrypt(member.getPhoneNumber());

            // JdbcTemplate으로 직접 업데이트 — Converter를 거치지 않아 이중 암호화 방지
            jdbcTemplate.update(
                "UPDATE member SET email = ?, phone_number = ? WHERE id = ?",
                newEncEmail, newEncPhone, member.getId()
            );
        }
        return members.getNumberOfElements();
    }
}
```

### 고급 전략: Envelope Encryption

**AWS KMS, GCP KMS, HashiCorp Vault**와 같은 Key Management Service를 사용하면:
- 데이터 암호화에 사용하는 **DEK(Data Encryption Key)**를 KMS의 **KEK(Key Encryption Key)**로 한번 더 암호화하여 DB에 저장합니다.
- 키 교체 시 DEK만 재암호화하면 되므로 전체 데이터 재처리가 불필요합니다.
- 대규모 운영 서비스의 표준적인 접근 방식입니다.

---

## 12. 실무 체크리스트 및 주의사항

1. **DB 컬럼 길이 고려**:
   - `IV(12 bytes) + CipherText + Auth Tag(16 bytes)` → Base64 인코딩으로 길이가 크게 늘어납니다.
   - 원문이 15글자라도 암호문은 60~80자 이상이므로 컬럼 길이를 넉넉히(`VARCHAR(500)`) 잡아야 합니다.

2. **`@Converter(autoApply = true)` 절대 금지**:
   - `autoApply = true`로 설정하면 **모든 `String` 타입 필드**에 자동으로 암호화가 적용됩니다.
   - `name`, `password` 등 암호화해서는 안 되는 필드까지 암호화되어 기능이 완전히 망가집니다.
   - 반드시 기본값인 `autoApply = false`를 유지하고 `@Convert(converter = PrivacyEncryptConverter.class)`를 필요한 필드에만 명시하세요.

3. **이메일/전화번호 정규화 규칙 분리**:
   - 이메일의 점(`.`)과 `@` 기호는 도메인 식별자이므로 제거하면 안 됩니다.
   - 전화번호는 선두 `+`만 유지하고 나머지는 숫자만 남깁니다.
   - `generateEmailIndex()`와 `generatePhoneIndex()`를 **반드시** 구분해서 사용하세요.

4. **Race Condition 대응**:
   - 중복 사전 검증 후 저장 사이에 다른 스레드가 먼저 저장할 수 있습니다.
   - `DataIntegrityViolationException`을 catch하여 DB `UniqueConstraint`를 최후의 방어선으로 활용하세요.

5. **배치/Native Query 사용 시 주의**:
   - `AttributeConverter`는 JPA/Hibernate 영속화 시점에만 발동합니다.
   - MyBatis, JDBC Template, Spring Batch Native Query 사용 시에는 `AesGcmCrypto`를 직접 호출하여 암호화해야 합니다.

6. **로깅 및 마스킹**:
   - DTO의 `toString()`에서 개인정보가 노출되지 않도록 마스킹 처리합니다.
   - `@JsonIgnore`, `@JsonProperty(access = WRITE_ONLY)` 등으로 API 응답에서도 불필요한 노출을 막으세요.

7. **Blind Index 정규화 일관성**:
   - 인덱스 생성 시와 검색 시 반드시 동일한 정규화 로직을 적용해야 합니다.
   - `"010-1234-5678"`과 `"01012345678"`이 동일한 인덱스를 생성해야 검색이 정확합니다.

---

## 정리

Spring Boot 2.4.3, Spring Security, JPA 환경에서 개인정보 보호는 다음 네 축으로 완성할 수 있습니다:

1. **비밀번호**: Spring Security의 `BCryptPasswordEncoder`로 강력한 단방향 해싱.
2. **개인정보 암복호화**: `AES-256-GCM` + JPA `AttributeConverter`(`@Qualifier` 생성자 주입) + `HibernateConfig(SpringBeanContainer)`를 통해 비즈니스 로직을 오염시키지 않는 투명한 암복호화.
3. **개인정보 검색**: HMAC 기반 **Blind Index** 컬럼으로 암호화 보안성을 유지하면서도 B-Tree 인덱스 기반의 초고속 검색 보장. 이메일/전화번호 각각의 특성에 맞는 정규화 규칙 분리 적용.
4. **아키텍처 완성도**: 암호화 유틸을 **순수 POJO**로 작성하고 **`@Configuration + @Bean`** 패턴으로 관리하여 단위 테스트의 독립성과 보안 설정의 집중화를 달성. Race Condition 방어, `@JsonCreator` 기반 Jackson 호환, `@Qualifier` 기반 키 로테이션으로 실전 품질을 확보.
