---
layout: post
title:  NodeJS의 메모리 영역
date:  2023-09-05 22:00:00 +0900
categories: cs
tags: cs nodejs article
---

## NodeJS의 메모리 영역을 알아두어야 할까?

NodeJS로 개발을 할 때에는 이벤트 루프와 함께 메모리 영역과 작업의 처리 과정에 대해서는 알아두면 상당히 좋다.  
NodeJS에서는 이벤트루프와 메모리의 Stack/Heap을 사용해서 함수를 처리하고 데이터를 쌓는다. 이 과정중에 원치 않는 순서대로 작업이 수행되거나, 원하는 함수가 전혀 실행되지 않기도 한다. 의도대로 프로그램이 실행되지 않아 곤혹스러운 때를 회피할 수 있게 된다.  

### NodeJS에서의 작업을 처리할 때 사용하는 메모리 영역

NodeJS에서는 작업을 처리하기 위해 2개의 공간을 사용한다.

- Queue: 요청된 작업이 쌓이는 공간
- Stack: Queue에서 실제 처리될 작업들의 Callback이 등록되는 공간

NodeJS에서 js파일을 읽으면, 라인 단위로 작업을 해체해서 작업 queue에 등록한다. NodeJS는 이 작업queue에 등록된 작업을 하나씩 빼내어 처리하게 된다.  
너무나도 억지스럽지만, `a`, `b`, `c` 순서대로 콘솔에 찍기 위한 아래 예제를 참조해보자. 하지만 실행 결과는 `setTimeout`에 의해 원하는 내용과 다르게 나온다.

```js
// 소스코드
function print(text) { console.log(text); }

print('a')
setTimeout(() => print('b'), 0);
print('c')

// 실행 결과
'a'
'c'
undefined // 안나오는 경우도 있음
'b'
```

`b` 가 뒤늦게 콘솔에 찍히게 되는데, 이유는 NodeJS의 event loop에서 아래와 같은 순서대로 작업을 처리하기 때문이다.

- step #1
  - print, setTimeout, print를 queue에 등록
- step #2
  - pool 단계에서 print('a') 처리
  - pool 단계에서 setTimeout( ... ) 의 내용을 loop에 등록하여 event loop의 timer 단계까지 대기한다. setTimeout의 결과인 undefined가 콘솔에 찍힐 때가 있다.
  - pool 단계에서 print('c') 처리
- step #3
  - timer 단계에서 step #2에서 등록된 setTimeout 함수를 처리. delay가 0이니 해당 함수의 콜백 print('b')를 queue에 등록
  - poll 단계에서 queue에 등록된 print('b')를 처리

만약 함수 내에서 다른 함수를 계속 호출하는 아래와 같은 모양이라면 stack은 이렇게 생기게 되며, heap 영역에 저장되는 데이터는 없다.

```js
// 코드
function print(text) { console.log(text); }
function calla() { print('call - a'); }
function callb() { calla(); }


setTimeout(() => callb() , 0);

/*
// 메모리 영역
--- stack ---
|   print   |
|   calla   |
|   callb   |
| anonymous |
-------------
--- heap  ---
--- young --- | ---  old  ---
|             |             |
-----------------------------
*/
```

혹시 이 과정이 어떻게 처리되는지 동적인 그림으로 보고 싶다면 이 조각글 하단 `Reference`의 [Javascript Visualizer 9000](https://www.jsv9000.app/) 에서 한번 실행해보는것도 좋다.

## NodeJS에서의 데이터를 저장할 때 사용하는 메모리 영역

NodeJS에서는 데이터 저장을 위해 Stack과 Heap을 사용하며, Heap은 GC를 위해 여러 개의 영역으로 나뉘어 관리된다.

- Stack
  - 로컬 변수(argument, return value) 및 객체에 대한 포인터 정보를 저장한다.  여기서 말하는 포인터는 단순 객체에 대한 포인터와 Global scope(= Global frame)에 대한 포인터를 모두 포함한다.
  - OS에서 관리되고 처리된다.
- Heap
  - Stack의 포인터에 의해 참조되는 객체 정보(object type, function)를 저장한다.  primitive type(int, string과 같은)은 포인터에 의해 참조되는 객체 정보가 아니기 때문에 Heap이 아닌 Stack에 저장된다.
  - V8 엔진이 관리하고 처리하며, 이를 위해 2가지 GC를 사용한다.
  - Young Generation과 Old Generation으로 나뉜다.
     Young Generation: nursey/intermediate 로 나뉘며, 새 객체는 nursey에 생성되어 저장되고, intermediate는 Scavenge 때 활용된다.
    - old generation: Young Generation에서 GC가 2번 진행될 동안 객체의 포인터가 살아있는 경우, 이 객체들이 이동되는 장소이다.

## GC (Garbage Collection)

NodeJS에서 GC의 경우에는 개발자가 이를 손댈 수 없다. V8 엔진에서 모든 GC작업을 알아서 처리하기 때문이다. 그럼에도 우리는 언제 GC가 실행되고 이로 인하여 우리의 어플리케이션에 어떠한 영향을 주는지 기억해두는게 좋다.  
GC에는 2가지가 있는데, Young Generation만을 처리하는 Scavenge와 전체 Heap 영역을 처리하는 Major GC로 나뉜다.

### Scavenge(Minor GC) 의 동작 과정

![V8 Memory](https://v8.dev/_img/trash-talk/02.svg#center)
[V8 Scavenge](https://v8.dev/_img/trash-talk)
{:.image-caption}

- 새 객체들이 생성되면 nursey에 할당된다.
- GC가 시작될 때, 모든 객체의 포인터를 확인하고, 포인터가 살아있는 모든 객체들을 마킹한다.
- 마킹된 모든 객체들을 immediate로 이동시킨다.
  - 이 때 sweeping이 되면서 모든 객체 정보가 가지런히 저장된다. 파편화 제거가 목적이다.
  - 옮겨지는 모든 객체들에는 한번 GC가 되었음을 마킹해둔다.
- 참조하고 있는 pointer정보가 유효하지 않으니 모든 pointer의 주소를 업데이트한다.
- immediate의 모든 내용을 nursey로 옮긴다.
- 두번의 GC(GC가 한번 되었음에도 불구하고 살아남은 모든 객체의 데이터)는 old-generation으로 이동되며, 이후 Major GC가 이루어질 때 까지 heap에 계속 존재하게 된다.
- 만약 scavenge가 실행되는 과정중에 새로운 객체가 생성된다면 이 객체는 nursey에 저장된다.

### Major GC

전체 heap 영역을 처리한다.  
heap에 저장되는 모든 데이터는 tree구조로 만들어져있다. heap에 데이터가 저장될 때 루트에서 뻗어나간 어딘가의 리프로서 저장된다.
GC가 시작될 때 루트를 시작으로 DFS 방식으로 모든 객체들을 탐색하며 포인터가 있는 모든 객체들에 마킹을 하게 된다.
그 후에 모든 메모리영역을 순회하며 마킹되지 않은 모든 객체들에 대해 메모리를 해제하고 필요에 따라 파편화 방지를 위해 메모리영역을 압축한다.

## Reference

- [V8: trash-talk](https://v8.dev/blog/trash-talk)
- [V8: Orinoco: Young Generation garbage collection](https://v8.dev/blog/orinoco-parallel-scavenger)
- [Javascript Visualizer 9000](https://www.jsv9000.app/)
