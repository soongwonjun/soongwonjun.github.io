---
layout: post
title:  "Kotlin 테스트 코드에서 capture를 사용해보자"
date:  2024-02-08 01:00:00 +0900
categories: kotlin
tags: kotlin test
---

## Kotlin + Junit5 + Mockk

요즘에 Kotlin을 사용하면서, Junit5와 Mockk를 사용하여 테스트 코드를 작성하고 있다. 대부분은 쉽게 처리가 되었는데, 함수 처리에서 시간을 상당히 소모했다.  이를 해결하면서 `argumentCapture` 기능을 새로 알게 되었기에 잠깐 기록해본다.

## argumentCapture란?

argument에 어떤 값이 들어가는지 확인하는 단계이며, 이 argument에 함수가 들어가는 부분도 캡쳐할 수 있다.  
테스트코드를 작성하면서 난감했던 함수 체크를 이 argumentCapture를 사용해 검증할 수 있게 된다.
이런 부분에 대해서는 역시 코드가 설명하기 더 쉬우니, 간단한 예제를 만들어 보자.

```kotlin
class TestClass {
    fun testFunc(value: Int, calculator: (v1: Int) -> Int) =
        calculator(value)
}
```

간단한 테스트 함수를 작성해 보았다. value와 calculator를 인자로 받아서, calculator를 통해 value를 계산하는 함수이다.  
이제 이 함수에 대한 테스트 코드를 넣어본다.  

```kotlin
class CaptureTest {
    @Test
    fun `capture test`() {
        // 캡쳐를 하기 위한 대상을 모킹한다.
        // 원래 테스트코드에서는 모킹한 대상을 직접 호출하는건 의미가 없지만, 여기서는 설명을 위해 중간 단계를 모두 건너뛰어 본다.
        val t = mockk<TestClass>()

        // calculator에 들어가는 함수를 확인하기 위해서 slot을 만들고 capture를 사용하여 argument를 확인한다.
        val slot = slot<Int.() -> Int>()
        every { t.testFunc(any(), capture(slot)) } returns 10

        // 여기서 설정되는 lambda 함수가 캡쳐된다.
        t.testFunc(1) { it + 1 }

        // 함수가 캡쳐됬는지 확인하고, 실제 캡쳐된 함수가 어떻게 동작하는지 확인한다.
        // 여기서는 넘겨받은 인자에 +1을 해주는 함수이므로, 캡쳐된 함수에 들어가는 숫자가 1이 증가되는지 확인한다.
        assertTrue(slot.isCaptured)
        assertEquals(2, slot.captured(1))
        assertEquals(11, slot.captured(10))
    }
}
```

mockito의 홈페이지에서는 value 값, vararg 값을 체크하는 등 많은 방법으로 argumentCapture를 사용하는 방법을 예제로 만들어 두었다.
- [ArgumentCapture](https://www.javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/ArgumentCaptor.html)
