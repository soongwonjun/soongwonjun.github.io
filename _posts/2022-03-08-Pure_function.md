---
layout: post
title:  "Pure Function"
date:  2022-03-08 12:00:00 +0900
categories: cs
tags: fp cs
description: Pure Function에 대한 얕은 지식
summary: node.js, javascript에 대한 코드를 고민하다가 Pure Function에 대해 공부한 내용을 요약, 정리해 보았다.
---

## Function, 함수에 대해서

### Function?

- arguments를 input으로 받아 호출되며, return value를 돌려주는 일련의 프로세스를 의미한다.
- procedure, i/o process 와 같은 작업들 또한 함수에 포함될 수 있다.

### Pure Function?

- 항상 같은 input이 들어가면, 항상 같은 output이 나온다.
- 이 작업이 이루어질 때, side-effect가 없어야 한다.
  - side-effect란 output이 변화할 수 있는 모든 경우의 수를 의미한다.  
    함수가 실행되는 동안의 변수의 변화, Exception의 발생, 예정되지 않은 I/O의 발생 등이 포함된다.
- input과 output이 항상 일치하므로, arguments와 return value를 사용하여 value map을 만들 수 있다.

### Pure Function에 대한 결론은?

- Functional Programming에서 Pure Function은 상당히 중요한 의미를 가진다.
이는 Pure Function은 자신만의 독립적인 구조를 만들고(Isolate) 실행되기 때문인데, 이 의미는 함수의 내부 상태가 외부의 영향에서 완전히 독립됨을 말한다. 이는 공유가 가능하기 때문에 발생하는 다양한 문제로부터 해방될 수 있음을 의미한다.
Pure Function은 재사용성이 뛰어난 함수를 만들 수 있도록 도와준다.
Pure Function은 확실한 동시성 제어(Concurrency)를 제공하며, 테스트 코드 작성에 있어 중요한 의미를 가진다.
코드의 분석에 있어서도 좋은 결과를 가져온다. 모든 함수가 Pure Function 일 경우, 사실상 함수명 혹은 함수에 같이 작성되어 있는 주석 정도만으로도 충분히 함수를 이해할 수 있을 것이다.

## Pure Function과 Inpure Function

### Pure Function의 경우

- 아래 함수를 만들었다고 하였을 때 이 함수는 pure function이 될 수 있다.
  - 항상 arguments에 따라 return value가 바뀌며, 같은 argument의 경우 항상 같은 return value를 가진다.  
  중간에 다른 side-effect가 발생할 일은 없다.  
  return value를 만듦에 있어, 다른 로직이 끼여들 여지가 없다.

const add = (a, b) => {
    return a + b;
}

```

- 또다른 대표적인 예로 자주 사용하는 아래의 function 들 또한 pure function이라고 할 수 있다.
```typescript
// console.log
console.log('print this');
// Math functions
const max = Math.max(1, 2, 3);
```

### Inpure Function의 경우

- 아래의 함수를 만들었다고 하였을 때 이 함수는 pure function이 될 수 없다.
  - add 함수 내에서 참조하는 a, b 변수는 global 변수로 언제든지 변경될 수 있다.  
  arguments에 영향 없이, add함수는 항상 다른 return value를 가질 수 있다.  
  심지어 a, b는 add 함수가 호출될 때 까지 초기화되지 않았을 수 있다.

```javascript

let a, b;
const add = () => {

    return a + b;
}

```

- 대표적인 예로 자주 사용하는 아래의 function 들은 pure function이 될 수 없다.

```javascript
// Date 객체의 now 함수는 항상 현재 시각을 알려준다. argument는 없지만 항상 결과값이 바뀐다.
const now = Date.now();
// Random 함수는 입력값에 따라 항상 결과값이 바뀐다.
Math.random();
```

## 상태 공유와 Pure Function

- 상태 공유의 경우에 속할 수 있는 대상은 변수, 객체, 메모리 영역 등 다양한 부분이 있을 수 있다.
OOP에서는 클래스에서 사용중인 object가 또다른 object에서 자신의 property가 수정될 수 있음을 허용한다.
JS에서 primitive type이 아닌 모든 변수는 reference로서 작동한다.
또한 object는 scope단위로 정의되는데, 이 때 scope는 global scope 가 될 수도, closure scope가 될 수도 있다.
- 이 때 고려해야 하는 가장 큰 문제는 side-effect와 프로그램 수정에 따른 추가 비용이다.
  - function의 효과를 이해하기 위해서 관련된 모든 function의 정보를 알아야만 side-effect를 피할 수 있다는 점이다.
  - 프로그램이 작동중일 때, 어디선가 변경된 data가 몇몇 function에 지속적으로 영향을 준다면, 전체적인 프로그램에 문제를 야기할 수 있다.
- 때문에 가능한 한 이러한 상태 공유에 속하는 내용이 적거나 없을 수록 프로그램은 견고해진다.
- 아래의 예제는 name과 items를 받아서 storage를 만드는 예제이다. 여기에서 inpure function인 makeStorage function을 pure function으로 만들고자 한다.

```typescript
const makeStorage = (name: string, items: string[]) => {
    const storage = {
        name: name,
        items: items
    };
    return storage;
}
```

- 위의 코드에서는 makeStorage 안에 들어가는 items 변수는 array타입이기 때문에 storage 객체에서 reference를 참조하게 된다.
또한 arguments로 들어온 items array는 객체는 다른 어디선가 변화될 수 있다. 따라서 현재로서는 makeStorage의 return value 객체는 불변성을 가지지 못한다.
이에 storage에 들어가는 items에 변경을 가하여, makeStorage의 return value에 불변성을 부여해보도록 한다.

```typescript
const makeStorage = (name: string, items: string[]) => {
    const storage = {
        name: name,
        items: _.cloneDeep(items)
    };
    return storage;
}
```

- arguments로 넘어온 items를 lodash의 cloneDeep을 사용하여 전혀 다른 object로 만든 후, 해당 객체를 storage에 넣었다.
이렇게 되면 arguments의 items와 storage의 items는 별개의 reference를 가지게 되므로, storage의 items는 불변성을 가지게 된다.
또한 makeStorage는 항상 같은 input에 같은 output을 생성하므로, 이는 Pure Function이라고 할 수 있다.
