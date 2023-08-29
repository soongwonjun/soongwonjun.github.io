---
layout: post
title:  "Java Generic"
date:  2022-04-14 23:00:00 +0900
categories: cs
tags: java cs
description: Java Generic에 대해서
---

## 개요

JDK 5.0부터 추가된 기능으로서 컴파일 시에 더 많은 버그를 찾아내어 코드 안정성을 높이고 코드를 더 쉽게 작성할 수 있게 해준다. Generic은 객체 유형을 추상화시킬 수 있으며, 이를 통하여 프로그래머는 실제 의도를 더 잘 표현할 수 있다. 이렇게 표현된 Generic은 컴파일 때 formal type으로 결정된다. 또한 Generic은 type을 객체 내부에서는 제거하고 이를 외부에서 정할 수 있게 하여 유연하고 재사용성 높은 코드를 작성할 수 있도록 해준다.  
이러한 Generic을 가장 쉽게 볼 수 있는 부분은 계층 구조이다. 아래 코드에서는 간단한 list에서 데이터를 다루는 과정이다.

```java
List myList = new ArrayList();
myList.add(new Integer(0));
Integer x = (Integer) myList.iterator().next();
```

위 코드에서는 myList에서 데이터를 획득할 때 casting을 해주어야 한다. 그렇지 않으면 에러가 발생하는데 이유는 next() 메소드가 반환하는 객체가 Object 타입이기 때문이다. 또한 myList에서 데이터를 다룰 때 코딩을 하는 프로그래머는 항상 myList가 Integer를 다루어야 한다고 알고 있어야 한다.

```java
List<Integer> myList = new ArrayList<Integer>();
myList.add(new Integer(0));
Integer x = myList.iterator().next();
```

새로이 고친 코드는 훨씬 깔끔하고 직관적이다. myList를 다룰 때 캐스팅이 사라졌으며, myList가 다루는 데이터가 어떠한 유형인지 직관적으로 볼 수 있게 되었다. 또한 myList에 할당된 List 객체의 내부 자료구조는 Integer를 사용하게 될 것이다. generic 메소드는 호출될 때 실제 argument를 formal 타입의 argument로 전달되어 메소드를 실행한다. 여기서는 Integer type으로 대체되어 iterator()와 next()가 호출될 것이다.  
이로서 코드의 가독성이 올라가며, 컴파일러는 Generic이 선언된 객체의 유형을 항상 보장하여 코드를 더욱 견고하게 만들어 줄 것이다. 캐스팅을 피할 수 있으니 `ClassCastException`도 회피가 가능하다.

## Generic의 선언

Generic을 정의할 때에는 다이아몬드(<>)를 사용하며, 일반적으로 유형을 선언할 수 있는 모든 곳에서 사용할 수 있다.  

```java
public interface List<E> {
  void add(E x);
  Iterator<E> iterator();
}
```

또한 argument/return type 모두 Generic으로 받는 경우도 있다.

```java
public interface MyObject {
  public <E> get(E element);
}
```

### 다수의 Generic 선언

Generic은 한번에 여러 Generic을 사용할 수 있다. 대표적으로 key/value를 사용하는 map이 있다.  

```java
public interface Map<K, V> {
  Set<K> keySet();
  V get(K key);
}
```

### Generic의 subtype과 wildcard

Generic은 상속과 subtype을 제공한다. 때문에 아래의 코드는 문제없이 작동할 수 있다.  

```java
class Shape {}
class Circle extends Shape {};
class Rectangle extends Shape {};

public static void main(String args[]) {
  List<Shape> myList = new ArrayList<Shape>();
  myList.add(new Circle());
  myList.add(new Rectangle());
}
```

단, Java는 다이아몬드 안의 내용에 대해서는 `is a` 관계를 파악하지 못한다. 때문에 아래 코드의 Case#1같은 경우는 처리되지 못하고 에러가 발생한다.  
이를 회피하기 위해 Case#2와 같이 와일드카드를 사용하면 문제없이 실행되는 코드를 작성할 수 있다.

