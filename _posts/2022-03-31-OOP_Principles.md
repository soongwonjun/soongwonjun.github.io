---
layout: post
title:  "OOP의 5 Principles"
date:  2022-03-31 01:00:00 +0900
categories: cs
tags: oop cs
description: 5 Principles(SOLID)에 대해 알아보자
---
## 요약

SOLID 원칙은 OOP에서 설계적인 면, 코드적인 면에서 좀 더 명확한 객체를 만들고 객체를 적재적소에 잘 사용하여 더 나은 형태의 캡슐화, 확장성, 다형성을 확보하고, 개발 과정에 있어 좀 더 쉬운 코드 분석, 더 나은 유지보수성을 추구하는데 그 목표가 있다.  
이 개념은 반드시 필요한 내용만을 담아 잘 만들어진 추상화 객체를 사용하고, 각 레이어별로 명확하게 분리하여 가능한 한 느슨한 커플링 구조를 유지하고, 객체 단위의 확장성과 다양성을 추구하여 보다 견고한 코드를 작성하도록 도와준다.

## 5 Principles (SOLID)

### Single Responsibility Principle (SRP)

`1개의 객체는 하나의 책임만을 가져야 한다.`  
module, class, function 등 하나의 프로그램 객체은 단 하나의 functionality를 가져야 하며, 만들어진 객체는 수정될 때 단 하나의 이유만으로 수정되어야 함을 의미한다. 이는 하나의 객체는 하나의 작업만을 수행하며, 하나의 목적만을 위해 만들어져야 한다는 의미이다.  
이 원칙이 잘 지켜진다는 의미는 객체가 잘 캡슐화 되었다고 할 수도 있다. 캡슐화가 동일한 속성과 행위(데이터 필드와 이를 조적하는 메소드)를 하나로 묶고, 실제 구현에 대해서는 은닉하는 작업임을 생각한다면 SRP는 캡슐화를 잘 한다고 할 수 있다.  
아래의 예제는 SRP를 위반한 경우이다. Animal 클래스의 howl 메소드에서는 모든 동물들의 울음소리를 표현하려 한다. 이렇게 하나의 클래스가 여러 목적을 가지게 되면 확인되는 모든 동물과에 대해 일일이 대응해야 하며, 메소드가 추가되거나 수정되게 되면 모든 경우의 수를 체크하고 또 보완해야 한다.

```java
class Animal {
  public void howl(AnimalFamily animalFamily) {
    switch (animalFamily) {
      case DOG:
        System.out.println('Bow-wow');
        break;
      case CAT:
        System.out.println('Meow');
        break;
      default:
        throw new NotDefinedAnimalFamilyException(animalFamily.toString() + ' is not defined');
    }
  }
}
```

이를 아래처럼 개선해보자.  Animals 클래스를 상속받은 Dog/Cat 클래스는 각각 개과, 고양이과에 대한 정의만을 가지고 있으며, 개과에 대한 수정을 반영할 때에는 개과를 대표하는 Dog 클래스만을 수정하게 되며, Cat 클래스는 이 수정에 영향을 받지 않는다.  
이렇게 만들어진 코드는 각 클래스의 목표에 맞는 테스트와 개선 작업만을 진행하면 되므로, 기존보다 코드가 탄탄해지고 유지보수가 쉬워진다. 서로에 대한 의존성도 매우 낮아지며, 객체가 단순해지고, 필요한 메소드들을 슈퍼클래스에서 지정하니 상속 클래스를 만들기도 쉬워진다.

```java
abstract class Animal {
  abstract void howl();
}
class Dog {
  public void howl() {
    System.out.println('Bow-wow');
  }
}
class Cat {
  public void howl() {
    System.out.println('Meow');
  }
}
```

### Open Close Principle (OCP)

`확장에 대해서는 열려 있어야 하며, 수정에 대해서는 닫혀 있어야 한다.`  
module, class, function 등 객체를 확장하여 새로이 구현할 때 해당 객체의 동작을 수정해서는 안된다. 데이터 필드를 추가하거나, 새 기능을 추가하는 것은 환영하지만, 클래스를 확장할 때 항상 기존 코드가 수정되면 안된다. 여기서 기존 코드는 상속받고 있는 슈퍼 클래스 혹은 이를 사용하는 다른 클래스나 메소드도 포함된다. 이는 객체의 안정성을 높이고 보다 용이한 유지보수성을 제공한다.

아래 예제에서는 Animal 클래스를 사용한 Play 클래스를 구현하였다. 억지스러운 예제이지만, 현재로서는 Dog/Cat 클래스에 따라 Play 클래스가 계속 영향을 받는다.

