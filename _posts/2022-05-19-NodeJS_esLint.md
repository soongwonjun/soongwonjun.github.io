---
layout: post
title:  "ESLint 적용하기"
date:  2022-05-18 22:00:00 +0900
categories: nodejs
tags: nodejs eslint vscode
description: "VSCode에서 NodeJS 프로젝트에 eslint를 적용해서 자동으로 convention을 맞춰보자"
---

## Eslint 설치하고 설정파일 만들기

### 프로젝트에 세팅하기

먼저 프로젝트에 eslint를 dev로 설치해준다.

```shell
npm i -D eslint
```

이후 eslint 설정파일을 만든다. 아래는 설정 중 본인이 설정한 내용을 나타낸다. 설정이 끝나면 project root에 설정한 내용대로 .eslintrc 파일이 생긴다. 여기서는 config file format을 yaml로 선택했기 때문에 .eslintrc.yml이 생성된다.

```shell
$ npm init @eslint/config
...
✔ How would you like to use ESLint? · style
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · none
✔ Does your project use TypeScript? · No / Yes
✔ Where does your code run? · browser
✔ How would you like to define a style for your project? · guide
✔ Which style guide do you want to follow? · standard
✔ What format do you want your config file to be in? · YAML
Checking peerDependencies of eslint-config-standard@latest
The config that you've selected requires the following dependencies:

@typescript-eslint/eslint-plugin@latest eslint-config-standard@latest eslint@^8.0.1 eslint-plugin-import@^2.25.2 eslint-plugin-n@^15.0.0 eslint-plugin-promise@^6.0.0 @typescript-eslint/parser@latest
✔ Would you like to install them now with npm? · No / Yes
Successfully created .eslintrc.yml file in /Users/sjun/Workspaces/practice/ts_apply_lint

# 이후 lint가 출력한 패키지를 설치해준다.
$ npm i -D eslint-config-standard
$ npm i -D @typescript-eslint/eslint-plugin@latest eslint-config-standard@latest eslint@^8.0.1 eslint-plugin-import@^2.25.2 eslint-plugin-n@^15.0.0 eslint-plugin-promise@^6.0.0 @typescript-eslint/parser@latest
```

### terminal을 통해 확인하기 위해 package.json에 script 추가하기

```json
"scripts": {
  "lint": "eslint src/**/*.ts",
  "lint-fix": "eslint --fix src/**/*.ts"
}
```

### ESLint가 제대로 적용되는지 확인하기

아래처럼 lint의 에러를 간직한 파일을 예제로 작성해본다. 해당 파일에는 indent문제, 정의만 되고 사용되지 않은 객체, indent 오류, semicolon 오류 등을 포함한다.

```ts
// app.ts
class Xxx {
   private x: any;
}
```

위 파일에 대해 lint를 실행하면 아래처럼 에러가 발생한다.

```terminal
$ npm run lint

> lint-example@1.0.0 lint
> eslint src/**/*.ts


/Users/sjun/Workspaces/practice/ts_apply_lint/src/app.ts
  1:7   error  'Xxx' is defined but never used                no-unused-vars
  2:1   error  Expected indentation of 2 spaces but found 3   indent
  2:18  error  Extra semicolon                                semi
  3:2   error  Newline required at end of file but not found  eol-last
```

--fix 옵션을 사용하면 eslint가 수행될 때 강제로 수정내용이 소스코드에 반영된다. 다만 소스코드의 구조적 수정이 필요한 일부 에러의 경우(no-unused-vars같은) 반영되지 않는다.

```terminal
$ npm run lint-fix

> lint-example@1.0.0 lint-fix
> eslint --fix src/**/*.ts


/Users/sjun/Workspaces/practice/ts_apply_lint/src/app.ts
  1:7  error  'Xxx' is defined but never used  no-unused-vars

✖ 1 problem (1 error, 0 warnings)
```

## VSCode에서 eslint가 자동으로 적용되도록 하기

### VSCode에서 eslint extension 설치하기

[VSCode extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) 페이지에서 설치하거나, VSCode의 extension 탭에서 eslint를 설치한다.  
eslint extension은 자동적으로 working root에서 eslintrc, eslintignore를 읽어서 프로젝트에 적용한다. 초기 설정에는 소스파일의 수정 및 저장 시에도 자동적으로 적용되도록 기본값이 들어있다.

## Troubleshooting

### VSCode에서 eslint가 자동으로 실행되지 않을 때

VSCode의 settings -> eslint.validate 에 다음을 추가한다.

```json
{
  "eslint.validate": [
    "javascript",
    "typescript"
  ]
}
```

### Error: Failed to load parser '@typescript-eslint/parser'

VSCode에서 module을 load할 수 없다며 eslint가 자동으로 실행되지 않는 문제가 있을 수 있는데, 이 때는 VSCode의 eslint extension을 disable하고 재시작 후 enable하면 해결된다.

```text
Error: Failed to load parser '@typescript-eslint/parser' declared in '.eslintrc.yml': Cannot find module '@typescript-eslint/parser'

```

## Reference & References

- [ESLint Getting Start](https://eslint.org/docs/user-guide/getting-started)
- [ESLint Rules](https://eslint.org/docs/rules/)
- [VSCode eslint plugin](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
