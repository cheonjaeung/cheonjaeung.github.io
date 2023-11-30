---
title: "Kotlin Explicit API 옵션으로 더 엄격한 문법 규칙 적용하기"
summary: "라이브러리와 같이 타입 정의가 중요한 경우 Explicit API라는 컴파일러 기능을 활용 할 수 있다."
date: 2022-10-07
categories: ["Kotlin"]
---

Kotlin 1.4버전이 출시할 때, **Explicit API** 라는 기능이 컴파일러에 추가되었다.
Explicit은 '명시적' 이라는 뜻의 영어 단어다. 번역하자면 '명시적 API' 정도가 되겠다.

## 왜 필요할까?

이 기능은 기존의 유연한 코틀린의 문법 검사를 엄격하게 바꿔 '명시적'으로 코드에 적도록 하는 옵션이다.
코틀린은 유연하고 간결한 문법을 자랑하는데 왜 엄격한 모드를 제공할까?

유연한 문법은 코드 생산성을 높여주는 아주 편리한 기능이지만 엄격하게 관리해야 하는 경우에는 오히려 독이 될 때가 있다.
예를 들면 라이브러리 개발이다.
내가 어떤 함수를 만들어 두었는데 반환 타입을 추론하도록 만들었다고 해보자.
그런데 패치를 하다 보니 나도 모르게 수정되어서 다른 타입을 추론하게 될 수도 있다.
오픈 소스나 팀이 개발하는 경우 제 3자가 수정했는데 미처 파악하지 못했을 수도 있다.
이럴 때는 엄격한 문법이 도움이 된다.

## Explicit API를 활성화하면 바뀌는 것들

### 접근제한자 명시

기존 코틀린 문법에서는 접근 제한자를 달지 않으면 자동으로 `public`으로 만든다.
하지만 Explicit API가 활성화되면 `public`이든 `protected`든 어떤 접근 제한자이든 명시해 주어야 한다.
어떤 클래스, 함수, 필드가 공개되어 있는지 숨겨져 있는지 명확하게 보이게 된다.

```kotlin
// 기존에 이렇게 사용했다면,
fun remove(index: Int)

// 이렇게 수정해야 한다.
public fun remove(index: Int)
```

### 반환 타입 명시

코틀린 컴파일러는 타입 추론 기능이 있어서 함수 등을 선언할 때 타입이 명확하다면 굳이 타입을 명시하지 않아도 무방하다.
하지만 Explicit API가 활성화되면 반드시 함수나 프로퍼티 등에 반환 타입을 명시해야 한다.

```kotlin
// 기존에 이렇게 사용했다면,
fun isAllEmpty() = list1.isEmpty() && list2.isEmpty()

// 이렇게 수정해야 한다.
fun isAllEmpty(): Boolean = list1.isEmpty() && list2.isEmpty()
```

## Explicit API 활성화하기

이 컴파일러 옵션을 활성화 하는 방법은 간단하다.
코틀린 컴파일러에게 `-Xexplicit-api=strict|warning`을 옵션으로 전달하면 된다.
`strict`를 사용하면 규칙에 위배되는 코드를 오류로 표시하고, `warning`으로 하면 경고로 표시한다.
오류는 아예 컴파일이 불가능하고 경고는 컴파일은 가능하나 계속 경고를 띄운다.

```kotlin
tasks.withType<KotlinCompile> {
    kotlinOptions.freeCompilerArgs += "-Xexplicit-api=strict"
}
```
예를 들어 `build.gradle.kts` 파일에서 이 옵션을 활성화한다면 위 코드처럼 할 수 있겠다.
