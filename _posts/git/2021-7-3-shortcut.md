---
title: "git 단축키 및 설정"
excerpt: "애용하는 git 단축키 및 설정"
categories:
    - git
tags:
    - git
last_modified_at: 2021-7-3T00:00:00+09:00
toc: true
toc_sticky: true
---
## 단축키 등록

```shell
git config --global alias.b "branch"
git config --global alias.ch "cherry-pick"
git config --global alias.co "checkout"
git config --global alias.cm "commit"
git config --global alias.d "diff"
git config --global alias.m "merge"
git config --global alias.r "restore"
git config --global alias.s "status"
git config --global alias.st "stash"
git config --global alias.l "log --oneline --graph -30 --pretty=format:'%C(Yellow)%h%C(reset) %s %C(auto)%d%C(reset) %C(dim white)%an%C(reset) %C(dim cyan)%ci%C(reset)'"
git config --global alias.ll "log --oneline --graph  --all -30 --pretty=format:'%C(Yellow)%h%C(reset) %s %C(auto)%d%C(reset) %C(dim white)%an%C(reset) %C(dim cyan)%ci%C(reset)'"
```