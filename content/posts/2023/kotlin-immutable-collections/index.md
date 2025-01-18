---
title: "Kotlin Collections의 불변성과 Kotlinx Collections Immutable 라이브러리"
summary: "Kotlin 표준 라이브러리의 컬렉션은 사실 읽기 전용이지 불변적인 타입이 아닙니다. Kotlin에서 불변적인 컬렉션을 사용하는 방법에 대해서 알아봅니다."
coverAlt: "Kotlin"
coverCaption: "Cover by Author, Logo from [Jetbrains](https://kotlinlang.org/docs/kotlin-brand-assets.html)"
date: 2023-12-10
categories: ["Kotlin"]
---

Kotlin에는 대표적으로 `List`, `Set`, `Map`과 같은 표준 라이브러리 컬렉션 타입들이 있습니다.
이 타입들은 모두 인터페이스입니다.
예를 들어 `List`는 아래와 같이 만들어져 있습니다.

```kotlin
interface List<out E> : Collection<E> {
    override val size: Int
    override fun isEmpty(): Boolean
    override fun contains(element: @UnsafeVariance E): Boolean
    override fun iterator(): Iterator<E>
    override fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean
    operator fun get(index: Int): E
    fun indexOf(element: @UnsafeVariance E): Int
    fun lastIndexOf(element: @UnsafeVariance E): Int
    fun listIterator(): ListIterator<E>
    fun listIterator(index: Int): ListIterator<E>
    fun subList(fromIndex: Int, toIndex: Int): List<E>
}
```

잘 살펴보면 `List` 인터페이스에는 수정과 관련 된 기능이 정의되어 있지 않습니다.
수정 관련 기능은 `List`를 상속받은 `MutableList`에 정의되어 있죠.
일반적으로 우리는 컬렉션의 변화를 방지하기 위해 위 인터페이스들을 사용합니다.

## Kotlin Standard Collections는 정말로 불변적일까?

그런데 `List`, `Set`, `Map` 타입들은 정말로 불변적일까요?
실제로 위 타입들은 완전히 불변적인 타입이 아닙니다.
대신 읽기 전용(Read Only)이라고 보는 편이 더 정확합니다.
아래 예시를 통해 살펴봅시다.

```kotlin
suspend fun main() = coroutineScope {
    val numStrings = mutableListOf<String>()
    launch {
        printListEverySeconds(numStrings, 1000)
    }
    delay(500)
    for (i in 1..1000) {
        println("add $i to list")
        numStrings.add(i.toString())
        delay(1000)
    }
}

private suspend fun <T> printListEverySeconds(list: List<T>, times: Int) {
    repeat(times) {
        println("List=$list, size=${list.size}")
        delay(1000)
    }
}
```

우선 `numStrings`라는 `MutableList`를 생성합니다.
그리고 매 초마다 리스트를 출력하는 `printListEverySeconds`를 호출하고 `List` 타입으로 `numStrings`를 넘겨줍니다.
500 밀리초 후 매 초마다 `numStrings`에 새 아이템을 추가합니다.
과연 결과는 어떻게 출력될까요?

```
List=[], size=0
add 1 to list
List=[1], size=1
add 2 to list
List=[1, 2], size=2
add 3 to list
List=[1, 2, 3], size=3
add 4 to list
List=[1, 2, 3, 4], size=4
add 5 to list
List=[1, 2, 3, 4, 5], size=5
...
```

결과는 위와 같습니다.
`printListEverySeconds`는 불변적으로 보였던 `List` 타입으로 인스턴스를 넘겨받았지만 실제 구현체는 가변적인 `MutableList`였고,
같은 인스턴스를 참조하고 있는 다른 곳에서 아이템을 변경할 수 있습니다.
하지만 `printListEverySeconds` 함수 안에서는 `List` 타입이기 때문에 리스트를 조작하지 못합니다.

즉, Kotlin의 기본 컬렉션 타입인 `List`, `Set`, `Map` 등은 완전히 불변적인 타입이 아닌, **읽기 전용(Read Only)** 이라는 것을 알 수 있었습니다.

## 불변 컬렉션을 제공하는 Kotlinx Collections Immutable 라이브러리

