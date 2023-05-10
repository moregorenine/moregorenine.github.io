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
