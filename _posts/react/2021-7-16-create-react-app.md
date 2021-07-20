---
title: "Create React App 정리"
excerpt: "Create React App 정리"
categories:
    - react
tags:
    - react
last_modified_at: 2021-7-16T14:00:00+09:00
toc: true
toc_sticky: true
---

## Getting Started
### App 생성
create-react-app 구조를 따르는 my-app 프로젝트를 하나 생성합니다. 
```shell
npx create-react-app my-app
```
만약 typescript를 사용한다면
```shell
npx create-react-app my-app --template typescript
```
### 실행
my-app 프로젝트를 실행하면 shell에 로컬에서 접속가능한 주소가 표시될 것입니다.  
Local: [http://localhost:3000](http://localhost:3000)  
```shell
npm start
```

## Deployment
다양한 환경에 대한 배포를 지원합니다. 우선 Github에 배포해봅시다.
### Github Pages
#### Step 1: Add homepage to package.json
`package.json`파일에 github repository 주소를 추가합니다.
```shell
"homepage": "https://myusername.github.io/my-app",
```
#### Step 2: Install gh-pages and add deploy to scripts in package.json
gh-pages를 설치합니다.
```shell
npm install --save gh-pages
```
`package.json`에 배포관련 script를 추가합니다.  
`depoly`를 실행하면 `predeploy`가 먼저 자동으로 실행될 것입니다. 
```shell
  "scripts": {
+   "predeploy": "npm run build",
+   "deploy": "gh-pages -d build",
    "start": "react-scripts start",
    "build": "react-scripts build",
```
만약에 GitHub project page가 아니라 user page에 배포를 한다면
 ```shell
  "scripts": {
    "predeploy": "npm run build",
-   "deploy": "gh-pages -d build",
+   "deploy": "gh-pages -b master -d build",
 ```
#### Step 3: Deploy the site by running npm run deploy
deploy를 실행하면 github project repository에 gh-pages란 branch가 생성되고 해당 project page로 접속이 가능할 것입니다.
```shell
npm run deploy
```
## Learn React
React를 배울 때 두가지 접근 방법을 제공합니다.
- **직접 구현해보면서 학습하는 것** : [practical tutorial.](https://reactjs.org/tutorial/tutorial.html)
- **개념을 차근차근 익히며 학습하는 것** : [guide to main concepts.](https://reactjs.org/docs/hello-world.html)  

## links
react 공식 홈페이지 : [https://reactjs.org/](https://reactjs.org/)  
create-react-app : [https://create-react-app.dev/](https://create-react-app.dev/)