Kotlin에서는 완벽하게 불변적인 컬렉션을 위해 [kotlinx.collections.immutable](https://github.com/Kotlin/kotlinx.collections.immutable)이라는 라이브러리를 오픈 소스로 개발하고 있습니다.
이 라이브러리는 읽기 전용일 뿐만 아니라 실제로 인스턴스의 값을 바꿀 수 없는 컬렉션을 제공합니다.

kotlinx.collections.immutable 라이브러리에는 크게 2가지 종류의 컬렉션 인터페이스가 있습니다.

- **Immutable**: 완전하게 불변적이게 구현된 컬렉션 타입.
- **Persistent**: Immutable에 값이 수정된 새로운 컬렉션을 반환할 수 있는 기능을 추가로 확장한 컬렉션 타입.

`Immutable`과 `Persistent` 모두 Kotlin 표준 라이브러리 컬렉션처럼 인터페이스 타입으로 구현되어 있습니다.
다만, 이들은 구현체가 불변적이라는 전제가 뒤에 깔려 있습니다.

## Immutable Collections 사용하기

### 새로운 불변 컬렉션 생성하기

```kotlin
val list = persistentListOf(1, 2, 3, 4, 5)
```

Kotlin의 표준 컬렉션과 비슷하게 편의성 함수가 만들어져 있습니다.

{{< alert "circle-info" >}}
특이한 점은 `persistentListOf`나 `persistentSetOf`처럼 `persistent`로 시작하는 함수만 존재한다는 점입니다.
`immutableListOf`, `immutableSetOf`와 같이 `immutable`로 시작하는 함수들은 Deprecate 된 함수입니다.
{{< /alert >}}

### 다른 컬렉션을 불변 컬렉션으로 변환하기

```kotlin
val immutableList = list.toImmutableList()
val persistentList = list.toPersistentList()
```

컬렉션 인터페이스를 상속한 모든 타입은 `toImmutable`과 `toPersistent` 함수를 통해 불변적인 컬렉션으로 변환이 가능합니다.
변환 시 새로운 불변 컬렉션 인스턴스를 생성하는 방식으로 동작해 원본 컬렉션의 가변 여부에 영향을 받지 않습니다.

### PersistentCollection 수정하기

```kotlin
val list1 = persistentListOf(1, 2, 3, 4, 5)
val list2 = list1.add(6)
println("list1=$list1") // list1=[1, 2, 3, 4, 5]
println("list2=$list2") // list2=[1, 2, 3, 4, 5, 6]
```

`Persistent`로 시작하는 컬렉션들은 `add`와 같은 수정 메소드를 가지고 있습니다.
하지만 표준 라이브러리 컬렉션의 Mutable과 다르게 원본 인스턴스를 수정하기 않고 결과가 반영 된 새로운 컬렉션 인스턴스를 반환합니다.
따라서 원본의 불변성을 유지하면서 변경된 값을 활용 할 수도 있습니다.

### 불변 컬렉션을 표준 라이브러리 컬렉션 타입으로 사용하기

kotlinx.collections.immutable 라이브러리의 컬렉션은 모두 표준 라이브러리의 인터페이스를 상속하고 있습니다.
따라서 불변 컬렉션들은 `List`, `Set`과 같은 표준 타입으로 취급할 수 있기 때문에 사용에 큰 문제가 없습니다.

## 정리하며

이번 글에서 Kotlin의 표준 라이브러리 컬렉션 타입들이 실제론 불변한(Immutable) 인스턴스가 아니라 읽기 전용(Read Only)이라는 점, 완전한 불변성을 위해서라면 kotlinx.collections.immutable 라이브러리를 사용할 수 있다는 것을 알았습니다.

하지만 모든 상황에서 표준 라이브러리 컬렉션을 Immutable 라이브러리로 교체해야 하는 것은 아닙니다.
대부분의 상황에서는 읽기 전용이라는 제약만으로도 꽤 많은 사이드 이펙트를 방지 할 수 있습니다.
다만, 동시성 문제나 완벽한 불변성을 요구하는 경우라면 Immutable의 컬렉션을 고려해볼 수 있겠습니다.
또한 Jetpack Compose에서는 인터페이스 타입을 불안정하게 여기지만 Immutable 라이브러리 컬렉션은 안정된 타입으로 여기기 때문에 충분히 도입할 이유가 되기도 합니다.
(Jetpack Compose의 안정성 시스템에 대해서는 [이 글](https://cheonjaeung.com/posts/jetpack-compose-recomposition-and-stability-system)을 참고해 주세요.)

Kotlin 프로젝트에서 완전 불변한 컬렉션이 필요하다면 [kotlinx.collections.immutable](https://github.com/Kotlin/kotlinx.collections.immutable) 사용을 고려해 보세요.
