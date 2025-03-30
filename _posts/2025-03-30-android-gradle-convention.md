---       
layout: post
title: "[Android] Gradle Convention Plugin 으로 멀티 모듈 효율적으로 관리하기"
date: 2025-03-30
categories: android
photos: /assets/post_images/android/13.png
tags: [android, gradle, gradle_plugin, convention_plugin, multi-module, modularization, build]
description: "멀티 모듈 구조의 프로젝트에서 gradle convention plugin 을 통해 여러 모듈들의 gradle 설정을 간결하고 공통되게 관리해보고, 더 잘 사용할 수 있는 몇 가지 방법들을 살펴봅시다."
---

# 개요

멀티 모듈 구조의 프로젝트를 진행하다보면, 모듈들이 점점 많아지면서 각 모듈에서 사용되는 gradle 설정들도 중복해서 작성되기 마련입니다. 이렇게 되면 보일러 플레이트가 증가할 뿐만 아니라 gradle 설정을 공통으로 관리하기 어려워져 유지보수 비용이 크게 늘어나게 될 수도 있습니다. 마치 **버전 카탈로그**를 이용해 라이브러리 의존성 버전을 전역에서 일관되게 관리했던 것처럼, gradle 설정 또한 이렇게 관리할 수 없을까요? 바로 그 솔루션인 **Gradle Convention Plugin**에 대해 알아봅시다.

<br>

# Convention Plugin 만들고 적용하기

대부분의 모듈에 적용이 필요해서 중복으로 설정되고 있는 gradle 설정이 뭐가 있을까요? **아마 가장 떠올리기 쉬운 것은 바로 lint 관련 설정**일 것입니다. Kotlin 의 가장 대표적인 lint 툴인 **spotless** 를 적용하는 상황을 예로 들어봅시다.

```kotlin
// app/build.gradle.kts

plugins {
    id("com.diffplug.spotless")
}

spotless {
    kotlin {
        target("**/*.kt")
        targetExclude("**/build/**/*.kt")
        licenseHeader(licenseHeader)
        ktlint(libs.versions.ktlint.get())
    }
    java {
        target("**/*.java")
        targetExclude("**/build/**/*.java")
        licenseHeader(licenseHeader)
    }
    format("kts") {
        target("**/*.kts")
        targetExclude("**/build/**/*.kts")
        licenseHeader(licenseHeader, "(^(?![\\/ ]\\*).*$)")
    }
    format("xml") {
        target("**/*.xml")
        targetExclude("**/build/**/*.xml")
    }
}

private val licenseHeaderKotlin = buildString {
        append("/*\n")
        append(" * Copyright 2025 yjyoon-dev\n")
        append(" *\n")
        append(" * Licensed under the Apache License, Version 2.0 (the \"License\");\n")
        append(" * you may not use this file except in compliance with the License.\n")
        append(" * You may obtain a copy of the License at\n")
        append(" *\n")
        append(" *     http://www.apache.org/licenses/LICENSE-2.0\n")
        append(" *\n")
        append(" * Unless required by applicable law or agreed to in writing, software\n")
        append(" * distributed under the License is distributed on an \"AS IS\" BASIS,\n")
        append(" * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.\n")
        append(" * See the License for the specific language governing permissions and\n")
        append(" * limitations under the License.\n")
        append(" */\n")
        append("\n")
    }
```

우선 app 모듈에 spotless 를 적용해보았습니다. 하지만 아직 남아있는 모듈이 많습니다. 모든 모듈의 gradle 파일에 위 코드를 복붙하지 않기 위해, **spotless 를 위한 convention plugin** 을 만들어봅시다.

