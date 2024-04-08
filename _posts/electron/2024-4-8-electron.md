---
title: "Electron reload"
excerpt: "Electron reload"
categories:
  - Electron
tags:
  - Electron
last_modified_at: 2024-4-7T00:00:00+09:00
toc: true
toc_sticky: true
---

Electron을 로컬에서 개발하면서 코드 변경 시 자동으로 Electron 애플리케이션을 재시작하게 하는 것이 가능합니다.
이를 위해서는 개발 의존성으로 `electron-reload` 또는 `nodemon` 같은 도구를 사용할 수 있습니다.
여기서는 각각의 방법에 대해 간단히 설명하겠습니다.

### electron-reload 사용하기

`electron-reload`는 Electron 애플리케이션 개발 시 코드 변경을 감지하고 자동으로 애플리케이션을 재시작하는 간단한 방법을 제공합니다.

1. `electron-reload`를 개발 의존성으로 설치합니다.

   ```bash
   npm install electron-reload --save-dev
   ```

2. 메인 프로세스 파일(예: `main.js`)에서 `electron-reload`를 요구합니다.

   ```javascript
   require("electron-reload")(__dirname, {
     electron: require(`${__dirname}/node_modules/electron`),
   });
   ```

이 설정을 완료하면, Electron 애플리케이션 개발 중에 소스 코드가 변경될 때마다 자동으로 애플리케이션을 재시작합니다.

### nodemon 사용하기

`nodemon`은 Node.js 애플리케이션을 위해 만들어진 유틸리티로, 파일 변경을 감지하고 자동으로 애플리케이션을 재시작합니다. Electron에서도 사용할 수 있습니다.

1. `nodemon`을 개발 의존성으로 설치합니다.

   ```bash
   npm install nodemon --save-dev
   ```

2. `package.json`에서 Electron 애플리케이션을 시작하는 스크립트를 `nodemon`을 사용하도록 수정합니다.

   ```json
   "scripts": {
     "start": "nodemon --watch *.* --exec electron ."
   }
   ```

3. `npm start` 명령어로 애플리케이션을 실행합니다.

이 설정을 통해, 파일 변경 시 `nodemon`이 Electron 애플리케이션을 자동으로 재시작합니다.

두 방법 모두 개발 과정에서 코드 변경을 쉽게 관리하고, 개발 효율성을 높일 수 있도록 도와줍니다. 개발 환경에 맞는 방법을 선택하여 사용하시면 됩니다.
