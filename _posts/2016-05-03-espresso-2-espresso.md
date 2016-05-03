---
layout: blog
category: blog
splash: ""
tags: ""
published: false
title: "Espresso #2 - Espresso 기초"
---
Espresso API는 테스트 작성자에게 사용자가 어플리케이션과 상호 작용 할 때 무엇을 할 것인지 대해 생각하도록 격려한다 - UI 요소들을 위치에 두고 그들과 상호작용하는 것. 동시에 이 프레임워크는 어플리케이션의 Activity들과 View들에 직접 접근하는 것을 막는다. UI 스레드에서 시작된 이 객체들을 잡고 작업하는 것이 비정상적인 테스트의 주된 원인이기 때문이다. 그러므로, 당신은 Espresso API에서 getView나 getCurrentActivity같은 메소드들을 볼 수 없을 것이다. 하지만 당신은 여전히 당신의 _ViewAction_들과 _ViewAssertions_들을 구현함을 통해 View에서 안전하게 작업을 할 수 있다.

다음은 Espresso의 주요 구성 요소들의 개요이다:

* **Espresso** – View들과의 상호 작용의 진입점(onView와 onData를 통해서). 또한 어떤 뷰와도 묶일 필요가 없는 APIs들을 노출한다(pressBack).
* **ViewMatchers** – _Matcher<? super View>_ 인터페이스를 구현한 객체들의 모임. 현재 View 계층 내에 위치한 view의 위치를 찾기 위해 하나 또는 그 이상의 ViewMatchers를 _onView_ 메소드에게 전달할 수 있다.
* **ViewActions** – _ViewInteraction.perform()_ 메소드에게 전달할 수 있는 _ViewAction들의_ 모임 (예를 들어, click())
* **ViewAssertions** – _ViewInteraction.check()_메소드에 전달할 수 있는 _ViewAssertion_들의 모임. 대부분 시간, 당신은 matches assertion을 사용할 것이다. 이것은 현재 선택된 view의 상태를 assert하기 위해 View matcher를 사용한다.

예:

    onView(withId(R.id.my_view))      // withId(R.id.my_view)은 ViewMatcher
      .perform(click())               // click()은 ViewAction
      .check(matches(isDisplayed())); // matches(isDisplayed())은 ViewAssertion

# onView로 view를 찾기

막대한 경우들에서, onView메소드는 현재 view 계층내에서 유일한 한가지와 일치함이 예상되는 hamcrest matcher를 받는다. Matcher들은 강력하다. 그리고 Mockito나 JUnit에서 이것들을 사용하는 사람들에게 친숙할 것이다. 만약 당신이 hamcrest matcher에 익숙하지 않다면 [이 슬라이드](http://www.slideshare.net/shaiyallin/hamcrest-matchers)를 먼저 훑어 보기를 권장한다.

보통 희망하던 view는 유일한 R.id를 가지며 단순한 withId matcher는 view 검색을 줄일 것이다. 하지만 테스트를 개발할 때 R.id를 확정할 수 없는 정당한 경우들이 많이 있다. 예를 들어, 특정 view는 R.id를 가지고 있지 않거나 R.id가 유일하지 않다. 이는 view에 접근하기 위한 (_findViewById()_를 통한) 일반적인 방식이 동작하지 않기 때문에 일반적인 Instrumentation 테스트를 불안정하고 작성하기 복잡하게 만들 수 있다. 이와 같이 당신은 view를 가지고 있는 Activity나 Fragment의 private 멤버들에 접근하거나 특정 view를 위해 알려진 R.id로 컨테이너를 찾고 그것의 내용을 다룰 필요가 있을 것이다.

Espresso는 기존의 ViewMatcher들이나 당신이 커스텀한 ViewMatcher들 어느 것이든 사용하여 view를 좁힐 수 있도록 허용하여 이 문제를 깔끔하게 처리한다.


View의 R.id로 이를 찾는 것은 아주 간단하다:

	onView(withId(R.id.my_view))

가끔, _R.id_ 값은 여러 view들에서 공유될 수 있다. 이런 일이 생기면 특정 _R.id_를 사용한 시도는 당신에게 (예를 들어) _AmbiguousViewMatcherException_를 준다. The exception message provides you with a text representation of the current view hierarchy, which you can search for and find the views that match the non-unique R.id:

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
    
View들의 다양항 속성들을 살펴보면, 유일하게 인식 가능한 속성을 찾을 수 있을 것이다 (위 예제에서는, view들 중 하나는 "Hello!" 텍스트를 가진다). 이를 사용하여 matcher들의 조합을 사용하여 당신의 검색을 좁힐 수 있다.

	onView(allOf(withId(R.id.my_view), withText("Hello!")))

어느 matcher들을 반전하기 위해 _not_을 사용할 수도 있다:

	onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))