우선 이 convention plugin 이라는 것은 프로젝트 어디에 위치해야 할까요? Android 개발의 바이블인 [Now in Android](https://github.com/android/nowinandroid) 프로젝트에서는 **`build-logic` 이라는 폴더 안에 `convention` 이라는 이름의 모듈을 만들어 plugin 들을 관리하고 있습니다.** 그리고 이 `build-logic` 폴더를 빌드에 포함시키기 위해 `build-logic/settings.gradle.kts` 파일을 작성하고, 루트의 `settings.gradle.kts` 에서 해당 폴더를 `includedBuild` 시켜줍니다.

```kotlin
// build-logic/settings.gradle.kts

pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
    }
}

dependencyResolutionManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}

rootProject.name = "build-logic"
include(":convention")
```

```kotlin
// settings.gradle.kts

pluginManagement {
    includeBuild("build-logic")
    /* . . . */
}
```

`build-logic` 폴더를 만들었다면 해당 폴더 안에 `convention` 모듈을 포함시켜줍니다. `convention` 모듈의 `build.gradle.kts` 는 아래와 같습니다.

```kotlin
// build-logic/convention/build.gradle.kts

plugins {
    `kotlin-dsl`
}

group = "dev.yjyoon.example.buildlogic.convention"

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

kotlin {
    compilerOptions {
        jvmTarget = JvmTarget.JVM_17
    }
}

dependencies {
    implementation(libs.spotlessGradlePlugin)
 // implementation(libs.bundles.plugin)
}
```
한 가지 확인해야 할 점은 **내가 만드려는 플러그인이 의존하는 gradle plugin 의 의존성을 convention gradle 모듈에 추가해주어야 한다는 점입니다.** 위 코드에서 `libs.spotlessGradlePlugin`은 `com.diffplug.spotless:spotless-plugin-gradle` 패키지를 의미합니다. 앞으로 convention plugin 이 추가될수록 필요한 의존성도 많아지기 때문에, 주로 **버전 카탈로그의 `bundles` 기능**을 사용하여 gradle 플러그인 의존성을 bundle 로 묶어 한 번에 추가하는 편이 일반적입니다.

자 이제 spotless 적용을 위한 convention plugin 을 만들어 봅시다.

```kotlin
// build-logic/convention/SpotlessConventionPlugin.kt

class SpotlessConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.diffplug.spotless")
            }
            extensions.configure<SpotlessExtension> {
                kotlin {
                    target("**/*.kt")
                    targetExclude("**/build/**/*.kt")
                    licenseHeader(licenseHeaderKotlin)
                    ktlint(libs.versions.ktlint.get())
                }
                java {
                    target("**/*.java")
                    targetExclude("**/build/**/*.java")
                    licenseHeader(licenseHeader)
                }
                format("kts") {
                    target("**/*.kts")
                    targetExclude("**/build/**/*.kts")
                    licenseHeader(licenseHeader, "(^(?![\\/ ]\\*).*$)")
                }
                format("xml") {
                    target("**/*.xml")
                    targetExclude("**/build/**/*.xml")
                }
            }
        }
    }
}

private val licenseHeaderKotlin = buildString {
        append("/*\n")
        append(" * Copyright 2025 yjyoon-dev\n")
        append(" *\n")
        append(" * Licensed under the Apache License, Version 2.0 (the \"License\");\n")
        append(" * you may not use this file except in compliance with the License.\n")
        append(" * You may obtain a copy of the License at\n")
        append(" *\n")
        append(" *     http://www.apache.org/licenses/LICENSE-2.0\n")
        append(" *\n")
        append(" * Unless required by applicable law or agreed to in writing, software\n")
        append(" * distributed under the License is distributed on an \"AS IS\" BASIS,\n")
        append(" * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.\n")
        append(" * See the License for the specific language governing permissions and\n")
        append(" * limitations under the License.\n")
        append(" */\n")
        append("\n")
    }
```

그런데 위 파일을 작성하고 나면, **버전 카탈로그의 libs 객체에 접근할 수 없어** ktlint 버전을 가져오지 못하는 것을 발견하실 수 있습니다. `build-logic` 패키지 내부에 있는 `convention` 모듈에서는 버전 카탈로그에 바로 접근할 수 없으므로, 편리하게 convention plugin 을 작성하기 위해선 이를 위한 유틸 확장 함수를 따로 선언해야 합니다.

```kotlin
// build-logic/convention/VersionCatalogDsl.kt

internal val Project.libs: VersionCatalog
    get() = extensions.getByType<VersionCatalogsExtension>().named("libs")

internal fun VersionCatalog.version(name: String): String = findVersion(name).get().requiredVersion

internal fun VersionCatalog.library(name: String): MinimalExternalModuleDependency = findLibrary(name).get().get()

internal fun VersionCatalog.plugin(name: String): PluginDependency = findPlugin(name).get().get()

internal fun VersionCatalog.bundle(name: String): ExternalModuleDependencyBundle = findBundle(name).get().get()
```

이와 같이 `Project` 객체와 `VersionCatalog` 객체에 확장 함수를 정의하면 `convention` 모듈에서 보다 편리하게 버전 카탈로그를 이용할 수 있습니다. 이제 위에서 작성했던 `SpotlessPlugin` 에서 ktlint 버전을 설정하는 부분을 아래와 같이 수정하면 정상적으로 동작하는 것을 확인하실 수 있습니다.

```kotlin
// ktlint(libs.versions.ktlint.get())
ktlint(libs.version("ktlint"))
```

이렇게 만들 플러그인을 다른 모듈에서 사용하기 위해서는 이를 `gradlePlugin` 에 등록해주어야 합니다. `convention` 모듈의 `build.gradle.kts` 파일에 아래와 같이 추가해 줍시다.

```kotlin
// build-logic/convention/build.gradle.kts

 register("spotless") {
    id = "example.convention.spotless"
    implementationClass = "SpotlessConventionPlugin"
}
```

여기서 `id` 필드는 다른 모듈에서 plugin 을 추가할 때의 식별자가 되고, `implementationClass` 는 실제 플러그인의 구현체 클래스를 의미합니다. 이제 다른 모듈에서 `id("example.convention.spotless")` 와 같은 방식으로 convention plugin 을 추가할 수 있습니다.

```kotlin
// core/whatever/build.gradle.kts

plugins {
    id("example.convention.spotless")
}
/* . . . */
```

<br>

# 모듈 유형별 의존성 분기 걸기

이번엔 spotless 와 같이 대부분의 모듈에 포함되기 쉬운 **Hilt** 를 convention plugin 을 이용해 적용해봅시다.

```kotlin
// build-logic/convention/HiltConventionPlugin.kt

class HiltConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            apply(plugin = "com.google.devtools.ksp")

            dependencies {
                "ksp"(libs.library("hilt.compiler"))
            }

            // Add support for Jvm Module
            pluginManager.withPlugin("org.jetbrains.kotlin.jvm") {
                dependencies {
                    "implementation"(libs.library("hilt.core"))
                }
            }

            // Add support for Android modules
            pluginManager.withPlugin("com.android.library") {
                apply(plugin = "dagger.hilt.android.plugin")
                dependencies {
                    "implementation"(libs.library("hilt.android"))
                }
            }
        }
    }
}

// build-logic/convention/build.gradle.kts

gradlePlugin {
    register("hilt") {
        id = "example.convention.hilt"
        implementationClass = "HiltConventionPlugin"
    }
}
```

Spotless convention plugin 에서는 보지 못했던 패턴이 보입니다. 바로 **모듈 유형별로 의존성을 분기 처리 하는 것입니다.** Hilt 를 사용하기 위해 필수적인 `ksp` 플러그인과  `hilt-compiler` 의존성은 공통으로 추가하고, 모듈에 걸려있는 plugin 종류에 따라 각기 다른 hilt 관련 의존성을 추가해줍니다. 위 코드의 예시로 `kotlin("jvm")` 의존성이 걸려있다면 안드로이드 의존성이 없는 순수 JVM 모듈이기 때문에 `hilt-core` 에 대한 의존성만 추가하고, `com.android.library` 의존성이 걸려있는 안드로이드 모듈에 경우 이에 필요한 `hilt-android` 의존성을 추가해줄 수 있습니다.

<br>

# 상위 convention 들을 묶어 하위 convention 만들기

특정 gradle 설정을 담는 **convention plugin 여러개가 모여서 또 하나의 convention plugin 을** 만들어내기도 합니다. 예를 들어, **feature 기반 멀티 모듈 구조**의 안드로이드 프로젝트에서 **어느 한 feature 모듈을 가정**해봅시다.

```kotlin
// feature/home/build.gradle.kts

plugins {
    id("example.convention.android.library")
    id("example.convention.android.compose")
    id("example.convention.android.hilt")
    id("example.convention.android.spotless")
    id("org.jetbrains.kotlin.plugin.serialization")
    /* . . . */
}

dependencies {
    implementation(projects.core.ui)
    implementation(projects.core.model)
    implementation(projects.core.data)

    implementation(libs.androidx.lifecycle.viewModelCompose)
    implementation(libs.androidx.navigation.compose)
    /* . . . */
}
/* . . . */
```

위에서 만든 spotless 와 hilt 외에도 android, compose 관련 설정들 또한 convention plugin 으로 만들어 적용했다고 가정한 모습입니다. 이 정도만 되어도 convention plugin 을 사용하지 않았을 때보다 충분히 코드량이 많이 줄었고 재사용성도 높아졌지만, **더 공통화 시킬 수 있는** 건덕지가 보이는 것 같습니다. `feature` 모듈이라면 위 코드와 같이 android 및 compose 관련 플러그인이나 `ui` 및 `data` 와 같은 핵심 core 모듈, 그리고 ViewModel 과 Navigation 에 대한 의존성은 **필수가 아닐까요?** 이번엔 이들을 묶어서 **feature 모듈을 위한 공통 convention plugin**을 만들어봅시다.

```kotlin
// build-logic/convention/FeatureConventionPlugin.kt

class FeatureConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                id("example.convention.android.library")
                id("example.convention.android.compose")
                id("example.convention.android.hilt")
                id("example.convention.android.spotless")
            }
        }

        dependencies {
                "implementation"(projects(":core:ui"))
                "implementation"(projects(":core:model"))
                "implementation"(projects(":core:data"))

                "implementation"(libs.findLibrary("androidx.lifecycle.viewModelCompose"))
                "implementation"(libs.findLibrary("androidx.navigation.compose"))
        }
    }
}

// build-logic/build.gradle.kts

gradlePlugin {
    register("feature") {
        id = "example.convention.feature"
        implementationClass = "FeatureConventionPlugin"
    }
}
```

이제 새로운 feature 모듈을 만들 때 번거롭게 반복적인 gradle 설정을 할 필요 없이, 아래와 같이 **feature convention plugin 을 적용하는 것으로 생산성과 일관성을 동시에 높여줄 수 있습니다.**

```kotlin
// feature/whatever/build.gradle.kts

plugins {
    id("example.convention.feature")
}
/* . . . */
```
<br>

# 마치며

이렇게 한 번 개발한 convention plugin 은 특정 프로젝트에서 뿐만 아니라, **다른 프로젝트에서도 충분히 재사용이 가능하다는 이점**을 갖고 있습니다. 멀티 모듈 구조의 프로젝트를 개발할 때의 가장 일반적인 문제 중 하나가 바로 **코드양이 늘어남으로써 오버헤드가 발생할 수 있다는 점**인데, **gradle convention plugin 이 이를 해결하는 데에 아주 결정적인 역할을 하는 것 같습니다.** 만약 모듈화를 진행하신다면 반드시 적용해보시는 것을 추천드립니다!
