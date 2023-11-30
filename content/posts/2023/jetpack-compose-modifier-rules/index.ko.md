---
title: "Jetpack Compose Modifier의 규칙들"
summary: "Jetpack Compose에서 공식적으로 제공하는 Modifier 사용 규칙에 대해서 알아보자."
coverAlt: "Cover"
coverCaption: Image from [Android Developers Blog](https://android-developers.googleblog.com/2022/05/whats-new-in-jetpack-compose.html)
date: 2023-08-11
categories: ["Jetpack Compose"]
---

Jetpack Compose에서 [Modifier](https://developer.android.com/jetpack/compose/modifiers)는 아주 광범위하게 사용된다.
Modifier를 통해서 위치와 크기를 조절하고 이펙트를 추가하는 등 컴포넌트를 디자인한다.
또한 Modifier를 통해 컴포넌트의 클릭, 이펙트, 드래그 등 행동을 정의한다.
Jetpack Compose로 UI 그릴 때 계속 마주쳐야 하는 것이 바로 Modifier다.
하지만, 자주 사용하는 만큼 제시하는 몇 가지 기본적인 사항들을 지켜야 깨끗한 UI 코드를 만들 수 있을 것이다.

## 항상 Modifier를 넘겨라

아주 기본적이며 간단한 규칙이다.
모든 Composable 함수는 **항상** Modifier를 받아야 한다.

Jetpack Compose 개발진은 GitHub Repository 내 문서로 Compose API 가이드라인을 제시하고 있다.
해당 문서에서 Modifier 관련 부분을 보면 ([링크](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-api-guidelines.md#elements-accept-and-respect-a-modifier-parameter)) 다음과 같이 적혀 있다.

> Element functions MUST accept a parameter of type Modifier...

UI 구성요소 함수는 반드시(MUST) Modifier 매개변수를 받아야 한다고 말한다.
그 이유에 대해서는 다음과 같이 서술하고 있다:

> Modifiers are the standard means of adding external behavior to an element in Compose UI and allow common behavior to be factored out of individual or base element API surfaces. This allows element APIs to be smaller and more focused, as modifiers are used to decorate those elements with standard behavior.
>
> An element function that does not accept a modifier in this standard way does not permit this decoration and motivates consuming code to wrap a call to the element function in an additional Compose UI layout such that the desired modifier can be applied to the wrapper layout instead. This does not prevent the developer behavior of modifying the element, and forces them to write more inefficient UI code with a deeper tree structure to achieve their desired result.
>
> Modifiers occupy the first optional parameter slot to set a consistent expectation for developers that they can always provide a modifier as the final positional parameter to an element call for any given element's common case.

Modifier는 UI 컴포넌트에 외부로부터의 동작을 추가하는 수단이며 기본적인 요소들을 작게 만들기 위한 표준적인 API이라고 정의한다. 다시 말해, Modifier라는 표준 API를 통해서 UI 요소의 동작과 구성을 정의함으로 인해 UI 컴포저블을 더 작고 목적에 집중한 형태로 만들 수 있다.

Modifier를 받지 않는다는 것인 이러한 장식 등을 일체 허용하지 않는다는 의미이며, 본래 Modifier로 할 수 있던 일을 우회해서 하기 위해 함수를 감싸거나 수많은 매개변수를 정의해야 하는 등 비효율적인 UI 트리를 만들도록 유도한다.
따라서 Modifier는 Composable 함수에서 항상 매개변수로 주어져야 한다.

## Modifier 매개변수 이름은 항상 "modifier"로 해라

모든 Composable 함수는 Modifier를 매개변수로 받으며 그 이름은 반드시 "modifier" 여야 한다.

Composable 함수는 항상 Modifier를 받을 수 있도록 열려 있어야 한다.
즉, 표준적인 개발 룰을 따르는 개발자라면 언제나 Modifier를 넘길 수 있다고 생각하고 있다.
여기서 어떤 Composable 함수는 Modifier를 "modifier"이라는 이름으로 정의하고 어떤 함수는 "m"로 했다고 가정해보자.
그 개발자는 이제 프로젝트 전체에 있는 Modifier가 "modifier"라는 것을 믿을 수 없다.
이는 개발 경험에 부정적인 영향을 준다.

## Modifier 매개변수는 반드시 첫 번째 선택적 매개변수여야 한다

Composable 함수를 만들다 보면 선택 매개변수로 주어지는 것이 상당히 많다.
Modifier는 그 중 가장 첫 번째여야 한다.
이 규칙 역시 개발 경험을 위한 규칙이다.
예시를 들어 보자.

Do:
```kotlin
@Composable
fun SmallTextButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true
) { ... }
```

위의 SmallTextButton은 text와 onClick 람다를 필수 매개변수로 받는다.
그리고 modifier와 enabled를 선택 매개변수로 가지고 있다.
여기서 modifier가 가장 처음에 위치한 선택적 매개변수이다.

Don't:
```kotlin
@Composable
fun SmallTextButton(
    modifier: Modifier = Modifier,
    text: String,
    onClick: () -> Unit,
    enabled: Boolean = true
) { ... }

@Composable
fun SmallTextButton(
    text: String,
    onClick: () -> Unit,
    enabled: Boolean = true,
    modifier: Modifier = Modifier
) { ... }
```

두 번째 SmallTextButton은 modifier를 필수 매개변수보다 위에 두어 틀렸다.
세 번째 SmallTextButton은 선택 매개변수 중 2번째에 위치해 틀렸다.

## Modifier 매개변수의 기본값은 항상 Modifier이다.

Modifier 매개변수는 반드시 선택적 매개변수여야 하며, 그 기본값은 항상 Modifier여야 한다.
즉, 언제나 `modifier: Modifier = Modifier`의 형태로 정의되어야 한다.

Modifier 매개변수를 `modifier: Modifier = Modifier`형태가 아닌 커스텀 된 형태로 정의한다면 사용자가 다른 Modifier를 재정의해 전달했을 때 기존의 정의가 덮어씌워져 사라진다.
만약 기본적인 Modifier 속성을 정의하고 싶다면 매개변수로 받은 Modifier 뒤에 이어서 붙이면 된다.
예를 들어 Jetpack Compose Material Components 라이브러리의 Surface의 경우 아래처럼 정의되어 있다:

```kotlin
@Composable
fun Surface(
    modifier: Modifier = Modifier,
    shape: Shape = RectangleShape,
    ...
    border: BorderStroke? = null,
    content: @Composable () -> Unit
) {
    ...
    Box(
        modifier = modifier
            .surface(
                shape = shape,
                backgroundColor = surfaceColorAtElevation(
                    color = color,
                    elevation = absoluteElevation
                ),
                border = border,
                shadowElevation = shadowElevation
            )
            .semantics(mergeDescendants = false) {
                 isContainer = true
            }
            .pointerInput(Unit) {},
        propagateMinConstraints = true
    ) {
        content()
    }
}
```

modifier에 `.surface`, `pointerInput` 같은 Modifier 함수를 Chaining 형태로 적용하고 있는 것을 확인할 수 있다.
이렇게 매개변수의 기본값은 항상 빈 Modifier로 유지하고 기본 옵션들을 Method Chaining으로 정의하면 사용자 정의 Modifier가 들어와도 기본 옵션들을 그대로 적용할 수 있다.

## 링크

- [Modifier Documents on Android Developers](https://developer.android.com/jetpack/compose/modifiers)
- [Compose API Guideline](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-api-guidelines.md)
