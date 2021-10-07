---
title: "Eclipse Decompiler 설치 오류 해결"
excerpt: "Eclipse Decompiler 설치시 Unable to read repository at https://ecd-plugin.github.io/update/content.xml. 오류 발생 해결 방법"
categories: 
  - idea
tags: 
  - dev
  - eclipse
  - decompiler
last_modified_at: 2019-03-08T00:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
프로젝트 투입시 Eclipse에 가장 먼저 설치하는 plugin인 Enhanced Class Decompiler 설치시 오류가 발생했다.  
Eclipse Version은 Oxygen.3a (4.7.3a) 였다. 검색해본 결과 해당 Version에만 해당하는 내용은 아닌 듯 했다.
github에서는 #11에 proxy 관련 문제라고들 하는데 해결이 되지 않았다. 그래서 그냥 github 프로젝트를 다운로드해 local에서 설치해서 해결 했다.

## github ecd 프로젝트 직접 다운로드
issue #11에 제기된 내용이지만 proxy로 해결이 되지 않아. #38에 등록된 RobertZenz의 답변을 참조해 해결 했다.  
[ecd github - issue #38](https://github.com/ecd-plugin/ecd/issues/38 "issue #38")  
ecd의 github 프로젝트를 직접 다운로드 받은 후 eclipse에서 다운로드 받은 파일로 직접 plugin 설치를 수행하며 아래 과정대로 진행했다.

1. Download the whole update repository and add it as source on the local filesystem.
  1. Download the the zip of the repository.
  2. Unpack it to somewhere on the filesystem.
  3. Go to "Install new software" and enter the location of that as source, file://YOURLOCATION.
  
## JDK8 설정
설치시 repository 접속 방법이 https 로 변경되면서 JDK1.7 이하 버젼에서 발생할 수 있는 문제일 수도 있나보다.
이럴 경우 -vm 환경 변수로 JDK8 이상 버젼을 지정해 해결하는 케이스도 있나보다.  
[JDK 버젼 문제 - stackoverflow.com](https://stackoverflow.com/questions/45319531/eclipse-luna-shows-error-unable-to-read-repository-at/51359313#51359313 "JDK 버젼 문제 - stackoverflow.com")

## Class 보기 동작 안할때
특히 oxygen 버젼일 경우 필요한 설정이다. class 파일들에 대한 연결이 아마 다른 plugin으로 설정 되어있을 텐데 Class Decompiler Viewer를 Default로 설정해줘야 한다.
- Click on "Window > Preferences > General > Editors > File Associations"
- Change default to your for both .class association.
- "*.class" : "Class Decompiler Viewer" is selected by default.
- "*.class without source" : "Class Decompiler Viewer" is selected by default.
