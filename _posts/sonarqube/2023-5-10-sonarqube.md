---
title: "SonarQube 설치"
excerpt: "SonarQube 설치"
categories:
    - SonarQube
tags:
    - SonarQube
    - docker
last_modified_at: 2023-5-10T00:00:00+09:00
toc: true
toc_sticky: true
---

## SonarQube 이미지 설치하기
<https://hub.docker.com/_/sonarqube>
```sh
docker pull sonarqube
```

## SonarQube 실행
SonarQube에서 default로 사용하는 9000 port를 호스트의 9000 port와 연결
```sh
docker run --name sonarqube -p 9000:9000  sonarqube
```

## SonarQube 접속
<http://localhost:9000> 으로 접속.  
default username, password인 admin / admin 으로 접속합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/ae1821d3-411e-4ccd-898d-0a0945af691f)

## SonarQube Project 생성
Jenkins에 자동연동 하는 방법도 있으나 local에서 Intellij 와 연동해 테스트 진행하기 위해 `Manually`로 생성합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/ca533f5a-8a1c-4a79-a96e-ff5a4793b026)  

Project 생성합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/b56efc4f-cd5e-43a7-b54f-4fa30d9448aa)

## Token 생성
`Locally`를 선택합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/041d39b6-09e3-4712-a6f7-cad7e731ac3c)  

`Generate`버튼으로 생성 후, `Token` 값을 복사해 둡니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/dc45571b-a9a0-4827-ab7b-9b5efe8f089a)

### Run analysis on your project
순수하게 Maven 이나 Gradle로 <http://localhost:9000> 로 직접 연계해서 분석도 가능합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/78a118cb-3c82-4cfd-a17e-79f7ade824dc)  

![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/6033c23f-2f74-440c-92f9-fb1dcba17977)

## IntelliJ SonarQube 연동
### 플러그인 설치
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/cf01a07c-784c-488b-bcc7-1d2ac7df0274)

### SonarQube 연동
Intellij에서 SonarQube를 연동해보겠습니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/e9677ad6-1fb2-4da5-9699-cc7247deeda6)  

SonarQube 서버의 url 주소를 입력합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/5a05e9b5-0f11-453e-8492-c8bd28624761)  

위에서 `Token` 생성 후 저장했던 값을 입력합니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/17f4adc3-7100-4bc1-a8ea-dfa58dd07bcc)  

연동이 완료 되었습니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/96b91e0d-6673-4e16-916a-1baaefe7b3c6)  

SonarQube 서버 방문없이 Intellij 에서 바로 결과를 확인할 수 있게 되었습니다.
![image](https://github.com/moregorenine/moregorenine.github.io/assets/6086965/af4ab27c-fbcf-4e19-b500-ad79f911f581)






