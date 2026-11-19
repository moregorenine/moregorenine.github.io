---
title: "IntelliJ 사용자를 위한 VS Code 단축키 및 설정 가이드"
excerpt: "IntelliJ의 주요 기능을 VS Code에서 그대로 사용하기 위한 단축키 대응표 및 설정 정리"
categories:
  - vscode
  - productivity
tags:
  - vscode
  - keybindings
  - shortcuts
  - intellij
  - settings
  - developer-tools
  - guide
last_modified_at: 2026-03-18T00:00:00+09:00
toc: true
toc_sticky: true
---

## 📝 IntelliJ 전문가를 위한 VS Code 단축키 가이드

IntelliJ의 강력한 기능을 VS Code에서 그대로 재현하기 위한 단축키 대응표입니다. 원격 접속 환경이나 VS Code 환경에 적응 중인 개발자를 위해 정리했습니다.

### 1. 코드 편집 및 선택 (Editing)

| 기능                      | IntelliJ             | **VS Code (Win/Linux)** | **VS Code (macOS)**      |
| :------------------------ | :------------------- | :---------------------- | :----------------------- |
| **선택 영역 확장**        | `Ctrl + W`           | `Shift + Alt + →`       | `Ctrl + Shift + Cmd + →` |
| **줄 복제 (Duplicate)**   | `Ctrl + D`           | `Shift + Alt + ↓`       | `Shift + Opt + ↓`        |
| **한 줄 삭제**            | `Ctrl + Y`           | `Ctrl + Shift + K`      | `Cmd + Shift + K`        |
| **줄 위/아래 이동**       | `Ctrl + Shift + ↑/↓` | `Alt + ↑/↓`             | `Opt + ↑/↓`              |
| **멀티 커서 (다음 일치)** | `Alt + J`            | `Ctrl + D`              | `Cmd + D`                |
| **에디터 최대화/최소화**  | `Ctrl + Shift + F12` | `Ctrl + K Ctrl + M`     | `Cmd + K Cmd + M`        |

### 2. 코드 정리 및 탐색 (Navigation & Refactor)

| 기능               | IntelliJ         | **VS Code (Win/Linux)** | **VS Code (macOS)** |
| :----------------- | :--------------- | :---------------------- | :------------------ |
| **코드 자동 정렬** | `Ctrl + Alt + L` | `Shift + Alt + F`       | `Shift + Opt + F`   |
| **임포트 최적화**  | `Ctrl + Alt + O` | `Shift + Alt + O`       | `Shift + Opt + O`   |
| **선언부로 이동**  | `F4` 또는 `F12`  | `F12`                   | `F12`               |
| **최근 파일 목록** | `Ctrl + E`       | `Ctrl + Tab`            | `Ctrl + Tab`        |
| **탭 좌/우 이동**  | `Alt + ←/→`      | `Ctrl + PgUp/PgDn`      | `Cmd + Opt + ←/→`   |

### 3. 전문가를 위한 고급 팁

- **Search Everywhere (`Shift` 2번) 대응:**
  VS Code에서는 **`Ctrl + P`** 하나로 통합되어 있습니다. 입력창에 `@`를 치면 심볼 검색, `:`를 치면 라인 이동이 가능합니다.
- **Run Anything (`Ctrl` 2번) 대응:**
  모든 명령을 실행하는 **Command Palette (`Ctrl + Shift + P`)**를 사용하세요. 원격 접속 시 키 연타가 인지되지 않을 때 가장 확실한 대안입니다.
- **파일 저장 시 자동화:**
  `settings.json`에 아래 설정을 추가하면 IntelliJ처럼 저장 시 자동 정렬과 임포트 정리가 실행됩니다.
  ```json
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": "explicit"
  }
  ```

### 4. 모든 창(사이드바·하단 패널) 한 번에 숨기기/보이기

