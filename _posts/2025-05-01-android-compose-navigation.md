---       
layout: post
title: "[Android/KMP] Compose Navigation 멀티 모듈 환경에서 제대로 사용하기"
date: 2025-05-01
categories: android
photos: /assets/post_images/android/14.png
tags: [android, kmp, compose, navigation, multi-module, architecture]
description: "멀티 모듈 구조의 안드로이드 및 KMP 프로젝트에서 compose navigation 을 이용해 모듈 간 캡슐화와 의존성을 고려한 화면 네비게이션 기능을 제대로 구현해보자"
---

<br>

# 개요

최근에 사이드로 진행했던 KMP 프로젝트를 단일 모듈 구조에서 feature 기반 멀티 모듈 구조로 마이그레이션하는 작업을 했었습니다. 당시 저는 Navigation 및 ViewModel 에 대한 솔루션으로 KMP 를 지원하는 [Voyager](https://github.com/adrielcafe/voyager) 를 사용했었는데, 이는 navigate 동작 시 타겟 Screen 을 반드시 직접 참조해야했기 때문에 feature 별 모듈 분리에 제약이 있었습니다. 따라서 프로젝트 전반에 퍼져있는 **Voyager 를 걷어내고 이를 이제 막 KMP 를 alpha 버전으로 지원하기 시작한 compose-navigation 으로 마이그레이션하는 대공사**를 진행했습니다. 이 과정에서 **feature 기반 멀티 모듈 프로젝트에서 어떻게 해야 compose-navigation 을 제대로 적용할 수 있을지** 고민해본 내용에 대해 정리해보았습니다.

<br>

# 모듈 간 순환 종속

먼저 단일 모듈에서 가장 단순한 형태의 compose navigation 예시를 살펴봅시다.

```kotlin
@Serializable
data object HomeRoute

@Serializable
data class CheckoutRoute(val bookId: Long)

@Composable
fun MyApp() {
    val navController = rememberNavController()
    NavHost(navController, startDestination = Home) {
        composable<HomeRoute> {
            HomeScreen(
                viewModel = koinViewModel<HomeViewModel>(),
                onNavigateToCheckout = { bookId ->
                    navController.navigate(route = Checkout(bookId))
                }
            )
        }
        composable<CheckoutRoute> { backStackEntry ->
            val bookId = backStackEntry.toRoute().bookId
            CheckoutScreen(
                bookId = bookId,
                onNavigateToHome = {
                    navController.navigate(route = Home)
                }
            )
        }
    }
}
```
단일 모듈의 경우 애플리케이션 최상위의 `NavHost` 에서 직접 `HomeScreen`, `HomeViewModel`, `CheckoutScreen` 을 참조합니다. 하지만 feature 별로 모듈화 된 멀티 모듈 프로젝트의 경우, `:home` 모듈과 `:checkout` 모듈이 나눠져있을 것이고, 각 모듈에 Screen 과 ViewModel 이 선언되어 있을 것입니다. 그럼 이제 위 네비게이션 동작이 멀티 모듈 구조에서도 동작하도록 각 모듈에 네비게이션 코드를 작성해보려 하면, **뭔가 이상함**을 깨달을 수 있습니다. 

<img width="33%" src="/assets/post_images/android/14-1.png" />

위 네비게이션 코드를 보면 `HomeScreen` 에서 `CheckoutScreen` 으로 이동할 수 있고, 반대로 `CheckoutScreen` 에서도 `HomeScreen` 으로 이동할 수 있습니다. 즉, 각 모듈에서 **양방향**으로 이동할 수 있습니다. 따라서 네비게이션 코드를 각 모듈에서 구현하려고 해도 **`:home` 모듈과 `:checkout` 모듈이 서로 참조해야 하기 때문에 순환 종속이 일어나 구현이 불가능합니다.** 또한, 모듈을 분리한 이상 최대한 
**모듈 간 결합력을 낮추는 것**이 중요하기 때문에 이렇게 두 가지 **모듈 간 직접 통신하는 것 자체가 아키텍쳐 관점에서도 바람직하지 않습니다.**

그렇면 어떻게 해야할까요? 가장 직관적이면서도 깔끔한 해결책은 바로 **모든 feature 모듈을 참조하고 있는 모듈에 네비게이션 기능을 위임하는 것**입니다. 일반적으로 이 역할을 맡고 있는 모듈은 **`:app` 모듈**이며, 필요 시 별도로 **`:navigation` 모듈** 등을 따로 두어 모든 feature 모듈들을 참조하도록 하는 경우도 있습니다. 

<img width="95%" src="/assets/post_images/android/14-2.png" />

그러면 위 네비게이션 코드를 단순히 `:app` 모듈에 선언하기만 하면 되는걸까요? 아쉽게도 한 가지 고려해야 할 부분이 남아있습니다.

<br>

# 모듈 별 캡슐화

위에서 모듈 간 직접 참조하고 통신하는 것은 아키텍쳐 관점에서 바람직하지 않다고 했습니다. 이를 프로그래매틱한 차원에서 방지하는 방법은 **모듈 별 캡슐화**를 잘 해주는 것입니다. Kotlin 에서는 이를 위해 모듈 내의 클래스 또는 메서드를 외부 모듈에 노출시키지 않도록 하는 **`internal`** 접근제어자를 제공하고 있습니다. 방금과 같은 순환 종속 발생과 모듈 간 직접적인 통신을 하려는 시도를 막으려면 모듈의 화면을 구성하는 Screen 과 비즈니스 로직을 포함하는 ViewModel 은 굳이 **다른 모듈에 공개하지 않고 숨기는 것**이 좋아보입니다. Screen 과 ViewModel 에 `internal` 키워드를 붙여봅시다.

```kotlin
// feature/home

@Composable
internal fun HomeScreen(
    viewModel: HomeViewModel,
    onNavigateToCheckout: (Long) -> Unit
) {
    /* . . . */
}

internal class HomeViewModel(
    /* . . . */
): ViewModel() {
    /* . . . */
}
```
```kotlin
// feature/checkout

@Composable
internal fun CheckoutScreen(
    bookId: Long,
    onNavigateToHome: () -> Unit
) {
    /* . . . */
}
```

그런데 이렇게 적용하고 나니 또 **뭔가 이상합니다.** 바로 `:app` 모듈에서도 feature 내 Screen 및 ViewModel 접근이 불가능해진다는 것입니다. 그럼 어쩔 수 없이 모듈의 캡슐화는 어느정도 포기해야 하는걸까요? 여기서도 해결책이 존재하는데, 바로 **`NavGraphBuilder`의 확장**입니다.

```kotlin
// feature/home/navigation

@Serializable
data object HomeRoute

fun NavGraphBuilder.homeScreen(
    onNavigateToCheckout: (Long) -> Unit
) {
    composable<HomeRoute> {
        HomeScreen(
            viewModel = koinViewModel<HomeViewModel>(),
            onNavigateToCheckout = onNavigateToCheckout
        )
    }
}
```
```kotlin
// feature/checkout/navigation

@Serializable
data class CheckoutRoute(val bookId: Long)

fun NavGraphBuilder.checkoutScreen(
    onNavigateToHome: () -> Unit
) {
    composable<CheckoutRoute> { backStackEntry ->
        val bookId = backStackEntry.toRoute().bookId
        CheckoutScreen(
            bookId = bookId,
            onNavigateToHome = onNavigateToHome
        )
    }
}
```

`internal` 로 선언된 Screen 과 ViewModel 을 같은 모듈 내의 **`NavGraphBuilder` 확장 함수**에서 호출하고, 해당 **확장 함수와 route 클래스를 외부로 공개**합니다. 이렇게 하면 **모듈 별 캡슐화를 지키면서도 `:app` 모듈의 탐색 그래프에서 feature 모듈 간 네비게이션을 구현할 수 있습니다.** 필요하다면 아래와 같이 `NavController`의 확장 함수를 통해 navigate 코드를 단순화할 수도 있습니다.

```kotlin
fun NavController.navigateToCheckout(bookId: Long) = navigate(CheckoutRoute(bookId))
```

> 이 때, 해당 확장 함수를 구현했다고 해서 route 클래스까지 `internal` 키워드를 통해 숨기면 `:app` 모듈에서 해당 route 로 직접 `navigate()` 하거나 `startDestination` 로 지정하는 등의 작업이 불가능하기 때문에 유의해야 합니다.

최종적으로 `:app` 모듈은 아래와 같은 형태를 갖습니다.

```kotlin
@Composable
fun MyNavHost(
    navController: NavController = rememberNavController(),
    modifier: Modifier = Modifier
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier
    ) {
        homeScreen(
            onNavigateToCheckout = { bookId ->
                navController.navigateToCheckout(bookId)
            }
        )
        
        checkoutScreen(
            onNavigateToHome = {
                navController.navigate(Home)
            }
        )
    }
}

@Composable
fun MyApp() {
    MyNavHost()
}
```

<br>

# 부록

## 중첩 그래프 탐색

추가적으로 한 모듈에 여러 스크린이 포함된 경우, `navigation` 컴포넌트를 이용한 중첩 그래프 탐색을 통해 하나의 확장 함수로 간단히 구현할 수 있습니다.

```kotlin
// feature/friend

@Serializable
sealed class FriendGraph {

    @Serializable
    data object ListRoute : FriendGraph()

    @Serializable
    data class DetailRoute(val friendId: Long) : FriendGraph()
}

fun NavGraphBuilder.friendGraph(
    navController: NavController,
    onNavigateToHome: () -> Unit
) {
    navigation<FriendGraph>(
        startDestination = FriendGraph.ListRoute
    ) {
        composable<FriendGraph.ListRoute> {
            FriendListScreen(
                onNavigateToHome = onNavigateToHome,
                onNavigateToFriendDetail = { friendId ->
                    navController.navigate(FriendGraph.DetailRoute(friendId))
                }
            )
        }

        composable<FriendGraph.DetailRoute> {
            FriendDetailScreen(
                viewModel = koinViewModel<FriendDetailViewModel>(),
                onNavigateToBack = {
                    navController.popBackStack()
                }
            )
        }
    }
}

internal class FriendDetailViewModel(
    savedHandleState: SavedHandleState,
    private val friendRepository: FriendRepository
): ViewModel() {
    
    val friendId = savedHandleState.toRoute<FriendGraph.DetailRoute>().friendId

    /* . . . */
}

/* . . . */
```
```kotlin
// app

@Composable
fun MyNavHost(
    navController: NavController = rememberNavController(),
    modifier: Modifier = Modifier
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier
    ) {
        /* . . . */
        
        friendGraph(
            navController = navController,
            onNavigateToHome = {
                navController.navigate(Home)
            }
        )
    }
}
```

<br>

## 복잡한 데이터 넘기기

Route 클래스에 `Long` 과 같은 원시 타입이 아닌 `data class` 와 같은 데이터를 넘길 경우 **typeMap** 을 지정해줘야 합니다. typeMap 은 넘기려는 데이터에 대한 serialization 정보를 매핑해줍니다. 여기서는 `kotlinx-serialization-json` 을 이용한 방법을 살펴봅니다.

```kotlin
inline fun <reified T : Any?> serializableTypeOf(
    isNullableAllowed: Boolean = false,
    json: Json = Json,
) = object : NavType<T>(isNullableAllowed = isNullableAllowed) {

    override fun get(bundle: Bundle, key: String) = bundle.getString(key)?.let<String, T>(json::decodeFromString)
    
    override fun parseValue(value: String): T = json.decodeFromString(value)

    override fun serializeAsValue(value: T): String = json.encodeToString(value)

    override fun put(bundle: Bundle, key: String, value: T) = bundle.putString(key, json.encodeToString(value))
}

@Serializable
data class Friend(
    val id: Long,
    val name: String,
    val profileImageUrl: String
) {
    companion object {
        val typeMap = mapOf(typeOf<Friend>() to serializableTypeOf<Friend>())
    }
}
```


```kotlin
@Serializable
data class FriendDetailRoute(val friend: Friend)

fun NavGraphBuilder.friendDetailScreen(
    onNavigateToHome: () -> Unit
) {
    composable<FriendDetailRoute>(
        typeMap = Friend.typeMap
    ) { backStackEntry ->
        val friend = backStackEntry.toRoute().friend
        FriendDetailScreen(
            friend = friend,
            onNavigateToHome = onNavigateToHome
        )
    }
}
```
위와 같이 `kotlin-serialization-json` 의 encoding 및 decoding 을 이용하여 `NavType` 의 제네릭한 구현체를 만들어주는 `serializableType` 공통 함수를 작성해두면 편리하게 다양한 데이터에 대한 typeMap 을 만들 수 있습니다.

**하지만 위 예제는 매칭되는 navigation destinaion 이 없다는 오류를 내며 제대로 동작하지 않습니다.** 놀랍게도 그 이유는 바로 **`Friend` 클래스의 `profileImageUrl` 필드**에 있습니다.

<br>

### 데이터에 URL 형식의 문자열이 포함되어 있을 때

Compose navigation 의 route 는 내부적으로 **결국 URL 형식의 문자열**로 이루어져 있습니다. Route 클래스나 그 안에 담기는 데이터가 serializable 해야하는 이유도 바로 이 때문입니다. Route 가 최종적으로 중복되지 않고 유일한 url 형식의 문자열로 변환되야 하기 때문입니다.

> android-app://androidx.navigation/\${route}/\${arguments}

그렇기 때문에 url 형식의 문자열이 포함된 데이터를 넘기게 될 경우, **url 형식으로 변환된 route 문자열에 영향을 주게 되어 url 이 변질되고 이에 맞는 navigation destination 을 찾을 수 없다는 오류가 발생**하게 되는 것입니다.

이와 같은 현상을 피하려면 아래와 같이 데이터 안에 포함된 url 형식의 문자열에 대하여 인코딩 및 디코딩을 해줘야 합니다.

```kotlin
object UriSerializer : KSerializer<String> {
    override val descriptor = PrimitiveSerialDescriptor("Uri", PrimitiveKind.STRING)

    override fun serialize(encoder: Encoder, value: String) {
        encoder.encodeString(URLEncoder.encode(value, StandardCharsets.UTF_8.name()))
    }

    override fun deserialize(decoder: Decoder): String {
        return URLDecoder.decode(decoder.decodeString(), StandardCharsets.UTF_8.name())
    }
}

@Serializable
data class Friend(
    val id: Long,
    val name: String,
    @Serializable(with = UriSerializer::class)
    val profileImageUrl: String
) {
    companion object {
        val typeMap = mapOf(typeOf<Friend>() to serializableTypeOf<Friend>())
    }
}
```
이 때, `URLEncoder` 및 `URLDecoder` 의 경우 java 패키지에서 지원하는 클래스이기 때문에 KMP 환경에서는 사용할 수 없으므로, URL 인코딩/디코딩을 지원하는 별도의 third-party 라이브러리를 사용하셔야 합니다.

<br>

# 마치며

중간중간 `hiltViewModel` 이 아닌 `koinViewModel` 이 나와 당황하신 분들이 계실 수 있을텐데요. KMP 프로젝트를 기반으로 작성된 포스트라 KMP 를 지원하는 koin 을 예시로 사용하게 되었습니다. 얼른 compose-navigation 도 KMP 를 정식 지원하게 되면 좋겠네요!