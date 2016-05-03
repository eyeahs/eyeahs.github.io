---
layout: post
category: blog
title: "Espresso #2 - Espresso 기초"
date: "2016-04-18 16:07:46 +0900"
published: true
splash: ""
tags: ""
---

[원본](https://google.github.io/android-testing-support-library/docs/espresso/basics/index.html)

Espresso API는 테스트 작성자에게 사용자가 (UI요소들을 위치에 두고 그들과 상호 작용하는) 어플리케이션과 상호 작용하는 동안 무엇을 할 것인지의 관점으로 생각하는 것을 돕는다. 동시에 이 프레임워크는 어플리케이션의 Activity들과 View들에 직접 접근하는 것을 막는다. 이 객체들을 잡고 UI 스레드에서 시작된 이것들에 작업을 하는 것이 비정상적인 테스트의 주된 원인이기 때문이다. 그러므로, 당신은 Espresso API에서 getView나 getCurrentActivity같은 메소드들을 볼 수 없을 것이다. 하지만 당신은 여전히 당신의 ViewAction들과 ViewAssertions들을 구현함을 통해 View에서 안전하게 작업을 할 수 있다.

다음은 Espresso의 주요 구성 요소들의 개요이다:

* **Espresso** – View들과의 상호 작용의 진입점(onView와 onData를 통해서). 또한 어떤 뷰와도 묶일 필요가 없는 APIs들을 노출한다(pressBack).
* **ViewMatchers** – Matcher<? super View> 인터페이스를 구현한 객체들의 모임. 현재 View 계층 내에 위치한 view의 위치를 찾기 위해 하나 또는 그 이상의 ViewMatchers를 onView 메소드에게 전달할 수 있다.
* **ViewActions** – ViewInteraction.perform() 메소드에게 전달할 수 있는 ViewAction들의 모임 (예를 들어, click())
* **ViewAssertions** – ViewInteraction.check()메소드에 전달할 수 있는 ViewAssertion들의 모임. 대부분 시간, 당신은 matches assertion을 사용할 것이다. 이것은 현재 선택된 view의 상태를 assert하기 위해 View matcher를 사용한다.

예:

    onView(withId(R.id.my_view))      // withId(R.id.my_view)은 ViewMatcher
      .perform(click())               // click()은 ViewAction
      .check(matches(isDisplayed())); // matches(isDisplayed())은 ViewAssertion

# onView로 view를 찾기

막대한 경우들에서, onView메소드는 현재 view 계층내에서 유일한 한가지와 일치함이 예상되는 hamcrest matcher를 받는다. Matcher들은 강력하다. 그리고 Mockito나 JUnit에서 이것들을 사용하는 사람들에게 친숙할 것이다. 만약 당신이 hamcrest matcher에 익숙하지 않다면 [이 슬라이드](http://www.slideshare.net/shaiyallin/hamcrest-matchers)를 먼저 훑어 보기를 권장한다.

보통 희망하던 view는 유일한 R.id를 가지며 단순한 withId matcher는 view 검색을 줄일 것이다. 하지만 테스트를 개발할 때 R.id를 확정할 수 없는 정당한 경우들이 많이 있다. 예를 들어, 특정 view는 R.id를 가지고 있지 않거나 R.id가 유일하지 않다. 이는 view에 접근하기 위한 (findViewById()를 통한) 일반적인 방식이 동작하지 않기 때문에 일반적인 Instrumentation 테스트를 불안정하고 작성하기 복잡하게 만들 수 있다. 이와 같이 당신은 view를 가지고 있는 Activity나 Fragment의 private 멤버들에 접근하거나 특정 view를 위해 알려진 R.id로 컨테이너를 찾고 그것의 내용을 다룰 필요가 있을 것이다.

Espresso는 기존의 ViewMatcher들이나 당신이 커스텀한 ViewMatcher들 어느 것이든 사용하여 view를 좁힐 수 있도록 허용하여 이 문제를 깔끔하게 처리한다.


View의 R.id로 이를 찾는 것은 아주 간단하다:

	onView(withId(R.id.my_view))

가끔, R.id 값은 여러 view들에서 공용 될 수 있다. 이런 일이 생기면 특정 R.id를 사용한 시도는 당신에게 (예를 들어) AmbiguousViewMatcherException를 던져 준다. 이 예외 메시지는 당신에게 당신이 검색에 사용하는 현재 view 계층과 유일하지 않은 R.id와 일치하는 view들의 텍스트 묘사를 제공한다.

    java.lang.RuntimeException:
    com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
    This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
…

    +----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
    is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
    ****MATCHES****
    |
    +------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
    is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
	****MATCHES****
    
View들의 다양한 속성들을 살펴보면, 유일하게 인식 가능한 속성을 찾을 수 있을 것이다 (위 예제에서는, view들 중 하나는 "Hello!" 텍스트를 가진다). matcher들의 조합을 통해 당신의 검색을 좁히기 위해 이를 사용할 수 있다.

	onView(allOf(withId(R.id.my_view), withText("Hello!")))

어떤 matcher들을 반전하기 위해 not을 사용할 수도 있다:

	onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))