```java
class Dog {}
class Cat {}
class Play {
  public void play(Dog dog) {
    System.out.println('Dog is playing with a disc');
  }
  public void play(Cat cat) {
    System.out.println('Cat is playing with a fishing rod');
  }
}
```

이를 아래처럼 개선해보자. Dog와 Cat을 위해 Animal이라는 추상화된 interface를 정의하였고, Dog/Cat 클래스가 이를 구현하도록 하였다. 그리고 Dog/Cat 클래스는 항상 자신만의 play 함수를 구현하게 되었다. 또한 Play 클래스는 Animal을 구현한 객체의 play를 호출하는 정도로 수정하였다.  
이제 Play 클래스는 더 이상 Dog/Cat 클래스로부터 영향을 받지 않으며, 오버로딩으로 여러번 만들어진 play 메소드도 하나로 줄었다. 또한 추후에 Hamster나 기타 새로운 클래스를 추가할 때, 동일하게 Animal 인터페이스를 구현하여 사용한다면 Play 클래스가 영향을 받지는 않는다.  
현재는 Strategy Pattern에 해당되는 코드로 만들어졌지만, Factory/Observer pattern에도 이 정의가 녹아 있다.

```java
interface Animal {
  public void playWithToys();
}
class Dog implements Animal {
  public void playWithToys() {
    System.out.println('Dog is playing with a disc');
  }
}
class Cat implements Animal {
  public void playWithToys() {
    System.out.println('Cat is playing with a fishing rod');
  }
}
class Play {
  public void play(Animal animal) {
    animal.play();
  }
}
```

### Liskov Substitution Principle

`sub-type은 side-effect 없이 base-type으로 실행될 수 있어야 한다.`  
sub-type은 항상 base-type의 행위를 할 수 있어야 한다. 때문에 base-type으로 동작하는 모든 코드들은 sub-type이 들어왔을 때에도 문제없이 실행될 수 있어야 한다. 이 조건을 만족하기 위해서는 sub-type이 구현한 모든 메소드의 내용은 base-type에서 정의된 스펙(메소드의 argument, 리턴 타입)과 동일해야 한다.
이는 실제 구현된 sub-type을 사용하지 않고, base-type을 사용하여 객체를 처리하기 위함이다.

우리 세상에는 날 수 있는 동물이 있고, 날 수 없는 동물이 있다. 그리고 우리는 날 수 있는 동물들로 하늘을 채워보려 한다.

```java
interface Animal {
  public void fly();
}
class Sparrow extends Animal {
  public void fly() {
    System.out.println('fly the sky');
  }
}
class Cat extends Animal {
  public void fly() {
    throw new Exception('Can not fly');
  }
}
class FillSky {
  public void fly(Animal[] animals) {
    for (animal : animals) {
      animal.fly();
    }
  }
}
```

위 예제에서 FillSky.fly에 Cat 객체가 포함된 상태로 전달된 경우에는 Exception이 발생한다. 고양이는 하늘을 날 수 없기에 Exception이 발생하여 기대한 결과가 실행되지 못한다. 때문에 이에 대해 좀 더 세분화된 상속 구조가 나와야만 한다.

```java
interface Animal { ... }
interface Bird extends Animal { 
  public void fly();
}
interface Mammal extends Animal {}
class Sparrow implements Bird { ... }
class Cat implements Mammal { ... }
class FillSky {
  public void fly(Bird bird) {
    bird.fly();
  }
}
```

이제 더 이상 날지 못하는 동물인 고양이는 FillSky에 전달되지 못한다. Cat은 Mammal를 구현하였기 때문이고, FillSky.fly는 Bird만을 받을 수 있게 되었기 때문이다.

LSP는 사실 코드 자체의 영역보다 디자인의 영역에 좀 더 치중되어 있다. 객체를 디자인할 때 객체들의 특성과 객체에 요구되는 요구사항을 정확하게 녹여내지 않으면 클래스 구조가 맞지 않게 된다. 그리고 이를 사용하는 쪽에서 base-type을 사용하는 경우 문제가 발생하기 쉽다. 일부 경우를 마주치고 예외 회피를 위해 `instanceof`와 같은 로직을 추가한다면 유지보수성은 더더욱 어려워진다. 때문에 얼마나 정확하게 객체를 디자인하고 상속구조를 구축하는가는 LSP에서 매우 중요한 요소로 다가온다.
잘 디자인된 객체 구조는 재사용성이 용이하며, 유지보수성도 좋다.
LSP의 강점을 느낄 수 있는 또다른 분야는 테스트이다. LSP가 지켜진 객체는 모두 동일한 결과를 얻을 수 있기에 테스트를 진행할 때 base-type으로 수많은 하위 객체를 테스트할 수 있다.