VS Code 기본 기능에는 **기본 사이드바, 보조 사이드바, 하단 패널**을 한 번의 단축키로 동시에 숨기거나 다시 여는 단일 명령어가 없습니다. `keybindings.json`에 `runCommands` 기능을 추가하면 IntelliJ의 `Hide All Tool Windows`(`Ctrl + Shift + F12`)처럼 단축키 하나로 세 창을 동시에 제어할 수 있습니다.

#### 1) `keybindings.json` 열기
1. `Ctrl + Shift + P`를 눌러 명령 팔레트를 엽니다.
2. `Preferences: Open Keyboard Shortcuts (JSON)` (한글판: `기본 설정: 바로가기 키 열기(JSON)`)를 입력하고 `Enter`를 누릅니다.
3. 열린 `keybindings.json` 파일의 대괄호 `[`와 `]` 사이에 아래 코드를 추가하고 저장(`Ctrl + S`)합니다. (기존 설정이 있다면 쉼표 `,`로 구분)

#### 2) 한 번에 닫고 여는 토글 방식 (권장)
같은 단축키(`Ctrl + K Ctrl + M`)를 한 번 누르면 세 창이 모두 닫히고, 다시 누르면 모두 열리도록 `when` 조건을 두 개로 나누어 등록합니다.

```json
[
    {
        "key": "ctrl+k ctrl+m",
        "command": "runCommands",
        "args": {
            "commands": [
                "workbench.action.closeSidebar",
                "workbench.action.closeAuxiliaryBar",
                "workbench.action.closePanel"
            ]
        },
        "when": "!editorPartMultipleEditorGroups && (sideBarVisible || auxiliaryBarVisible || panelVisible)"
    },
    {
        "key": "ctrl+k ctrl+m",
        "command": "runCommands",
        "args": {
            "commands": [
                "workbench.action.toggleSidebarVisibility",
                "workbench.action.toggleAuxiliaryBar",
                "workbench.action.togglePanel"
            ]
        },
        "when": "!editorPartMultipleEditorGroups && !sideBarVisible && !auxiliaryBarVisible && !panelVisible"
    }
]
```

- **첫 번째 블록 (모두 닫기):** 에디터 창이 1개일 때(`!editorPartMultipleEditorGroups`), 세 창 중 하나라도 열려 있으면(`sideBarVisible || auxiliaryBarVisible || panelVisible`) 모두 닫습니다.
- **두 번째 블록 (모두 열기):** 에디터 창이 1개일 때, 세 창이 모두 닫혀 있으면(`!sideBarVisible && !auxiliaryBarVisible && !panelVisible`) 세 창을 동시에 다시 엽니다.
- **에디터 분할 시 충돌 방지:** 에디터 창이 2개 이상 분할되어 있을 때는 기존 최대화 단축키(`View: Toggle Maximize Editor Group`)가 충돌 없이 정상 작동합니다.
- **단축키 변경:** `"key"` 값을 원하는 다른 단축키(예: `"ctrl+alt+m"`)로 변경해 사용할 수 있습니다.

#### 3) 참고: 닫기 전용 설정 및 개별 다시 열기
토글 없이 **모두 닫기만** 원한다면 아래 단일 블록만 추가하면 됩니다.

```json
{
    "key": "ctrl+k ctrl+m",
    "command": "runCommands",
    "args": {
        "commands": [
            "workbench.action.closeSidebar",
            "workbench.action.closeAuxiliaryBar",
            "workbench.action.closePanel"
        ]
    },
    "when": "editorTextFocus && !editorPartMultipleEditorGroups"
}
```

- **작동 조건:** 에디터에 커서가 있고(`editorTextFocus`), 창이 1개일 때(`!editorPartMultipleEditorGroups`)만 작동합니다.
- **개별 창 다시 열기 단축키:**
  - 기본 사이드바: `Ctrl + B`
  - 보조 사이드바: `Ctrl + Alt + B`
  - 하단 패널(터미널): `Ctrl + J`


