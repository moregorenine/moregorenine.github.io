---
title: "Google Antigravity CLI (agy) 설치 및 시작 가이드"
excerpt: "터미널 환경에서 가볍고 빠르게 Google Antigravity 에이전트를 사용할 수 있는 agy CLI의 OS별 설치 방법, 초기 인증, 권한 확인 없이 실행하는 모드 옵션(--mode, --dangerously-skip-permissions) 및 기본 설정 방법을 정리합니다."
categories:
    - AI
    - Development
tags:
    - antigravity
    - agy
    - cli
    - google
    - ai-coding
    - developer-tools
    - permissions
    - guide
last_modified_at: 2026-11-25T02:00:00+09:00
toc: true
toc_sticky: true
---

**Google Antigravity CLI(`agy`)**는 별도의 IDE나 데스크톱 앱을 띄우지 않고도 터미널 환경에서 Antigravity AI 에이전트를 빠르게 실행하고 상호작용할 수 있는 경량 CLI 도구입니다.

이 글에서는 **macOS, Linux, Windows 환경별 `agy` 설치 방법**과 **초기 인증, 기본 사용법**, 그리고 **권한 확인을 줄이거나 생략하고 실행하는 모드 옵션 및 영구 설정 방법**을 정리합니다.

---

### 1. 운영체제별 설치 방법

#### macOS / Linux
터미널에서 공식 설치 스크립트를 실행합니다.
```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

#### Windows (PowerShell)
PowerShell에서 아래 명령어를 실행합니다.
```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

#### Windows (Command Prompt / CMD)
PowerShell 스크립트 실행이 제한된 환경이라면 명령 프롬프트(CMD)에서 다음 명령어를 실행합니다.
```cmd
curl -fsSL https://antigravity.google/cli/install.cmd -o install.cmd && install.cmd && del install.cmd
```

---

### 2. 실행 및 초기 인증 (`agy`)

설치가 완료되면 새 터미널을 열거나 현재 창에서 `agy`를 실행합니다.

```bash
agy
```

- **초기 인증 (Google Sign-In):** 처음 실행 시 화면 안내에 따라 Google 계정 로그인을 진행합니다.
  - **로컬 환경:** 기본 웹 브라우저가 자동으로 열리며 로그인을 완료할 수 있습니다.
  - **원격 SSH 환경:** 터미널에 출력되는 인증 URL을 복사해 로컬 브라우저에서 열어 로그인하면 됩니다.

---

### 3. 기본 명령어 및 종료 방법

- **CLI 옵션 및 서브커맨드 확인:**
  ```bash
  agy --help
  ```
- **TUI 내부 슬래시 명령어 확인:** `agy` 실행 상태에서 `/help` 입력
- **종료하기:** `Ctrl + D`를 두 번 누르거나(`/exit` 또는 `/quit` 입력)

---

### 4. 권한 확인 없이 실행하기 (실행 모드 및 자동 승인)

작업할 때마다 파일 수정이나 명령 실행 승인 프롬프트가 뜨는 것을 줄이고 싶다면, 목적과 운영체제 환경에 맞는 실행 모드 플래그를 사용하거나 설정 파일에 기본 모드를 고정할 수 있습니다.

#### 1) 터미널 실행 옵션 (일회성)

실행 시 `--mode` 또는 `--dangerously-skip-permissions` 플래그를 넘기면 해당 세션에만 임시로 적용됩니다.

| 옵션 | 동작 방식 | 적합한 상황 |
| :--- | :--- | :--- |
| `agy --mode auto` | 워크스페이스 파일 수정을 자동 승인하고, 터미널 명령은 OS 샌드박스 안에서 자동 실행합니다. 샌드박스 밖 실행이 필요한 명령은 안전 여부를 자동 평가해 안전한 명령만 무확인 실행합니다. | 일상적인 개발 자동화 (안전성과 편의성 균형) |
| `agy --mode accept-edits` | 워크스페이스 내 파일 수정·생성만 자동 승인하고, 터미널 명령어 등은 기존 권한 정책에 따라 확인을 받습니다. | 코드 편집은 빠르게 맡기되 명령 실행은 직접 통제할 때 |
| `agy --dangerously-skip-permissions` | 샌드박스나 OS 지원 여부와 무관하게 **모든 도구 권한 확인 프롬프트를 생략**하고 즉시 자동 승인합니다. | 격리된 테스트 환경 또는 `auto` 모드가 지원되지 않는 환경에서 완전 자동화가 필요할 때 |

