---
title: "vscode 단축키 및 설정"
excerpt: "vscode 단축키 및 설정"
categories: 
  - idea
tags: 
  - vscode
last_modified_at: 2026-03-18T00:00:00+09:00
toc: true
toc_sticky: true
---

## 📝 IntelliJ 전문가를 위한 VS Code 단축키 가이드

IntelliJ의 강력한 기능을 VS Code에서 그대로 재현하기 위한 단축키 대응표입니다. 원격 접속 환경이나 VS Code 환경에 적응 중인 개발자를 위해 정리했습니다.

### 1. 코드 편집 및 선택 (Editing)

| 기능 | IntelliJ | **VS Code (Win/Linux)** | **VS Code (macOS)** |
| :--- | :--- | :--- | :--- |
| **선택 영역 확장** | `Ctrl + W` | `Shift + Alt + →` | `Ctrl + Shift + Cmd + →` |
| **줄 복제 (Duplicate)** | `Ctrl + D` | `Shift + Alt + ↓` | `Shift + Opt + ↓` |
| **한 줄 삭제** | `Ctrl + Y` | `Ctrl + Shift + K` | `Cmd + Shift + K` |
| **줄 위/아래 이동** | `Ctrl + Shift + ↑/↓` | `Alt + ↑/↓` | `Opt + ↑/↓` |
| **멀티 커서 (다음 일치)** | `Alt + J` | `Ctrl + D` | `Cmd + D` |

### 2. 코드 정리 및 탐색 (Navigation & Refactor)

| 기능 | IntelliJ | **VS Code (Win/Linux)** | **VS Code (macOS)** |
| :--- | :--- | :--- | :--- |
| **코드 자동 정렬** | `Ctrl + Alt + L` | `Shift + Alt + F` | `Shift + Opt + F` |
| **임포트 최적화** | `Ctrl + Alt + O` | `Shift + Alt + O` | `Shift + Opt + O` |
| **선언부로 이동** | `F4` 또는 `F12` | `F12` | `F12` |
| **최근 파일 목록** | `Ctrl + E` | `Ctrl + Tab` | `Ctrl + Tab` |
| **탭 좌/우 이동** | `Alt + ←/→` | `Ctrl + PgUp/PgDn` | `Cmd + Opt + ←/→` |

### 3. 전문가를 위한 고급 팁

* **Search Everywhere (`Shift` 2번) 대응:**
    VS Code에서는 **`Ctrl + P`** 하나로 통합되어 있습니다. 입력창에 `@`를 치면 심볼 검색, `:`를 치면 라인 이동이 가능합니다.
* **Run Anything (`Ctrl` 2번) 대응:**
    모든 명령을 실행하는 **Command Palette (`Ctrl + Shift + P`)**를 사용하세요. 원격 접속 시 키 연타가 인지되지 않을 때 가장 확실한 대안입니다.
* **파일 저장 시 자동화:**
    `settings.json`에 아래 설정을 추가하면 IntelliJ처럼 저장 시 자동 정렬과 임포트 정리가 실행됩니다.
    ```json
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": "explicit"
    }
}
```

---

이 포스팅 내용 중에 더 구체적으로 보강하고 싶은 단축키나, GitPages용으로 **포스팅 제목(Title)**이나 **태그(Tags)**를 추가해 드릴까요?
