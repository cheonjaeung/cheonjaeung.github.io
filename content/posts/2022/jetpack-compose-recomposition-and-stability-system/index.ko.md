---
title: "Jetpack Compose의 Smart Recomposition과 안정성 시스템 이해하기"
summary: "Jetpack Compose로 만든 UI를 최적화 하기 위해 안정성 시스템을 이해해 보자."
coverAlt: "Composition Lifecycle"
coverCaption: Jetpack Compose의 수명 주기 이미지 from [Android Developers](https://developer.android.com/jetpack/compose/lifecycle)
date: 2022-12-17
categories: ["Jetpack Compose"]
---

Jetpack Compose는 특별한 컴파일러 플러그인을 통해서 기존의 Kotlin 코드로 할 수 없었던 동작을 구현했고,
이를 통해서 UI를 동작시킨다.
Compose 컴파일러 플러그인에 의해 특별하게 취급되는 것 중 하나가 컴포저블 함수다.

컴포저블 함수는 `@Composable` 어노테이션이 달린 함수로, Compose 컴파일러 플러그인에 의해서 특별하게 취급된다.
컴파일이 수행될 때 Compose 컴파일러 플러그인에 의해서 컴포저블 함수 내부에 특별한 코드들이 삽입된다.
컴포저블 함수는 이 코드들로 인해서 일반 함수와 다르게 Compose 수명주기에 따라 여러 번 재실행하면서 UI를 그려낸다.

Compose 생명 주기에 따르면 컴포저블 함수는 최초에 컴포지션을 통해 1번 UI를 그린 후, 상태 변화에 따라서 0 ~ n번 리컴포지션(재구성)을 통해서 UI를 업데이트 해 나간다.
하지만 상태 하나가 바뀔 때 마다 수 많은 UI를 재구성 하는 것은 상당히 비효율적이다.
그래서 Compose 컴파일러에는 스마트 리컴포지션(Smart Recomposition) 이라는 기능이 있다.

스마트 리컴포지션은 굳이 다시 그리지 않아도 될 컴포저블 함수는 리컴포지션을 뛰어 넘는 기능이다.
이를 통해서 꼭 갱신이 필요한 UI만 재구성해서 최대한 효율적으로 UI를 그려 나간다.
스마트 리컴포지션을 파악하기 위해서는 우선 2가지 개념을 알아 봐야 한다.

## 안정성 시스템

Compose 에서는 컴파일 시점에 객체들의 "**안정성(Stability)**"을 확인한다.
그리고 어떤 컴포저블 함수에 사용 된 모든 인자가 안정적이라면, 그 컴포저블 함수는 "**생략 가능하다(Skippable)**"라고 본다.
리컴포지션이 발생했을 때, 어떤 컴포저블 함수의 모든 인자가 안정적(Stable)이고 그 값이 전혀 변하지 않았다면 리컴포지션을 생략한다.
이것이 스마트 리컴포지션의 기본 원리다.

여기서 알수 있는 점은 Compose에서는 모든 객체를 "**안정된(Stable)**"상태 또는 "**불안정한(Unstable)**"상태로 볼 수 있다는 점이다.
그리고 컴포저블 내에서 사용할 객체들은 안정된 상태로 만들어 주어야 최적화된 UI를 그릴 수 있다는 점이다.

## 안정의 조건

Compose에서는 크게 3가지 기준을 통해 안정성을 판단한다.[^skipping]

- 두 인스턴스가 같은 상태라면 두 객체의 비교 결과(`equals`)는 항상 같아야 한다.
- 객체가 가진 모든 public 필드는 안정된 상태여야 한다.
- 만약 값이 변경된다면 컴포저블에 알려져야 한다.

추가적으로 Compose 컴파일러에 의해서 기본적으로 안정적이라고 판단하는 것들이 있다.

- 원시 타입
- 문자열
- 함수 (람다)

위 조건 중 "값이 변경되었을 때 컴포저블에 알려져야 한다'는 내용은 조금 추상적으로 느껴질 수 있다.
조금 더 알아보자.

값이 바뀌었을 때 리컴포지션이 일어난다는 뜻은 값 객체의 변경여부를 감시하거나 객체가 Composer에게 자기가 바뀌었다고 알려 주어야 가능하다.
Jetpack Compose에서는 `State`객체가 그 역할을 수행한다.
예를 들어 설명해보자.

```kotlin
@Composable
fun Sample1() {
    var message = ""
    TextField(
        value = message,
        onValueChange = { str -> message = str }
    )
}
```
```kotlin
@Composable
fun Sample2() {
    var message by remember { mutableStateOf("") }
    TextField(
        value = message,
        onValueChange = { str -> message = str }
    )
}
```

위에 2가지 예시가 있다.
둘 모두 `TextField`에 메세지를 입력하는 동작을 하는 UI를 그린다.
다만, `Sample1`은 메세지 문자열을 단순히 `var`로 관리하고, `Sample2`에서는 `MutableState`로 관리한다.
`Sample1`은 아무리 타이핑해도 UI가 업데이트 되지 않는다.
하지만 디버깅 해 보면 `message` 값은 계속 바뀐다.
즉, 값은 바뀌는데 리컴포지션이 발생하지 않았다는 뜻이다.
그런데 `Sample2`는 리컴포지션이 잘 일어난다.

`Sample1`에서는 값 입력 시 `onValueChange`를 통해 변경된 문자열이 넘어왔고, 그것을 `message`에 저장했다.
하지만 `message`는 단순 `String` 객체일 뿐이고 Compose는 값이 바뀐지 모른다.
`Sample2`는 다 똑같지만 `message`가 `MutableState`다.
`State`는 값이 바뀌면 Compose에게 그 사실을 알린다.
따라서 값이 바뀐 영역인 `Sample2` 컴포저블부터 다시 리컴포지션을 수행한다.

정리해 보자면, 가변적인 값을 다룰 때는 어떤 방법이든지 `State`로써 감싸져 있는 편이 좋다.
그래야 업데이트 시 리컴포지션이 일어나고, 나머지 조건까지 따져서 UI 업데이트 여부를 결정할 수 있다.

### 원시 타입과 문자열

위에서 알아보았듯이 `Char`, `Boolean`, `Int`, `Long`, `Double`, `Float` 등등의 원시 타입과 `String`은 안정적이다.

### 인터페이스

인터페이스의 안정성에 대해서 생각해보자.
자주 사용하는 인터페이스를 뽑아 보자면 `List`, `Map` 등이 있을 수 있다.
인터페이스는 안정적일까 불안정적일까?

인터페이스가 안정적이려면 기본적으로 public 필드가 안정되어야 한다.
하지만 인터페이스는 자신을 구현하는 클래스의 public 필드를 강제하지 않는다.
즉, 구현체는 얼마든지 불안정한 필드가 있을 수 있다.
예를 들어 보자면 `List`를 상속하는 `MutableList`는 내부 값을 변경할 수 있는 함수들을 가지고 있다.
컴포저블 영역 밖에서 `MutableList`의 인스턴스를 참조하고 있다가 값을 바꾼다면 Compose 컴파일러는 알 수 없다.
결론적으로 인터페이스는 구현체의 안정성을 보장할 수 없기 때문에 Compose에서는 기본적으로 "**모든 인터페이스는 불안정하다**"고 판단한다.

다만 Compose 1.2 이후부터 예외적으로 [kotlinx.collections.immutable](https://github.com/Kotlin/kotlinx.collections.immutable) 라이브러리의 컬렉션들은 안정적으로 취급한다.
Kotlin 내장 라이브러리에 포함된 기능은 아니지만, Kotlin 측에서 공식적으로 개발하는 오픈 소스 라이브러리다.
기본 내장 컬렉션과 다르게 완벽하게 불변적인 컬렉션을 구현하는 라이브러리이기 때문에 안정적으로 취급하는 듯 하다.

{{< alert "circle-info" >}}
[Kotlin Collections의 불변성과 Kotlinx Collections Immutable 라이브러리](https://woong.io/posts/kotlin-immutable-collections)
포스트에서 kotlinx.collections.immutable 라이브러리에 대해 간단히 알아볼 수 있습니다.
{{< /alert >}}

### 클래스

클래스는 3가지 안정성 조건을 충족 한다면 안정적인 객체로 취급한다.
몇 가지 예시를 통해 알아보자.

```kotlin
data class User1(val username: String, val password: String)
```

위의 `User1` 클래스는 모든 필드가 `val`이고 타입이 안정적인 `String`이다.
데이터 클래스이기 때문에 `equals`도 문제 없고, 값이 바뀌려면 반드시 새 인스턴스를 만들어 넘겨줘야 하기 때문에 Compose가 알기 쉽다.
즉, 위 클래스는 안정적이다.

```kotlin
data class User2(val username: String, var password: String)
```

이번에는 `password`를 `var`로 바꿨다.
모든 public 필드의 타입은 `String`으로 안정적이지만 `password`는 가변적이다.
게다가 외부에서 인스턴스를 참조하고 있다가 언제든지 값을 바꿀 수 있다.
그렇다면 Compose가 그 변경 사항을 알 수 없다.
즉, 위 클래스는 불안정하다.

```kotlin
data class User3(val username: String, val tags: List<String>)
```

위의 `User3`는 `List`라는 인터페이스 타입의 필드가 있다.
위에서 보았듯 인터페이스는 불안정하게 취급하기 때문에 자연스레 `User3`도 불안정하다.
예시로 든 `User3`는 `tags`의 타입을 kotlinx collections immutable의 `ImmutableList`로 바꾼다면 안정될 수 있을 것이다.

```kotlin
class ProfileUiState(
    val name: String,
    val level: Int
) {
    var isLoading: Boolean
        private set

    fun startLoading() { isLoading = true }

    fun finishLoading() { isLoading = false }
}
```

이번에는 데이터 클래스가 아닌 일반 클래스 `ProfileUiState`로 예시를 들어보자.
클래스도 데이터 클래스와 큰 차이가 없다.
이 클래스는 `name`, `level`, `isLoading`으로 총 3가지 공개 필드가 있다.
이 중 `name`, `level`은 `val`이자 원시 타입이라 안정적이다.
하지만 `isLoading`은 `var`이기 때문에 불안정하다.
즉, public 필드 중 일부가 불안정하기 때문에 이 클래스도 불안정하다.

### 함수 (람다)

위에서 알아보았듯, 함수(람다)는 기본적으로 안정된 객체로 취급한다.
그런데 간혹 람다가 안정적이지 않아 보이는 경우가 나타난다.
그 이유를 이해하고 싶다면 컴파일러의 동작을 조금 더 알아보아야 한다.

Compose에서 사용 할 수 있는 람다는 크게 2가지로 나누어 볼 수 있다.
하나는 일반 람다이고 나머지 하나는 컴포저블 람다이다.
컴포저블 람다의 경우 Compose 컴파일러의 도움으로 강력한 최적화를 받게 되고 별다른 수정 없이 최적화된 리컴포지션을 수행할 수 있다.
문제는 일반 람다를 사용할 때다.
일반 람다는 컴파일 시점에 어떻게 처리되는지 조금 더 자세히 파헤쳐 보자.

Kotlin 컴파일이 일어날 때, 람다는 2가지로 나누어 최적화가 진행된다.

```kotlin
val lambda1 = { println("Hello, World!") }
```

Kotlin에서 모든 함수는 사실 `Function`이라는 이름의 객체로 취급한다.
Kotlin이 함수를 일반 객체처럼 다룰 수 있는 이유이기도 하다.
그 이유로 `lambda1` 람다는 컴파일 후에 아래와 같은 형태가 된다.

> 참고: 실제로는 `Function0`, `Function1` 등 몇가지 바리에이션이 있지만 `Function`으로 통일해 설명한다.

```kotlin
class Lambda1 : Function<Unit> {
    override operator fun invoke() {
        println("Hello, World!")
    }
}
```

람다는 `Function` 객체로 감싸졌고 `invoke` 함수를 오버라이드 하는 방식으로 변환된다.
반환 타입은 없기 때문에 `Unit`이다.
위처럼 람다 외부에 아무런 참조도 하지 않는 함수는 언제나 같은 결과를 반환하기 때문에 Kotlin 컴파일러는 이 람다를 싱글톤으로 저장해 최적화한다.

이번엔 아래와 같은 람다를 컴파일 해 보자.

```kotlin
val count: Int = 4
...
val lambda2 = { println("count: $count") }
```

위의 람다는 람다 밖에 있는 `count`라는 변수를 참조한다.
이렇게 람다 밖의 값을 참조하는 것을 **캡쳐링(Capturing)** 이라고 한다.
값을 캡쳐링하는 람다가 컴파일 된다면 아래처럼 바뀔 수 있다.

```kotlin
class Lambda2(private val count: Int) : Function<Unit> {
    override operator fun invoke() {
        println("count: $count")
    }
}
```

외부 변수를 참조하기 때문에 캡쳐링하는 변수를 생성자에서 받도록 변환한다.
참조하는 값의 변화에 따라 계속 변화할 수 있기 때문에 Kotlin 컴파일러는 이 람다를 싱글톤으로 저장하지 않는다.
대신 람다를 사용할 때 `Lambda2` 인스턴스를 생성하고 `invoke`를 실행하는 방식으로 컴파일한다.

여기까지가 일반 Kotlin 컴파일러의 동작이다.
Compose 컴파일러 플러그인은 여기에 더해 추가적인 최적화를 한다.

컴포저블 영역에서 사용하는 람다에서 외부 값을 캡쳐하지 않는 `Lambda1`같은 경우는 이미 싱글톤으로 최적화 되어 있다.
람다가 절대 바뀔 일이 없으므로 불변적이고 Compose 컴파일러는 이를 안정적이라고 판단한다.

안정적인 외부 값을 캡쳐하는 `Lambda2`같은 경우 캡쳐하는 값과 람다를 `remember`로 감싸 최적화한다.
`remember`의 동작에 따라서 캡쳐한 값이 변경되었을 때에만 새 람다 객체를 생성한다.
**문제는 불안정한 값을 캡쳐하는 람다에 있다**.
캡쳐링하는 외부 값이 불안정하다면 `remember` 최적화가 적용되지 않는다.
즉, 불안정한 값을 캡쳐링하는 람다는 매 리컴포지션마다 새 인스턴스를 만드는 방식으로 동작한다.

가능하면 람다에서 불안정한 것을 캡쳐하는 일을 찾아서 줄이는 게 좋다.
예시를 들자면 `ViewModel`이 있다.
람다 내에서 `ViewModel`의 프로퍼티나 함수 등을 사용하는 경우 그 람다는 `ViewModel`을 캡쳐한다.
`ViewModel`은 거의 대부분 불안정한 상태이기 때문에 람다는 `remember`의 도움을 받지 못하는 방향으로 컴파일된다.
이를 해결하려면 람다가 안정적인 값만 캡쳐하도록 수정하거나 캡쳐하는 값을 안정화 시켜주어야 한다.

## Stable Marker

Compose에서는 기본적으로 불안정한 타입들을 안정적으로 사용할 수 있도록 도와주는 기능이 있다.
이것을 "**Stable Marker**"라고 부른다.
Stable Marker는 [`@Immutable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Immutable)과
[`@Stable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Stable)이라는 2가지 어노테이션을 통해 사용한다.

### @Immutable

`@Immutable`은 한번 객체가 생성된 후 모든 public 필드가 **절대** 바뀌지 않음을 표시하는 마커다.
Kotlin에도 불변적인 값을 저장하기 위해 `val` 키워드가 있지만, `@Immutable`은 그것보다 더 강력한 불변성을 보장해야 한다.

```kotlin
val list = mutableListOf(1, 2, 3)
list.add(4)
```

위 예시를 보면 리스트 자체는 `val`로 선언되어 재할당 할 수 없지만 리스트 내부의 값들 까지는 제약하지 않는다.
`@Immutable`을 사용하고 싶다면 이런 경우 없이 완전히 불변적이여햐 한다.

### @Stable

`@Stable`은 객체가 생성된 후에도 값이 바뀔 수 있지만, 안정 조건을 따른다고 표시하는 마커다.
`@Stable`이 추가 된 객체는 무조건 안정적이라고 판단하고 리컴포지션 발생 시 실제로 값이 바뀌었는지 체크한다.
그리고 값이 바뀌지 않았다면 리컴포지션을 건너 뛸 수 있다.

```kotlin
@Stable
class ProfileUiState(
    val name: String,
    val level: Int
) {
    var isLoading: Boolean
        private set

    fun startLoading() { isLoading = true }

    fun finishLoading() { isLoading = false }
}
```

위 예시의 `ProfileUiState`는 `var`로 된 public 필드가 있어서 원래대로라면 불안정한 상태로 취급되어야 한다.
원래대로라면 `ProfileUiState`가 컴포저블 함수로 넘어갈 때 마다 리컴포지션이 일어나야 하지만,
`@Stable`로 인해 안정적으로 바뀌고 public 필드 일부가 실제로 변경되었을 때에만 리컴포지션을 일으킨다.

### 주의할 점

Stable Marker는 컴파일러에게 약속하는 기능이지 실제로 검사를 수행하지 않는다.
마치 Kotlin에서 `!!`를 사용하면 그 이후로 별도의 검사 없이 NotNull로 취급하는 것과 비슷하다.
`!!`이 사용된 코드에 `null`이 사용되면 문제가 생기듯, Stable Marker를 사용했지만 실제로 불안정하다면 예상 못한 결과가 나올 수도 있다.[^compose-api-guideline]
그러므로 Stable Marker는 Compose 컴파일러가 안정성을 추론하지 못하는 경우에만 한정적으로 사용해야 한다.

### 안정성 전파

Compose의 안정성은 전파되는 특징이 있다.

```kotlin
@Immutable
interface IntList

class ImmutableIntList(vararg values: Int) : IntList

fun intListOf(vararg values: Int): IntList = ImmutableIntList(values)
```

위 예시를 살펴보자.
위 코드에서 각 타입들의 안정성 상태는 아래와 같다.

- `IntList`는 인터페이스이기 때문에 원래대로라면 불안정하게 취급된다.
  하지만, `@Immutable`이 붙어있기 때문에 불변적이라고 취급하게 된다.
- `ImmutableIntList`은 불변적인 `IntList`를 구현한다.
  안정적인 타입을 상속하기 때문에 이 클래스도 안정적이라고 판단한다.
- `intListOf` 함수 역시 안정적인 `IntList`를 반환한다.
  이 함수 역시 안정적이라 판단한다.

여기서 알 수 있는 점은, `ImmutableIntList`와 `intListOf`는 Stable Marker 없이도 안정적이라고 인식한다는 점이다.

```kotlin
interface IntList

@Immutable
class ImmutableIntList(vararg values: Int) : IntList

fun intListOf(vararg values: Int): IntList = ImmutableIntList(values)
```

만약 `@Immutable` 어노테이션을 `ImmutableIntList`로 옮긴다면 어떻게 될까?

- `IntList`는 인터페이스이고 Stable Marker도 없기 때문에 불안정하게 취급된다.
- `ImmutableIntList`는 불안정한 인터페이스를 상속하지만, `@Immutable`이 달려서 불변적으로 취급된다.
- `intListOf`는 내부적으로 불변적인 `ImmutableIntList`를 만들어 반환하지만, 반환 타입은 불안정한 `IntList`이다.
  따라서 이 함수는 불안정한 함수가 된다.

## 정리

Compose에서는 주로 아래 3가지 조건에 의해 안정성을 판단한다.

- 두 인스턴스가 같은 상태라면 두 객체의 비교 결과(`equals`)는 항상 같아야 한다.
- 객체가 가진 모든 public 필드는 안정된 상태여야 한다.
- 만약 값이 변경된다면 컴포저블이 그것을 알 수 있어야 한다.

원시 타입과 `String`, 함수는 안정적으로 취급한다.
불변적인 것은 보통 안정적으로 취급하고 가변적인 것은 불안정하게 취급한다.
불안정한 객체가 컴포저블에 전달되면 Compose는 그 UI를 매번 리컴포지션 한다.
따라서 불안정한 부분들을 안정적으로 만드는 것이 중요하다.

또한 람다에서도 안정적인 값들만 캡쳐링 할 수 있도록 만드는 것이 중요하다.
불안정한 값을 캡쳐링한 람다는 최적화의 도움을 받지 못해 성능상의 문제가 될 수 있다.

불안정한 객체를 임의로 안정화 시킬 수 있는 Stable Marker가 있다.
Stable Marker는 `@Stable`과 `@Immutable` 두 어노테이션으로 사용한다.
`@Stable`은 가변적인 객체에 사용하고, `@Immutable`은 불변적인 객체에 사용한다.
Stable Marker는 편리하지만 컴파일러가 검사해주지 않기 때문에 주의해서 사용해야 한다.
사용하기 전에 꼼꼼히 안정성을 따져 보고 써야 한다.

## 참조

- [Lifecycle of Composables (Android Developers)](https://developer.android.com/jetpack/compose/lifecycle)
- [State and Jetpack Compose (Android Developers)](https://developer.android.com/jetpack/compose/state)
- [State holders and UI State (Android Developers)](https://developer.android.com/topic/architecture/ui-layer/stateholders)
- [Compose API Guideline (AndroidX Compose Repository)](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-api-guidelines.md)
- [Exploring Kotlin: Lambda Bytecode by Christian C. Carroll](https://medium.com/@christian.c.carroll/exploring-kotlin-lambda-bytecode-8c2d15afd490)

[^그림1]: 그림1 출처 https://developer.android.com/jetpack/compose/lifecycle
[^skipping]: https://developer.android.com/jetpack/compose/lifecycle#skipping
[^compose-api-guideline]: 구글에서는 [API 가이드라인](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-api-guidelines.md#stable-types)을 통해서 Stable과 Immutable을 주의 깊게 사용하기를 권장한다.
