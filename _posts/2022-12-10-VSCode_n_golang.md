---
layout: post
title:  "VSCode에 Golang을 세팅하고 작업해보자"
date:  2022-12-10 00:00:00 +0900
categories: vscode
tags: vscode
description: ""
---

## vscode extension 설치

- [Go](https://marketplace.visualstudio.com/items?itemName=golang.Go)
- [Go Nightly](https://marketplace.visualstudio.com/items?itemName=golang.go-nightly)

## settings.json 설정

```json
{
    "go.gopath": "/Users/sjun/.gvm/pkgsets/go1.18/global",
    "go.goroot": "/Users/sjun/.gvm/gos/go1.18",
    "go.coverOnSave": true, // unittest, coverage를 작성하고 실행할 때 사용하자
}
```

## 참고

- [Go Standard Layout](https://github.com/golang-standards/project-layout)
