---
layout: post
title:  "Monorepo의 101"
date:  2023-03-10 22:00:00 +0900
categories: nodejs
tags: nodejs monorepo
description: NodeJS환경에서 Yarn을 사용해서 Monorepo를 구성하여 봅시다.
---

## Yarn을 활용한 Monorepo의 구성 방법

### yarn 초기화

```terminal
yarn init
```

### repository의 내용을 package.json에 작성하기

```json
{
    "workspaces": {
    "packages": [   // 여기에 프로젝트들이 실제로 개발되는 내용이 들어가는 디렉토리를 적는다. 본인은 ~/apps 밑에 모든 application을 넣을 예정이다.
      "apps/**"     //
    ]
  },
  "private": true   // monorepo를 위해서는 private option이 true여야 한다.
}
```

### 작성된 repository에 따라 초기화 해주기

위에서 package.json에서는 apps 밑에 여러개의 서비스를 만들 수 있도록 명명하였다. 지금은 아직 시작이니, server 하나만 만들고 index.ts까지 만들어본다.

```terminal
mkdir apps
mkdir apps/server
cd apps/server
yarn init
mkdir src
touch src/index.ts
```

여기까지 하면 폴더구조는 아래처럼 나온다.

```
.
├── README.md
├── apps
│   └── server
│       ├── README.md
│       ├── package.json
│       └── src
│           └── index.ts
├── package.json
└── yarn.lock
```

## 개발을 위한 기반 작업

### 필요한 라이브러리의 설치

global로 설치할 때에는 일반적인 add 명령어와 동일하지만, 특정 workspace 밑에 설치할 때에는 다른 명령어가 필요하다.

```
# workspace 공용으로 사용하는 경우, ts-node를 설치해보자
yarn add ts-node

# 특정 workspace에서 사용하는 경우, express를 설치해보자
yarn workspace server add express
yarn workspace server add -D @types/express
```

### 실행 script 작성

server의 package.json에 script는 아래처럼 넣어두었으며, 이를 사용해 실행해보도록 한다.

```json
{
  "scripts": {
    "local": "ts-node src/index.ts"
  }
}
```

### 대망의 실행

```terminal
yarn workspace server local
```

## 번외 - zero-install에 대해

yarn v2부터는 yarn berry와 pnp를 사용하여 zero-install 상태를 만들 수 있다.  
모든 패키지 의존성이 `.yarn/cache`에 압축파일로 저장되어, 저장소에 올리고, 다운로드받음으로서 패키지 설치 시간이 극도로 줄어들게 되며 이를 통해 개발환경 설정 혹은 빌드 시간을 단축시킬 수 있다.

### 필수 요구사항, zipfs

<https://marketplace.visualstudio.com/items?itemName=arcanis.vscode-zipfs>

### pnp 사용에 대해

```terminal
# yarn init 이후 아래 명령어를 통해 yarn berry를 활성화한다.
# package.json에 packageManager 필드가 가 생기며, .yarn, .yarnrc.yml파일이 생성된다.
yarn set version berry

# 이 이후에 package를 설치하고나면 .pnp.cjs가 생성된다
# 그리고 .yarn/cache를 확인해보면 의존성 패키지들이 zip파일로 압축된 채로 저장된다.
yarn workspace {repo-name} {package name} add

# vscode에서 pnp 활성화
# yarn berry를 활성화만 시킨 상태에서는 의존성을 찾지 못한다. 때문에 적절한 명령어와 세팅 방법이 필요하다.
# 아래는 vscode일 경우 필요한 명령어이며, 이후 vscode에서 typescript version을 명기해주어야 한다.
yarn dlx @yarnpkg/sdks vscode

# transpile-only 옵션을 켜서 프로젝트를 실행시킨다. 컴파일 단계에서 캐싱된 의존성 라이브러리를 찾지 못해 에러가 나기 때문에 이를 회피하기 위함이다.
# 옵션을 주면 syntax error만 체크하기 때문에 기존에 발생하는 type checking/import 등은 넘어가게 된다.
ts-node --transpile-only src/index.ts

# 번외. yarn berry 비활성화 & 패키지 관리툴을 node_modules로 변경을 원한다면 아래 명령어로 해결 가능하다.
# 이 이후에는 .yarn 폴더를 삭제하고 다시금 yarn install을 통해 패키지 설치를 진행해야 한다.
yarn config set nodeLinker node-modules
```

개발툴에 따라 세팅법이 다르니 아래 링크를 참고하여 세팅하도록 하자. 필자의 주력 툴은 vscode이므로 본문에서는 vscode를 사용한 세팅을 다룬다.

- [yarn editor-sdks](https://yarnpkg.com/getting-started/editor-sdks)
- [typescript transpile only](https://www.npmjs.com/package/typescript-transpile-only)
