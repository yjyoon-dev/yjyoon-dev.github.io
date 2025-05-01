---       
layout: post
title: "[Android] 멀티 모듈 프로젝트에서 Compose Navigation 제대로 사용하기"
date: 2025-05-01
categories: android
photos: /assets/post_images/android/14.png
tags: [android, compose, navigation, multi-module, architecture]
description: "멀티 모듈 구조의 안드로이드 및 KMP 프로젝트에서 compose navigation 을 이용해 모듈 간 캡슐화와 의존성을 고려한 화면 네비게이션 기능을 제대로 구현해보자"
---

# 개요

Android 및 KMP 프로젝트에서 멀티 모듈을 도입한다고 가정해봅시다.
여러 멀티 모듈 구조 중에서도 feature 별로 모듈화를 진행하는 프로젝트라고 한다면, feature 간의 화면 이동을 구현하려면 어떻게 해야할까요?
feature 별 Screen 과 ViewModel 을 공개하고 이를 다른 feature 에서 직접 참조해도 괜찮을까요?

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
                viewModel = hiltViewModel<HomeViewModel>(),
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

위 네비게이션 코드를 보면 `HomeScreen` 에서 `CheckoutScreen` 으로 이동할 수 있고, 반대로 `CheckoutScreen` 에서도 `HomeScreen` 으로 이동할 수 있습니다. 즉, 각 모듈에서 **양방향**으로 이동할 수 있습니다. 따라서 네비게이션 코드를 각 모듈에서 구현하려고 해도 **`:home` 모듈과 `:checkout` 모듈이 서로 참조해야 하기 때문에 **순환 종속**이 일어나 구현이 불가능합니다.** 또한, 모듈을 분리한 이상 최대한 
**모듈 간 결합력을 낮추는 것**이 중요하기 때문에 이렇게 두 가지 **모듈 간 직접 통신하는 것 자체가 아키텍쳐 관점에서도 바람직하지 않습니다.**

그렇면 어떻게 해야할까요? 가장 직관적이면서도 깔끔한 해결책은 바로 **모든 feature 모듈을 참조하고 있는 모듈에 네비게이션 기능을 위임하는 것**입니다. 일반적으로 이 역할을 맡고 있는 모듈은 **`:app` 모듈**이며, 필요 시 별도로 **`:navigation` 모듈** 등을 따로 두어 모든 feature 모듈들을 참조하도록 하는 경우도 있습니다. 

<img width="75%" src="/assets/post_images/android/14-2.png" />

그러면 위 네비게이션 코드를 단순히 `:app` 모듈에 선언하기만 하면 되는걸까요? 아쉽게도 한 가지 고려해야 할 부분이 남아있습니다.

<br>

# 모듈 별 캡슐화

위에서 모듈 간 직접 참조하고 통신하는 것은 아키텍쳐 관점에서 바람직하지 않다고 했습니다. 이를 프로그래매틱한 차원에서 방지하는 방법은 **모듈 별 캡슐화**를 잘 해주는 것입니다. Kotlin 에서는 이를 위해 모듈 내의 클래스 또는 메서드를 외부 모듈에 노출시키지 않도록 하는 **`internal`** 접근제어자를 제공하고 있습니다. 방금과 같은 순환 종속 발생과 모듈 간 직접적인 통신을 하려는 시도를 막으려면 모듈의 화면을 구성하는 Screen 과 비즈니스 로직을 포함하는 ViewModel 은 굳이 다른 모듈에 공개하지 않고 숨기는 것이 좋아보입니다. Screen 과 ViewModel 에 `internal` 키워드를 붙여봅시다.

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
    savedHandleState: SavedHandleState
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
            viewModel = hiltViewModel<HomeViewModel>(),
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

`internal` 로 선언된 Screen 과 ViewModel 을 같은 모듈 내의 **`NavGraphBuilder` 확장 함수**에서 호출하고, 해당 **확장 함수와 route 클래스를 외부로 공개**합니다. 이렇게 하면 **모듈 별 캡슐화를 지키면서도 `:app` 모듈의 탐색 그래프에서 feature 모듈 간 네비게이션을 구현할 수 있습니다.** 필요하다면 아래와 같이 `NavController`의 확장 함수를 통해 navigate 코드를 단순화할 수도 있습니다. 이 때, 해당 확장 함수를 구현했다고 해서 route 클래스까지 `internal` 키워드를 통해 숨기면 `:app` 모듈에서 해당 route 로 직접 `navigate()` 하거나 `startDestination` 로 지정하는 등의 작업이 불가능하기 때문에 유의해야 합니다.

```kotlin
fun NavController.navigateToCheckout(bookId: Long) = navigate(CheckoutRoute(bookId))
```

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

# 중첩 그래프 탐색

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
                viewModel = hiltViewModel<FriendDetailViewModel>(),
                onNavigateToBack = {
                    navController.popBackStack()
                }
            )
        }
    }
}

@HiltViewModel
internal class FriendDetailViewModel @Inject constructor(
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