```java
// Case#1
List<Rectangle> myList1 = new ArrayList<Rectangle>();
List<Shape> myList2 = new ArrayList<Shape>();
myList2 = myList1; // 에러 발생, Type mismatch가 발생한다.
                   // List의 Generic에는 type이 들어가 있으며, 이 타입에 대해서는 sub-type을 처리할 수 없다.

// Case#2
List<? extends Shape> myList1 = new ArrayList<Rectangle>();
List<? extends Shape> myList2 = new ArrayList<Shape>();
myList2 = myList1; // Unbound Wildcard를 사용하였기 때문에 다이아몬드 내에서도 sub-type을 처리할 수 있다.
```

또한 아래처럼, 특정 조건을 걸어둘 수도 있다.

```java
// Shape를 상속받는 객체만으로 Generic type을 제한한다.
void drawAll(Collection<? extends Shape> c) { ... }

// Rectangle과 Rectangle의 상위 클래스로 Generic type을 제한한다.
void drawAll(Collection<? super Rectangle> c) { ... }
```

다만 와일드카드는 type-safety하지 않다. 와일드카드는 Unknown type임을 명시하며, 그 어떠한 타입도 될 수 있음을 의미한다. 와일드카드는 컴파일 타임에서의 type-safety를 느슨하게 만들어 컴파일을 통과하게 만들지만 잠재적으로는 런타임 에러의 위험성을 가지고 있기 때문에 사용에는 주의를 하여야 한다.  
와일드카드는 컴파일러가 타입을 추론하지 못하는 경우도 있다. 또한 type parameter는 extends 로만 제한할 수 있으나, 와일드카드는 extends와 super 모두 가능하다.  

```java
List<Shape> list; // list는 Shape에 대한 List Object이다
List<?> list;     // list는 어떤한 타입도 매칭될 수 있는 성격을 가진 List Object이다.
```

아래는 타입을 추론하지 못해 발생하는 와일드카드 에러이다.  
컴파일러는 i 파라메터를 Object로 처리한다. 이 때 List.set(int, E)라고 코딩하면 컴파일러는 이것을 허용하지 않는다. 컴파일러가 잘못된 변수를 할당했다고 판단하기 때문이다. 이 부분은 Java의 type safety와 관련이 있다. 이를 회피하기 위해서는 type을 추론할 수 있는 helper method를 작성하고 이를 호출하도록 하는 방법을 제안하고 있다.

```java
public void foo(List<?> i) {
  i.set(0, i.get(0));
}

// 컴파일 시, 발생하는 에러
error: method set in interface List<E> cannot be applied to given types;
    i.set(0, i.get(0));
     ^

// 위 문제를 회피하기 위해 helper method를 작성, 이를 호출한다.
// helper method에서는 컴파일러가 객체의 type을 추론할 수 있도록 해준다.
public void foo(List<?> i) {
  fooHelper(i);
}
private <T> void fooHelper(List<T> l) {
  l.set(0, l.get(0));
}
```

## 사용의 기준

Generic의 목적은 코드의 가독성을 올리고, 코드의 타입을 강제하여 코드를 더 견고하게 만들어주는 역할을 한다. 그렇다면 우리는 이 Generic을 선언하는 기준을 어떻게 잡아야 할까?  
Generic은 객체의 외부에서 객체 내부의 타입을 결정지을 수 있는 강력한 기능히다. 내부의 자료구조를 외부에서 결정지을 수 있기 때문이다. 때문에 우리는 사용하는 쪽에서 가장 적절한 객체를 사용하여야 한다.  
물론 primitive type을 사용하면 모든 객체를 받을 수 있으니 좋겠지만, 이는 2가지 문제로 사용할 수 없다. Java 컴파일 엔진에서 허용을 하지 않으며, 허용한다 한들 코드의 가독성을 낮추고 버그 포인트를 만드는 과정이 될 뿐이다. 때문에 Generic을 사용하는 쪽에서 어떻게 사용할 것인지를 항상 고민해야 한다.

## Appendix & References

- [Oracle Java Doc, Generic](https://docs.oracle.com/javase/tutorial/extra/generics/index.html)