### Interface Segregation Principle

`클라이언트와 관련 없는 인터페이스를 구현하지 말아야 한다.`  
하나의 객체 명세는 관련 없는 객체 명세를 가지지 말아야 한다. 인터페이스가 모든 케이스에 대한 내용을 정의할 필요는 없다.
이번에는 조류 중에서 참새와 펭귄의 예를 들어보자. 참새는 날 수 있지만, 펭귄은 날 수 없다. 또한 참새는 수영을 할 수 없지만 펭귄은 수영을 할 수 있다. 그래서 새를 정의하는 Bird 인터페이스에 fly와 swim 메소드를 만들었다.
하지만 Bird 인터페이스에 fly, swim 메소드를 둘 다 정의한다면 이는 ISP를 위반한 셈이 된다. 참새는 swim이라는 메소드를 필요로 하지 않으며, 펭귄은 fly 메소드를 필요로 하지 않는다.  
이로서 불필요한 구현과 불필요한 exception, try/catch가 더 이상 코드에 등장하지 않는다.

```java
interface Bird extends Animal {
  public void fly();
  public void swim();
}
```

따라서 아래처럼 새로이 인터페이스를 만들어 보도록 한다. 참새를 위한 인터페이스는 FlyingBird로, 펭귄을 위한 인터페이스는 SwimmingBird를 만들었다.
인터페이스를 각각 특정 역할로서 구분하고, 그 역할에 따라 두 개의 인터페이스를 만들게 되었다. 구현체는 필요로 하는 인터페이스를 상속받아 구현한다.
물론 필요에 따라 두 인터페이스 모두를 구현해야 할 때도 있다. 오리를 생각해보자. 오리는 날 수 있으며, 수영도 할 수 있다. 그렇기에 오리는 FlyingBird와 SwimmingBird 두 인터페이스 모두를 구현해야 한다.

```java
interface Bird extends Animal { ... }
interface FlyingBird extends Bird {
  public void fly();
}
interface SwimmingBird extends Bird {
  public void swim();
}
class Sparrow implements FlyingBird { ... }
class Penguin implements SwimmingBird { ... }
class Duck implements FlyingBird, SwimmingBird { ... }
````

이제 더 이상 Penguin 클래스는 무의미한 fly 메소드를 구현할 필요 없으며, 이에 대해서 유지보수할 필요 또한 없어졌다. 반대로 Sparrow 클래스 또한 무의미한 swim 메소드와는 결별할 수 있게 되었다. Duck 클래스는 fly/swim 두 메소드 모두 사용할 수 있다.  
우리는 이제 훨씬 구조화되고, 분명하게 필요한 부분만 구현된 객체를 볼 수 있으며, 이에 따라 객체가 어떠한 역할을 할지 명확하게 알 수 있게 되었다.
ISP는 SRP와 비슷한 목적을 가진다. 객체의 목적에 집중하며, 객체가 필요로하는 최소한의 내용을 포함하는 응집력 높은 객체를 만드는 것이 그것이다. 즉 잘 구현된 캡슐화를 목적으로 한다. 차이점이라면 SRP는 클래스에 초점이 맞추어져 있으며, ISP는 인터페이스에 초점이 맞추어져 있다는 정도이다.

### Dependency Inversion Principle

`high-level module은 low-level module에 의존적이어서는 안된다.`  
먼저 여기서 말하는 Dependency Inversion은 Spring과 같은 곳에서 말하는 Dependency Injection 과는 다르다. Dependency Injection은 클래스 간의 의존성을 낮추어 실제 코드는 선언된 상태로 작성되며, 실행 단계에서 의존 관계게 명확하게 결졍되어 동작할 수 있도록 하는 것에 그 목표가 있다.  
high-level module은 class/interface를 사용하여 무언가 수행하거나, 다수의 low-level module을 오케스트레이션해주는 module을 말하기도 하며, 특정 객체에 대해 추상화된 객체 또는 인터페이스가 되기도 한다. low-level module은 아주 작은 객체로서, 매우 작은 작업을 수행하거나 혹은 객체에 대한 정의만 담고 있는 단순한 객체를 의미하기도 하며, 추상화된 특정 객체를 실제 구현/확장한 객체를 말하기도 한다.  
high-level module이 low-level module에 대해 의존적이어서는 안되는 이유는 실제 객체를 구현하거나 사용하는 세부 내용은 low-level module의 구현체에 따라 달라지게 되기 때문이다. 그렇기 때문에 low-level module에 의존하는 high-level module을 만들 경우, low-level module의 수정에 민감하게 반응해야 하며, 이 low-level module이 늘어감에 따라 high-level module은 더욱 큰 타격을 받게 된다.  

반려동물 분양을 위한 루틴을 만들고 싶다. 여기에서 고양이 혹은 강아지만을 분양하는 AnimalHouse와 Adopt를 DIP에 위반되도록 구현해보았다. 아래에서 Adopt는 AnimalHouse를 확장한 DogHouse, CatHouse, DogAndCatHouse 3개 객체에 대해 매우 강한 의존성을 가지고 있다.
Adopt 객체는 high-level module이다. 또한 이 Adopt 객체는 AnimalHouse를 상속받은 3개 객체들에 대해 강한 커플링을 맺고 있다.
또한 반려동물 입양을 위해 만든 adoptDog 및 adoptCat 메소드 덕분에 Adopt 클래스는 Dog와 Cat에 대해서도 강한 커플링을 맺고 있다.

```java
class AnimalHouse {
  private static List<Animal> animals;
  constructor() {
    this.animals = new List<Animal>();
  }
  public List<Animal> getAnimalList() { ... }
}

