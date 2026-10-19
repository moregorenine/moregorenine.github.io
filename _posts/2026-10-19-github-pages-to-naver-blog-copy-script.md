---
title: "GitHub Pages(Jekyll) 글을 서식 깨짐 없이 네이버 블로그로 옮기는 방법 — MD2Naver 웹 도구"
excerpt: "네이버 스마트에디터 ONE에 글을 붙여넣을 때 코드 블록이 풀리고, 줄간격이 벌어지며, 소제목마다 거대한 구분선이 생기는 원인을 분석하고 이를 원클릭으로 해결하는 오픈소스 웹 도구(md-to-naver)를 소개합니다."
categories:
  - blog
  - javascript
tags:
  - github-pages
  - jekyll
  - minimal-mistakes
  - naver-blog
  - smarteditor
  - clipboard
  - javascript
  - bookmarklet
last_modified_at: 2026-10-19T15:32:00+09:00
toc: true
toc_sticky: true
---

GitHub Pages(Jekyll)로 기술 블로그나 개인 블로그를 운영하면서, 한국어 검색 유입을 위해 **네이버 블로그에도 동일한 글을 발행(Cross-posting)**하고 싶을 때가 있습니다.

하지만 네이버 블로그는 스팸·어뷰징 방지를 이유로 과거에 지원하던 **글쓰기 Open API와 XML-RPC를 전면 종료**했고, 에디터(스마트에디터 ONE)에서 **HTML 소스 직접 입력 모드도 제거**했습니다.

