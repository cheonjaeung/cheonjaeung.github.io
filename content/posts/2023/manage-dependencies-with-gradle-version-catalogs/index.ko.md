---
title: "Gradle Version Catalogs로 의존성 관리하기"
summary: "Gradle 7.4부터 새로운 버전 관리 방법이 도입되었습니다. Version Catalogs를 이용해 의존성을 관리하는 방법을 알아봅니다."
coverAlt: "Gradle Logo"
coverCaption: "Gradle Logo"
date: 2023-11-29
categories: ["Gradle"]
---

Gradle 7.4 버전부터 [Version Catalogs(버전 카탈로그)](https://docs.gradle.org/current/userguide/platforms.html)라는 새로운 기능이 추가되었습니다.
버전 카탈로그를 사용하면 전체 프로젝트에서 모듈 간의 의존성들과 플러그인들을 확장성 있게 관리할 수 있습니다.
또한 이전 방식들 보다 간편하게 모듈 간에 의존성 공유가 가능합니다.

## 버전 카탈로그를 위한 준비

버전 카탈로그는 Gradle 7.4 이후부터 추가된 기능입니다.
Gradle 버전이 낮다면 우선 업그레이드를 해야 합니다.

```shell
./gradlew wrapper --gradle-version=x.y
```

버전 카탈로그는 Gradle 8.0부터 기본적으로 활성화 되도록 바뀌었습니다.
만약 프로젝트의 Gradle 버전이 8 이하라면, 별도로 활성화 시켜 주어야 합니다.
프로젝트의 `settings.gradle.kts` 또는 `settings.gradle` 파일에 아래와 같이 추가해 주세요.

```kotlin
enableFeaturePreview("VERSION_CATALOGS")
```

## 의존성을 정의할 파일 만들기

Gradle은 기본적으로 `libs.versions.toml`이라는 파일에서 의존성 정보를 가져옵니다.
따라서 우선 `libs.versions.toml` 파일을 만들어야 합니다.
다른 이름을 사용하는 것도 가능하지만, 추가적인 작업이 필요합니다.
자세한 내용은 [공식 문서](https://docs.gradle.org/current/userguide/platforms.html)를 확인하세요.

기본적으로 버전 카탈로그 파일은 프로젝트의 `gradle` 폴더 안에 위치합니다.
따라서 `gradle/libs.versions.toml`와 같은 경로를 가집니다.

## 버전 카탈로그 작성하기

버전 카탈로그의 기본적인 형식은 아래와 같습니다.

```toml
[versions]

[plugins]

[libraries]
```

각 섹션에 대한 내용은 아래와 같습니다.

- versions: 라이브러리나 플러그인 등의 버전을 가지는 변수를 지정합니다. 일반적으로 plugins와 libraries 섹션에서 사용합니다.
- plugins: 프로젝트에서 사용하는 Gradle 플러그인을 정의합니다.
- libraries: 프로젝트에서 사용하는 라이브러리 의존성을 정의합니다.

아래 샘플은 [GridLayout for Compose](https://github.com/cheonjaeung/gridlayout-compose)에서 사용한 버전 카탈로그의 예시입니다.

```toml
[versions]
kotlin = "1.9.0"
android-gradle-plugin = "8.1.1"
compose-multiplatform = "1.5.1"
compose-android = "1.5.0"
compose-android-compiler-plugin = "1.5.2"

[plugins]
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
android-library = { id = "com.android.library", version.ref = "android-gradle-plugin" }
android-application = { id = "com.android.application", version.ref = "android-gradle-plugin" }
compose-multiplatform = { id = "org.jetbrains.compose", version.ref = "compose-multiplatform" }
maven-publish = { id = "com.vanniktech.maven.publish", version = "0.25.3" }

[libraries]
androidx-core = { module = "androidx.core:core", version = "1.10.1" }
compose-multiplatform-runtime = { module = "org.jetbrains.compose.runtime:runtime", version.ref = "compose-multiplatform" }
compose-multiplatform-foundation = { module = "org.jetbrains.compose.foundation:foundation", version.ref = "compose-multiplatform" }
compose-android-runtime = { module = "androidx.compose.runtime:runtime", version.ref = "compose-android" }
compose-android-foundation = { module = "androidx.compose.foundation:foundation", version.ref = "compose-android" }

# Test dependencies
junit4 = { module = "junit:junit", version = "4.13.2" }
compose-android-ui-test-junit4 = { module = "androidx.compose.ui:ui-test-junit4", version.ref = "compose-android" }
compose-android-ui-test-manifest = { module = "androidx.compose.ui:ui-test-manifest", version.ref = "compose-android" }

# Sample dependencies
androidx-appcompat = { module = "androidx.appcompat:appcompat", version = "1.6.1" }
androidx-activity-compose = { module = "androidx.activity:activity-compose", version = "1.7.2" }
compose-android-ui = { module = "androidx.compose.ui:ui", version.ref = "compose-android" }
compose-multiplatform-material3 = { module = "org.jetbrains.compose.material3:material3", version.ref = "compose-multiplatform" }
compose-android-material3 = { module = "androidx.compose.material3:material3", version = "1.1.1" }
```

plugins나 libraries에서 버전을 사용할 때, 하드코딩 하는 경우 `version`을, versions에서 정의한 변수를 사용하는 경우 `version.ref`를 사용하는 것을 볼 수 있습니다.
또한 플러그인은 `id`를 사용해 Gradle 플러그인의 아이디를 정의하고, 라이브러리는 `module`로 그룹 네임과 아티팩트 네임을 정의하는 것을 볼 수 있습니다.

## 버전 카탈로그 사용하기

이제 프로젝트의 의존성 선언 블록에서 버전 카탈로그를 사용할 수 있습니다.

```kotlin
dependencies {
    implementation(libs.androidx.core)
}
```

Gradle 플러그인에 적용하는 경우 조금 다릅니다.
우선 프로젝트 최상위 `build.gradle`에 프로젝트에서 사용하는 플러그인을 아래와 같이 정의합니다.

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm) apply false
}
```

그 후 플러그인을 실제로 적용하는 모듈에서 아래와 같이 적용합니다.

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
}
```

## 추가 팁

버전 카탈로그의 versions 섹션에 라이브러리 외의 변수도 저장할 수 있습니다.

```toml
[versions]
minsdk = "21"
compose-compiler = "1.5.2"
```

```kotlin
android {
    defaultConfig {
        minSdk = libs.versions.minsdk.get().toInteger()
    }

    composeOptions {
        kotlinCompilerExtensionVersion = libs.versions.compose.android.compiler.plugin.get()
    }
}
```

## 레퍼런스

- [Sharing dependency versions between projects](https://docs.gradle.org/current/userguide/platforms.html) - Gradle Official Documents
- [Migrate your build to version catalogs](https://developer.android.com/build/migrate-to-catalogs) - Android Developers
