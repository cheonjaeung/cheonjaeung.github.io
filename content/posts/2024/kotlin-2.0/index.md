---
title: "Kotlin 2.0 살펴보기"
summary: "Kotlin 2.0에 어떤 변화가 있었는지 살펴봅니다."
coverAlt: "Kotlin_2.0"
coverCaption: "Cover Design by Author, Kotlin Logo from [Jetbrains](https://kotlinlang.org/docs/kotlin-brand-assets.html)"
date: 2024-07-19
categories: ["Kotlin"]
---

Kotlin 2.0이 2024년 5월 21일에 정식 릴리즈 되었습니다.
저는 안드로이드 개발자인 만큼 Kotlin/JVM과 관심가는 변경점들에 중점을 두고 어떤 점들이 달라졌는지 살펴볼까 합니다.

## K2 컴파일러 도입

![new-frontend](https://blog.jetbrains.com/wp-content/uploads/2024/05/k2-compiler-architecture.png)
<figcaption>Image from <a href="https://blog.jetbrains.com/ko/kotlin/2024/05/celebrating-kotlin-2-0-fast-smart-and-multiplatform/" target="_blank" rel="noreferrer">JetBrains Blog</a></figcaption>

K2 컴파일러가 Kotlin 2.0에서 stable 버전이 되며 정식 도입되었습니다.
Kotlin 2.0은 사실 언어 기능적 변화보다는 내부적인 변화가 큰 업데이트입니다.
그 중 가장 큰 변화가 K2 컴파일러의 도입이라고 할 수 있습니다.

K2 컴파일러는 Kotlin 팀이 오랫동안 준비한 컴파일러이고, 이전 Kotlin 컴파일러에서 부족했던 프론트엔드를 대체하는 새로운 컴파일러라고 합니다.
K2를 도입하면서 크게 다음과 같은 이점을 얻게 되었다고 합니다.

- **컴파일 속도 향상**: 기존 컴파일러는 전반적으로 유의미할 정도의 컴파일 속도 향상이 이루어졌다고 합니다.
- **모듈화된 컴파일러 개발**: 처음부터 체계적으로 모듈화 해 개발했기 때문에 코드 수정이 더 쉽고, 따라서 언어 기능 개발에 더욱 속도를 높일 수 있다고 합니다.

조금 더 자세히 알아보자면, Kotlin 컴파일러의 구조는 크게 2가지로 나누어 볼 수 있습니다.
프론트엔드와 백엔드이죠.

프론트엔드는 개발자가 작성한 코드를 가장 먼저 읽고 해석해, 백엔드에 전달할 수 있는 중간 결과물을 생성하는 역할을 합니다.
Kotlin 소스 코드를 읽어 문법을 검사하고, 해석해서 Syntax Tree라는 것을 만듭니다.
이 Syntax Tree는 코드를 분해해 컴파일러가 다루기 편하게 구조화한 객체입니다.
그리고 Syntax Tree를 보고 여러 가지 정보를 파악하고 저장합니다.
예를 들면 "이 함수는 어디에 있는 함수인지?", "이 제네릭의 타입은 무엇인지?" 같은 정보들입니다.
이런 일련의 과정을 거쳐 프론트엔드는 백엔드가 읽을 수 있는 중간 결과물을 생성합니다.

그 다음은 백엔드의 차례입니다.
컴파일러 백엔드는 각 타겟에 맞춰 중간 결과물로부터 최종 바이너리를 생성하는 역할을 담당합니다.
Kotlin은 멀티플랫폼 언어이니 JVM, Javascript, WebAssembly, 그리고 여러 플랫폼별 Native로 컴파일이 가능합니다.
당연하게도 각 플랫폼마다 필요한 바이너리는 다르며, 백엔드는 각각에 맞춰 컴파일을 해야 합니다.
JVM은 Java ByteCode를 생성해야 하고, 웹은 Javascript나 WebAssembly로 만들어야 하며, Native는 Windows, macOS, Linux 등등 플랫폼마다 다르지만 기계어나 어셈블리 코드를 생성할 겁니다.

이 중 백엔드는 Kotlin이 JVM 생태계만의 언어에서 멀티플랫폼 언어가 되면서 한 번 대규모로 변경된 적이 있습니다.
하지만 프론트엔드는 대규모 개선이 없었다고 하죠.
당연하게도 언어가 크게 변하는 동안 쌓인 기술 부채나 구조적 한계가 많았고 언어 기능 개발이나 성능 향상에 걸림돌이 되었을 겁니다.
그래서 K2 컴파일러 개발이 시작되었고 이번에 정식 도입이 되며 Kotlin 2.0이 되었습니다.

## Smart Cast 기능 향상

기존에도 Kotlin에서는 스마트 캐스트라는 기능이 있었습니다.
어떤 타입이 특성 상황에서 특정한 타입일 수 밖에 없다고 추론 가능한 경우, 그 타입을 자동으로 특정 타입으로 취급하는 기능입니다.
이 기능을 활용해 불필요하게 타입을 검사하거나 캐스팅하는 코드를 써야 하는 경우가 줄어들었습니다.
예를 들면:

```kotlin
val cat: Cat? = Cat()
if (cat != null) {
    cat.meow() // Cat? 타입이 Cat 타입으로 스마트 캐스트 됩니다.
}
```

위 코드는 nullable 타입인 `Cat?`을 조건문을 통해 null 검사를 수행합니다.
여기서 조건문 안쪽까지 진행되는 경우는 반드시 null이 아니기 때문에 자동으로 not-null 인 `Cat`으로 스마트 캐스트 됩니다.

하지만 이전 Kotlin 버전에서는 스마트 캐스트에 한계가 있었습니다.
이번에 2.0버전으로 넘어오면서 K2 컴파일러가 도입되었고, 더 많은 경우에 스마트 캐스트를 사용할 수 있게 되었다고 합니다.

- [지역 변수를 통한 더 넓은 범위의 스마트 캐스트](#1-지역-변수를-통한-더-넓은-범위의-스마트-캐스트)
- [OR 논리 연산자를 통한 타입 검사](#2-or-논리-연산자를-통한-타입-검사)
- [인라인 함수에서의 스마트 캐스트](#3-인라인-함수에서의-스마트-캐스트)
- [함수 타입 스마트 캐스트 버그 수정](#4-함수-타입-스마트-캐스트-버그-수정)
- [에러 처리와 스마트 캐스트](#5-에러-처리와-스마트-캐스트)
- [증감 연산자를 통한 스마트 캐스트](#6-증감-연산자를-통한-스마트-캐스트)

### 1. 지역 변수를 통한 더 넓은 범위의 스마트 캐스트

```kotlin
// Kotlin 1.9에서의 코드
fun meowIfCat(animal: Any) {
    if (animal is Cat) {
        animal.meow()
    }
}

// Kotlin 2.0부터 가능한 코드
fun meowIfCat2(animal: Any) {
    val isCat = animal is Cat
    if (isCat) {
        animal.meow()
    }
}

class Cat {
    fun meow() {
        println("Meow")
    }
}
```

기존에도 조건문을 통해 타입을 검사하고 해당 타입으로 조건문 내부에서 스마트 캐스트된 변수를 사용하는 것은 가능했습니다.
하지만 조건문 바깥에서 검사한 결과를 지역 변수에 저장하고 그것을 활용한 스마트 캐스트는 지원하지 않았습니다.

예를 들어 위 코드의 `meowIfCat`는 `if`문에서 직접 `animal`의 타입을 검사합니다.
이 경우 Kotlin 1.9에서도 스마트 캐스트가 작동했습니다.
`animal`은 `Cat`임이 자명하고 따라서 `if`문 내에서 `animal`은 `Cat`으로 취급됩니다.

하지만 Kotlin 1.9에서 `meowIfCat2`는 사용할 수 없습니다.
`animal`이 `Cat` 타입인지 검사하긴 하지만 그 검사는 `if`문 바깥에서 수행합니다.
그리고 그 결과를 `isCat`이라는 변수가 저장합니다.
`if`를 통해 `isCat`의 조건을 걸어 코드를 수행하지만 위 코드는 컴파일 에러가 발생합니다.
논리적으로는 문제가 없으나 1.9 버전까지의 컴파일러는 `isCat`과 `animal`이 `Cat`타입인 것을 연관짓지 못했기 때문입니다.
따라서 컴파일러 입장에서 `animal`은 여전히 `Any` 타입이며 `Any`에는 `meow` 메소드가 없기 때문에 컴파일 하지 못합니다.

이 부분이 Kotlin 2.0에서 개선되었습니다.
이제 지역 변수를 활용해 타입을 검사해도 더 넓은 범위에서 스마트 캐스트를 사용할 수 있게 됩니다.
컴파일러가 지역 변수에 대해서 더 많은 정보를 수집하게 되면서 그 이후에서 일어나는 `if`, `while`, `when`문과 같은 조건문에서 스마트 캐스트를 활용할 수 있다고 합니다.

### 2. OR 논리 연산자를 통한 타입 검사

```kotlin
fun doIfFelidae(animal: Any) {
    if (animal is Tiger || animal is Lion) {
        // Kotlin 1.9에서는 에러, Kotlin 2.0부터는 가능
        animal.grrr()
    }
}

interface Felidae {
    fun grrr()
}

class Cat : Felidae {
    override fun grrr() {
        println("meow")
    }
}

class Tiger : Felidae {
    override fun grrr() {
        println("grrr...")
    }
}

interface Lion : Felidae {
    override fun grrr() {
        println("grrr...")
    }
}
```

위 코드에서는 `animal`이 `Tiger` 또는 `Lion`인지 확인하고 참인 경우 공통 부모 타입인 `Felidae`의 메소드를 실행합니다.
Kotlin 1.9에서는 조건문을 통과 하더라도 컴파일러가 타입을 특정하지 못해 `animal`은 여전히 `Any`타입으로 취급했습니다.
그렇기 때문에 `Any`에는 `grrr`라는 메소드가 없어 컴파일 에러가 발생하죠.
Kotlin 2.0부터는 `||` 연산으로 타입을 검사하는 경우, 가장 가까운 부모 타입으로 스마트 캐스트 하도록 개선되었습니다.
따라서 `animal`이 `Tiger` 또는 `Lion`인지를 검사하면 `Tiger`와 `Lion`의 가장 가까운 공통 부모 타입인 `Felidae`로 스마트 캐스트 됩니다.

### 3. 인라인 함수에서의 스마트 캐스트

```kotlin
val processors = arrayOf(PrintProcessor(), PrintlnProcessor())
var processorIndex = 0

fun main() {
    var p: Processor? = null
    do {
        p = runProcessor(p)
    } while (p != null)
}

interface Processor {
    fun process()
}

class PrintProcessor : Processor {
    override fun process() {
        print("Process")
    }
}

class PrintlnProcessor : Processor {
    override fun process() {
        println("Process")
    }
}

inline fun inlineAction(f: () -> Unit) = f()

fun nextProcessor(): Processor? {
    val p = processors.getOrNull(processorIndex)
    processorIndex++
    return p
}

fun runProcessor(initProcessor: Processor?): Processor? {
    var processor: Processor? = initProcessor
    inlineAction {
        if (processor != null) {
            // Kotlin 1.9까지는 processor?.process()와 같이 null safe call이 필요
            processor.process()
        }

        processor = nextProcessor()
    }

    return processor
}

```

위의 예시 코드는 프로세서를 하나씩 꺼내 `runProcessor`를 실행하고 더 이상 프로세서가 없으면 종료하는 코드입니다.
`runPRocessor`는 `Processor.process`를 실행하고 다음 프로세서를 꺼내 반환합니다.

Kotlin 1.9에서는 위 코드 실행 시 다음과 같은 컴파일 에러가 발생합니다.

```
Smart cast to 'Processor' is impossible, because 'processor' is a local variable that is captured by a changing closure
```

`runProcessor`에서는 `processor`라는 지역 변수를 가지고 있고, `inlineAction`이라는 인라인 함수 스코프 내부에서 `processor`의 타입을 검사하고 실행합니다.
Kotlin 1.9에서는 `processor`가 만들어진 스코프와 타입 검사 및 실행하는 스코프가 다르기 때문에 스마트 캐스트 할 수 없다고 에러가 발생했습니다.

Kotlin 1.9까지의 컴파일러는 `inlineAction` 스코프 바깥에서 어떤 일이 일어날지 모르니 스코프 바깥에서 정의된 `processor`는 언제든지 변경될 가능성이 있고 따라서 스마트 캐스트를 지원하지 않았습니다.
사람이 보기에는 그럴 가능성이 없지만, 컴파일러는 엄밀한 정확성이 필요하기에 스코프 바깥의 변수의 스마트 캐스트를 원천 차단했던 것이죠.

Kotlin 2.0에서는 인라인 함수의 경우에 대해서 스마트 캐스트가 개선되었습니다.
인라인 함수는 컴파일 시 사용 부분이 직접 작성한 것 처럼 삽입되어 최적화하기 위한 수단입니다.
즉, 인라인 함수 내부에서 일어난 일은 인라인 함수를 수행한 함수의 범위를 벗어날 수 없다는 뜻입니다.
이 점을 살려 2.0부터는 컴파일러가 인라인 함수 내부에서 사용된 변수에 대해 더 구체적으로 분석해 외부의 영향이 없다고 판단되는 변수에 대해서 스마트 캐스트가 가능하도록 개선되었다고 합니다.

### 4. 함수 타입 스마트 캐스트 버그 수정

```kotlin
class Holder(val provider: (() -> Unit)?) {
    fun process() {
        if (provider != null) {
            // Kotlin 1.9까지는 provider?.invoke()로 실행해야 문제가 없었음
            // Kotlin 2.0부터는 정상적으로 실행 가능
            provider()
        }
    }
}
```

Kotlin 1.9까지는 함수 타입에 대한 스마트 캐스트에 버그가 있었다고 합니다.
Kotlin 2.0부터는 의도한 대로 스마트 캐스트가 가능합니다.

### 5. 에러 처리와 스마트 캐스트

```kotlin
var stringInput: String? = null
stringInput = ""
try {
    println(stringInput.length)

    stringInput = null

    if (2 > 1) throw Exception()
    stringInput = ""
} catch (exception: Exception) {
    // Kotlin 1.9까지는 stringInput이 null이 아니라고 분석해 safe-call을 할 필요 없다고 판단함
    // 하지만 실제로는 nullable 하기 때문에 문제 발생 가능
    println(stringInput?.length)
}
```

`stringInput`이라는 `String?` 타입 변수가 있고, 여기에 `""`를 대입하면 `String`으로 스마트 캐스트 됩니다.
하지만 다시 `null`을 대입하면 `String?`으로 또 한번 스마트 캐스트 됩니다.
결국 `catch` 문에서 `stringInput`은 `String`일지 `null`일지 확실하지 않습니다.

Kotlin 1.9에서는 try-catch 이전에 `String`으로 스마트 캐스트 되었기 때문에 `catch`에서도 `String`이라고 판단해 safe-call 할 필요 없다고 메세지를 출력합니다.
Kotlin 2.0부터는 정상적으로 safe-call 하는 것이 옳다고 판단합니다.

### 6. 증감 연산자를 통한 스마트 캐스트

```kotlin
fun main() {
    var num = One()
    println(num)
    num++
    // Kotlin 1.9에서는 One이 Two로 바뀐 것을 컴파일러가 알지 못함
    // 하지만 2.0부터는 타입 변화를 감지하며, One에서 Two로 스마트 캐스트
    println(num.toEnglish())
}

open class One {
    operator fun inc(): Two = Two()

    override fun toString(): String = "1"
}

class Two: One() {
    fun toEnglish(): String = "two"

    override fun toString(): String = "2"
}
```

Kotlin은 연산자 오버로딩을 통해 코드를 편하게 만들 수 있는 기능을 제공합니다.
하지만 Kotlin 1.9까지는 증감 연산자를 오버로딩 했을 때 타입이 변경될 수 있다는 점을 처리하지 못했고, 스마트 캐스트도 지원하지 않았습니다.
Kotlin 2.0부터는 증감 연산을 통한 타입 변화를 컴파일러가 감지하고 스마트 캐스트를 지원합니다.

## Compose Compiler 버전관리 주체 변경

그 동안은 Jetpack Compose Compiler Plugin과 Kotlin이 별도로 관리되어서 버전 맞추기로 인한 불편함이 있었죠.
Compose Compiler Plugin은 Kotlin 컴파일러에 맞춰 제작되다 보니 버전이 맞지 않으면 제대로 사용할 수 없었기 때문입니다.
하지만 2.0부터 Compose Compiler가 Kotlin repository에 통합된다고 합니다.

- [Kotlin 2.0 소개 문서](https://kotlinlang.org/docs/whatsnew20.html#compiler-plugins-support)
- [Android Developers 블로그 글](https://android-developers.googleblog.com/2024/04/jetpack-compose-compiler-moving-to-kotlin-repository.html)

더 이상 Kotlin 버전과 Compose 버전이 달라 불편했던 경험은 하지 않아도 될 것 같습니다.

## Kotlin Power Assert Plugin (실험적 기능)

실험적인 기능이지만 디버깅과 테스트에 더 자세한 정보를 알려주는 Power-assert 플러그인이 도입되었다고 합니다.
아직 실험적인 기능이기 때문에 언제든 바뀔 수 있으니 주의해 사용해야 합니다.

```kotlin
plugins {
    kotlin("plugin.power-assert") version "2.0.0"
    // or
    id("org.jetbrains.kotlin.plugin.power-assert") version "2.0.0"
}
```

사용하려면 Gradle 빌드스크립트에 위와 같은 플러그인을 추가하면 됩니다.

```kotlin
class AssertTest {
    @Test
    fun assertTest() {
        val hello = "Hello"
        val world = "World!"
        assert(hello.length == world.substring(1, 4).length) { "Incorrect length" }
    }
}
```

Power Assert 플러그인을 사용하면 `assert` 함수가 강화됩니다.
기존에는 위 코드를 실행하면 아래와 같은 메세지가 출력됩니다.

```text
Incorrect length
java.lang.AssertionError: Incorrect length
	at AssertTest.assertTest(AssertTest.kt:8)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
```

하지만 Power Assert가 활성화되면 아래와 같이 바뀝니다.

```text
Incorrect length
assert(hello.length == world.substring(1, 4).length) { "Incorrect length" }
       |     |      |  |     |               |
       |     |      |  |     |               3
       |     |      |  |     orl
       |     |      |  World!
       |     |      false
       |     5
       Hello
```

훨씬 더 자세한 결과를 보여주도록 `assert`가 강화되었습니다.
물론 아직 실험적 기능이다 보니 부족한 부분이 있을 수 있지만 코드 디버깅에 훨씬 도움이 될 것 같습니다.

자세한 내용은 [공식 문서](https://kotlinlang.org/docs/power-assert.html)에서 확인해 보세요.

## 그 외

그 밖에도 Kotlin Multiplatform 관련 개선사항, Standard Library의 일부 기능 안정화, Kotlin Gradle DSL 등 여러가지 자잘한 변경들이 있습니다.
[공식 문서](https://kotlinlang.org/docs/whatsnew20.html)를 확인해 보세요.
