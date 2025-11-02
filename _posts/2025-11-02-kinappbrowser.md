---       
layout: post
title: "[KMP] 인앱 브라우저 라이브러리 KInAppBrowser 개발기"
date: 2025-11-02
categories: kmp
photos: /assets/post_images/kmp/01.png
tags: [kotlin, kmp, library, publish]
description: "KMP 인앱 브라우저 라이브러리 KInAppBrowser를 개발하고, Maven Central에 제 첫 라이브러리를 배포한 경험을 공유합니다"
---

<br>

# 개요

최근 KMP를 이용해 히스토리 기반 음식 메뉴 추천 서비스인 [모목지](https://github.com/yjyoon-dev/momokzi-kmp)를 개발하던 중, 인앱 브라우저에 대한 동작이 필요하게 되었고 이에 대한 마땅한 오픈소스 라이브러리를 찾지 못해 직접 구현하게 되었습니다.
이렇게 구현한 코드를 바탕으로 이 참에 라이브러리로 만들어 배포해보자는 결심을 하게되었고, 제 생애 처음으로 Maven Central에 직접 만든 라이브러리를 배포하게 되었습니다.

<br>

# 왜 만들었을까?

위에서 언급한 모목지 앱을 개발하며 소셜 OAuth 로그인 기능 구현이 필요했는데, 이를 위해 [supabase](https://supabase.com/)를 사용하게 되었습니다.
Supabase는 본래 DB 서버 제공이 주 목적이지만, 인증 기능에 대한 솔루션도 제공하며 이때 이메일 로그인 뿐만 아니라 여러 provider 들에 대한 OAuth 소셜 로그인 기능을 제공합니다.
또, **supabase 는 [Kotlin Multiplatform 에서 사용할 수 있는 클라이언트 라이브러리](https://github.com/supabase-community/supabase-kt)를 제공하기 때문에 KMP 서비스를 개발할 때 DB, 인증, 비즈니스 로직들을 앱 개발자들이 손쉽게 구현할 수 있습니다.**

<img width="67%" src="https://raw.githubusercontent.com/supabase-community/flutter-auth-ui/main/screenshots/supabase_auth_ui.png"/>

Supabase 를 통해 Android 앱에서는 구글 로그인, iOS 앱에서는 애플 로그인, 그리고 양 쪽 플랫폼 에서 카카오 로그인을 제공하기로 했습니다.
그런데 kmp 용 supabase 클라이언트 라이브러리는 로그인 기능을 각 provider가 제공하는 네이티브 SDK를 사용하지 않고 REST API 를 통해 로그인을 수행합니다.
즉, 로그인 시 브라우저를 통해 URL 을 열고, 인증이 완료되면 브라우저에서 다시 앱으로 리다이렉트 해주는 방식이므로 **앱 -> 브라우저 -> 앱** 플로우를 거치게 됩니다.
이때, **이 플로우가 UX 를 상당히 해치게 되는데 브라우저가 앱 외부에서 열리기 때문에 앱을 아예 벗어나게 되고, 브라우저에서 앱을 돌아올 때도 "모목지 앱으로 계속하시겠습니까?" 와 같은 다이얼로그가 뜨고 유저가 확인 버튼을 눌러주어야 정상적으로 앱으로 다시 돌아올 수 있었습니다.**
설상가상으로, OAuth 인증을 위해 접속했던 웹페이지가 기본 브라우저 앱에 그대로 남아있어 사용자에게 한 층 더 불쾌감을 줄 수 있었습니다.

첫 출시를 하기 전까지는 당장은 기능에 문제가 없으니 마이너한 이슈로 판단하고, 추후 유저가 유입되기 시작한 이후에 고쳐도 된다고 생각했었습니다.
그렇게 나머지 기능들을 모두 개발하고 드디어 대망의 스토어 심사를 받게 되었습니다.

*그런데...*

<img width="80%" src="/assets/post_images/kmp/01-1.png"/>

**애플 앱스토어 심사에서 사용자 경험을 해친다는 사유로 리젝을 받게 되었습니다.**
애플의 심사가 구글의 심사에 비해 월등히 빡세다는 소문을 몸소 체감할 수 있는 순간이었습니다.
심사 결과 내용 중에는 앱에서 url 을 직접 여는 경우에 iOS의 인앱 브라우저 API 인 Safari View Controller 를 사용할 것을 권장하고 있었습니다.

> 구글 플레이 스토어 심사에서는 이를 문제 삼지 않고 심사가 통과 되었습니다.

사실 로그인 과정은 거의 supabase 클라이언트 라이브러리에 의존하고 있었기 때문에 이 문제에 대해 직접 해결이 가능한건지에 대한 확신도 없었기 때문에 상당히 막막했습니다. 마음을 가다듬고 supabase 쪽 코드를 살펴보던 중 아래와 같이 **`UrlLauncher` 를 커스텀 할 수 있다는 것을 알게 되었습니다.**

```kotlin
createSupabaseClient(
    supabaseUrl = SUPABASE_URL,
    supabaseKey = SUPABASE_KEY,
) {
    install(Auth) {
        host = REDIRECT_URL_HOST
        scheme = REDIRECT_URL_SCHEME
        urlLauncher = UrlLauncher { supabase, url ->
            // 플랫폼 별 인앱 브라우저에서 오픈
        }
    }
    // . . .
}
```

정말 다행이라고 느끼고 KMP 용 인앱 브라우저 오픈소스 라이브러리를 바로 찾아보았습니다만...
**어째서인지 그런 오프소스는 찾을 수 없었습니다.** 어차피 대부분 웹뷰를 쓰기 때문에 인앱 브라우저는 수요가 적어서인지, 아니면 이미 공개된 오픈소스가 있지만 제가 못찾은 것인지는 알 수 없지만 그렇게 결국 **직접 구현하게 되었습니다.**

<br>

# 개발 과정

### Android

먼저 Android 쪽의 **Chrome Custom Tabs** 와 같은 경우 `androidx.browser` 의존성만 추가하면 어렵지 않게 구현할 수 있었습니다만, **문제는 Activity 관리였습니다.** Custom Tabs 의 경우 **URL을 열 때 반드시 Activity Context가 필요했기 때문에**, URL 을 열 때마다 context 를 전달하거나 인앱 브라우저 구현체에서 context 를 들고 있어야 했습니다. 그런데 위에서 코드로 언급한 supabase client 의 경우 대부분 DI 를 위한 싱글톤 객체를 만들어 사용하기 때문에, MainActivity의 context 를 전달하긴 힘들었습니다. 또 한 가지 방법은 MainActivity context 를 주입받아 사용하는 것인데, 대부분 koin 은 `Application` 클래스에서 초기화해서 Application Context 를 주입받아 사용할 수 있도록 하는 것이 일반적이고 안정적이었습니다. 따라서 **인앱 브라우저를 MainActivity 에서 초기화함으로써 activity context 를 직접 관리하는 방식을 선택했습니다.** 이는 사실 순수 안드로이드 개발 측면에서 바라봤을 때 **메모리 누수 가능성이 높아 위험한 방식**이었습니다만, 아래와 같은 근거로 안정성을 보장할 수 있다고 판단했습니다.

 - KMP 기반 앱에서 Android 구현은 일반적으로 **싱글 액티비티 구조**이다
 - 혹여나 있을 엣지 케이스에 대해서는 **WeakReference**로 충분히 커버 가능하다

따라서 아래와 같이 인앱 브라우저를 MainActivity 에서 한 번 초기화하면, 코드의 위치 제약 없이 어느 곳에서든 인앱 브라우저를 실행할 수 있는 패턴으로 설계하게 되었습니다.

```kotlin
import dev.yjyoon.kinappbrowser.KInAppBrowser

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // KInAppBrowser 초기화, 내부적으로 WeakReference 를 통해 관리
        KInAppBrowser.init(this)
        
        // 코드 위치 제약 없이 url 열기 가능
        KInAppBrowser.open("https://github.com/yjyoon-dev/KInAppBrowser")
    }
}
```


<br>

### iOS

iOS의 경우에는 사실 지식이 거의 없어서 AI의 힘을 많이 빌리게 되었는데 구현상 Custom Tabs 처럼 activity 같은 특정 인스턴스를 필요로 하는 일도 없었기 때문에 비교적 간단했습니다. 하지만 아래와 같은 **예상치 못한 문제점들이 있었습니다.**

 - 인앱 브라우저 내에서 앱으로 리다이렉트 될 때 **브라우저가 자동으로 닫히지 않는다**
 - 인앱 브라우저를 통해 여는 최초 URL 의 스키마는 **반드시 HTTP/HTTPS 이어야 한다**

첫 번째 문제의 경우, 인앱 브라우저를 통해 인증이 완료된 이후 앱 스키마 url을 통한 리다이렉트까지는 정상적으로 수행되었지만 **Safari 뷰 컨트롤러가 닫히지 않아** 사용자가 직접 빈 페이지의 인앱 브라우저를 닫아주지 않는 이상 로그인이 완료되었다는 것을 인지하기 힘들었습니다. 다행히 Safari 뷰 컨트롤러는 **수동으로 브라우저를 닫는 API** 를 제공하고 있어, Android 와 다르게 `close()` 라는 API 를 제공해 개발자가 수동으로 닫을 수 있도록 했습니다.

> Custom Tabs 의 경우, 인앱 브라우저를 수동으로 닫는 API 는 따로 제공하지 않습니다.

```kotlin
import dev.yjyoon.kinappbrowser.KInAppBrowser

// 별도 초기화 없이 URL 열기 가능
KInAppBrowser.open("https://github.com/yjyoon-dev/KInAppBrowser")

// Safari 뷰 컨트롤러 수동 닫기
KInAppBrowser.close()
```

두 번째 문제의 경우 인앱 브라우저를 통해 `HTTP/HTTPS` 를 제외한 스키마를 갖는 URL 은 열 수가 없었습니다. 다만, 이건 **최초로 여는 URL에 한해서**이고 OAuth 과정처럼 브라우저에서 접속한 **페이지 내에서 일어나는 리다이렉션에 경우에는 해당이 안되기 때문에** 로그인 이후 앱으로 무사히 돌아올 수 있습니다. iOS 진영에서 이러한 앱 스키마 URL의 경우 `UIApplication.sharedApplication.openURL()` 를 통해 열 수 있기 때문에, URL 오픈 시 스키마를 구분하여 `HTTP/HTTPS` 프로토콜이 아닌 경우 `UIApplication`을 이용하도록 내부적으로 분기 처리하였습니다.

```kotlin
import dev.yjyoon.kinappbrowser.KInAppBrowser

// HTTP/HTTPS 외의 스키마일 경우 내부적으로 UIApplication 을 통해 open
KInAppBrowser.open("example://page=1")
```

<br>

# Maven Central 배포 & CI/CD

이렇게 개발된 라이브러리를 Maven Central 에 배포하고 오픈소스 저장소 관리를 용이하도록 하기 위한 작업을 했습니다.
**제 생에 첫 라이브러리 배포였기 때문에, Maven Central 에 계정을 만드는 것이 첫 번째 작업이었습니다.** 항상 누군가가 만든 라이브러리를 확인하러 오는 곳이었는데 이렇게 직접 계정을 등록하니 감회가 새로웠습니다. Namespace 의 경우 깃허브 계정으로 로그인하면 `io.github.username` 을 기본적으로 제공하지만, 저는 `yjyoon.dev` 도메인을 직접 갖고 있었기 때문에 이를 등록할 수 있었습니다. 

<img width="50%" src="/assets/post_images/kmp/01-2.png"/>

KMP 라이브러리를 Maven Central 에 배포하는 방법은 Android 라이브러리 배포 방법과 사실 동일하며,
**공식 문서에서 [KMP 라이브러리 배포 튜토리얼](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-publish-libraries.html)**을 제공하고 있기 때문에 자세한 내용은 본 링크를 참조해주세요.

그 외의 GitHub Actions 를 이용한 CI/CD 의 경우 아래와 같은 내용들을 구현했습니다.
자세한 workflow 파일은 아래에서 제공하는 repository 링크에서 확인해주세요.

 - **spotless** 를 이용한 코드 스타일 체크
 - **dokka** 를 이용한 API 문서 생성 및 GitHub Pages 자동 배포
 - PR 및 push 시 **플랫폼 별 빌드 및 테스트 코드 수행**
 - GitHub Release 생성 시 **자동 Maven Central 배포**

<br>

# 마무리

<img width="55%" src="/assets/post_images/kmp/01-3.png"/>

이렇게 제 첫 오픈소스 라이브러리 **KInAppBrowser** 가 배포되어 Kotlin Multiplatform 커뮤니티에 기여할 수 있게 되었습니다. 언젠가 한 번 직접 만들어보고 싶다는 막연한 생각만 있었는데 이렇게 직접 만들고 배포까지 하니 그 뿌듯함이 정말 큰 것 같습니다. 앞으로도 오픈소스와 Kotlin 커뮤니티에 많이 기여할 수 있으면 좋겠습니다. 읽어주셔서 감사합니다!

- **KInAppBrowser GitHub**: [https://github.com/yjyoon-dev/KInAppBrowser](https://github.com/yjyoon-dev/KInAppBrowser)