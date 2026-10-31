---
title: "VS Code Rojo 확장 설치 & Studio 연결 문제 해결 (Can't parse JSON, Aftman 오류)"
excerpt: "VS Code Rojo 확장으로 설치했을 때 만나는 Can't parse JSON, rojo 명령어 인식 불가, Aftman 오류 해결기"
date: 2026-10-31 09:30:00 +0900
categories:
  - Roblox
tags:
  - Roblox
  - Rojo
  - Aftman
  - VSCode
toc: true
toc_sticky: true
---

Roblox 프로젝트를 VS Code + Rojo로 개발하려고 세팅하면서 겪은 문제와 해결 방법을 정리했습니다.

## 환경

- Windows
- VS Code + [Rojo 확장 (evaera.vscode-rojo)](https://marketplace.visualstudio.com/items?itemName=evaera.vscode-rojo)
- Rojo 7.7.1 (Aftman으로 설치)
- Roblox Studio

## 1. Rojo 확장 설치

1. VS Code 확장 탭에서 **Rojo - Roblox Studio Sync** 설치
2. 프로젝트 폴더를 열고 `default.project.json`이 있는 상태에서 Rojo 확장으로 서버 시작 (`Rojo: Start Server`)
3. 확장이 [Aftman](https://github.com/LPGhatguy/aftman)으로 Rojo를 설치하고, 프로젝트에 `aftman.toml`을 생성합니다.

```toml
[tools]
rojo = "rojo-rbx/rojo@7.7.1"
```

## 2. 문제 ① Studio 연결 시 "Can't parse JSON"

`http://localhost:34872`로 Connect 했더니 Studio 플러그인에서 `Can't parse JSON` 메시지가 떴습니다.

### 원인

- `default.project.json` 문법 오류가 **아니었습니다**.
- 서버(`/api/rojo`)는 정상 응답했고 (Rojo 7.7.1, 프로토콜 버전 5), 응답은 MessagePack 바이너리 형식이었습니다.
- Studio에 설치된 **Rojo 플러그인이 서버와 버전이 맞지 않아** 응답을 해석하지 못한 것이 원인이었습니다.
- `%LOCALAPPDATA%\Roblox\Plugins`에는 Rojo 플러그인이 없었고, 예전에 Toolbox 등으로 설치한 구버전 플러그인이 쓰이고 있었습니다.

### 해결

1. Studio에서 기존 Rojo 플러그인 제거 (Manage Plugins)
2. 서버와 같은 버전의 플러그인 설치

   ```powershell
   rojo plugin install
   ```

   `%LOCALAPPDATA%\Roblox\Plugins\RojoManagedPlugin.rbxm`이 생성됩니다.
3. Studio를 **완전히 종료 후 재실행**하고 Connect

## 3. 문제 ② 터미널에서 `rojo` 명령어가 실행되지 않음

VS Code 확장으로 설치했더니 터미널에서 `rojo plugin install`이 실행되지 않았습니다.

### 원인

Aftman이 설치되면서 `~\.aftman\bin`이 PATH에 추가되는데, **설치 전에 열려 있던 VS Code/터미널은 예전 PATH를 그대로 가지고 있었습니다.**

### 해결

- VS Code를 **완전히 종료했다가 다시 실행** (터미널만 새로 여는 것으로는 부족)
- 그래도 안 되면 전체 경로로 실행

  ```powershell
  & "$env:USERPROFILE\.aftman\bin\rojo.exe" plugin install
  ```

## 4. 문제 ③ Aftman error: no aftman.toml files list this tool

```text
Aftman error: Tried to run an Aftman-managed version of rojo, but no aftman.toml files list this tool.
To run rojo from this directory, add it to one of these files:
- C:\Users\<user>\.aftman\aftman.toml
```

### 원인

rojo가 설치되지 않은 것이 아닙니다. Aftman은 **현재 디렉터리와 상위 폴더의 `aftman.toml`에 적힌 도구만** 실행합니다. 프로젝트 폴더가 아닌 곳(예: 홈 디렉터리)에서 `rojo`를 실행해서 발생한 오류입니다.

### 해결

**A. 프로젝트 폴더에서 실행 (권장)**

```powershell
cd <프로젝트 폴더>
rojo plugin install
```

**B. 어디서나 쓰고 싶다면 전역 등록**

```powershell
aftman add --global rojo-rbx/rojo@7.7.1
```

## 정리

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| Can't parse JSON | Studio 플러그인과 Rojo 서버 버전 불일치 | `rojo plugin install` 후 Studio 재시작 |
| `rojo` 명령어 인식 불가 | 기존 세션의 PATH 미갱신 | VS Code 완전 재시작 |
| Aftman: no aftman.toml lists this tool | 프로젝트 폴더 밖에서 실행 | 프로젝트 폴더로 이동 또는 `aftman add --global` |

## 참고: .gitignore

`.vscode/` 폴더와 `sourcemap.json`은 개인 설정이나 생성 파일이므로 `.gitignore`에 추가해 두면 좋습니다.

```gitignore
sourcemap.json
.vscode/
```