class DogHouse extends AnimalHouse { ... }
class CatHouse extends AnimalHouse { ... }
class DogAndCatHouse extends AnimalHouse { ... }

class Adopt {
  List<AnimalHouse> dogHouses = [dogHouse, dogAndCatHouse];
  List<AnimalHouse> catHouses = [catHouse, dogAndCatHouse];
  public Dog adoptDog(Cons cons) {
    for (dogHouse : this.dogHouses) { ... }
  }
  public Cat adoptCat(Cons cons) {
    for (catHouse : this.catHouses) { ... }
  }
}
```

이에 대해 AnimalHouse에서 알아서 고객에게 분양할 수 있는 반려동물을 찾도록 하고, Adopt는 해당 객체에 대해 추상화된 Animal을 사용하도록 한다. 또한 입양을 원하는 반려동물을 찾을 때에도 Adopt 객체가 처리하지 않고, 각 객체들이 알아서 처리할 수 있도록 한다. 상세한 내용은 각 클래스들이 구현할 것이므로, Adopt 클래스는 거기까지 신경쓰지 않도록 한다.
이로서 Adopt라는 high-level module은 AnimalHouse/Animal이라는 추상 레벨에 대해 의존성을 가지게 되고 이를 구체화환 객체와는 더 이상 의존성을 가지지 않는다.

```java
class AnimalHouse {
  private List<Animal> animals;
  public Animal adopt(Cons cons) { ... }
  public boolean isAdoptable(Cons cons) { ... }
}

class DogHouse extends AnimalHouse { ... }
class CatHouse extends AnimalHouse { ... }
class DogAndCatHouse extends AnimalHouse { ... }

class Adopt {
  List<AnimalHouse> animalHouses;
  public Animal adoptAnimal(Cons cons) {
    Animal animal = null;
    for (animalHouse : this.animalHouses) {
      if (animalHouse.isAdoptable(cons)) {
        animal = animalHouse.adopt(cons);
        break;
      }
    }
    return animal;
  }

  public void registerAnimalHouse(AnimalHouse animalHouse) {
    this.animalHouse.push(animalHouse);
  }
}
```

위의 개선 코드를 보면 OCP, LSP와 매우 비슷하게 보인다. 하지만 정의로 보면 정말 다르다.  
OCP는 단일 모듈을 디자인할 때의 확장성을 이야기하며, DIP는 다수의 모듈을 오케스트레이션할 때 의존성을 느슨하게 만드는데 그 목적이 있다. 예를 들어 Animal객체를 확장할 때에는 OCP 원칙을 따르지만, 이를 사용하는 Adopt 객체나 AnimalHouse를 수정할 때에는 DIP 원칙을 따르게 된다. OCP는 단일 모듈을 디자인하는 영역에 있으며, DIP는 다수의 모듈을 오케스트레이션 하는 영역에 있기 때문이다.  
LSP은 클래스/인터페이스의 상속/구현의 디자인에 대해 이야기하고 있으며, DIP는 그러한 객체를 사용하게 되는 레이어 레벨에 대해 이야기한다.

DIP를 적용함으로서 우리는 객체 간의 관계를 매우 느슨하게 만들 수 있다. 그렇게되면 확장성이 용이하고, 코드 유지보수성은 단연 더 높아진다.
DIP가 가장 빛을 발하는 분야를 꼽아보자면 테스트일 것이다. 의존성이 약하니, mock 객체를 만들어서 Adopt 클래스를 테스트하기가 매우 수월해진다.
