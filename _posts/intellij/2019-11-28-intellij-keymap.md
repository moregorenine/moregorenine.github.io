---
title: "Intellij 단축키 및 설정"
excerpt: "Intellij 단축키 및 설정"
categories: 
  - intellij
tags: 
  - intellij
last_modified_at: 2019-11-28T00:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
eclipse, vscode, intellij 혼합해서 사용하다 보니 단축키가 자주 헷갈린다.
내가 종종 사용하는 기능이 인터넷 올라와 있는 여러 단축키 글들 중에서 없는 것도 많아.
개인적인 참고용으로 작성한다.

## Code Assist 기능 사용시 대/소 문자 구분 해제
Code Assist 기능 사용시 대/소 문자 구분하도록 세팅되어있다. 핵불편...<br>
Settings → Editor → General → Code Completion → Match case: 체크 해제

## auto build
소스 코드 변경시 자동 빌드 후 재실행 시켜준다.
- File → Settings → Build, Execution, Deployment → Compiler : build project automatically 체크
- **Ctrl + Alt + Shift + /** 검색창에서 Registry 선택 : compiler.automake.allow.when.app.running 체크

## Editor Actions
- **Ctrl + D** : (Duplicate Line or Selection)
  - ★★★★★
  - 커서 위치한 라인이나 선택영역을 복사생성한다.
- **Ctrl + Shift + U** : (Toggle Case)
  - ★★★☆☆
  - 선택영역 대소문자 변경 토글

## Main menu
- **Ctrl + Shift + UP/Down** : (Code → Move Statement Down)
  - ★★★★★
  - 선택영역을 화살표 방향으로 이동 line영역, 블럭 영역, 함수 단위 영역 다 가능
- **Ctrl + Alt + O** : (Code → Optimize Imports)
  - ★★★★★
  - 자동으로 Imports 시켜준다.
- **Ctrl + Alt + L** : (Code → Reformat Code)
  - ★★★★★
  - 코드 자동 정렬
- **Alt + J** : (Edit → Find → Add Selection for Next Occurrence)
  - ★★★★★
  - 선택영역이랑 동일한 다음 문자열 선택
- **Alt + Shift + J** : (Edit → Find → Unselect Occurrence)
  - ★★★★★
  - 선택영역이랑 동일한 다음 문자열 선택 undo
- **Alt + W** : (Edit → Extend Selection  )
  - ★★★★★
  - 커서 기준으로 선택영역을 확장한다.
- **Ctrl + Alt + S** (File → Settings...)
  - ★★☆☆☆
  - Settings 화면을 띄운다.
- **Ctrl + N** (Navigate → Class...)
  - ★★★☆☆
  - Class 검색 화면을 띄운다.
- **Ctrl + Shift + N** (Navigate → File...)
  - ★★★☆☆
  - File명 검색 화면을 띄운다.
- **Ctrl + F8** (Run → Toggle Line Breakpoint)
  - ★☆☆☆☆
  - Break Point toggle
- **Ctrl + Shift + E** (View → Recent Locations)
  - ★★★★★
  - 최근에 코드의 조회 위치 내역들을 보여준다. 한번더 클릭시 최근 코드 수정 기준으로 보여준다.
- **Ctrl + Shift + F12** (Window → Active Tool Window → Hide All Tool Windows)
  - ★★☆☆☆
  - 모든 tool windows를 숨김으로서 코드 editor창만이 Maximize되는 효과
- **Ctrl + Shift + '** (Window → Active Tool Window → Maximize Tool Window)
  - ★☆☆☆☆
  - 선택된 tool창 사이즈 max toggle

## Tool Windows
- **Alt + 0~9** : (Tool Windows)
  - ★☆☆☆☆
  - 번호에 따라 tool창에 포커싱 된다. 
  
  
