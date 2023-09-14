---
layout: post
title:  Javascript와 type에 대해서
date:  2023-09-13 22:00:00 +0900
categories: cs
tags: cs article
---

## Javascript는 type을 지원할까?

혹자는 Javascript에는 타입을 지원하지 않는다고 한다. 하지만 Javascript는 이미 여러가지 타입들을 지원하고 있으며, 이 타입을 판별하는 방법도 제공한다. 그리고 이 타입을 사용하여 value/function을 설정하고 프로그램을 작성하고 실행시킨다. 그런데 왜 javascript에는 타입이 없다고 하는 것일까?  
이것은 Javascript의 개념 중 `value`와 `variable`을 명확하게 구분하지 못하기 때문에 생기는 오해이다.

- value: javascript의 identifier가 바라보는 값에 저장되어 있는 값 정보이다. 이 value에는 type 정보가 있다. type의 명세에 대해서는 [이쪽](https://262.ecma-international.org/10.0/#sec-type)을 참고하자
- variable: identifier를 만들기 위한 property이며, type 정보가 없다. (var, let, const)

이제 우리는 `Javascript는 type을 정의하고 지원한다. 그리고 Javascript의 variable에는 type이 할당되지 않는다` 라고 말할 수 있게 되었다. 어째서 이런 구조를 가지느냐에 대해서는 ECMAScript의 배경과 관련이 있다.

### ECMAScript에 대해서

ECMAScript는 Javascript와 같은 스크립트 언어의 표준화를 위해 만들어진 문서이다. 기본적으로 이 언어는 객체 기반이며 ECMAScript는 객체간의 커뮤니케이션의 모음이다. ECMAScript에서 Object는  attribute를 가진 0개 이상의 property의 모음이다. 또한 property는 다른 객체, primitive value, function을 가질 수 있다.  

### ECMAScript가 집중하는 부분

ECMAScript 문법은 상당히 사용하기 쉬운 스크립트 언어라는데 초점을 맞추고 있다. 변수는 타입을 정의할 필요가 없으며, property는 type 연결되지 않으며(value와 연결된다) 정의된 함수는 호출되기 전까지 그 내용이 표현될 필요가 없다는 부분이다.  

### 번외. ECMAScript에서 변수가 선언되고 나서 어떻게 될까?

Javascript를 실행하기 위해서 작성된 코드가 먼저 평가된다. 이 때 Lexical Environment가 생성되고, Initializer에 의해 Identifier가 만들어진다. 이 이후 Identifier의 값이 Lexical Environment 저장된 후 Lexical Binding 정보가 생성된다. Lexical Binding 정보는 Environment Record에 기록되고 이 때 identifier와 value가 매핑되어 Lexical Environment에 접근이 가능해진다. 예외적으로 Let 변수는 identifier에 undefined가 바인딩된다.
우리가 선언한 변수는 Lexical Environment의 어딘가를 바라보는 하나의 프로퍼티라고 할 수 있다. 조금 더 깊게 들어가면 코드에 대한 평가, Realm, Execution Context, Queue, Agent 등 많은 개념이 나오지만 이는 본 토막글과는 주제가 다르니 다른 토막글로 정리해보자.

## References

- [Wiki, ECMAScript](https://en.wikipedia.org/wiki/ECMAScript)
  - [Wiki, ECMA스크립트](https://ko.wikipedia.org/wiki/ECMA%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8)
- [ECMAScript, Overview](https://262.ecma-international.org/10.0/#sec-ecmascript-overview)
- [ECMAScript, Type](https://262.ecma-international.org/10.0/#sec-type)
- [ECMAScript, Variable statement](https://262.ecma-international.org/10.0/#sec-declarations-and-the-variable-statement)

## Appendix

- Lexical Environment(LE): identifier와 value/function의 관계를 정의하는 명세를 가진 `lexical nesting structure` 이다. LE는 Envorinment Record와 외부 LE 참조에 대한 null로 구성되어 있다. 매번 코드가 평가될 때 마다 새로운 Environment Record가 생성되고 평가된 코드를 위해 identifier가 바인딩된다.
- Lexical nesting structure: Execution Contexts에서 Scope와 Identifier를 관리하는 기능
- Environment Record(ER): identifier와 LE를 바인딩하는 record를 모아둔 것. 내부적으로 크게 3개의 파트로 나누어진다
  - Declarative Environment Records: ECMAScript의 함수선언, 블록구문, Try구문의 Catch와 같은 특별한 구문과 연결된다
  - Object Environment Records: ECMAScript의 특정 객체에 바인딩되는 Identifier bindings를 저장한다.
  - Global Environment Records: 함수 내에서 top-level 선언 혹은 전역 레벨 선언되었다고 평가된 항목들을 저장한다.
- 바인딩: identifier와 value를 서로 연결해 두는 것
- Execution Contexts: 함수가 실행될 때 마다 설정되는 컨텍스트. 관련 코드의 실행 상황을 추적하는데 필요한 모든 내용이 들어간다.
- 코드의 평가: 구문과 표현식을 평가할 규칙을 작성해 두고, 그 규칙에 맞게 작성되었는지 확인하고 실행하는 작업
- top-level declaration: Global object의 property/method
- attribute: object를 구성하는 property의 특성을 나타내는 value를 나타낸다. ECMAScript에서는 method 또한 하나의 value이므로, javascript에서는 method 또한 attribute에 포함된다.
- method: object를 구성하는 property 중 function과 같이 특정 작업을 수행할 수 있는 attribute를 말한다.
- function: 기능을 수행하도록 구현된 문장의 집합
