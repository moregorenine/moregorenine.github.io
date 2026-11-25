---
title: "Google Antigravity CLI (agy) 설치 및 시작 가이드"
excerpt: "터미널 환경에서 가볍고 빠르게 Google Antigravity 에이전트를 사용할 수 있는 agy CLI의 OS별 설치 방법, 초기 인증, 권한/실행 모드 옵션(--mode, --dangerously-skip-permissions), 그리고 하단 statusLine에 토큰 및 한도(5시간/주간) 사용량을 표시하는 방법을 정리합니다."
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
    - statusline
    - guide
last_modified_at: 2026-11-25T19:18:00+09:00
toc: true
toc_sticky: true
---

**Google Antigravity CLI(`agy`)**는 별도의 IDE나 데스크톱 앱을 띄우지 않고도 터미널 환경에서 Antigravity AI 에이전트를 빠르게 실행하고 상호작용할 수 있는 경량 CLI 도구입니다.

이 글에서는 **macOS, Linux, Windows 환경별 `agy` 설치 방법**과 **초기 인증, 기본 사용법**, **권한 확인을 줄이거나 생략하고 실행하는 모드 옵션**, 그리고 **하단 상태 표시줄(`statusLine`)에 토큰 및 사용 한도를 실시간으로 표시하는 커스터마이징 방법**을 정리합니다.

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

### 5. 하단 상태 표시줄(`statusLine`) 커스터마이징 — 토큰·한도 표시

`agy` 하단의 상태 표시줄은 에이전트 상태가 바뀔 때마다 외부 스크립트를 호출해 원하는 정보를 출력하도록 커스터마이징할 수 있습니다. 스크립트 `stdin`으로 전달되는 JSON에는 `context_window`(토큰 사용량·비율)와 `quota`(모델 그룹별 한도 및 갱신 남은 시간)가 포함됩니다.

> **참고: Antigravity 쿼터 구조 (`/usage`)**  
> Antigravity의 모델 쿼터는 일간(24시간) 단위가 아니라 **5시간 한도(`5h`)**와 **주간 한도(`weekly`)** 두 가지로 관리됩니다(`/usage` 또는 `/quota`로 확인 가능). Gemini 계열은 `gemini-5h` / `gemini-weekly`, Claude·GPT 계열은 `3p-5h` / `3p-weekly` 버킷으로 전달됩니다.

#### 1) `statusLine` 설정 구조 (`settings.json`)

`~/.gemini/antigravity-cli/settings.json`에 `statusLine` 블록을 추가하면 기본 상태 표시줄을 대체하거나 아래에 한 줄을 덧붙일 수 있습니다.

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.gemini/antigravity-cli/statusline.cmd",
    "stack_with_default": true
  }
}
```

| 속성 | 기본값 | 설명 |
| :--- | :--- | :--- |
| `command` | — | 실행할 셸 명령 또는 스크립트 경로 (`sh -c`, Windows는 `cmd /c`로 실행) |
| `type` | `"command"` | 현재 `"command"`만 지원 |
| `stack_with_default` | `false` | `true`로 두면 기본 상태 표시줄을 유지하고 **바로 아래 줄**에 커스텀 출력을 추가 |
| `padding` | `0` | 상태 표시줄 위쪽 빈 줄 수 |
| `enabled` | `true` | `false`로 바꾸면 설정은 유지한 채 기본 상태 표시줄만 표시 |

> **Windows 경로 주의사항**  
> `agy`는 `statusLine.command` 문자열의 **맨 앞에 있는 `~`만** 홈 디렉터리로 치환합니다. `node ~/.gemini/...`처럼 중간에 `~`를 넣으면 Windows `cmd /c`에서 확장되지 않으므로, `"~/.gemini/antigravity-cli/statusline.cmd"`처럼 래퍼 스크립트를 두거나 절대 경로를 지정하는 것이 안전합니다.

#### 2) 토큰 사용량 + 5시간/주간 사용률·남은 시간 출력 스크립트

외부 패키지(`jq` 등) 없이 Node.js만으로 `stdin` JSON을 파싱해 `토큰 14.2% · 149.3k │ 5h 1% · 4h45m │ 주간 11% · 6d2h` 형태로 출력하는 예시입니다.

**`~/.gemini/antigravity-cli/statusline.mjs`**
```javascript
import fs from 'node:fs';

const R = '\x1b[0m';
const BOLD = '\x1b[1m';
const GRAY = '\x1b[90m';
const GREEN = '\x1b[92m';
const YELLOW = '\x1b[93m';
const RED = '\x1b[91m';
const CYAN = '\x1b[96m';
const WHITE = '\x1b[97m';

