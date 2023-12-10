---
title: "Kotlin Immutable Collections: Kotlin Collections are read-only not immutable"
summary: "Kotlin standard collections are not actually immutable. They are just read-only. Let's know how to use immutable collections."
coverAlt: "Kotlin"
coverCaption: "Cover by Author, Logo from [Jetbrains](https://kotlinlang.org/docs/kotlin-brand-assets.html)"
date: 2023-12-10
categories: ["Kotlin"]
---

There are some collection types in Kotlin standard library like `List`, `Set` and `Map`.
They are all interface.
For example, the `List` is implemented like this:

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

`List` doesn't have any mutating function like `add` or `remove`.
Mutating functions are included in `MutableList` that extends the `List` interface.

## Does Kotlin standard collections actually immutable?

Actually, Kotlin standard collections are not immutable, but they are read only.
Let's check following code:

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

Firstly, it creates a `MutableList` named `numStrings`.
And it prints list elements every second using `printListEverySeconds`.
The `printListEverySeconds` function have `list` parameter that is typed as `List`.
Meanwhile, it adds a new element to `numStrings` every second.
Let's expect the result.

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

The result will look like this.

`printListEverySeconds` takes a list as `List` type, but actual implementation is `MutableList`.
`printListEverySeconds` can't change list elements but the main function can.
Because `printListEverySeconds` use the list as a read-only type `List`, but main function use the same list instance as `MutableList` type.

So, the standard collection types are **read only**, not immutable.

## The Kotlinx Collections Immutable library

Kotlin Organization is developing [kotlinx.collections.immutable](https://github.com/Kotlin/kotlinx.collections.immutable) library as an open source.
This library contains actual immutable collections for Kotlin.

There introduce 2 types of collection:

- **Immutable**: They specify by their contract the real immutability of their implementors.
- **Persistent**: They extend immutable collections and provide efficient modification operations that return new instances of persistent collections with the modification applied.
  The returned collections can share parts of data structure with the original persistent collections.

Both `Immutable` and `Persistent` extend standard collection types.
But implementation is immutable.

## How to use immutable collections

### Creating a new immutable collection

```kotlin
val list = persistentListOf(1, 2, 3, 4, 5)
```

This library contains convenience functions like standard library.
you can just make a new instance with `persistent` prefixed functions.

{{< alert "circle-info" >}}
Note that there are only `persistent` prefixed functions.
`immutable` prefixed functions (like `immutableListOf` or `immutableSetOf`) are deprecated.
{{< /alert >}}

### Convert collections to immutable collections

```kotlin
val immutableList = list.toImmutableList()
val persistentList = list.toPersistentList()
```

All collection types can convert to immutable collections with `toImmutable` and `toPersistent`.

### Modifying immutable collection

```kotlin
val list1 = persistentListOf(1, 2, 3, 4, 5)
val list2 = list1.add(6)
println("list1=$list1") // list1=[1, 2, 3, 4, 5]
println("list2=$list2") // list2=[1, 2, 3, 4, 5, 6]
```

`Persistent` collections have modification operators.
Unlike mutable collections of standard library, they return a new instance collection with the modification applied.

### Use immutable collections as standard collection type

kotlinx.collections.immutable library collections are extends standard library collection types.
So, all immutable collections can be considered as standard collection type like `List`.

## Conclusion

In this article, we understood that the Kotlin standard library collections are not immutable but read only.
And we can use kotlinx.collections.immutable library for immutable collections.

Standard collections are not actually immutable.
However, read only is useful enough to reduce side effects.
You can consider immutable collection for perfect immutability.
And also immutable collections for Jetpack Compose is great.
Because Jetpack Compose Compiler thinks immutable collections as stable type.
(Standard collection types are unstable.)