Espresso가 제공하는 view matcher를 위해 [ViewMatchers](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/matcher/ViewMatchers.java)를 보라.

Note: 품행이 단정한 어플리케이션에서는, 사용자가 상호 작용할 수 있는 모든 뷰들은 descriptive text 또는 content description 둘 중 어느 하나를 가져야 한다([Android accessibility guidelines](http://developer.android.com/intl/ko/guide/topics/ui/accessibility/apps.html)를 보라. 만약 당신이 'withText'나 'withContentDescription'을 사용하여 onView 검색을 좁힐 수 없다면 이를 접근성 버르고 다루는 것을 고려하라)

Note: 당신이 찾고 있는 view 하나를 검색하기 위해 최소한의 descriptive matcher를 사용하라. 과도하게 명시하지 마라. 이는 프레임워크가 필요 이상으로 많은 일을 하도록 강제한다. 예를 들어, 만약 view가 그것의 텍스트로 유일하게 인식 가능하다면, view가 TextView로 지정할 수 있는지를 명시할 필요가 없다. 많은 view들을 위해서는 view의 R.id로 충분할 것이다.

Note: 만약 타겟 view가 (_ListView_, _GridView_, _Spinner_같은) _AdapterView_ 내부에 있다면 onView 메소드는 동작하지 않을 것이다. 이를 위해서는 _onData_ 메소드를 대신 사용하는 것을 권장한다.

# View에 행위를 수행하기

타겟 view를 위한 적절한 matcher를 찾았다면, 여기에 perform 메소드를 사용하여 _ViewAction_을 수행할 수 있다.

예를 들어, view를 클릭:

	onView(...).perform(click());

한 perform 호출에 하나 이상의 행위를 수행할 수 있다:

	onView(...).perform(typeText("Hello"), click());

을 요구하는 (click()이나 typeText()같은) 앞선 행위를 고려하라.
만약 당신이 작업하고 있는 view가 ScrollView 내부에 존재한다면 (수직이든 수평이든), click()이나 typeText()처럼 scrollTo()를 통해 뷰가 화면에 표시되어야 함을 요구하는 선행 행위를 고려하라. 이것은 다른 행위를 수행하기 전에 view가 화면에 표시됨을 보장한다.

	onView(...).perform(scrollTo(), click());

Note: 만약 view가 이미 화면에 표시되었다면 _scrollTo()_은 

will have no effect if the view is already displayed so you can safely use it in cases when the view is displayed due to larger screen size (for example, when your tests run on both smaller and larger screen resolutions).

See ViewActions for the view actions provided by Espresso.

# Checking if a view fulfills an assertion

Assertions can be applied to the currently selected view with the check() method. The most used assertion is the matches() assertion, it uses a ViewMatcher to assert the state of the currently selected view.

For example, to check that a view has the text “Hello!”:

	onView(...).check(matches(withText("Hello!")));

Note: Do not put “assertions” into the onView argument - instead, clearly specify what you are checking inside the check block. For example:

If you want to assert that “Hello!” is content of the view, the following is considered bad practice:

    // Don't use assertions like withText inside onView.
    onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));

