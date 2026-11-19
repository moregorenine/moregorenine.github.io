---
title: "VS Code 설정을 다른 PC에서도 그대로 사용하는 방법 (Settings Sync, 프로필, 수동 백업)"
excerpt: "새 PC나 다른 환경에서도 기존 VS Code 설정, 단축키, 확장 프로그램, 스니펫을 그대로 가져와 사용하는 세 가지 방법 정리"
categories:
  - vscode
  - productivity
tags:
  - vscode
  - settings-sync
  - profiles
  - backup
  - settings
  - developer-tools
  - guide
last_modified_at: 2026-11-19T22:51:00+09:00
toc: true
toc_sticky: true
---

새로운 PC를 세팅하거나 집과 회사 등 여러 환경에서 작업할 때, 기존에 맞춰 둔 VS Code 설정(`settings.json`), 단축키(`keybindings.json`), 확장 프로그램(Extensions), 스니펫을 그대로 가져와 사용하는 방법은 크게 세 가지가 있습니다.

| 방법 | 추천 대상 | 동기화 범위 | 특징 |
| :--- | :--- | :--- | :--- |
| **Settings Sync** | 일반 사용자 (가장 권장) | 설정, 단축키, 확장, 스니펫, UI 상태, 프로필 | GitHub/Microsoft 계정 로그인만으로 실시간 자동 동기화 |
| **Profiles 내보내기/가져오기** | 계정 연동 없이 배포/공유 | 설정, 단축키, 확장, 스니펫, UI 상태 | `.code-profile` 파일이나 Gist URL로 한 번에 적용 |
| **수동 백업 (파일 + CLI)** | 사내 폐쇄망 / 오프라인 환경 | 설정 파일 직접 복사 + CLI로 확장 일괄 설치 | 외부 클라우드 연동 없이 로컬 파일로만 이관 |

---

### 1. Settings Sync 사용하기 (가장 간편한 방법)

VS Code에 기본 내장된 **Settings Sync**를 켜두면 계정 로그인만으로 다른 PC에서도 동일한 개발 환경이 자동으로 복원됩니다.

#### 기존 PC에서 설정 켜기
1. 좌측 하단 **관리(톱니바퀴 아이콘)** → **Backup and Sync Settings...** 클릭 (또는 `Ctrl + Shift + P` 후 `Settings Sync: Turn On...` 실행).
2. 동기화할 항목 선택 (기본적으로 Settings, Keyboard Shortcuts, User Snippets, Extensions, UI State, Profiles 모두 포함).
3. **Sign in & Turn on**을 누르고 **GitHub** 또는 **Microsoft** 계정으로 로그인합니다.

#### 새 PC에서 불러오기
1. 새 PC에 VS Code를 설치한 뒤 좌측 하단 **관리(톱니바퀴 아이콘)** → **Backup and Sync Settings...**에서 동일한 계정으로 로그인합니다.
2. 기존 설정이 감지되면 아래 중 하나를 선택해 적용합니다.
   - **Merge:** 현재 PC 설정과 클라우드 설정을 병합
   - **Replace Local:** 클라우드(기존 PC) 설정으로 현재 PC를 덮어쓰기 *(새 PC 세팅 시 권장)*

> **팁: 특정 PC 전용 설정은 동기화 제외하기**  
> OS별 폰트나 로컬 경로처럼 PC마다 달라야 하는 항목은 `settings.json`에 `settingsSync.ignoredSettings`로 지정해 동기화에서 제외할 수 있습니다.
> ```json
> "settingsSync.ignoredSettings": [
>   "editor.fontSize",
>   "terminal.integrated.defaultProfile.windows"
> ]
> ```

---

### 2. Profiles 내보내기 / 가져오기

로그인 기반 실시간 동기화를 원하지 않거나, 특정 프로젝트/팀원에게 설정 세트를 그대로 공유하고 싶을 때 유용합니다.

#### 내보내기 (Export)
1. `Ctrl + Shift + P` → `Profiles: Export Profile...` 실행.
2. 포함할 항목(Settings, Keyboard Shortcuts, Snippets, Extensions 등)을 확인한 뒤 **Export** 클릭.
3. **GitHub Gist**로 업로드하거나 **Local File**(`.code-profile`)로 저장합니다.

#### 새 PC에서 가져오기 (Import)
1. `Ctrl + Shift + P` → `Profiles: Import Profile...` 실행.
2. 저장해 둔 `.code-profile` 파일을 선택하거나 Gist URL을 입력하면 확장 프로그램까지 한 번에 설치 및 적용됩니다.

---

### 3. 수동 백업 및 복원 (폐쇄망 / 오프라인 환경)

외부 계정을 연결할 수 없는 사내망이나 파일 단위로 직접 관리하고 싶다면, 설정 폴더와 확장 목록을 직접 백업해 옮길 수 있습니다.

#### 1) 설정 파일 복사
아래 경로의 `User` 폴더 안의 파일(`settings.json`, `keybindings.json`, `snippets/` 등)을 복사해 새 PC의 같은 위치에 붙여넣습니다.

- **Windows:** `%APPDATA%\Code\User\`
- **macOS:** `~/Library/Application Support/Code/User/`
- **Linux:** `~/.config/Code/User/`

> **참고:** 확장 프로그램 자체 바이너리는 보통 `~/.vscode/extensions`에 설치되지만 용량이 크고 OS/아키텍처 의존성이 있으므로, 확장 파일 폴더를 통째로 복사하기보다 아래 CLI 명령어로 목록만 뽑아 재설치하는 것이 깔끔합니다.

#### 2) 확장 프로그램 목록 내보내기 및 일괄 설치

- **기존 PC에서 목록 추출:**
  ```bash
  code --list-extensions > extensions.txt
  ```

- **새 PC에서 일괄 설치 (macOS / Linux / Git Bash):**
  ```bash
  xargs -n 1 code --install-extension < extensions.txt
  ```

- **새 PC에서 일괄 설치 (Windows PowerShell):**
  ```powershell
  Get-Content extensions.txt | ForEach-Object { code --install-extension $_ }
  ```

> **참고: USB 등에 담아 쓰는 Portable Mode**  
> 아예 설치 없이 폴더째 들고 다니고 싶다면 VS Code `.zip` 압축 버전을 받은 뒤 실행 파일과 같은 경로에 `data` 폴더를 만들어 [Portable Mode](https://code.visualstudio.com/docs/editor/portable)로 실행하면 모든 설정과 확장이 해당 `data` 폴더 안에 저장됩니다.
