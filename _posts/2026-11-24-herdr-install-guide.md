---
title: "Herdr 설치 가이드 — AI 코딩 에이전트를 위한 터미널 런타임"
excerpt: "Claude Code, Codex 등 여러 AI 코딩 에이전트를 한 번에 실행하고 백그라운드 세션으로 유지해 주는 Herdr의 OS별(macOS, Linux, Windows) 설치 방법 및 빠른 시작 정리"
categories:
    - AI
    - Development
tags:
    - herdr
    - terminal
    - ai-coding
    - claude-code
    - codex
    - developer-tools
    - guide
last_modified_at: 2026-11-24T00:36:00+09:00
toc: true
toc_sticky: true
---

**Herdr**는 Claude Code, Codex 등 여러 AI 코딩 에이전트를 동시에 실행하고 관리할 수 있도록 설계된 **터미널 멀티플렉서 및 런타임**입니다. 클라이언트 창을 닫거나 연결이 끊겨도 백그라운드 서버에서 에이전트 세션이 계속 유지되는 것이 특징입니다.

이 글에서는 macOS, Linux, Windows 환경별 **Herdr 설치 방법**과 기본 실행 방법을 정리합니다.

---

### 1. macOS & Linux 설치

환경에 따라 아래 방법 중 편한 방식을 선택해 설치할 수 있습니다.

#### 방법 A: 공식 설치 스크립트 (권장)
가장 빠르게 최신 안정 버전 바이너리를 설치하고 `PATH`에 등록하는 방법입니다.
```bash
curl -fsSL https://herdr.dev/install.sh | sh
```

#### 방법 B: Homebrew
Homebrew를 사용 중이라면 다음 명령어로 설치합니다.
```bash
brew install herdr
```
*(macOS에서 데스크톱 GUI 앱 래퍼를 함께 설치하려면 `brew install --cask herdr-app`을 사용합니다.)*

#### 방법 C: Mise
`mise`로 개발 도구를 관리하는 경우 전역 설치할 수 있습니다.
```bash
mise use -g herdr
```
*(레지스트리에서 `herdr`를 찾지 못한다는 오류가 나오면 먼저 `mise`를 최신 버전으로 업데이트하세요.)*

#### 방법 D: Nix (Flake)
Nix 환경에서는 Flake를 이용해 설치할 수 있습니다.
```bash
nix profile install github:herdrdev/herdr
```
*(특정 릴리스 버전을 지정하려면 `github:herdrdev/herdr/v0.x.y` 형태로 태그를 붙여 실행합니다.)*

---

### 2. Windows 설치 (Preview / Beta)

#### 방법 A: PowerShell (권장)
PowerShell을 열고 아래 명령어를 실행합니다.
```powershell
powershell -ExecutionPolicy Bypass -c "irm https://herdr.dev/install.ps1 | iex"
```

#### 방법 B: Command Prompt (CMD)
사내 보안 정책 등으로 파일리스 PowerShell 스크립트 실행이 차단된 환경이라면, **명령 프롬프트(CMD)**에서 아래 명령어를 실행합니다.
```cmd
curl.exe -fsSLo install.cmd https://herdr.dev/install.cmd && install.cmd && del install.cmd
```

---

### 3. 설치 확인 및 빠른 시작

#### 설치 확인
터미널을 새로 열거나 현재 창에서 버전을 출력해 정상 설치 여부를 확인합니다.
```bash
herdr --version
```

#### 실행하기
1. 작업할 프로젝트 디렉토리로 이동합니다.
   ```bash
   cd /path/to/your/project
   ```
2. `herdr`를 실행합니다.
   ```bash
   herdr
   ```
   기본 백그라운드 세션이 시작되거나 기존 세션에 연결됩니다. 이후 터미널 창을 닫더라도 실행 중인 프로세스와 코딩 에이전트 작업은 백그라운드 서버에서 그대로 유지됩니다.

---

### 참고
- 공식 웹사이트: [https://herdr.dev](https://herdr.dev)
