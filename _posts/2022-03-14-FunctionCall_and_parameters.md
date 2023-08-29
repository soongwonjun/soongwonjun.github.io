---
layout: post
title:  "함수 호출과 Parameter 전달"
date:  2022-03-14 20:00:00 +0900
categories: cs
tags: cs nodejs java
description: Call by Value, Call by Reference를 정리해보자
summary: 함수를 호출할 때 변수를 전달하는 방법(Call by Value, Call by Reference)과 작동에 대해서
---

## 일반적인 의미

### Call by value

parameter의 값만을 전달하며, caller와 callee간의 공유가 발생하지 않는다.  
특히 thread와 같은 동시성 이슈가 발생할 경우에는 call by value를 통해서 이슈를 회피해야 한다.  

### Call by reference

parameter의 값이 아닌 주소를 전달하며, callee가 parameter의 값을 변경하면 caller도 영향을 받는다.  
전달받은 parameter의 내부 내용이 바뀔 수 있으므로 코드 전체를 이해하고 있어야 하며, 문제가 발생할 소지가 다분하다.

## NodeJS와 Java에서는 어떻게 작동할까?

### NodeJS에서

primitive type은 call by value를, object type은 call by reference를 사용한다.

### Java에서

Java에서 Caller는 Callee에게 전달하는 모든 parameter는 call by value형태로 전달한다. 이는 object type도 예외는 아니다.  
caller에서의 arg의 instance 주소를 참조하는 새로운 변수 arg#1을 만들고, arg#1이 바라보는 메모리 주소를 arg와 동일하게 복사한다.  
새로이 만들어진 arg#1을 callee에게 전달하는 용도로 사용하고, caller는 arg를 계속 사용한다.  
caller와 callee가 arg를 보는 메모리상 주소는 같지만, 실제 instance의 주소를 가진 변수는 각각 arg, arg#1로 다르다.  

```java
public class TestOne {
  boolean isInit = false;

  public static void main(String args[]) {
    TestOne testOne = new TestOne();
    this.init(testOne);
  }
  
  public static void init(TestOne testOne) {
    // ... do something
  }
}
/*
   heap --------> #1 {testOne: {isInit: false}}  <--
              |                                    |
   main -> testOne (heap #1)                       |
   init -> testOne (heap #1) -----------------------
*/
```

때문에 return value로 새 object의 포인터를 반환하지 않는 한, caller가 바라보는 object는 변경되지 않는다.  
하지만 testOne 안쪽의 field가 변경되면, caller가 바라보는 testOne instance의 field도 같이 바뀐다. callee에서 바라보는 object의 instance가 caller가 바라보는 object의 instance이기 때문이다.
