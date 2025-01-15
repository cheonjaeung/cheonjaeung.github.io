---
title: "How to manage dependencies with Gradle Version Catalogs"
summary: Gradle Version Catalogs is added since Gradle 7.4. Let's check how to use it.
coverAlt: "Gradle Logo"
coverCaption: "Gradle Logo"
date: 2023-11-29
categories: ["Gradle"]
---

After the Gradle version 7.4, a new feature named [Version Catalogs](https://docs.gradle.org/current/userguide/platforms.html) is added.
Version Catalogs helps to manage dependencies and plugins in scalable way.
It makes easier to share dependencies and plugins between modules in whole project.

## Ready to use Version Catalogs

Version Catalogs is added since Gradle 7.4.
If your project's Gradle version is old, you need to upgrade Gradle first.

```shell
./gradlew wrapper --gradle-version=x.y
```

After the Gradle 8.0, Version Catalogs is enabled as default.
But if your project's Gradle version is below than 8.0, you need to enable Version Catalogs feature:

```kotlin
enableFeaturePreview("VERSION_CATALOGS")
```

## Creating a file for declaring dependencies

Gradle searches version catalogs from `libs.versions.toml` file.
So, we need to create `libs.versions.toml` file first.
If you want to use other name, you need to fix build script.
Checkout [official documents](https://docs.gradle.org/current/userguide/platforms.html) for more details.

By default, `libs.versions.toml` is placed in the root project's `gradle` directory.
The path will look like `gradle/libs.versions.toml`.

## Declaring dependencies

Following code shows the sections of Version Catalogs.

```toml
[versions]

[plugins]

[libraries]
```

Each sections means that:

- versions: Define variables for versions of dependencies and plugins. The variables used in plugins and libraries section.
- plugins: Define Gradle plugins for this project.
- libraries: Define dependencies for this project.

Following is version catalogs of my [GridLayout for Compose](https://github.com/cheonjaeung/gridlayout-compose) project.

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

When set a version, the hard coded version uses `version` and the variable version uses `version.ref`.
And plugins uses `id` for declaring Gradle plugin ID.
And the libraries use `module` for declaring group name and artifact name.

## Using Version Catalogs

Now we can use version catalogs in `dependencies` block in build script.

```kotlin
dependencies {
    implementation(libs.androidx.core)
}
```

Applying plugins is little different.
First, set all plugins of your project like following code.

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm) apply false
}
```

And second, set plugin at the actual target module like this.

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
}
```

## Tips

You can use `versions` for other variables.

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

## References

- [Sharing dependency versions between projects](https://docs.gradle.org/current/userguide/platforms.html) - Gradle Official Documents
- [Migrate your build to version catalogs](https://developer.android.com/build/migrate-to-catalogs) - Android Developers
