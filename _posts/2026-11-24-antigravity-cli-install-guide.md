---
title: "Google Antigravity CLI (agy) 설치 및 시작 가이드"
excerpt: "터미널 환경에서 가볍고 빠르게 Google Antigravity 에이전트를 사용할 수 있는 agy CLI의 OS별 설치 방법, 초기 인증, 기본 명령어 및 설정 경로 정리"
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
    - guide
last_modified_at: 2026-11-24T00:40:00+09:00
toc: true
toc_sticky: true
---

**Google Antigravity CLI(`agy`)**는 별도의 IDE나 데스크톱 앱을 띄우지 않고도 터미널 환경에서 Antigravity AI 에이전트를 빠르게 실행하고 상호작용할 수 있는 경량 CLI 도구입니다.

이 글에서는 **macOS, Linux, Windows 환경별 `agy` 설치 방법**과 **초기 인증, 기본 사용법, 설정 파일 위치**를 정리합니다.

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

### 4. 설정 파일 및 공식 문서

- **설정 파일 경로:** `~/.gemini/antigravity-cli/settings.json`
- **공식 문서 참고:**
  - [Antigravity CLI Reference](https://antigravity.google/docs/cli/reference.md)
  - [Antigravity CLI Best Practices](https://antigravity.google/docs/cli/best-practices.md)
