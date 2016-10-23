---
layout: post
category: blog
published: false
title: Loader And Presenter
---
# 런타임 변경시 객체 보존하기

RetainedFragment만들어서 객체 보존

- 어느 객체든 저장할 수 있지만, `Activity`에 묶여 있는 객체는 절대로 전달하면 안 됩니다. 예를 들어 `Drawable`, `Adapter`, `View` 또는 `Context`와 연관된 기타 모든 객체가 이에 해당됩니다. 이런 것을 전달하면, 원래 액티비티 인스턴스의 모든 보기와 리소스를 몽땅 누출시킵니다.

# Loader

Activity 또는 Fragment에서 비동기식으로 데이터를 쉽게 로딩할 수 있게 해줌.

- 모든 Activity와 Fragment에 사용할 수 있습니다.
- 데이터의 비동기식 로딩을 제공
- 데이터 출처를 모니터링하여 그 콘텐츠가 변경되면 새 결과를 전달
- 구성 변경 후에 재생성된 경우, 마지막 로더의 커서로 자동으로 다시 연결됨. 따라서 데이터를 다시 쿼리하지 않아도 됨.

**Loader** - 데이터의 비동기식 로딩을 수행하는 추상 클래스. 로더의 기본 클래스

- Activity의 onCreate(), 또는 Fragment의 onActivityCreated() 메서드에서 Loader 초기화
  (support 24.0.0부터는 Fragment의 onCreate()에서도 가능)
  - getLoaderManager().initLoader(0 // 로더 식별 고유 ID , null // 로더에 제공할 인수 , this // LoaderCallbacks 구현);
    - ID가 지정한 로더가 이미 존재하면, 마지막으로 생성된 로더 재사용,
      아니면 LoaderManager.LoaderCallbacks#onCreateLoader() 호출

**LoaderManager** - Activity또는 Fragment와 연관된 추상 클래스. 하나 이상의 Loader 인스턴스를 관리하는데 사용.

* 로더 다시 시작 - getLoaderManager().restartLoader(0, null, this);

**LoaderManager.LoaderCallbacks** - 클라이언트에 대해 LoaderManager와 상호 작용하도록 하는 콜백 인터페이스.

- onCreateLoader() - 주어진 ID에 대하여 인스턴트화하고 새 Loader 반환
- onLoadFinished() - 이전에 생성된 로더가 휴식 중이라서 해당되는 데이터를 사용할 수 없는 경우 호출
- onLoaderReset() - 이전에 생성된 로더가 휴식 중이라서 해당되는 데이터를 사용할 수 없는 경우 호출

# Loader의 목표와 Configuration change

[원본](https://medium.com/google-developers/making-loading-data-on-android-lifecycle-aware-897e12760832#.w7v39wljw)

* Loader는 configuration change에도 살아남는다.
* Loader는 영원히 지속되지 않는다. Activity나 Fragment가 destory되면 클린업된다.
* Loader는 Activity나 Fragment를 포함하는 참조를 가지고 있어서는 안된다. (static class로 구현할 것)
* Loader는 ContentObserver, FileObserver, OnSharedPreferenceChangeListener같은 broadcast receiver가 존재하기에 완벽한 곳이다.

Loader는 단 하나의 목표만을 가지고 있다 : 최신 정보를 당신에게 알려주는 것이다. 단말의 configuration change에 살아남고 자신의 데이터 관찰자를 포함하는 것으로 그것을 한다. 이것는 당신의 나머지 Activity/Fragment가 이들의 디테일에 대한 필요가 없음을 의미한다. (또한 당신의 Loader는 데이터가 어떻게 사용되는지에 대해 전혀 알 필요가 없다.)

만약 당신이 configuration change동안 데이터를 저장하기 위해 retained Fragment를 사용하고 있다면, **Retained Fragment를 Loader로 변경하도록 강력하게 권고**한다. Retained Fragment은 Activity 생명주기를 완전히 인지하지만, 이는 완전히 독립적인 독립체로 보아야 한다. 반면 Loader는 Activity나 Fragment의 생명주기(심지어 child fragment에도!)와 직접적으로 엮여있고 화면에 표시하기 위한 데이터를 정확하게 얻기에 훨씬 적합하다. 다이나믹하게 Fragment를 추가하고 제거하는 경우를 생각해보면 -Loader는 당신이 로드 프로세스를 생명주기와 엮으면서도 configuration change에도 로드된 데이터가 파괴되는 것을 피할 수 있다.

단일 초점은 **로딩을 UI와 분리하여 테스트할 수 있음**을 의미한다.여기 예제는 Context만을 전달한다, but you can certainly pass in any required classes (or mocks thereof!) to ease testing. Being entirely event driven, it is also possible to determine exactly what state the Loader is in at any time as well as expose additional state solely for testing.

> **Note**: while there’s a [LoaderTestCase](http://developer.android.com/reference/android/test/LoaderTestCase.html?utm_campaign=adp_series_loaders_020216&utm_source=medium&utm_medium=blog) designed for framework classes, you’ll need to make a Support Library equivalent from the [LoaderTestCase source code](https://android.googlesource.com/platform/frameworks/base/+/master/test-runner/src/android/test/LoaderTestCase.java?utm_campaign=adp_series_loaders_020216&utm_source=medium&utm_medium=blog) if you want to do something similar with the Support v4 Loader (something [Nicholas Pike](https://medium.com/u/295ee3666612) [already has done](http://www.npike.net/2016/unit-testing-loaders?utm_campaign=adp_series_loaders_020216&utm_source=medium&utm_medium=blog)!). This also gives you a good idea of how to interact with a *Loader* without a *LoaderManager*.

이제, **Loader는 반응적이며, 데이터의 수령자이다.**라고 말하는 것은 중요한다. But for what they do, they do fill a needed gap of lifecycle aware components that survive configuration changes and get your data to your UI.

# Presenter surviving orientation changes with Loaders

[원본](https://medium.com/@czyrux/presenter-surviving-orientation-changes-with-loaders-6da6d86ffbbf#.n6d76b1te)

기본적으로 화면 회전같은 단말의 상태 변경configuration changes은 전체 Activity를 재시작하게 한다.

Loader의 생명주기는 시스템에 의해 관리되고 Activity나 Fragment가 영구히 파괴되어야 할 때 자동적으로 클린업된다. 이는 그것들이 언제 삭제되어야 할 지를 당신의 애플리케이션에 추가할 필요가 없음을 의미한다.

원래 Loader의 주요 유스케이스는 AsyncTaskLoader와 CursorLoader의 상속을 통해 백그라운드에서 어떤 데이터를 로드하는 것이다. 특히 content provider와 SQLite 데이터베이스같은 것에서 특히 빛난다. 