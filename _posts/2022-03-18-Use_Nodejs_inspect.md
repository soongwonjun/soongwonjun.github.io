---
layout: post
title:  "NodeJS inspector 사용하기"
date:  2022-03-18 23:00:00 +0900
categories: nodejs
tags: nodejs debugging
description: NodeJS에 typescript를 작성하고 inspector를 붙여서 디버깅을 해보자
---

## 테스트용 샘플 코드 작성하기

여기서는 nodejs의 inspect를 확인하는 용도이기 때문에 간단한 코드를 작성한다

```typescript
const calculate = (a: number, b: number) => {
  const c = a + b;
  console.log(c);
  return c;
}

setInterval(() => {
  calculate(10, 20);
}, 1000);
```

## 실행하기

실행 옵션에 `--inspect` 옵션을 넣고 실행한다.  
실행하면 chrome/edge와 같은 브라우저에서 inspect할 수 있는 경로가 아래와 같이 나온다.

```terminal
$ npm run build
$ node --inspect dist/app.js
Debugger listening on ws://127.0.0.1:9229/89777fd1-7250-4b8c-a9d7-aaacc3e06ee1
For help, see: https://nodejs.org/en/docs/inspector
```

모던 브라우저를 사용하는 경우에는 이 경로가 굳이 필요 없다.  
여기에서는 주로 사용하는 chrome의 경우를 설명하며, 다른 브라우저의 경우에는 [NodeJS: Debugging](https://nodejs.org/en/docs/guides/debugging-getting-started/) 페이지에 자세히 적혀있다.  
`chrome://inspect` 에 들어가면 `Remote Target` 항목이 있는데, 여기에 `ìnspect`를  클릭하면 실행중인 nodejs app의 inspect 창이 바로 뜬다.
![Chrome_inspect_main](/images/20220318/chrome_inspect.png)

이제 실행중인 앱에 대해서 프로파일링도 가능하고, 코드 단위 디버깅까지 진행할 수 있는 환경이 되었다.
![Chrome_inspect_app](/images/20220318/chrome_inspect_app.png)

## Appendix & References

[NodeJS: Debugging](https://nodejs.org/en/docs/guides/debugging-getting-started/)
