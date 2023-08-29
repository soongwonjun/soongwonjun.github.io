---
layout: post
title:  NestJS Monorepo에서 공용 라이브러리 폴더를 만들어보자
date:  2023-08-01 22:00:00 +0900
categories: nodejs
tags: nodejs nestjs cheatsheet
description: NestJs Monorepo를 더 잘 써봅시다
---

## 폴더 및 구조

```shell
$ tree
.
├── README.md
├── apps
│   ├── app01
│   │   ├── src
│   │   │   ├── app01.controller.ts
│   │   │   ├── app01.module.ts
│   │   │   ├── app01.service.ts
│   │   │   ├── app01.service.spec.ts
│   │   │   └── main.ts
│   │   ├── test
│   │   └── tsconfig.app.json
│   └── app02
│       ├── src
│       │   ├── app02.controller.ts
│       │   ├── app02.module.ts
│       │   ├── app02.service.ts
│       │   ├── app02.service.spec.ts
│       │   └── main.ts
│       ├── test
│       └── tsconfig.app.json
├── libs
│   └── greeting.ts
├── nest-cli.json
├── package.json
├── pnpm-lock.yaml
├── tsconfig.build.json
└── tsconfig.json
```

## nest-cli.json

- sourceRoot 삭제
- root 삭제
- 아래 항목 수정

```json
{
  "compilerOptions": {
    "tsConfigPath": "tsconfig.json",
  }
}
```

## tsconfig

`./tsconfig.json`에 `Source path, include` 추가

```json
{
  "compilerOptions": {
    "paths": {
      "apps/*": ["./apps/*"],
      "libs/*": ["./libs/*"]
    },
  },
  "include": ["apps/**/*.ts", "libs/**/*.ts"]
}
```

이후 소스코드에서 libs의 내용을 import 할 때에는 아래처럼 만들어진다.

- libs/greeting.ts

```typescript
export const greeting = (serviceName: string): string =>
  `Greeting! Here is ${serviceName}`;
```

- apps/app01/src/app01.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { greeting } from 'libs/greeting';  // 절대경로처럼 import되며 이 내용은 이제 nest가 잘 연결해준다.

@Injectable()
export class App01Service {
  getHello(): string {
    return greeting(`app01`);
  }
}
```

## 실행 명령

```shell
# nest 명령 실행 시
$ nest start -d --sourceRoot apps/${app}/src ${app}

# node 명령 실행 시
$ node dist/apps/${app}/main
```

## jest를 위한 설정

### tsconfig.json

```json
{
  "jest": {
   "moduleNameMapper": {
      "^libs/(.*)$": "<rootDir>/libs/$1",
      "^apps/(.*)$": "<rootDir>/apps/$1"
    },
    "roots": [
      "<rootDir>/libs/",
      "<rootDir>/apps/"
    ]
  }
}
```
