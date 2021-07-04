---
title: "hackathon node boilerplate 보일러 플레이트"
excerpt: "node boilerplate hackathon study"
categories:
    - node
tags:
    - node
last_modified_at: 2021-7-3T00:00:00+09:00
toc: true
toc_sticky: true
---
## hackathon-starter github
link : [hackathon-starter github](https://github.com/sahat/hackathon-starter)

## build
npm install시 오류가 발생한다면
```shell
npm cache clean --force
rm package-lock.json
rm -rf ./node_modules/
npm --verbose install
```
