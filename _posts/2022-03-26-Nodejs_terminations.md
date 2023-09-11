---
layout: post
title:  "NodeJS에서 app의 종료와 후처리에 대해서"
date:  2022-03-26 00:00:00 +0900
categories: nodejs
tags: nodejs
description: NodeJS에서 App이 종료되는 경우와 그에 따른 처리에 대해서 찾아보자
---

## 가장 간단하게 App을 종료시킬 때

`process.exit`을 사용하는 방법이 있다. 이 함수를 사용하면 가능한 한 빠르게 app이 종료된다.  
exit 함수를 호출할 때 exit code를 넣는데, 이 exit code에 따라서 process event가 다르게 등록된다. 0으로 넣는 경우에는 process event가 exit으로 들어간다.  
또한 event loop에 추가 작업이 더 스케쥴링되지 않도록 하여, callback 혹은 OS I/O 등 다음 작업이 이어지지 않도록 한다. 이는 파일 쓰기, 혹은 Network 처리 등이 도중에 종료됨을 의미한다.  
exit 함수의 exitCode는 정상일 때는 0을 사용하며, 에러와 관련된 exitCode는 [여기](https://nodejs.org/api/process.html#exit-codes)를 참조하자.

```ts
process.exit(0);
```

또다른 방법으로는 `process.abort`를 사용하는 방법이 있다.  
이 함수는 exit과는 다르게 app을 즉각적으로 종료한다. 만약, OS레벨에서 dump를 지원한다면 dump를 만들어주기도 한다.

```ts
process.abort();
```

마지막으로, `process.kill`을 사용하여 특정 signal event를 발생시키면서 종료하는 방법이 있다.  
이 방법은 OS를 통해서 app을 종료하는 과정과 동일하기 때문에 OS에서 사용중인 signal을 같이 보낼 수 있다.

```ts
process.kill(process.pid, 'SIGTERM');
```

### process event에 대해서

nodejs에서는 app의 다양한 상황에 대처하기 위해 process event를 사용하여 app이 어떻게 종료되는지 확인할 수 있도록 해준다. process event에는 OS의 signal을 감지하는 signal event와 nodejs에서 구현한 exit event가 있다.  
process event는 nodejs의 event의 한 종류로 취급된다. 이벤트로 취급되기 때문에 다른 이벤트와 동일하게 libuv의 event loop에 의해 처리된다. 그리고 이렇게 등록된 이벤트는 다음 번 event loop에 의해 처리될 수 있도록 스케쥴링된다.  
정의된 process event는 [여기](https://nodejs.org/api/process.html#process-events)를 참조하자.  
아래 코드에서는 `process.kill`을 진행할 때, `SIGTERM`을 발생시키며, process에서는 `SIGTERM` 이벤트에 대한 handler를 등록하여 handler에 등록된 작업, 여기서는 console.info를 수행할 수 있도록 한다.

```ts
process.on('SIGTERM', () => {
  console.info('Process terminated');
});

process.kill(process.pid, 'SIGTERM');
```

## 비정상적인 종료를 감지하는 방법

app이 관리자의 의도 없이 종료되는 경우는 수없이 많다. 대표적으로 에러로 인한 서버 크래시가 있을 것이다. 그리고 대부분의 서버 app은 후처리가 꼭 필요할 것이다.  
대표적으로 javascript에서 에러가 발생하여 app의 crash로 인하여 종료될 경우 `uncaughtException` 이 발생한다. 다행이 nodejs에서는 signal event 이외에 nodejs app에서 크래시가 발생하며 죽는 경우에 대해 process event로 구현해 두었으며, 이를 이용하여 후처리를 진행할 수 있다.  
이런 추가적인 루틴을 event handler를 이용해 구현함으로서 우리는 이 app의 후처리를 구현할 수 있고, 어떤 문제가 발생하였는지 확인할 수 있으며, 발생한 내용에 대해서 로그를 남겨 시스템 리포트로 변환할 수 있다. 즉 우리는 app에서 크래시가 발생하더라도 graceful shutdown을 할 수 있다는 의미이다.

```ts
process.on('uncaughtException', (err, origin) => {
  console.error(`Caught Error: ${err}`);
});
```

## process event와 함께 하는 graceful shutdown과 logging

우리는 app을 만들 때 종료 작업에 대해 생각한다. app이 종료될 때 반드시 세팅해야 하는 정보가 있거나, 비정상 종료일 때 문제에 대한 리포팅을 하려는 경우가 여기에 속할 것이다.  
nodejs에서 process event를 사용해서 app이 종료될 때를 감지할 수 있고, process event에 handler를 붙임으로서 어느정도 대응을 할 수 있게 해준다.  
단 process 종료와 관련된 event에 등록된 handler에서 일반적인 작업(서버를 어떻게든 살리는 작업과 같은)을 수행하면 문제가 발생할 소지가 다분하다. nodejs app이 어떠한 이유에서든 shutdown 과정을 시작한 경우 event loop에 등록된 모든 event, i/o callback 등을 클린업하기 시작하기 때문이다. 그리고 이 handler에서 문제가 발생할 경우 제대로 된 후처리 루틴이 작동하지 않을 수 있으며, 최악의 경우에는 app이 제대로 죽지 않아 deadlock을 만들 수도 있다.  
아래에서는 app이 `SIGTERM`에 의해 죽거나 `uncaughtException`이 발생해서 죽을 때, 이 event에 handler를 등록하여 graceful shutdown을 진행하고, 추가로 문제에 대한 로깅을 할 수 있도록 한다.  

```ts
class App {
  private serviceId;
  private sharedCache;

  terminate(err?: any) {
    if (err) {
      console.error(`Caught Error: ${error}`);
    }
    sharedCache.setDead(this.serviceId);
  }
}
const app: App = new App();

process.on('SIGTERM', () => {
  app.terminate(() => process.exit(0));
});

process.on('uncaughtException', (err) => {
  app.terminate((err) => process.exit(0));
});
```

## Reference & References

- [geeksforgeeks: nodejs lifecycle](https://www.geeksforgeeks.org/nodejs-program-lifecycle/)
- [nodejs.dev: How to exit from a Node.js program](https://nodejs.dev/learn/how-to-exit-from-a-nodejs-program)
- [nodejs.dev: process-events](https://nodejs.org/api/process.html#process-events)
- [nodejs.dev: Process signal event](https://nodejs.org/api/process.html#signal-events)
