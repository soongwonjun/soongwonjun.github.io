---
layout: post
title:  NestJS Monorepo에서 실행 스크립트를 하나로 만들어보자
date:  2023-08-01 22:00:00 +0900
categories: nodejs
tags: nodejs nestjs cheatsheet monorepo
---

## 실행 스크립트를 단순화해보자

NestJS에서 monorepo를 개별적으로 실행할 때에는 실행 명령어가 조금 달라지게 된다. 이 때 마다 실행 명령어를 만들기가 너무 싫어서 하나의 명령어로 실행할 수 있도록 해보자

## 목표

- 빌드하기, 개발모드 실행하기, 운영모드 실행하기로 각각 명령어를 하나씩 만들어보자
- 서비스가 늘어나도 명령어는 변함이 없도록 해보자
- 서비스가 없으면 에러가 나오도록 해보자

```json
{
  "scripts": {
    "app-checker": "[ ! -d apps/$app/src ] && echo \"not exists app.\" && exit 1 || exit 0",
    "build": "app=$app pnpm app-checker && nest build $app",
    "run:dev": "app=$app pnpm app-checker && nest start -w -d --sourceRoot apps/$app/src $app",
    "run:prod": "app=$app pnpm app-checker && node dist/apps/$app/main"
  }
}
```

## 실행해보기

여기서는 pnpm을 사용한다.

```terminal
# 서비스 생성
$ nest g app svc

# 대상 서비스 없이 실행
$ pnpm run:dev
...
not exists app.
 ELIFECYCLE  Command failed with exit code 1.
 ELIFECYCLE  Command failed with exit code 1.

# 없는 서비스 실행
$ app=noapp pnpm run:dev
...
not exists app.
 ELIFECYCLE  Command failed with exit code 1.
 ELIFECYCLE  Command failed with exit code 1.

# 새로 만든 서비스 실행
$ app=svc pnpm run:dev
...
 Info  Webpack is building your sources...

webpack 5.87.0 compiled successfully in 217 ms
Debugger listening on ws://127.0.0.1:9229/6e146f57-30e7-47bd-aad0-42fa97b7f3da
For help, see: https://nodejs.org/en/docs/inspector
Type-checking in progress...
[Nest] 39172  - 08/09/2023, 11:29:23 AM     LOG [NestFactory] Starting Nest application...
[Nest] 39172  - 08/09/2023, 11:29:23 AM     LOG [InstanceLoader] GrpcClientModule dependencies initialized +7ms
[Nest] 39172  - 08/09/2023, 11:29:23 AM     LOG [RoutesResolver] GrpcClientController {/}: +8ms
[Nest] 39172  - 08/09/2023, 11:29:23 AM     LOG [RouterExplorer] Mapped {/, GET} route +1ms
[Nest] 39172  - 08/09/2023, 11:29:23 AM     LOG [NestApplication] Nest application successfully started +1ms
No errors found.
```
