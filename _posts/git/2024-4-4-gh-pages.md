---
title: "gh-pages를 통한 github pages 배포"
excerpt: "gh-pages를 통한 github pages 배포"
categories:
    - git
tags:
    - git
    - github
    - gh-pages
    - deploy
last_modified_at: 2024-4-4T00:00:00+09:00
toc: true
toc_sticky: true
---

`gh-pages`는 GitHub Pages로 빠르게 정적 사이트를 배포할 수 있도록 도와주는 간편한 커맨드라인 도구입니다. 이를 활용하면 빌드된 정적 파일들을 GitHub Pages로 쉽게 배포할 수 있습니다. 아래는 `gh-pages`를 사용하여 프로젝트를 GitHub Pages에 배포하는 기본적인 단계를 설명합니다.

### 1. `gh-pages` 패키지 설치

먼저, 프로젝트에 `gh-pages` 패키지를 개발 의존성으로 추가합니다. 프로젝트 디렉토리에서 다음 명령어를 실행하세요.

```bash
npm install gh-pages --save-dev
```

또는 `yarn`을 사용하는 경우:

```bash
yarn add gh-pages --dev
```

### 2. `package.json`에 배포 스크립트 추가

`package.json` 파일에 배포를 위한 스크립트를 추가합니다. 보통 빌드 스크립트를 실행한 후, `gh-pages`를 사용하여 `build` 디렉토리(또는 프로젝트의 빌드 결과물이 위치하는 디렉토리)를 GitHub Pages로 배포합니다.

예를 들어, React 애플리케이션의 경우 다음과 같이 설정할 수 있습니다.

```json
"scripts": {
  "predeploy": "npm run build",
  "deploy": "gh-pages -d build"
}
```

`predeploy` 스크립트는 `deploy` 스크립트가 실행되기 전에 자동으로 실행되며, 여기서는 애플리케이션을 빌드합니다. `deploy` 스크립트는 빌드된 결과물을 GitHub Pages로 배포합니다.

### 3. 배포하기

설정을 완료했다면, 다음 명령어를 실행하여 GitHub Pages로 배포할 수 있습니다.

```bash
npm run deploy
```

또는 `yarn`을 사용하는 경우:

```bash
yarn deploy
```

### 4. GitHub Pages 설정 확인

배포가 성공적으로 완료된 후, GitHub 저장소의 설정으로 이동하여 GitHub Pages 섹션을 확인합니다. 배포된 사이트의 URL을 이 섹션에서 확인할 수 있습니다.

### 추가 설정

- **Custom Domain**: GitHub Pages 설정에서 사용자 지정 도메인을 사용하려면, 저장소의 `gh-pages` 브랜치 루트에 `CNAME` 파일을 추가하고, 내부에 도메인 이름을 기재합니다.
- **Single Page Application 배포**: SPA(Single Page Application)를 배포할 때는 404 페이지를 `index.html`로 리다이렉트하는 설정이 필요할 수 있습니다. 이를 위해 `404.html` 파일을 `index.html`의 복사본으로 만듭니다.

이러한 단계를 통해 `gh-pages`를 사용하여 프로젝트를 GitHub Pages에 배포할 수 있습니다. GitHub Pages는 정적 사이트만 호스팅할 수 있으므로 서버 사이드 렌더링이 필요한 애플리케이션은 다른 배포 방법을 고려해야 합니다.
