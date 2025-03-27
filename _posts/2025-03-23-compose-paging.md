---       
layout: post
title: "[Android] Jetpack Compose 무한스크롤 직접 구현하고 성능 개선해보기"
date: 2025-03-27
categories: android
photos: /assets/post_images/compose/paging.png
tags: [android, compose]
description: "Compose 환경에서 서버로부터 받은 응답을 자연스럽게 무한스크롤 형태로 표시하기 위해 페이지네이션을 위한 데이터 모델을 설계하고 PagingColumn 을 직접 구현한 뒤 적절한 리컴포지션 관리로 성능을 개선해 봅시다."
---

# 개요

프로젝트를 진행하던 도중 서버에서 일정 갯수 만큼 내려주는 데이터를 사용자의 스크롤 액션에 따라 요청하고 불러온 데이터를 리스트 끝에 연속적으로 추가해주어야 하는 기능 구현이 필요했습니다. 즉, 무한 스크롤 구현이 필요했습니다. Android 에서는 이미 이와 관련된 [Paging](https://developer.android.com/jetpack/androidx/releases/paging) 이라는 라이브러리를 제공하고 있지만, 프로젝트에서 요구하는 기능만 간단하게 추가하고 싶었고 이는는 compose 에서 제공하는 기본 함수로도도 충분히 구현할 수 있을 것 같아 직접 만들어보게 되었습니다.

<br>

# 데이터 모델 만들기

## 서버 요청/응답 형식

본 프로젝트에서 무한 스크롤을 요구하는 서버의 요청 및 응답 형식은 아래와 같은 형식이었습니다.

### Request
```
GET /api/user?cursorId={cursorId}&limit={limit}
```

요청에서 `cursorId` 는 서버의 DB 상에서 데이터를 요청할 **시작점**을 의미하고, `limit` 는 해당 지점으로 부터 **몇 개**의 데이터를 요청할지를 의미합니다.

<br>

### Response
```json
// GET /api/user?cursorId=0&limit=3
{
    "data": [
        {
            "id": 1,
            "name": "코틀린"
        },
        {
            "id": 2,
            "name": "안드로이드"
        },
        {
            "id": 3,
            "name": "컴포즈"
        }
    ],
    "nextCursorId": 2,
    "isLast": false
}
```

응답에서는 `cursorId=0` 에서부터 `3` 개의 데이터를 요청했으므로 `data` 필드에 총 **3** 길이의 데이터가 내려옵니다. 그리고 클라이언트에서 다음으로 이어지는 데이터를 연속해서 요청할 수 있도록 `nextCursorId` 로 다음 번째의 cursorId 를 전달합니다.
마지막으로 데이터가 끝까지 도달했는지의 여부를 `isLast` 필드로 전달합니다. `isLast=true` 라면 클라이언트에선 데이터를 끝까지 받아왔다고 인지하고 더 이상 서버 요청을 하지 않아야 합니다.

<br>

## 클라이언트 요청/응답 모델

위의 서버 요청/응답을 클라이언트에서 처리할 수 있도록 모델을 만들어봅시다.

### Request

```kotlin
data class PagingRequest(
    val cursorId: Int,
    val limit: Int = DEFAULT_LIMIT
) {
    companion object {
        const val DEFAULT_LIMIT = 3
    }
}
```

페이징 요청에 지속적으로 사용될될 공통 모델로, 다음으로 요청할 기준점 `cursorId` 와, 데이터의 갯수 `limit` 필드를 갖습니다.

<br>

### Response

```kotlin
interface PagingResponse<T> {
    val datas: List<T>
    val nextCursorId: Int?
    val isLast: Boolean
}

@Serializable
data class UserResponse(
    override val datas: List<User>,
    override val nextCursorId: Int?,
    override val isLast: Boolean
) : PagingResponse<User>

@Serializable
data class User(
    val id: Long,
    val name: String
)
```
위에서 예시로 든 `json` 응답을 파싱하기 위한 모델입니다. 추후 무한스크롤을 위한 응답을 공통화 해야할 일이 반드시 생길 것 같으니 `CursorResponse` 인터페이스를 통해 추상화 해주었습니다.

무한 스크롤을 위해 API 를 지속적으로 호출할 수록 위 형식의 데이터가 점점 쌓이게 될텐데, 이를 어떻게 관리해서 UI 에 나타내면 좋을까요?

<br>

# UI 상태 모델 만들기

위와 같은 형식으로 지속적으로 쌓이게 될 데이터를 UI 에 표시하기 위해 하나의 공통된 상태 클래스를 만들어봅시다.

```kotlin
data class PagingState<T>(
    val datas: List<T> = emptyList(),
    val isLoading: Boolean = false,
    val nextCursorId: Int? = null,
    val isLast: Boolean = false
) {
    val canRequest: Boolean
        get() = !isLoading && !isLast
    
    fun nextRequest(limit: Int = DEFAULT_LIMIT) = 
        PagingRequest(cursorId = nextCursorId, limit = limit)

    fun merge(next: PagingResponse<T>) = copy(
        datas = datas + next.datas,
        nextCursorId = next.nextCursorId,
        isLast = next.isLast
    )
}
```

`PagingState` 는 여러 데이터를 UI 에 표시하기 위해 `ViewModel` 에서 들고 있을 상태 클래스이며, `datas` 필드에 `UserResponse` 와 같이 UI 를 구성하기 위한 직접적인 데이터들이 쌓이게 됩니다. 이때 `PagingState` 으로 다음 요청 정보인 `PagingRequest` 를 만들어내고, 이로부터 받은 또다른 `PagingState` 를 `merge` 메서드를 통해 현재 `PagingState` 와 합쳐줍니다. 결과적으로 새로운 `PagingState` 는 **`datas` 필드로 데이터가 뒤에 추가되고**, **`nextCursorId` 와 `isLast` 값은 최신값값으로 업데이트** 됩니다. 이 때, 다음 데이터를 요청하기 위해선 기존의 요청이 종료된 이후여야 하고, `isLast` 필드값이 `false` 여야 하므로 이를 간편하게 확인할 수 있도록 `canRequest` 프로퍼티를 따로 두었습니다.

<br>

```kotlin
fun <T> PagingResponse<T>.toPagingState() = PagingState(
    datas = datas,
    isLoading = false,
    nextCursorId = nextCursorId,
    isLast = isLast
)
```
서버로부터 받은 응답을 `PagingState` 로 변환해주는 mapper 를 확장 함수를 이용하여 작성해줍니다. 

<br>

<img src="/assets/post_images/compose/paging-01.png" />

그러면 위 사진과 같은 흐름으로 데이터를 순차적으로 요청하고 연속적으로 받아 UI 상태가 지속적으로 업데이트 되는는 구조가 완성됩니다.

<br>

# PagingColumn 구현하기


이제 위에서 만든 `PagerState` 내의 데이터를 표시하고 적절한 타이밍에 `nextRequest()` 를 발화해 줄 UI 를 만들어야 합니다. 

무한 스크롤 형태의 리스트에서 다음 데이터를 요청하는 시점은 **스크롤이 마지막 아이템까지 내려갔을 때**입니다. 이를 감지해서 다음 데이터를 요청하는 UI를 `LazyListState` 와 `LazyColumn` 을 이용해 구현해봅시다.

```kotlin
val listState = rememberLazyListState()

val lastVisibleItemIndex = listState.layoutInfo.visibleItemsInfo.lastOrNull()?.index

LaunchedEffect(lastVisibleItemIndex) {
    val lastIndex = listState.layoutInfo.totalItemsCount - 1
    if (lastVisibleItemIndex == lastIndex) {
        // load next data
    }
}

LazyColumn(
    state = listState
) {
    // show items from data
}
```

현재 보여지고 있는 마지막 아이템의 인덱스가 바뀔 때마다 리스트의 마지막 아이템 인덱스를 비교하여 이 둘이 일치할 때 새로운 데이터를 불러오고 있습니다. 코드 상으로 동작이 굉장히 직관적이고, 실제로도 잘 동작하지만 위 코드에는 한 가지 문제점이 있습니다. 바로 **과도한 리컴포지션이 발생하여 성능이 떨어진다**는 것입니다.

## derivedStateOf 로 개선하기

`LazyListState.layoutInfo` 는 **스크롤이 이루어지는 매 프레임마다** 바뀌기 때문에 이를 컴포저블 내의 필드에 직접 선언하게 되면 말그대로 스크롤하는 매 프레임마다 리컴포지션이 일어나게 됩니다. 우리에게 필요한 스크롤 탐지 시점은 수 많은 프레임 중에서 **단 한 프레임, 바로 스크롤로 인해 리스트의 마지막 아이템이 보이기 시작하는 임계점** 입니다. Compose에서는 이처럼 상태 객체가 UI가 업데이트되어야 하는 것보다 더 자주 변경되어 불필요한 리컴포지션이 발생하는 상황에서 사용할 수 있는 `derivedStateOf` 함수를 제공합니다.

```kotlin
val listState = rememberLazyListState()

val isLastItemVisible by remember { 
    derivedStateOf {
        val lastVisibleItemIndex = listState.layoutInfo.visibleItemsInfo.lastOrNull()?.index
        val lastIndex = listState.layoutInfo.totalItemsCount - 1
        lastVisibleItemIndex == lastIndex
    }
}

LaunchedEffect(isLastItemVisible) {
    if (isLastItemVisible) {
        // load next data
    }
}

LazyColumn(
    state = listState
) {
    // show items from data
}
```

위와 같이 `derivedStateOf` 를 적용하면 `listState.layoutInfo` 와는 무관하게 **오직 `lastVisibleItemIndex == lastIndex` 의 값이 달라질 때만** `isLastItemVisible` 값이 바뀌며 리컴포지션이 일어나게 됩니다. 이처럼 `derivedStateOf` 는 **스크롤 위치와 같이 연속적인 값들에서 특정 임곗값을 기준으로 상태가 바뀌어야 할 때 사용하면 성능적으로 큰 이점**을 가져갈 수 있습니다.

하지만 위와 같이 코드를 수정해도 한 가지 찜찜한 부분이 있습니다. 바로 **`LaunchedEffect` 블록이 쓸데없이 너무 자주 수행될 수 있고, 이 때마다 컴포저블 전체가 리컴포지션 될 수 있다는 것**입니다. `LaunchedEffect` 를 통해 매번 `isLastItemVisible` 을 체크하기보단, 깔끔하게 **`isLastItemVisible` 상태에 대한 변경에 대한 구독하게끔** 할 순 없을까요?

<br>

## snapshotFlow 로 개선하기

특정 값에 대한 변경을 구독한다고 했을 때 떠오르는 것이 있습니다. 바로 **Coroutines Flow** 입니다. Compose 에서는 `State` 를 `Flow` 로 변환해주는 `snapshotFlow` 를 제공하고 있습니다. `Flow` 를 `State` 로 바꿔주는 `collectAsState()` 와는 달리 반대로 변환해주는 기능을 하고 있습니다. 이를 통해 특정 상태 값을 **한 번 구독하면, 값들을 자유롭게 필터링하고 collect 할 수 있습니다.**

```kotlin
val listState = rememberLazyListState()

val isLastItemVisible by remember { 
    derivedStateOf {
        val lastVisibleItemIndex = listState.layoutInfo.visibleItemsInfo.lastOrNull()?.index
        val lastIndex = listState.layoutInfo.totalItemsCount - 1
        lastVisibleItemIndex == lastIndex
    }
}

LaunchedEffect(Unit) {
    snapshotFlow { isLastItemVisible }
        .distinctUntilChanged()
        .filter { it }
        .collect {
            // load next data
        }
}

LazyColumn(
    state = listState
) {
    // show items from data
}
```
이제 `isLastItemVisible` 을 매번 확인할 필요 없이 한 번의 구독으로 지속적으로 상태 값을 받아와 불필요한 리컴포지션 없이 다음 데이터를 로드할 수 있습니다. 

이제 구현한 것을 바탕으로 여러 화면에서 재사용할 수 있도록 composable 객체로 만들어 봅시다.

<br>

## PagingColumn

```kotlin
@Composable
fun PagingColumn(
    onPaging: () -> Unit,
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    content: LazyListScope.() -> Unit
) {
    val isLastItemVisible by remember {
        derivedStateOf {
            val lastVisibleItemIndex = state.layoutInfo.visibleItemsInfo.lastOrNull()?.index
            val lastIndex = state.layoutInfo.totalItemsCount - 1
            lastVisibleItemIndex == lastIndex
        }
    }

    LaunchedEffect(Unit) {
        snapshotFlow { isLastItemVisible }
            .distinctUntilChanged()
            .filter { it }
            .collect {
                onPaging()
            }
    }

    LazyColumn(
        state = state,
        modifier = modifier,
        content = content
    )
}
```

`PagingColumn` 을 만들어 보았습니다. `onPaging()` 람다 객체를 넘겨받아 다음 데이터를 로드합니다.

그런데 한 가지 더 고려해야 할 만한 부분이 있습니다.

> 아직 이전 데이터를 요청 중이거나, 데이터를 이미 끝까지 다 받아온 경우엔 어떻게 하지?

위 의문으로부터 무작정 스크롤 위치가 마지막 아이템에 있다고 해서 다음 데이터를 받아오면 안된다는 것을 알 수 있습니다. 다음 데이터를 로드할 수 있는 상황인지에 대한 파라미터를 받아 `isLastItemVisible` 의 조건을 조금 수정해줍시다.

```kotlin
@Composable
fun PagingColumn(
    onPaging: () -> Unit,
    modifier: Modifier = Modifier,
    enablePaging: Boolean = true,
    state: LazyListState = rememberLazyListState(),
    content: LazyListScope.() -> Unit
) {
    val isLastItemVisible by remember {
        derivedStateOf {
            val lastVisibleItemIndex = state.layoutInfo.visibleItemsInfo.lastOrNull()?.index
            val lastIndex = state.layoutInfo.totalItemsCount - 1
            enablePaging && lastVisibleItemIndex == lastIndex
        }
    }

    LaunchedEffect(Unit) {
        snapshotFlow { isLastItemVisible }
            .distinctUntilChanged()
            .filter { it }
            .collect {
                onPaging()
            }
    }

    LazyColumn(
        state = state,
        modifier = modifier,
        content = content
    )
}
```
이렇게 `PagingColumn` 이 완성되었습니다.

이제 앞서 만들었던 `PagingData` 와 함께 아래와 같이 무한스크롤을 구현할 수 있습니다.

<br>

# 전체 예제

```kotlin
interface UserRepository {
    fun loadUser(request: PagingRequest): PagingResponse<User>
}

class MainViewModel(private val userRepository: UserRepository) : ViewModel() {

    private val _pagingState = MutableStateFlow(PagingState<User>())
    val pagingState: StateFlow<PagingState<User>> = _pagingState.asStateFlow()

    fun loadUser() {
        if (pagingState.value.canRequest.not()) return
        viewModelScope.launch {
            val request = pagingState.value.nextRequest()
            val response = userRepository.loadUser(request)
            _pagingState.value = pagingState.value.merge(response)
        }
    }
}

@Composable
fun MainScreen(
    viewModel: MainViewModel,
    modifier: Modifier = Modifier
) {
    val pagingState by viewModel.pagingState.collectAsStateWithLifecycle()

    PagingColumn(
        onPaging = { viewModel.loadUser() },
        enablePaging = pagingState.canRequest,
        modifier = modifier
    ) {
        items(pagingState.datas) {
            // item composable
        }
    }
}
```

이렇게 서버로부터 페이징 형식의 데이터를 받는 모델을 설계하고, 이를 화면에 나타낼 수 있도록 재사용 가능한 컴포넌트를 구현한 뒤 UI 성능까지 개선해보았습니다. 무한스크롤 및 페이지네이션 화면을 구현하는 이들에게 도움이 되길 바랍니다.