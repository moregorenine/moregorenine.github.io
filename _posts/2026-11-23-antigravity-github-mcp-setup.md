---
title: "Antigravity에 GitHub MCP 서버(@modelcontextprotocol/server-github) 연동하기"
excerpt: "Antigravity의 mcp_config.json에 GitHub MCP 서버를 추가하고, GitHub Personal Access Token(Classic / Fine-grained PAT)을 발급해 이슈 등록 등 GitHub 도구를 활성화하는 방법 정리"
categories:
    - AI
    - Development
tags:
    - antigravity
    - mcp-server
    - github
    - pat
    - ai-coding
    - productivity
last_modified_at: 2026-11-23T00:58:00+09:00
toc: true
toc_sticky: true
---

Antigravity에서 AI 에이전트가 GitHub 저장소의 이슈를 등록하거나 PR, 코드 등을 직접 조회·관리하게 하려면 `mcp_config.json`에 **GitHub MCP 서버(`@modelcontextprotocol/server-github`)**를 연동해야 합니다.

이 글에서는 **GitHub Personal Access Token(PAT) 발급 → `mcp_config.json` 설정 → Antigravity UI에서 MCP 서버 재시작**까지의 전체 과정을 정리합니다.

---

### 1. GitHub Personal Access Token (PAT) 발급받기

GitHub MCP 서버가 저장소에 접근할 수 있도록 인증 토큰을 발급받아야 합니다. 보안상 필요한 저장소와 권한만 부여하는 **Fine-grained PAT**를 권장하며, 간편하게 쓰려면 **Classic PAT**를 사용할 수도 있습니다.

1. GitHub 로그인 후 우측 상단 프로필 → **Settings**로 이동합니다.
2. 왼쪽 사이드바 맨 아래 **Developer settings** → **Personal access tokens**를 선택합니다.
3. 아래 두 가지 방식 중 하나를 선택해 토큰을 생성합니다.

#### 방법 A: Fine-grained tokens (최소 권한 권장)
1. **Fine-grained tokens** → **Generate new token** 클릭.
2. **Repository access**에서 **Only select repositories**를 선택하고 대상 저장소(예: `moregorenine/voca-card`)를 지정합니다.
3. **Permissions → Repository permissions**에서 필요한 권한을 설정합니다.
   - 이슈 등록/조회만 필요하다면 **Issues: Read and write** 권한 부여
   - 코드 조회·커밋·PR 작업까지 함께 사용하려면 `Contents`, `Pull requests` 권한도 필요에 따라 추가
4. **Generate token**을 누르고 발급된 토큰을 복사해 둡니다. *(창을 닫으면 다시 확인할 수 없습니다.)*

#### 방법 B: Tokens (classic)
1. **Tokens (classic)** → **Generate new token (classic)** 클릭.
2. **Select scopes**에서 **`repo`** 항목을 체크합니다.
3. **Generate token**을 누르고 발급된 토큰을 복사해 둡니다.

---

### 2. `mcp_config.json`에 GitHub MCP 서버 설정 추가

이제 Antigravity의 MCP 설정 파일(`mcp_config.json`)에 `@modelcontextprotocol/server-github` 구성을 추가하고 발급받은 토큰을 입력합니다.

1. Antigravity 우측 에이전트 패널 상단 **Additional Options (`...`) → MCP Servers → Manage MCP Servers → View raw config**를 클릭해 `mcp_config.json`을 엽니다.
   - Windows 기본 경로: `C:\Users\<사용자명>\.gemini\antigravity\mcp_config.json`
2. `"mcpServers"` 객체 안에 아래 `github` 설정을 추가하고, `<YOUR_GITHUB_PAT>` 자리에 발급받은 토큰을 붙여넣은 뒤 저장(`Ctrl + S`)합니다. *(기존에 다른 MCP 서버가 있다면 쉼표 `,`로 구분해 추가합니다.)*

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_GITHUB_PAT>"
      }
    }
  }
}
```

> **Windows 환경 참고:** Node.js(`npx`)가 설치되어 있어야 합니다. 만약 `npx` 실행 오류가 발생한다면 `"command": "cmd"`, `"args": ["/c", "npx", "-y", "@modelcontextprotocol/server-github"]` 형태로 지정하거나 터미널에서 `node -v`, `npx -v`가 정상 인식되는지 확인하세요.

---

### 3. Antigravity UI에서 MCP 서버 새로고침(재시작)

설정 파일을 저장한 뒤 Antigravity가 변경 사항을 반영하도록 MCP 서버를 새로고침합니다.

1. Antigravity UI의 **Additional Options (`...`) → MCP Servers**로 이동합니다.
2. 목록에 표시된 `github` MCP 서버를 **새로고침(재시작)**합니다.
3. 서버 상태가 정상 연결로 바뀌면 `create_issue`, `list_issues`, `get_issue`, `search_code` 등 GitHub 관련 도구들이 활성화됩니다.

---

### 4. 동작 확인 및 보안 주의사항

- **동작 테스트:** 에이전트 채팅창에 *"moregorenine/voca-card 저장소에 테스트 이슈 하나 등록해줘"*처럼 요청하면 GitHub MCP의 `create_issue` 도구가 호출되어 이슈를 생성할 수 있습니다.
- **보안 주의:** `mcp_config.json`에는 GitHub PAT가 평문으로 저장되므로 해당 파일을 외부에 공유하거나 Git 저장소에 커밋하지 않도록 주의하세요. 또한 토큰 만료 주기를 설정하고, 가급적 대상 저장소와 최소 권한만 허용하는 Fine-grained PAT를 사용하는 것이 안전합니다.
