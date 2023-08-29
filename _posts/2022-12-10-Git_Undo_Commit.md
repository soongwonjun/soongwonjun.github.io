---
layout: post
title:  "Git 실수를 만회해보자"
date:  2022-12-10 00:00:00 +0900
categories: github
tags: cheatsheet github
description: "Git 커밋 되돌리기, 롤백하기"
---

## Push 하기 전에 commit을 수정하기

```terminal
git commit --amend
```

## Push 이후에 commit 내역을 삭제하기

HEAD로부터 수정하고자 하는 commit만큼 reset 한다.  
그 후에 empty commit을 넣으면 로컬의 CL부터 이후의 모든 commit은 지워진다.  
항상 작업하기 전에 현재 작업을 stash해두는 것을 잊으면 안된다.  
아래는 1개 commit을 되돌리는 작업이다  

```terminal
$ git stash
$ git reset HEAD~1
// 변경내용이 이때 다시 만들어지는데 필요에 따라 revert 혹은 stash를 하여 내용을 잘 없애 둔다
$ git commit -m "empty commit for undo" --allow-empty
$ git push --force
```

## 커밋이나 머지할 때 충돌난 부분을 수정해보자

```terminal
git checkout '${fromBranch}'
git fetch origin '${대상 line}'
git checkout '${toBranch}'
git pull
git merge --no-ff '${fromBranch}'
// 충돌내용을 수정하고 커밋 & 푸쉬하자
git push origin '${toBranch}'
```
