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

## Welcome to IntelliJ IDEA
intellij 실행시 기본적으로 마지막 작업 프로젝트가 자동으로 Open 된다. 아래 두 가지 방법이 있다.
- **File → Close Project** : 현재 프로젝트를 닫고 프로젝트 선택 화면을 띄운다.
- **File → Settings → Appearance & Behavior → System Settings → Reopen last project on startup** 체크 해제 : intellij 시작시 프로젝트 선택 화면을 띄운다.

## Code Assist 기능 사용시 대/소 문자 구분 해제
Code Assist 기능 사용시 대/소 문자 구분하도록 세팅되어있다. 핵불편...<br>
- **Settings → Editor → General → Code Completion → Match case** 체크 해제

## auto build
소스 코드 변경시 자동 빌드 후 재실행 시켜준다.
- File → Settings → Build, Execution, Deployment → Compiler : build project automatically 체크
- **Ctrl + Alt + Shift + /** 검색창에서 Registry 선택 : compiler.automake.allow.when.app.running 체크

## auto import 문제
**Ctrl + Alt + O** (Optimize Imports) 사용되지 않은 import 제거되나 새로운 패키지를 가져 오지 않음.
- **File → Settings → Editor → General → Auto Import** 세팅의 아래 두 항목 
  - Add unambiguous imports on the fly
  - Optimize imports on the fly (for current project)
  
## Indexing
intellij 가 종종 하단부에 Indexing 작업을 하는게 보일 것이다. 필요 없는 폴더는 exclude 시키자. 난 node_modules 를 exclude 시키기 위해서 찾아본 기능인데 target이랑 node_modules은 기본적으로 exclude가 되어 있었다.
- **File → Project Structure → Modules → Sources tab.**

## Typo: In word 없애기
**F2**(highlighted error) 검사시 스펠링 체크 해제
- Editor → Inspections → Spelling 체크 해제

## Editor Actions
- **Ctrl + D** : (Duplicate Line or Selection)
  - ★★★★★
  - 커서 위치한 라인이나 선택영역을 복사생성한다.
- **Ctrl + Shift + U** : (Toggle Case)
  - ★★★☆☆
  - 선택영역 대소문자 변경 토글

## Main menu
- **Alt + Insert** : (Code → Generate...)
  - ★★★★★
  - 생성도구
- **Ctrl + Shift + UP/Down** : (Code → Move Statement Down)
  - ★★★★★
  - 선택영역을 화살표 방향으로 이동 line영역, 블럭 영역, 함수 단위 영역 다 가능
- **Ctrl + Alt + O** : (Code → Optimize Imports)
  - ★★★★★
  - 자동으로 Imports 시켜준다.
- **Ctrl + Alt + L** : (Code → Reformat Code)
  - ★★★★★
  - 코드 자동 정렬
- **Ctrl + Shift + Enter** : (Edit → Complete Current Statement)
  - ★★☆☆☆
  - 커서 기준으로 마지막에 ;를 입력해준다. 
- **Alt + W** : (Edit → Extend Selection)
  - ★★★★★
  - 커서 기준으로 선택영역을 확장한다.
- **Alt + J** : (Edit → Find → Add Selection for Next Occurrence)
  - ★★★★★
  - 선택영역이랑 동일한 다음 문자열 선택
- **Alt + Shift + J** : (Edit → Find → Unselect Occurrence)
  - ★★★★★
  - 선택영역이랑 동일한 다음 문자열 선택 undo
- **Ctrl + Alt + S** (File → Settings...)
  - ★★☆☆☆
  - Settings 화면을 띄운다.
- **Ctrl + N** (Navigate → Class...)
  - ★★★☆☆
  - Class 검색 화면을 띄운다.
- **Ctrl + Shift + N** (Navigate → File...)
  - ★★★☆☆
  - File명 검색 화면을 띄운다.
- **Shift + F6** : (Refactor → Rename...)
  - ★★★★★
  - 이름 변경
- **Shift + F9** (Run → Debug)
  - ★★★★★
  - 디버그 모드 실행
- **Shift + F10** (Run → Run)
  - ★★★★★
  - 실행
- **Ctrl + F8** (Run → Toggle Line Breakpoint)
  - ★☆☆☆☆
  - Break Point toggle
- **Ctrl + P** (View → Parameter Info)
  - ★★★★☆
  - 함수에 사용되는 인자 목록을 보여준다.
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
  
## Other
- **Ctrl + Tab** : (Switcher)
  - ★★★★★
  - 이전 편집탭 돌아가기

  