Espresso가 제공하는 view matcher들은 [ViewMatchers](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/matcher/ViewMatchers.java)를 보라.

Note: 품행이 단정한 어플리케이션에서는, 사용자가 상호 작용할 수 있는 모든 뷰들은 descriptive text 또는 content description 둘 중 어느 하나를 가져야 한다([Android accessibility guidelines](http://developer.android.com/intl/ko/guide/topics/ui/accessibility/apps.html)를 보라. 만약 당신이 'withText'나 'withContentDescription'을 사용하여 onView 검색을 좁힐 수 없다면 이를 접근성 버르고 다루는 것을 고려하라)

Note: 당신이 찾고 있는 view 하나를 검색하기 위해 최소한의 descriptive matcher를 사용하라. 과도하게 명시하지 마라. 이는 프레임워크가 필요 이상으로 많은 일을 하도록 강제한다. 예를 들어, 만약 view가 그것의 텍스트로 유일하게 인식 가능하다면, view가 TextView로 지정할 수 있는지를 명시할 필요가 없다. 많은 view들을 위해서는 view의 R.id로 충분할 것이다.

Note: 만약 타겟 view가 (ListView, GridView, Spinner같은) AdapterView 내부에 있다면 onView 메소드는 동작하지 않을 것이다. 이를 위해서는 onData 메소드를 대신 사용하는 것을 권장한다.

# View에 행위를 수행하기

타겟 view를 위한 적절한 matcher를 찾았다면, 여기에 perform 메소드를 사용하여 ViewAction을 수행할 수 있다.

예를 들어, view를 클릭:

	onView(...).perform(click());

한 perform 호출에 하나 이상의 행위를 수행할 수 있다:

	onView(...).perform(typeText("Hello"), click());

을 요구하는 (click()이나 typeText()같은) 앞선 행위를 고려하라.
만약 당신이 작업하고 있는 view가 ScrollView 내부에 존재한다면 (수직이든 수평이든), click()이나 typeText()처럼 scrollTo()를 통해 뷰가 화면에 표시되어야 함을 요구하는 선행 행위를 고려하라. 이것은 다른 행위를 수행하기 전에 view가 화면에 표시됨을 보장한다.

	onView(...).perform(scrollTo(), click());

Note: 만약 view가 이미 화면에 표시되었다면 scrollTo()은 아무 영향도 주지 않는다. 그래서 큰 스크린 사이즈로 인해 view가 표시되어 있는 경우에도 안전하게 사용할 수 있다 (예를 들어, 당신의 테스트가 크고, 작은 스크린 해상도 모두에서 수행할 때)

Espresso가 제공하는 view actions은 [ViewActions](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/action/ViewActions.java)를 보라.

# View가 assertion을 만족시키는지 확인하기

Assertion들은 check() 메소드를 이용해 현재 선택된 view에 적용 될 수 있다. 가장 많이 사용되는 assertion은 현재 선택된 뷰의 상태를 assert하기 위해 ViewMatcher()를 사용하는 matches() assertion이다.

예를 들어, view가 "Hello!" 텍스트를 가지고 있는지 확인하기 위해:

	onView(...).check(matches(withText("Hello!")));

Note: onView의 인수에 "assertions"을 넣지 마라 - 대신, 당신이 check 블록 내부에서 확인하고자 하는 것을 명확하게 명시하라. 예를 들어:

만약 당신이 "Hello!"가 view의 내용인지를 assert하기를 원한다면, 다음은 bad practice로 간주된다:

    // onView내부에 withText같은 assertion들을 사용하지 마라.
    onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));

반면, 당신이 "Hello!"를 가진 텍스트가 visible한지를 assert하기를 원한다면 -예를 들어 view들의 visibility 플래그가 변경된 후- 이 코드는 괜찮다.

**Note**: view가 화면에 표시되지 않았는지에 대한 assert와 view가 view 계층에서 존재하지 않는지에 대한 assert간의 차이점에 관심을 가지도록 하라.

# onView를 사용하는 간단한 테스트로 시작하기

이 예제에서 SimpleActivity는 Button과 TextView를 포함한다. 버튼이 클릭되면 TextView의 내용이 "Hello Espresso!"로 변경된다. Espresso로 이를 어떻게 테스트하는지가 여기 있다:

### 1.버튼을 클릭하라

첫 단계는 버튼을 찾는 것을 돕는 속성을 찾는 것이다. SimpleActivity의 버튼은 유일한 R.id를 가진다 - 완벽하다!

	onView(withId(R.id.button_simple))

이제 click을 수행하기다:

	onView(withId(R.id.button_simple)).perform(click());

### 2.이제 TextView에 "Hello Espresso!"가 들어있는지 확인하라

검증할 텍스트를 가진 _TextView_ 역시 유일한 R.id를 가진다:

	onView(withId(R.id.text_simple))

