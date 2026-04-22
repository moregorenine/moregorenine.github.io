---
title: "특정 파일만 이전 커밋 상태로 되돌리기: git restore 활용법"
excerpt: "실수로 포함된 특정 파일만 골라 이전 커밋 상태로 되돌리고 싶을 때, git restore와 amend를 활용한 깔끔한 롤백 가이드"
categories:
    - git
    - dev-tips
tags:
    - git-restore
    - git-amend
    - version-control
    - rollback
    - productivity
    - terminal
last_modified_at: 2026-04-22T10:45:00+09:00
toc: true
toc_sticky: true
---

이미 커밋(a1)을 완료했는데, 다른 파일은 그대로 두고 `test.js` 하나만 커밋 이전 상태로 되돌려야 하는 상황이 생길 때가 있습니다. 이럴 때 가장 현대적이고 깔끔한 방법은 **git restore** 명령어를 사용하는 것입니다. 

내 상황에 맞는 최적의 경로를 선택해 보세요.

## 1. 커밋 기록 자체를 수정하기 (추천)

아직 원격 저장소에 `push`를 하지 않았고, `a1` 커밋에서 해당 파일이 수정되었다는 흔적조차 남기고 싶지 않을 때 사용하는 방식입니다.

### 1단계: 파일만 이전(HEAD~1) 상태로 되돌리기
```bash
git restore --source=HEAD~1 test.js
```

### 2단계: 변경된 상태를 현재 커밋에 덮어쓰기
```bash
git commit --amend --no-edit
```

**결과:** `a1` 커밋은 유지되지만, 내부적으로 `test.js`는 처음부터 수정하지 않았던 것처럼 감쪽같이 돌아갑니다.

---

## 2. 기존 커밋은 유지하고 되돌린 기록 새로 남기기

이미 `push`를 완료했거나, "수정했다가 다시 되돌렸다"는 명확한 이력을 남겨야 하는 협업 환경일 때 적합합니다.

### 1단계: 파일만 이전 상태로 가져오기
```bash
git restore --source=HEAD~1 test.js
```

### 2단계: 새로운 롤백 커밋 만들기
```bash
git commit -m "Rollback test.js to state before a1"
```

---

## 💡 핵심 포인트: 왜 HEAD~1 인가요?

현재 내가 위치한 커밋이 `a1`이므로, `HEAD`는 `a1`을 가리킵니다. 따라서 `test.js`가 수정되기 전의 온전한 상태는 `a1` 바로 이전인 **`HEAD~1`** (또는 `HEAD^`)에 존재하기 때문입니다.

---

## 🛠️ 만약 명령어가 작동하지 않는다면? (Legacy 방식)

사용 중인 Git 버전이 낮아 `restore` 명령어를 지원하지 않는다면, 익숙한 `checkout` 방식을 사용하세요. 결과는 동일합니다.

```bash
# 파일 복구
git checkout HEAD~1 -- test.js

# 커밋 수정 (선택)
git commit --amend --no-edit
```

이 방법을 익혀두면 실수로 포함된 디버깅 코드나 설정 파일을 커밋했을 때 당황하지 않고 대처할 수 있습니다.
