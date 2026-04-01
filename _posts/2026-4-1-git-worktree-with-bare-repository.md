---
title: "Git Worktree와 Bare 저장소: 멀티 브랜치 동시 작업을 위한 구조"
excerpt: "왜 일반 저장소보다 Bare 저장소가 Worktree와 궁합이 좋은지, 구조적 이점과 실무 설정법을 완벽 정리합니다."
categories:
  - git
tags:
  - git-worktree
  - bare-repository
  - productivity
  - terminal-tips
  - multi-branching
last_modified_at: 2026-04-01T10:35:00+09:00
toc: true
toc_sticky: true
---

## 1. 왜 Bare 저장소를 사용하는가?

일반적인 저장소(Non-bare)에서 git worktree를 쓰면, **메인 작업 폴더(Main Working Tree)**와 추가 작업 폴더(Secondary Worktrees) 사이의 위계질서가 생깁니다. 하지만 bare 저장소를 쓰면 모든 워크트리가 평등해집니다.

- **메인 폴더의 제약 해제**: 일반 저장소에서는 "현재 체크아웃된 브랜치"를 다른 워크트리에서 동시에 체크아웃할 수 없습니다. 하지만 bare 저장소는 스스로 체크아웃된 브랜치가 없으므로 어떤 브랜치든 자유롭게 워크트리로 뽑아낼 수 있습니다.
- **깔끔한 디렉토리 구조**: 프로젝트 폴더 안에 `.git` 폴더와 소스 코드가 섞이지 않고, 실제 소스 코드는 각각의 독립된 폴더에만 존재하게 됩니다.
- **실수 방지**: 메인 폴더를 실수로 지워버려도 bare 저장소(데이터 본체)만 무사하면 다른 모든 워크트리를 안전하게 유지하거나 복구할 수 있습니다.

## 2. Bare 저장소를 활용한 Worktree 설정 순서

새로운 프로젝트를 시작하거나 기존 프로젝트를 worktree 환경으로 바꿀 때 추천하는 워크플로우입니다.

### 1) Bare 저장소로 클론하기

프로젝트 이름으로 된 폴더를 만들고, 그 안에 `.bare`라는 이름으로 저장소 본체를 받아옵니다.

```bash
mkdir my-project && cd my-project
git clone --bare <원격-저장소-URL> .bare
```

### 2) .git 파일 생성 (연결 고리)

현재 폴더(루트)에서 Git 명령어를 인식할 수 있도록 `.bare`를 가리키는 설정 파일을 만듭니다.

```bash
echo "gitdir: ./.bare" > .git
# 원격(origin)의 모든 브랜치를 로컬의 remotes 목록에 매핑하라는 설정입니다.
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
```

### 3) 워크트리 추가

이제 필요한 브랜치들을 각각의 폴더로 뽑아냅니다.

```bash
git worktree add main       # main 브랜치용 폴더
git worktree add develop    # develop 브랜치용 폴더
git worktree add feat/login  # 작업 중인 기능 폴더
```

**최종 디렉토리 구조:**

```text
my-project/ (루트)
├── .bare/      (진짜 Git 데이터)
├── .git        (포인터 파일)
├── main/       (코드)
├── develop/    (코드)
└── feat/login/ (코드)
```

## 3. Worktree 사용 시 주의할 점

- **브랜치 중복 체크아웃 금지**: 아무리 bare 저장소라도, 현재 다른 워크트리 폴더에서 이미 열어놓은 브랜치를 또 다른 폴더에서 열려고 하면 에러가 납니다. 이는 데이터 무결성을 위한 Git의 보호 조치입니다.
- **삭제 시 주의**: 워크트리 폴더를 그냥 `rm -rf`로 지우면 Git은 여전히 그 워크트리가 존재한다고 인식합니다. 반드시 아래 명령어로 정리해줘야 합니다.

```bash
# 권장 방식
git worktree remove <폴더명>

# 만약 수동으로 폴더를 삭제했다면
git worktree prune
```

## 💡 실전 활용 팁

SvelteKit이나 Spring Boot 프로젝트를 진행할 때, `main` 브랜치에서는 서버를 띄워두고 `feat` 브랜치 폴더에서는 코드를 수정하는 식으로 동시에 작업할 수 있어 생산성이 비약적으로 올라갑니다.

특히 **IntelliJ**나 **VS Code**에서 각 폴더를 별도의 창으로 열어두면 브랜치를 왔다 갔다 하며 `stash` 하거나 `commit` 할 필요가 전혀 없어집니다. 환경 설정이나 의존성(`node_modules` 등)이 브랜치마다 다를 때 발생하는 충돌 문제도 깔끔하게 해결됩니다.

## 참고 자료

- [Git Worktree로 여러 피처 동시에 개발하기 | AI 코딩 시대의 필수 스킬](https://www.youtube.com/watch?v=JtA2JeqlTnI)
