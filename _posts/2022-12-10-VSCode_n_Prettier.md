---
layout: post
title:  "VSCode에 ESLint/Prettier를 적용하자"
date:  2022-12-10 00:00:00 +0900
categories: vscode
tags: vscode
description: ""
---

## VSCode의 설정내용 추가

VSCode의 setting에 아래를 추가한다.  
설정내용을 공유하고 싶을 때에는 `.vscode/settings.json`을 사용하자.

```json
{
  "editor.codeActionsOnSave": {
    "source.organizeImports": true, // 불필요한 import 제거
    "source.fixAll.eslint": true // save할 때 eslint 적용
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "eslint.validate": ["typescriptreact", "typescript"],
  "eslint.workingDirectories": ["project"]
}
```

## prettier 설정

Prettier ESLint extension 플러그인 설치하기

- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [Prettier - eslint](https://marketplace.visualstudio.com/items?itemName=rvest.vs-code-prettier-eslint)

설치하고 나면 하단의 파란 상태바에서 prettier 가 동작하는지 확인할 수 있다.
![vscode_prettier](/images/20221210/vscode_prettier.png)
코드에 오류가 있으면 prettier는 작동하지 않는다. prettier를 작동시키기 위해서는 코드가 정상적으로 컴파일되거나 동작할 수 있어야 하는 상태가 되어야 한다.

## troubleshooting

### prettier 모듈을 로드하지 못할 때

```terminal
[Info  - 10:58:52 AM] Failed to load plugin 'prettier' declared in '.eslintrc.js': Your application tried to access eslint-plugin-prettier, but it isn't declared in your dependencies; this makes the require call ambiguous and unsound.  Required package: eslint-plugin-prettier Required by: /Users/sjun/Workspaces/practics/  Require stack: - /Users/sjun/Workspaces/practics/.eslintrc.js Referenced from: /Users/sjun/Workspaces/practics/.eslintrc.js
(node:72357) Warning: [Warning] The runtime detected new informations in a PnP file; reloading the API instance (/Users/sjun/Workspaces/practics/.pnp.cjs)
(Use `Code Helper --trace-warnings ...` to show where the warning was created)
```

package의 devDependency에 eslint-plugin-prettier를 추가하면 된다.

```terminal
npm install eslint-plugin-prettier --save-dev
```

### yarn berry에서 root에는 tsconfig.json을 두지 않았을 때, 각 project에 생성한 tsconfig.json파일을 찾지 못하는 경우

tsconfig.json의 위치를 각각 지정해둔다

```json
{
  "parserOptions": {
    "project": "./**/tsconfig.json",
    "sourceType": "module"
  }
}
```