결국 렌더링된 GitHub Pages 웹 화면을 드래그해서 네이버 블로그 에디터에 붙여넣기(`Ctrl + C` → `Ctrl + V`)해야 하는데, 그냥 붙여넣으면 서식이 처참하게 깨집니다. 이번 글에서는 **네이버 스마트에디터 ONE에서 서식이 깨지는 정확한 기술적 원인**과, 이를 클릭 한 번으로 완벽하게 해결해 주는 전용 웹 도구 **[MD2Naver Studio (md-to-naver)](https://moregorenine.github.io/md-to-naver/)**를 소개합니다.

---

## 1. 왜 GitHub Pages 화면을 네이버 블로그에 붙여넣으면 깨질까?

네이버 스마트에디터 ONE은 외부에서 복사된 HTML 클립보드 데이터를 붙여넣을 때 자체 필터(Sanitizer)를 거쳐 독자적인 문서 구조로 변환합니다. 이 과정에서 Jekyll(`minimal-mistakes` 테마 등)의 HTML/CSS와 충돌하며 크게 **5가지 문제**가 발생합니다.

### ① 소제목(`<h2>`) 아래마다 거대한 구분선이 생기는 현상
가장 당황스러운 현상입니다. 마크다운 본문에 `---`(`<hr>`)를 쓰지 않았는데도, 네이버 블로그에 붙여넣으면 모든 `## 소제목` 아래에 높이가 엄청나게 큰 **네이버 구분선 컴포넌트**가 강제로 생성됩니다.

- **원인:** `minimal-mistakes` 테마의 CSS(`_page.scss`)에는 소제목 아래에 얇은 밑줄을 긋는 스타일이 기본 적용되어 있습니다.
  ```scss
  .page__content h2 {
    padding-bottom: 0.5em;
    border-bottom: 1px solid $border-color;
  }
  ```
- 크롬 브라우저에서 화면을 드래그해 복사하거나 일반적인 `document.execCommand('copy')`를 실행하면, 크롬은 페이지의 CSS(`main.css`)를 계산해 `<h2>` 태그에 `border-bottom: 1px solid ...` 스타일을 자동으로 입혀 클립보드에 담습니다.
- **네이버 스마트에디터는 태그에 `border-bottom`이 감지되면 무조건 그 아래에 거대한 자체 구분선(`hr`) 모듈을 박아버립니다.**

### ② 코드 블록(`<pre><code>`)이 일반 줄글로 풀려버림
Jekyll의 문법 강조기(Rouge)가 생성하는 `div.highlighter-rouge > pre > code` 구조를 네이버 에디터는 인식하지 못하고 배경색과 테두리를 모두 날린 채 일반 텍스트로 뭉개버립니다.

### ③ 리스트(`<li>`)와 인용구(`<blockquote>`) 줄간격 뻥튀기
마크다운 파서가 `<li><p>내용</p></li>` 형태로 렌더링한 경우, 네이버 에디터는 `<li>` 안의 `<p>`를 추가 줄바꿈으로 해석해 목록 사이가 두세 줄씩 벌어집니다.

### ④ 표(`<table>`) 테두리 증발
CSS로만 테두리가 지정된 표는 네이버에 붙여넣는 순간 선이 사라집니다. 고전적인 HTML 속성인 `border="1"`과 인라인 셀 스타일을 함께 넣어주어야 표가 유지됩니다.

### ⑤ 콘솔에서 `navigator.clipboard.write` 실행 시 포커스 오류
개발자 도구(F12) 콘솔에서 최신 비동기 클립보드 API(`navigator.clipboard.write`)를 실행하면, 포커스가 웹페이지 본문이 아닌 개발자 도구 창에 있기 때문에 아래 보안 오류가 발생하며 복사가 거부됩니다.

```text
Uncaught (in promise) NotAllowedError: Failed to execute 'write' on 'Clipboard': Document is not focused.
```

---

## 2. 해결 전략: 네이버 스마트에디터 맞춤형 DOM 세탁 + 순수 HTML 주입

위 5가지 문제를 한 번에 해결하기 위한 핵심 원리는 다음과 같습니다.

1. **코드 블록과 인용구를 '1칸짜리 표(`<table>`)'로 변환**: 네이버 에디터가 절대 깨뜨리지 못하는 요소가 바로 `border="1"`이 적용된 `<table>`입니다. 코드 블록은 회색 배경(`#f6f8fa`)의 1칸 표로, 인용구는 왼쪽 굵은 테두리(`border-left: 4px solid`)가 있는 1칸 표로 치환합니다.
2. **불필요한 태그 제거 및 `<p>` 언래핑(Unwrap)**: 목차(`.toc`), 제목 옆 앵커 아이콘(`.header-link`), `<hr>` 태그를 제거하고, `<li>`와 `<blockquote>` 내부의 `<p>` 태그를 벗겨내어 줄간격이 벌어지지 않게 합니다.
3. **`e.clipboardData.setData`로 외부 CSS(`main.css`) 오염 원천 차단**:
   - 포커스 오류가 없는 `document.execCommand('copy')`를 호출하되, `copy` 이벤트를 가로채서(`e.clipboardData.setData('text/html', clone.innerHTML)`) 크롬이 사이트의 `main.css`(`h2` 밑줄 등)를 클립보드에 몰래 섞지 못하도록 **순수하게 가공된 HTML 문자열만 클립보드에 직접 꽂아 넣습니다.**

---

## 3. 클릭 한 번으로 끝내기 — MD2Naver Studio 웹 도구

매번 개발자 도구 콘솔을 열고 스크립트를 입력하는 번거로움을 없애기 위해, 위 모든 변환 로직을 웹 브라우저에서 바로 사용할 수 있도록 **[MD2Naver Studio (md-to-naver)](https://moregorenine.github.io/md-to-naver/)**를 제작해 배포했습니다.

- 🚀 **웹 도구 바로가기:** [https://moregorenine.github.io/md-to-naver/](https://moregorenine.github.io/md-to-naver/)

### 주요 기능 및 특징

1. **🎯 복사할 영역 클래스명 / CSS 선택자 자유 지정**
   - `Minimal Mistakes (.page__content)`, `Jekyll 기본 (.post-content)`, `티스토리 (.contents_style)` 등 원하는 블로그의 본문 클래스명을 직접 입력하거나 프리셋 버튼으로 선택해 필요한 영역만 정확히 추출할 수 있습니다.
   - 목차(`.toc`), 공유 버튼(`.page__share`), 사이드바 등 제외할 요소 선택자도 자유롭게 커스터마이징 가능합니다.

2. **🌐 3가지 변환 모드 지원**
   - **웹 URL 추출 모드:** 배포된 GitHub Pages나 외부 기술 블로그의 포스트 URL만 입력하면 본문을 자동으로 긁어와 네이버 스마트에디터 맞춤형 서식으로 즉시 변환합니다.
   - **마크다운(`.md`) / HTML 직접 입력 모드:** 아직 블로그에 배포하기 전인 마크다운 원문(YAML Front Matter 자동 제거)을 붙여넣으면 실시간으로 렌더링 및 변환됩니다.
   - **맞춤형 북마클릿(Bookmarklet) 생성기:** 내가 입력한 클래스명 설정이 그대로 반영된 **원클릭 북마클릿 버튼**을 자동으로 만들어 줍니다. 크롬 북마크 바에 끌어다 놓으면 어떤 사이트에서든 버튼 클릭 한 번으로 복사가 끝납니다.

3. **⚙️ 네이버 스마트에디터 ONE 최적화 옵션 & 실시간 미리보기**
   - 구분선(`<hr>`) 제거, 소제목 밑줄 제거, 코드 블록 테마 선택(`GitHub Light` / `VS Code Dark`), 인용구 변환 옵션을 스위치로 켜고 끌 수 있습니다.
   - 우측 미리보기 화면에서 네이버 블로그에 붙여넣어질 실제 디자인을 눈으로 확인한 뒤 **[📋 네이버 블로그용 복사하기]** 버튼을 누르고 에디터에 `Ctrl + V`만 하시면 됩니다.

---

### 마무리

GitHub Pages나 마크다운으로 작성한 글을 네이버 블로그로 옮길 때마다 줄간격과 코드 블록이 깨져 스트레스받으셨다면, 지금 바로 **[MD2Naver Studio](https://moregorenine.github.io/md-to-naver/)**를 사용해 보세요!