이젠 테스트 내용을 검증하기다:

	onView(withId(R.id.text_simple)).check(matches(withText("Hello Espresso!")));

# Using onData with AdapterView controls (ListView, GridView, …)

AdapterView는 Adapter에 자신의 데이터를 동적으로 적재하는 특별한 형태의 위젯이다. 대부분의 일반적인 AdapterView의 예제는 ListView이다. LinearLayout처럼 정적 위젯과는 달리, 현재 view 계층에는 AdapterView 자식들의 부분 집합만 로드될 것이며, 단순한 onView() 검색은 현재 로드되지 않은 view는 찾지 못할 것이다. Espresso는 이를 adapter의 항목 또는 그것의 자식들 중 어딘가에게 작업을 수행하기 전에 먼저 질문하고 (그것에 초첨을 맞추어) 로드할 수 있는 분리된 onData() 진입점을 제공하여 처리한다.
 
Note: 초기에 스크린에 표시되는 Adapter view안의 항목에 대해서는 onData() 로딩 작업을 하는 것을 우회하는 선택을 할 수 도 있다. 이는 그들이 이미 로드되었기 때문이다. 하지만 onData()를 계속 사용하는 것이 더 안전한다.

**Warning:** AdapterView의 커스텀 구현들이 만약 상속된 계약들(특히 getItem() API)을 어기면 onData()메소드에 문제가 있을 수 있다. 이런 경우들 최선의 행동 방식은 당신의 어플리케이션 코드를 고치는 것이다. 만약 이렇게 할 수 없다면 당신은 일치하는 커스텀 AdapterViewProtocol을 구현할 수 있다. 더 상세한 정보를 위해서 Espresso에서 제공되는 기본 AdapterViewProtocols을 들여다 보라.

# onData를 사용하는 간단한 테스트로 시작하기

이 간단한 테스트는 어떻게 onData()를 사용하는지를 보여준다. SimpleActivity은 약간의 항목을 -커피 음료의 유형들을 나타내는 스트링들- 가진 Spinner를 포함한다. 항목이 선택되면, "One %s a day!"(%s은 선택된 항목)으로 변경되는 TextView가 있다. 이 테스트의 목적은 Spinner를 열고 특정 항목을 선택한 뒤 TextView가 item을 포함하는 지를 검증하는 것이다. Spinner 클래스가 AdapterView에 기반하고 있기 때문에 항목 매칭을 위해 onView() 대신 onData()를 사용하는 것이 권장된다.

### 1.항목 선택을 열기 위해 Spinner를 클릭하라

	onView(withId(R.id.spinner_simple)).perform(click());

### 2.“Americano” 항목을 클릭하라

항목 선택을 위해 Spinner는 자신의 내용들로 ListView를 생성한다. 이 항목은 매우 길어서 view 계층에 제공되지 않을 수 있다. 우리는 onData()를 사용해 우리가 원하는 항목이 view 계층에 존재하도록 강제한다. Spinner내부의 항목들은 String들이며, 우리리는 String이며 "Americano"와 동일한 String인 항목과 맞추기를 원한다.

	onData(allOf(is(instanceOf(String.class)), is("Americano"))).perform(click());

### 3.TextView가 스트링 "Americano"를 포함하는지를 검증하라

	onView(withId(R.id.spinnertext_simple))
	  .check(matches(withText(containsString("Americano"))));

# Debugging

Espresso는 테스트 실패시 유용한 디버깅 정보를 제공한다:

## Logging

Espresso는 logcat에 모든 view 행위들을 로그를 남긴다. 예를 들어:

	ViewInteraction: Performing 'single click' action on view with text: Espresso
 
## View hierarchy

Espresso는 onView()가 실패했을 때 예외 문구안에 view 계층을 출력한다. * 만약 onView()가 타겟 view를 찾지 못하면, NoMatchingViewException가 던져진다. 당신은 왜 matcher가 어떤 view들과도 일치하지 않았는지 분석하기 위해 예외 스트링에서 view 계층을 조사할 수 있다. * 만약 onView()가 주어진 matcher와 일치하는 view을 여러개 찾으면, AmbiguousViewMatcherException이 발생한다. View 계층이 출력되고 일치하는 모든 뷰들은 MATCHES 라벨로 마크될 것이다:

    java.lang.RuntimeException:
    com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
    This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
…

    +----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
    is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
    ****MATCHES****
    |
    +------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
    is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
    ****MATCHES****
    When dealing with a complicated view hierarchy or unexpected behavior of widgets it is always helpful to use the Android View Hierarchy Viewer for an explanation.

## AdapterView warnings

Espresso는 사용자에게 AdapterView 위젯들이 존재함을 경고한다. 만약 onView() 작업이 NoMatchingViewException을 던지고 AdapterView 위젯들이 view 계층에 존재한다면, 가장 흔한 해결책은 onData()를 사용하는 것이다. 예외 메시지는 adapter view들의 목록을 가진 경고를 포함할 것이다. 당신은 타겟 view를 로드하기 위해 onData를 부르는데 이 정보를 이용할 것 이다.