On the other hand, if you want to assert that a view with the text “Hello!” is visible - for example after a change of the views visibility flag - the code is fine.

Note: Be sure to pay attention to the difference between asserting that a view is not displayed and asserting that a view is not present in the view hierarchy.

# Get started with a simple test using onView

In this example SimpleActivity contains a Button and a TextView. When the button is clicked the content of the TextView changes to "Hello Espresso!". Here’s how to test this with Espresso:

1. Click on the button

The first step is to look for a property that helps to find the button. The button in the SimpleActivity has a unique R.id - perfect!

	onView(withId(R.id.button_simple))

Now to perform the click:

	onView(withId(R.id.button_simple)).perform(click());

2. Check that the TextView now contains “Hello Espresso!”

The TextView with the text to verify has a unique R.id too:

	onView(withId(R.id.text_simple))

Now to verify the content text:

	onView(withId(R.id.text_simple)).check(matches(withText("Hello Espresso!")));

# Using onData with AdapterView controls (ListView, GridView, …)

AdapterView is a special type of widget that loads its data dynamically from an Adapter. The most common example of an AdapterView is ListView. As opposed to static widgets like LinearLayout, only a subset of the AdapterView children may be loaded into the current view hierarchy and a simple onView() search would not find views that are not currently loaded. Espresso handles this by providing a separate onData() entry point which is able to first load the adapter item in question (bringing it into focus) prior to operating on it or any of its children.

Note: You may choose to bypass the onData() loading action for items in adapter views that are initially displayed on screen because they are already loaded. However, it is safer to always use onData().

Warning: Custom implementations of AdapterView can have problems with the onData() method, if they break inheritance contracts (particularly the getItem() API). In such cases, the best course of action is to refactor your application code. If you cannot do so, you can implement a matching custom AdapterViewProtocol. Take a look at the default AdapterViewProtocols provided by Espresso for more information.

# Get started with a simple test using onData

This simple test demonstrates how to use onData(). SimpleActivity contains a Spinner with a few items - Strings that represent types of coffee beverages. When an item is selected, there is a TextView that changes to "One %s a day!" where %s is the selected item. The goal of this test is to open the Spinner, select a specific item and then verify that the TextView contains the item. As the Spinner class is based on AdapterView it is recommended to use onData() instead of onView() for matching the item.

1. Click on the Spinner to open the item selection

	onView(withId(R.id.spinner_simple)).perform(click());

2. Click on the item “Americano”

For the item selection the Spinner creates a ListView with its contents - this can be very long and the element not contributed to the view hierarchy - by using onData() we force our desired element into the view hierarchy. The items in the Spinner are Strings, we want to match an item that is a String and is equal to the String “Americano”:

	onData(allOf(is(instanceOf(String.class)), is("Americano"))).perform(click());

3. Verify that the TextView contains the String “Americano”

	onView(withId(R.id.spinnertext_simple))
	  .check(matches(withText(containsString("Americano"))));

# Debugging

Espresso provides useful debugging information when a test fails:

## Logging

Espresso logs all view actions to logcat. For example:

	ViewInteraction: Performing 'single click' action on view with text: Espresso
 
## View hierarchy

Espresso prints the view hierarchy in the exception string when onView() fails. * If onView() does not find the target view, a NoMatchingViewException is thrown. You can examine the view hierarchy in the exception string to analyze why the matcher did not match any views. * If onView() finds multiple views that match the given matcher, an AmbiguousViewMatcherException is thrown. The view hierarchy is printed and all views that were matched are marked with the MATCHES label:

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

Espresso warns users about presence of AdapterView widgets. When an onView() operation throws a NoMatchingViewException and AdapterView widgets are present in the view hierarchy, the most common solution is to use onData(). The exception message will include a warning with a list of the adapter views. You may use this information to invoke onData to load the target view.