```bash
# 1. 자동 모드 (파일 편집 + 안전한/샌드박스 내 명령 자동 승인)
agy --mode auto

# 2. 파일 편집만 자동 승인 (명령 실행은 확인 요청)
agy --mode accept-edits

# 3. 모든 권한 확인 건너뛰기 (위험 명령까지 무확인 실행되므로 주의)
agy --dangerously-skip-permissions
```

> **Windows 및 샌드박스 미지원 환경 참고**  
> `auto` 모드는 OS 터미널 샌드박스 기능을 필요로 합니다. `v1.4.4+`부터는 Windows에서도 샌드박스 지원 환경(Windows 11 24H2 이후 + 최신 업데이트 등)이면 `auto` 모드를 사용할 수 있지만, **Windows 10·이전 Windows 11 빌드·일부 컨테이너처럼 샌드박스를 쓸 수 없는 환경이나 Business/Enterprise 계정에서는 `auto` 모드가 비활성화되어 `default` 모드로 동작**합니다.  
> 이런 환경에서 명령 확인 없이 실행해야 한다면 `--dangerously-skip-permissions`를 사용하거나 아래의 세부 권한 규칙(`permissions.allow`)을 설정하세요.

#### 2) 기본값으로 영구 설정하기 (`/config` & `settings.json`)

매번 CLI 플래그를 붙이기 번거롭다면 대화형 설정 메뉴나 설정 파일에서 기본 실행 모드를 고정할 수 있습니다. *(세션 실행 중 `Shift + Tab`을 눌러 모드를 순환 전환할 수도 있습니다.)*

1. **`/config` 메뉴로 변경 (권장)**  
   `agy` 실행 상태에서 `/config`를 입력한 뒤 **Agent Mode** 항목을 `auto`, `accept-edits`, `plan` 등 원하는 모드로 변경하면 설정 파일에 즉시 저장됩니다.

2. **`settings.json` 직접 수정**  
   `~/.gemini/antigravity-cli/settings.json` 파일을 열어 `agentMode` 값을 지정합니다. (`""`로 두면 기본 확인 모드인 `default`로 동작합니다.)

   ```json
   {
     "agentMode": "auto"
   }
   ```

> **보안 팁: 무조건 전체 우회 대신 세부 권한 규칙 활용하기**  
> 모든 권한 확인을 생략하는 `--dangerously-skip-permissions`(또는 `settings.json`의 `"toolPermission": "always-proceed"`)는 의도치 않은 삭제나 외부 요청까지 확인 없이 실행할 수 있어 주의가 필요합니다. 특정 CLI 명령어만 안전하게 자동 허용하고 싶다면 `agy` 내부에서 `/permissions`를 입력하거나 `settings.json`의 `permissions`(`allow` / `ask` / `deny`) 규칙으로 `command(git)`, `command(regex:npm run (build|lint|test))`처럼 허용 대상을 좁혀 두는 편이 훨씬 안전합니다.

---

### 5. 설정 파일 및 공식 문서

- **설정 파일 경로:** `~/.gemini/antigravity-cli/settings.json`
- **공식 문서 참고:**
  - [Antigravity CLI Reference](https://antigravity.google/docs/cli/reference.md)
  - [Antigravity CLI Execution Modes](https://antigravity.google/docs/cli/modes.md)
  - [Agent Permissions & Security](https://antigravity.google/docs/permissions.md)
  - [Antigravity CLI Best Practices](https://antigravity.google/docs/cli/best-practices.md)
