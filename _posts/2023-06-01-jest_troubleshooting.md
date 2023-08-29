---
layout: post
title:  NestJS에 Jest를 적용하는 과정에서의 troubleshooting
date:  2023-06-01 22:00:00 +0900
categories: nodejs
tags: nodejs cheatsheet jest
---

## Dependency를 제대로 찾지 못하는 문제

### error message

```terminal
● Validation Error:

  Directory /Users/soongwonjun/Workspaces/nest-monorepo/apps/api/src in the roots[0] option was not found.

  Configuration Documentation:
  https://jestjs.io/docs/configuration
```

### solution

package.json에 아래를 추가한다.

```json
{
  "jest": {
    "roots": [
      "<rootDir>/apps/api",  //  추가하기
      "<rootDir>/libs/"
    ],
  }
}
```

### directory tree

```terminal
.
├── README.md
├── apps
│   └── api
│       ├── src
│       │   ├── app.module.ts
│       │   └── main.ts
│       ├── test
│       │   ├── app.e2e-spec.ts
│       │   └── jest-e2e.json
│       └── tsconfig.json
└── libs
    └── api-logger
```

## redis-mock을 inject했지만 제대로 처리되지 않는 문제

### error message

```terminal
 Nest can't resolve dependencies of the MyService (?, ConfigService). Please make sure that the argument default_IORedisModuleConnectionToken at index [0] is available in the RootTestModule context.

    Potential solutions:
    - Is RootTestModule a valid NestJS module?
    - If default_IORedisModuleConnectionToken is a provider, is it part of the current RootTestModule?
    - If default_IORedisModuleConnectionToken is exported from a separate @Module, is that module imported within RootTestModule?
```

### solution

injectable() 에서 이름 매핑을 하는데 이에 대한 명명이 전혀 달라서 생기는 문제  

```ts
const module: TestingModule = await Test.createTestingModule({
  providers: [
    FooService,
    { provide: default_IORedisModuleConnectionToken, useValue: redis },
    { provide: ConfigService, useValue: configServiceMock }
  ],
}).compile();

service = module.get<FooService>(FooService);
```

서비스 코드

```ts
import { InjectRedis, Redis } from '@nestjs-modules/ioredis';
export class FooService {
  constructor(
    @InjectRedis() private readonly redis: Redis,
    configService: ConfigService,
  ) {}
}
```
