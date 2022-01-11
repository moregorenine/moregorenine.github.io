---
title: "Spring Boot Configuration 암호화"
excerpt: "Spring Boot Configuration with Jasypt"
categories: 
  - springboot
tags: 
  - springboot
last_modified_at: 2022-01-10T00:00:00+09:00
toc: true
toc_sticky: true
---

## Jasypt?
프레임워크를 사용하다보면 설정파일에 DB 접속정보 같은 노출되기를 꺼려하는 정보를 입력할 경우가 있습니다.  
이럴 경우 보통 해당 정보를 암호화하여 입력하게 되는데요.  
Jasypt(Java Simplified Encryption) Spring Boot는 Boot 애플리케이션의 속성 소스를 암호화하기 위한 유틸리티를 제공 합니다.  


## Jasypt 사용법
### 1. Install jasypt
jasypt library를 설치합니다.  

`maven`
```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.4</version>
</dependency>
```
`gradle`
```shell
implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.4'
```
### 2. Spring Boot 설정
**application.properties**  
@Configuration에 등록될 @Bean name을 설정합니다.
```shell
-- application.properties --

jasypt.encryptor.bean=encryptorBean // @Bean name
```

**PropertyEncryptConfig.java**  
Jasypt에서 암호화시에 암호화키를 사용할텐데 암호화키를 알면 복호화가 가능하기 때문에 코드상이나 DevOps 상에 노출이 되지 않도록 보통 서버 OS에 환경변수를 등록하여 사용합니다.  
`JASYPT_PASSWORD` 환경변수에 `암호화 키`를 설정합니다.  
```java
-- PropertyEncryptConfig.java --

@Configuration
@EnableEncryptableProperties
public class PropertyEncryptConfig {

    @Value("${JASYPT_PASSWORD}")
    String password;

    @Bean(name = "encryptorBean")
    public StringEncryptor stringEncryptor() {
        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword(password); // 암호화 키 값, 서버의 환경변수로 설정 추천
        config.setAlgorithm("PBEWithMD5AndDES"); // 알고리즘
        config.setKeyObtentionIterations("1000"); // 해싱 반복 횟수
        config.setPoolSize("2"); // 인스턴스 pool, 머신의 코어 수와 동일하게 설정 추천
        config.setProviderName("SunJCE");
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
        config.setStringOutputType("base64");
        encryptor.setConfig(config);
        return encryptor;
    }
}
```

**PropertyEncryptConfigTest.java**  
암호화된 문자열을 얻기 위해서 Test 코드를 생성했습니다. 저희가 필요로 하는건 `decryptedText` 변수값이 되겠죠?
```java
-- PropertyEncryptConfigTest.java --

    @Test
    public void stringEncryptor() {
        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword("password"); // 암호화 키 값, 서버의 환경변수로 설정 추천
        config.setAlgorithm("PBEWithMD5AndDES"); // 알고리즘
        config.setKeyObtentionIterations("1000"); // 해싱 반복 횟수
        config.setPoolSize("2"); // 인스턴스 pool, 머신의 코어 수와 동일하게 설정 추천
        config.setProviderName("SunJCE");
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
        config.setStringOutputType("base64");
        encryptor.setConfig(config);

        String privateData = "암호화 할 내용";
        String encryptedText = encryptor.encrypt(privateData); // 암호화
        String decryptedText = encryptor.decrypt(encryptedText); // 복호화

        assertEquals(privateData, decryptedText);
    }
```

**application-local.yml**  
저 같은 경우 Google Oauth2 로그인시 필요한 client-id, client-secret 정보를 노출시키기 싫어서 암호화를 했습니다.  
암호화된 문자열은 `ENC()`로 감싸줍니다.
```shell
-- application-local.yml --

spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ENC(j3ZwPJ6u+5Z63EslBXwaOmNdkOk/4moGgl1pc0eT3HMWjLleH2+mp9njjqJQkxzbFxTqm50ExqbGndKqsI1LPK9B4i0zgpcaNC1BBFO/Wslr5kv0m66x3w==)
            client-secret: ENC(aDZ6ufu8kbIOsASrxttVWjA31j32ltX29CK9tvo85r5zFmJmeSHfrZBZdmqvqwd0)
```

## 참조
<https://www.baeldung.com/jasypt>  
<https://www.baeldung.com/spring-boot-jasypt>