function formatTokens(n) {
  if (!Number.isFinite(n) || n <= 0) return '0';
  if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}M`;
  if (n >= 1_000) return `${(n / 1_000).toFixed(1)}k`;
  return String(Math.round(n));
}

function formatRemaining(bucket) {
  let secs = bucket?.reset_in_seconds;
  if (!Number.isFinite(secs) && bucket?.reset_time) {
    const target = Date.parse(bucket.reset_time);
    if (Number.isFinite(target)) {
      secs = Math.max(0, Math.floor((target - Date.now()) / 1000));
    }
  }
  if (!Number.isFinite(secs) || secs < 0) return '-';
  if (secs < 60) return '<1m';
  const days = Math.floor(secs / 86400);
  const hours = Math.floor((secs % 86400) / 3600);
  const mins = Math.floor((secs % 3600) / 60);
  if (days > 0) return hours > 0 ? `${days}d${hours}h` : `${days}d`;
  if (hours > 0) return mins > 0 ? `${hours}h${mins}m` : `${hours}h`;
  return `${mins}m`;
}

function colorForPct(pct) {
  if (pct >= 90) return RED;
  if (pct >= 60) return YELLOW;
  return GREEN;
}

function pickBuckets(quota, model) {
  if (!quota || typeof quota !== 'object') return { short: null, weekly: null };
  const modelText = `${model?.id ?? ''} ${model?.display_name ?? ''}`.toLowerCase();
  const isThirdParty = /(claude|gpt|opus|sonnet)/.test(modelText);
  const prefix = isThirdParty ? '3p' : 'gemini';

  const keys = Object.keys(quota);
  const findBucket = (suffix) =>
    quota[`${prefix}-${suffix}`] ??
    quota[keys.find((k) => k.endsWith(`-${suffix}`)) ?? ''] ??
    null;

  return { short: findBucket('5h'), weekly: findBucket('weekly') };
}

function formatQuotaPart(label, bucket, compact) {
  if (!bucket || bucket.disabled || !Number.isFinite(bucket.remaining_fraction)) {
    return `${GRAY}${label} -${R}`;
  }
  const usedPct = Math.max(0, Math.min(100, Math.round((1 - bucket.remaining_fraction) * 100)));
  const remain = formatRemaining(bucket);
  const c = colorForPct(usedPct);
  return compact
    ? `${GRAY}${label} ${c}${BOLD}${usedPct}%${R}${GRAY}(${remain})${R}`
    : `${GRAY}${label} ${c}${BOLD}${usedPct}%${R} ${GRAY}· ${WHITE}${remain}${R}`;
}

try {
  const raw = fs.readFileSync(0, 'utf8').trim();
  const data = raw ? JSON.parse(raw) : {};
  const cw = data.context_window ?? {};
  const usedPct = Number.isFinite(cw.used_percentage) ? cw.used_percentage : 0;
  const totalTokens = (cw.total_input_tokens ?? 0) + (cw.total_output_tokens ?? 0);
  const width = Number.isFinite(data.terminal_width) ? data.terminal_width : 100;
  const compact = width < 85;

  const pctColor = usedPct >= 90 ? RED : usedPct >= 60 ? YELLOW : CYAN;
  const tokenPart = compact
    ? `${GRAY}토큰 ${pctColor}${BOLD}${usedPct.toFixed(1)}%${R}${GRAY}(${formatTokens(totalTokens)})${R}`
    : `${GRAY}토큰 ${pctColor}${BOLD}${usedPct.toFixed(1)}%${R} ${GRAY}· ${WHITE}${formatTokens(totalTokens)}${R}`;

  const { short, weekly } = pickBuckets(data.quota, data.model);
  const shortPart = formatQuotaPart('5h', short, compact);
  const weeklyPart = formatQuotaPart(compact ? '주' : '주간', weekly, compact);
  const sep = `${GRAY} │ ${R}`;

  process.stdout.write(`${tokenPart}${sep}${shortPart}${sep}${weeklyPart}\n`);
} catch {
  process.stdout.write(`${GRAY}토큰 - │ 5h - │ 주간 -${R}\n`);
}
```

OS별 실행 래퍼는 아래처럼 연결합니다.

- **Windows (`~/.gemini/antigravity-cli/statusline.cmd`):**
  ```bat
  @echo off
  @node "%~dp0statusline.mjs"
  ```
- **macOS / Linux (`~/.gemini/antigravity-cli/statusline.sh` + `chmod +x`):**
  ```bash
  #!/bin/sh
  exec node "$(dirname "$0")/statusline.mjs"
  ```
  *(macOS/Linux에서는 `settings.json`의 `command`를 `"~/.gemini/antigravity-cli/statusline.sh"`로 지정합니다.)*

#### 3) TUI에서 켜기·끄기·원복하기 (`/statusline`)

- **상태 표시줄 끄기 / 켜기:** `/statusline off`, `/statusline on` (또는 인자 없이 `/statusline`으로 토글)
- **커스텀 설정 제거 후 기본값 복원:** `/statusline delete` (또는 `/statusline reset`)
- **참고:** 스크립트가 연속 30회 실패하거나 5초를 초과하면 세션 중 자동 비활성화됩니다.

---

### 6. 설정 파일 및 공식 문서

- **설정 파일 경로:** `~/.gemini/antigravity-cli/settings.json`
- **공식 문서 참고:**
  - [Antigravity CLI Reference](https://antigravity.google/docs/cli/reference.md)
  - [Antigravity CLI Execution Modes](https://antigravity.google/docs/cli/modes.md)
  - [Status line customization](https://antigravity.google/docs/cli/statusline.md)
  - [Model quotas (/usage)](https://antigravity.google/docs/cli/commands/usage.md)
  - [Agent Permissions & Security](https://antigravity.google/docs/permissions.md)
  - [Antigravity CLI Best Practices](https://antigravity.google/docs/cli/best-practices.md)
